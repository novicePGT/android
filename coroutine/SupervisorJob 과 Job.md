![스크린샷 2025-02-20 오후 2 23 19](https://github.com/user-attachments/assets/8b44536f-4f1f-4810-b06d-06396a3a46b5)

# Coroutine SupervisorJob
SupervisorJob 의 사용 이유는 자식 코루틴의 실패가 부모와 형제에게 영향을 주지 않아 사용함   
- Job의 Coroutine 의 자식이 실패되면 부모와 형제 코루틴도 취소가 됨( 부모에서 에러를 잡아야 함 )
- SupervisorJob 의 부모가 취소되면 자식도 전부 취소 됨( 자식 코루틴 각각에서 에러를 핸들링 가능 )
- Job은 주로 실패 전파가 필요한경우( *트랜잭션 )에 사용하고 SupervisorJob은 독립적인 작업들 관리 시( * UI 이벤트 핸들링 ) 사용함

</br>

## 예제로 SupervisorJob 확인하기
```kotlin
        val downloadScope = CoroutineScope(Dispatchers.IO + job)
        activity.lifecycleScope.launch { 
            downloadScope.launch { fileDownload("file1", 100) } 
            downloadScope.launch { fileDownload("file2", 200) } 
            downloadScope.launch { fileDownload("file4", 400) } 
            downloadScope.launch { fileDownload("file1", 300) } 
        }
        
        suspend fun fileDownload(fileName: String, spendTime: Int) {
            delay(spendTime.toLong())
            println(fileName)
        }
// file1
// file2
// file3
// file4
```

</br>

### 위 예제에서 아래와 같이 오류가 발생한다면?
```kotlin
        activity.lifecycleScope.launch { 
            downloadScope.launch { fileDownload("file1", 100) } 
            downloadScope.launch { fileDownload("file2", 200) } 
            downloadScope.launch { fileDownload("file4", 400) } 
            downloadScope.launch {
                throw RuntimeException()
                fileDownload("file1", 300)
            } 
        }
// file1
// file2
// Exception in thread
```
3번 파일을 다운 받는 중 오류가 발생하게 됨: 4번 파일은 다운 받지 않음

### Coroutine Exception Handler 를 적용해본다면?
```kotlin
        val handler = CoroutineExceptionHandler() { _, exception ->
            println("Caught $exception")
        }
        activity.lifecycleScope.launch {
            downloadScope.launch { fileDownload("file1", 100) }
            downloadScope.launch { fileDownload("file2", 200) }
            downloadScope.launch { fileDownload("file4", 400) }
            downloadScope.launch(handler) {
                throw RuntimeException()
                fileDownload("file1", 300)
            }
        }
// file1
// file2
// catch in Handler!!
```
그래도 똑같이 4번의 파일 다운로드는 불가능함

### SupervisorJob 을 사용한다면?
```kotlin
        val downloadScope = CoroutineScope(Dispatchers.IO + SupervisorJob())
        val handler = CoroutineExceptionHandler() { _, exception ->
            println("Caught $exception")
        }
        activity.lifecycleScope.launch {
            downloadScope.launch { fileDownload("file1", 100) }
            downloadScope.launch { fileDownload("file2", 200) }
            downloadScope.launch { fileDownload("file4", 400) }
            downloadScope.launch(handler) {
                throw RuntimeException()
                fileDownload("file1", 300)
            }
        }
// file1
// file2
// catch in handler!!
// file4
```
위 예제와 같이 SupervisorJob 을 적용하면 에러를 자식 각각의 코루틴에서 핸들링 가능하기 때문에 부모까지 영향을 미치지 않아 나머지 코루틴이 정상적으로 동작할 수 있게 된다
