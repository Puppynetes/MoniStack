<img src="https://capsule-render.vercel.app/api?type=waving&color=00C3FF&height=150&section=header" width="1000" />

<div align="center">
<h1 style="font-size: 36px;">🌱 Docker & K8s를 활용한<br> SpringBoot App 배포</h1>
</div>
</br>

## 목차
1. [🙆🏻‍♂️ 팀원](#%EF%B8%8F-팀원)
2. [🌱 프로젝트 개요: Docker & K8s를 활용한 SpringBoot App 배포](#-프로젝트-개요-docker--k8s를-활용한-springboot-app-배포)
3. [💻개발환경](#-개발환경)
4. [🛠️ 실습 과정](#%EF%B8%8F-실습-과정)
5. [📖 배운 점](#-배운-점)
6. [💜 회고](#-회고)

## 🙆🏻‍♂️ 팀원

#### 팀명 : Puppynetes

|<img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/103871252?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/179544856?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/98368034?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|:-:|
|김민성<br/>[@minsung159357](https://github.com/minsung159357)|김우현<br/>[@woody6624](https://github.com/woody6624)|이은준<br/>[@2EunJun](https://github.com/2EunJun)|장수현<br/>[@Aunsxm](https://github.com/Aunsxm)|

---

📝 프로젝트 개요
시스템 및 애플리케이션 모니터링 환경 구축을 목표
Prometheus와 Grafana를 기반으로 상태 시각화 및 부하 테스트 대응 확인

주요 내용
 Prometheus + Grafana 모니터링 스택 구성

 Node, MySQL, Spring Boot 메트릭 수집 및 시각화

 stress를 활용한 부하 테스트 수행

 Spring Boot /actuator/prometheus 연동

 재부팅 후 대시보드 상태 유지 여부 확인 및 예외 처리

 ---

### **설치 방식 선택 이유** <br>

1️⃣ **Prometheus - 바이너리 설치** <br>
- 설정 파일(prometheus.yml) 직접 수정하고 테스트하기 편함

 - 구조가 단순해서 디버깅, 포트 확인, 서비스 등록 등 제어권이 높음

- 패키지 매니저 의존 없이 버전 선택 자유로움

- 운영환경에 맞게 가볍게 커스터마이징 가능

👉 설치 과정이 조금 수동적이긴 하지만, 구성 구조를 정확히 이해하고 직접 다루기에 적합 <br>

2️⃣ **Grafana - APT 설치** <br>
- APT를 통해 빠르고 안정적으로 설치 가능

- 시스템 서비스 자동 등록 → systemctl로 바로 관리

- 의존성 설치도 자동 처리돼서 세팅 간편

- 업데이트도 apt upgrade로 손쉽게 가능

👉 빠르게 설치하고 바로 웹에서 대시보드 구성 시작할 수 있음.

---
## 💻 개발환경  

| 항목         | 사용 기술 및 도구          | 설명                                      |
|-------------|------------------|--------------------------------|
| **OS**       | Ubuntu           | 메인 서버 |
| **가상화**   | VMware, VirtualBox | 가상 머신 실행 및 관리 |
| **백엔드**   | Spring Boot      | 서버 애플리케이션   |
| **API 문서화** | Swagger          | API 명세 자동화 및 UI 제공 |
| **데이터베이스** | MySQL            | 데이터 저장 및 관리 |
| **모니터링**  | Prometheus       | 애플리케이션 및 시스템 메트릭 수집 |
| **시각화**   | Grafana          | Prometheus 데이터를 기반으로 시각화 |

## Spring Boot 애플리케이션 성능 및 부하 테스트

### 1. build.gradle에 Prometheus와 Swagger 관련 의존성 설정

1️⃣ **`springdoc-openapi-starter-webmvc-ui`**

- Swagger UI를 자동 설정하여 API 문서를 제공

2️⃣ **`micrometer-registry-prometheus`**

- 애플리케이션 메트릭을 Prometheus 형식으로 노출해 Grafana에서 모니터링 가능

3️⃣ **`spring-boot-starter-actuator`**

- `/actuator/metrics`, `/actuator/prometheus` 등 애플리케이션 상태 및 성능 모니터링용 엔드포인트 제공

### 2. Swagger 설정을 위한 Config 파일 생성

```java
@Configuration    // 스프링 실행시 설정파일 읽어드리기 위한 어노테이션
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI().addServersItem(new Server().url("/").description("부하테스트"))
                .info(apiInfo());
    }

    private Info apiInfo() {
        return new Info()
                .title("puppynetes")
                .description("puppynetes팀 부하테스트 - API 명세서")
                .version("1.0.0");
    }
}
```

### 3. 부하 테스트용 객체 생성·조회·삭제 API 구현

```java
@Service
@RestController("/api")
public class StreamService {

    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    private final List<Object> objectStorage = new CopyOnWriteArrayList<>();
    private ScheduledFuture<?> loadTask;

    // 1분 동안 객체를 계속 생성하는 API
    @GetMapping("/start-load")
    public String startHeavyLoad() {
        if (loadTask != null && !loadTask.isDone()) {
            return "부하 테스트가 이미 실행 중입니다.";
        }

        loadTask = scheduler.schedule(() -> {}, 1, TimeUnit.MINUTES); // 1분 타이머

        new Thread(() -> {
            long end = System.currentTimeMillis() + 60_000; // 1분 동안
            while (System.currentTimeMillis() < end) {
                // 계속해서 객체를 생성
                for (int i = 0; i < 1000; i++) {
                    objectStorage.add(new byte[1024 * 10]); // 10KB씩 계속 생성
                }

                try {
                    Thread.sleep(50); 
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }).start();

        return "1분 동안 객체를 계속 생성하는 부하 테스트를 시작했습니다.";
    }

    // 생성된 객체를 전부 제거하는 API
    @GetMapping("/stop-load")
    public String stopHeavyLoad() {
        objectStorage.clear(); // 메모리 비우기
        if (loadTask != null) {
            loadTask.cancel(true);
        }
        return "부하 테스트를 종료하고, 객체를 전부 제거했습니다.";
    }

    // 현재 메모리에 남아있는 객체 수 확인
    @GetMapping("/load-status")
    public String loadStatus() {
        return "현재 생성된 객체 수: " + objectStorage.size();
    }
}

```

### 4. Spring Boot 애플리케이션을 JAR 파일로 빌드한 후, Ubuntu 서버로 배포

Window CMD 사용

cd 프로젝트 경로 → `gradlew.bat clean bootJar`  → JAR 생성 완료(위치: 프로젝트/build/libs/xx.jar)

JAR파일을 MobaXterm을 이용하여 우분투 서버로 옮김

⚠️ 이때 우분투 서버에는 JDK 설치가 되어있어야 한다.

### 5. Prometheus 설정 파일 수정 후 서비스 재시작

```bash
sudo vi /etc/prometheus/prometheus.yml
######################################################
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: 'mysql-exporter'
    static_configs:
      - targets: ['localhost:9104']

	# ⚠️ 스프링 부트에 대한 설정
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']  # <== 여기 Spring Boot 앱 주소!

########################################################################
sudo systemctl restart prometheus.service
```

### 6. Prometheus에서 Spring Boot 앱 상태(Target health) 확인

![image (7)](https://github.com/user-attachments/assets/4d6f65b3-f4fd-4b9f-9947-2c3fd1d36886)

### 7. Swagger 기반 API 요청을 활용한 부하 테스트

주소 : `http://IP주소:8080/swagger-ui/index.html#/`
![image (8)](https://github.com/user-attachments/assets/74d70a75-b16b-48f0-8cf6-a7cc34fe506f)


### 8. Grafana 대시보드를 통한 모니터링 확인

1. [localhost:3000](http://localhost:3000) 으로 그라파나 접속
2. Dashboards → Import → 4701(스프링 부트 어플리케이션)입력 후 Load 버튼 클릭
3. 과부화 테스트의 결과 확인 
![image (9)](https://github.com/user-attachments/assets/891fd9d1-0875-4b24-abf7-f641845f383d)


### 9. 주의사항 - 객체를 생성만 하고 삭제하지 않을 경우

스프링 부트 애플리케이션이 **OutOfMemoryError(OOM)** 발생 후 자동 종료될 수 있음.
```java
Exception in thread "Thread-1" java.lang.OutOfMemoryError: Java heap space
        at java.base/java.util.Arrays.copyOf(Arrays.java:3481)
        at java.base/java.util.concurrent.CopyOnWriteArrayList.add(CopyOnWriteArrayList.java:432)
        at fisa.test.service.StreamService.lambda$startHeavyLoad$1(StreamService.java:33)
        at fisa.test.service.StreamService$$Lambda$1366/0x000078e4ec609638.run(Unknown Source)
        at java.base/java.lang.Thread.run(Thread.java:840)

Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "http-nio-8080-Poller"
Exception in thread "Catalina-utility-2" java.lang.OutOfMemoryError: Java heap space
Exception in thread "http-nio-8080-exec-3" java.lang.OutOfMemoryError: Java heap space
2025-04-04T06:00:44.866Z ERROR 20588 --- [test] [alina-utility-1] o.a.coyote.http11.Http11NioProtocol      : Error processing async timeouts

java.util.concurrent.ExecutionException: java.lang.OutOfMemoryError: Java heap space
        at java.base/java.util.concurrent.FutureTask.report(FutureTask.java:122) ~[na:na]
        at java.base/java.util.concurrent.FutureTask.get(FutureTask.java:191) ~[na:na]
        at org.apache.coyote.AbstractProtocol.startAsyncTimeout(AbstractProtocol.java:660) ~[tomcat-embed-core-10.1.39.jar!/:na]
        at org.apache.coyote.AbstractProtocol.lambda$start$0(AbstractProtocol.java:646) ~[tomcat-embed-core-10.1.39.jar!/:na]
        at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:539) ~[na:na]
        at java.base/java.util.concurrent.FutureTask.runAndReset(FutureTask.java:305) ~[na:na]
        at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:305) ~[na:na]
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136) ~[na:na]
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635) ~[na:na]
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63) ~[tomcat-embed-core-10.1.39.jar!/:na]
        at java.base/java.lang.Thread.run(Thread.java:840) ~[na:na]
Caused by: java.lang.OutOfMemoryError: Java heap space

Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "pool-2-thread-1"

Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "Catalina-utility-1"

```
