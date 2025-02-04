## 다운로드 매니저
- Android 시스템에서 파일 다운로드 작업을 처리하고 관리하기 위한 시스템 서비스
- 다운로드 매니저를 호라용하면 백그라운드에서 안정적으로 파일을 다운로드하고 다운로드 상태를 추적하거나 알림을 통해 사용자에게 정보를 제공할 수 있음

</br>
</br>

### 주요 기능
1. 백그라운드 다운로드: 앱이 종료되더라도 다운로드가 계속됨

2. 알림 제공: 다운로드 상태( 진행중, 완료됨 )를 알림으로 표시할 수 있음

3. 파일 저장 관리: 다운로드한 파일을 지정된 디렉터리에 저장

4. 다운로드 상태 확인: 다운로드 진행률, 상태 코드, 파일 크기 등을 확인할 수 있음

5. 다양한 요청 옵션: 파일이름, 저장 위치, 네트워크 연결 조건 등을 설정할 수 있음

</br>
</br>

### 예제 코드
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val downloadButton: Button = findViewById(R.id.btnDownload)
        downloadButton.setOnClickListener {
            val fileUrl =
                "https://example.com/sample.pdf" // 다운로드할 파일 URL
            val fileName = "sample_file.pdf"

            startDownload(fileUrl, fileName)
        }
    }

    private fun startDownload(url: String, fileName: String) {
        val request = DownloadManager.Request(Uri.parse(url)).apply {
            setTitle("파일 다운로드 중...")
            setDescription("다운로드 중입니다: $fileName")
            setDestinationInExternalPublicDir(Environment.DIRECTORY_DOWNLOADS, fileName)
            setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED)
            setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI or DownloadManager.Request.NETWORK_MOBILE)
        }

        val downloadManager = getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
        val downloadId = downloadManager.enqueue(request)

        Toast.makeText(this, "다운로드 요청 완료 (ID: $downloadId)", Toast.LENGTH_SHORT).show()
    }
}
```
- DownloadManager.Request() 를 통해 다운로드 요청을 생성
- 다운로드 파일 경로는 setDestinationInExternalPublicDir() 로 설정 가능
- 네트워크 유형( NETWORK_WIFI, NETWORK_MOBILE ) 설정 가능
- 다운로드 완료 후 알림 표시

</br>
</br>

### DownloadManager 주요 메서드
- enqueue(request: DownloadManager.Request): 다운로드 요청 큐에 추가
- remove(downloadId: Int): 특정 다운로드 작업 취소
- query(query: DownloadManager.Query): 다운로드 상태 조회

</br>
</br>
