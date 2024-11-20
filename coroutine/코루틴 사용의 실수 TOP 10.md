# 코루틴 사용의 실수 TOP 10 소개
Android 개발자들은 Kotlin 의 Corutine 은 비동기 프로그래밍 형태를 구현하기 위해 필수가 되었습니다   
Coroutine 은 동시 작업을 간소화하고 코드를 더 읽기 쉽게 만들며, 콜백이 난무하던 것을 피할 수 있게 되었습니다   
하지만, 코루틴을 잘못 사용하게 되면 버그, 충돌, 성능적인 문제가 발생할 수 있습니다   

이 자료에서는 코루틴 사용의 실수 상위 10가지를 알고 이해하여 비동기 코드를 작성하는 데 도움이 되고자합니다   
<br/>
<br/>

## 1. 메인 스레드 차단
실수: DISPATCHER 에서 장기 작업을 수행하게 되면 MAIN UI 가 정지되며, ANR( 응답없음 )이 발생할 수 있습니다   

발생 이유: DISPATCHER 을 지정하지 않고 사용하게 되면 현재 어떤 DISPATCHER 를 사용하고 있는지 명확하게 인지하지 못해 발생할 수 있습니다   

회피 방법: 항상 Coroutine 에 적합한 DISPATCHER 를 지정
```kotlin
// 실수
GlobalScope.launch {
  // 장기 실행 작업
}
// 회피
GlobalScope.launch(Dispatcher.IO) {
  // 장기 실행 작업
}
```
- DISPATCHER IO: 입출력 작업에 사용
- DISPATCHER MAIN: CPU 작업 또는 UI 변경이 일어날 수 있는 작업에 사용
<br/>
<br/>

## 2. Coroutine Scope 계층 무시
실수: Coroutine Scope 를 올바르게 구성하지 않으면 의도한 수명 주기를 넘어서 관리되지 않는 Coroutine 이 생성되어 메모리 누수나 충돌이 발생할 수 있습니다   

발생 이유: GlobalScope 를 무분별하게 사용하거나, 구성 요소가 종료되거나 파괴되었을 때 Coroutine 을 취소하지 않는 경우

회피 방법: Coroutine 을 특정 lifecycle 에 연결하여 구조화된 동시성을 사용합니다.   
- Activity 나 Fragment 에서는 lifecycleScope( viewLifecycleOwner.lifecycleScope ) 사용
- ViewModel 에서는 viewModelScope 를 사용
```kotlin
viewModelScope.launch {
  // 실행 작업
}
```
- 이렇게 사용하면 연관된 lifecycle 이 파괴 될 때 코루틴도 적절하게 취소됩니다
<br/>
<br/>

## 3. Coroutine 내부의 잘못된 예외 처리
실수: Coroutine 내에서 예외를 제대로 처리하지 못하면 예기치 않은 충돌이나 명백한 오류가 발생할 수 있습니다   

발생 이유: Coroutine 내부에서 try-catch 블록이 어떻게 작동하는지 모르는 경우에 발생할 수 있습니다   

회피 방법:    
- Coroutine CancellationException 을 주의해서 확인해야 하며, 일반적으로 코루틴이 제대로 취소되도록 다시 Throw 해야 합니다   
- 구조화된 동시성을 위해 발생한 Exceoption 은 부모에게 전파됩니다
```kotlin
viewModelScope.launch {
  try {
    // 예외를 던질 수 있는 suspended function
  } catch (e: Exception) {
    if (e ! is CancellationException) {
    // 예외 처리
    } else {
      throw e // 취소를 알리기 위해 다시 던지기
    }
  }
}
```
처리되지 못하고 던져진 에러에서는 CoroutineExceptionHandler 를 사용할 수 있습니다
```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
  if (throwable !is CancellationException {
    // 처리되지 않은 예외 처리
  }
}

viewModelScope.launch(exceptionHandler) {
  // 예외를 던질 수 있는 suspended function
}
```
<br/>
<br/>

## 4. 잘못된 Coroutine Builder 사용
실수: launch 와 async builder 를 혼동하여 결과가 누락되거나 불필요한 동시성과 같이 의도하지 않은 행동으로 이어질 수 있습니다

발생 이유: launch( Job 을 반환 )와 Async( 결과를 얻기 위한 )의 차이를 오해하고 있습니다

회피 방법:
- launch: 결과가 필요하지 않고 비동기 실행을 하고싶을 때 사용합니다
- async: 비동기적으로 값을 계산하고 결과를 얻기 위해 사용합니다
```kotlin
val deferredResult = async {
  calculateValue()
]
val result = deferredResult.await()
```
<br/>
<br/>

## 5. GlobalScope 를 과도하게 사용
실수: Coroutine 을 실행하는 데 GlobalScope 에 의존하면 필요 이상으로 Coroutine 이 오래 실행되고 관리하기 어려워 질 수 있습니다   

발생 이유: Coroutine 의 생명주기를 고려하지 않았거나, 예제와 튜토리얼에서 단순화를 위해 사용한 GlobalScope 를 단순히 복사했을경우

회피 방법: 절대적으로 필요하지 않는 한 GlobalScope 사용을 피하자 / 대신 적절한 범위로 구조화된 동시성을 사용하자
- lifecycleScope: UI 관련 구성요소
- viewModelScope: ViewModel 안에서 사용할 때
- CoroutineScope: 적절한 취소를 통한 사용자 정의
<br/>
<br/>

## 6. 스레드 안전성을 고려하지 않음
실수: 적절한 동기화 없이 여러 코루틴에서 공유되는 변경 가능한 데이터에 엑세스하거나 수정하면 Race Condition 이 발생한다

발생 이유: 코루틴이 스레딩을 처리한다고 가정하고 공유 리소스에서 스레드 안전성의 필요성을 무시한다

회피 방법:
- 스레드를 사용할 때 안전한 데이터 구조를 사용
- Mutex 액세스를 클래스와 동기화한다
- 변경 가능한 상태를 특정 스레드나 코루틴으로 제한한다
```kotlin
val mutex = Mutext()
var sharedResource = 0

CoroutineScope.launch {
  mutex.withLock {
    sharedResource++
  }
}
```
<br/>
<br/>

## 7. 코루틴 취소를 잊음
실수: 더 이상 필요하지 않은 코루틴을 취소하지 않으면 리소스가 낭비되거나 의도치않은 부작용이 발생할 수 있다

발생 이유: 취소 논리를 간과하거나 사용자 정의 범위에서 제대로 처리하지 않았을 경우

회피 방법: 
- 구조화된 동시성을 사용하면 코루틴이 자동으로 취소된다
- 사용자 정의 코루틴을 사용하는 경우 적절한 시기에 취소해준다
```kotlin
val job = CoroutineScope(Dispatchers.IO) {
  // job
}

// 완료되면 취소
job.cancel()
```
<br/>
<br/>

## 8. 코루틴 내부 블로킹
실수: Thread.sleep() 과 같은 차단 호출이나 코루틴 내부의 무거운 계산을 적절한 Dispatcher 로 전환하지 않고 사용하여 기본 스레드를 차단하는 경우

발생 이유: 코루틴이 가벼운 스레드라는 것을 모르고 코루틴 내에서 블로킹 작업이 안전하다고 생각했을 경우

회피 방법:
- 코루틴 내부에서 호출을 차단하지 않아야한다
- Thread.sleep() 대신 delay() 과 같은 일시 중단 함수를 사용한다
- 무거운 계산은 Dispathers.Default 에게 맡긴다
```kotlin
launch(Dispatchers.IO) {
  // 실수
  Thread.sleep(1000)
}
launch(Dispatchers.IO) {
  // 회피
  delay(1000)
}
```
<br/>
<br/>

## 9. withContext 의 오용
실수: withContext 중첩을 불필요하게 하거나 목적을 오해하는 등 잘못 사용하면 읽기 어렵거나 비효율적인 코드가 생성된다

발생 이유: Context switching 및 withContext 범위에 대한 혼란

회피 방법: 
- withContext 특정 코드 블록의 컨텍스트를 전환하는 데 사용한다
- withContext 의 불필요한 중첩 호출을 하지 않는다
- withContext 블록은 가능한 작게 유지한다
```kotlin
val result = withContext(Dispatchers.IO) {
  // IO 작업
}
```
<br/>
<br/>

## 10. 코루틴을 제대로 테스트하지 않음
실수: 코루틴 기반 코드에 대한 적절한 테스트를 작성하지 않거나 코루틴을 올바르게 처리하지 않은 테스트를 작성하여 불안정하거나 신뢰할 수 없느 테스트가 생성됨

발생 이유: 비동기 코드를 테스트하는 것은 복잡하며 개발자는 코루틴에 사용할 수 있는 테스트 도구에 익숙하지 않을 수 있다

회피 방법:
- 단위 테스트 코루틴의 경우 runBlockingTest 또는 .runTest kotlinx-coroutines-test 를 사용한다
- TestCoroutineScope 및 TestCoroutineDispatcher 를 활용하여 테스트에서 코루틴 실행을 제어한다
- 지연시간이나 시간 초과가 있는 코드를 테스트할 때는 시간을 적절히 조절한다

```kotlin
@Test 
fun  testCoroutine () = runTest { 
    val result = mySuspendingFunction() 
    assertEquals(expectedResult, result) 
}
```

## 정리
- 항상 올바른 Dispatcher 를 사용한다
- 코루틴을 적절한 lifeCycle 에 연결한다
- 예외를 신중하게 처리한다
- Coroutine Scope 와 취소에 주의한다
- 코루틴 테스트 코드를 철저히 테스트한다
