<img src="https://capsule-render.vercel.app/api?type=waving&color=00C3FF&height=150&section=header" width="1000" />

<div align="center">
<h1 style="font-size: 36px;">🖥️ Spring Boot & MySQL 성능 테스트를 위한 <br>Prometheus-Grafana 통합 모니터링</h1>
</div>
</br>

## 목차
1. [🙆🏻‍♂️ 팀원](#%EF%B8%8F-팀원)
2. [🖥️ 프로젝트 개요: Spring Boot & MySQL 성능 테스트를 위한 Prometheus-Grafana 통합 모니터링](#%EF%B8%8F-프로젝트-개요-spring-boot--mysql-성능-테스트를-위한-prometheus-grafana-통합-모니터링--)
3. [💻개발환경](#-개발환경)
4. [🛠️ Spring Boot 애플리케이션 성능 및 부하 테스트](#%EF%B8%8F-spring-boot-애플리케이션-성능-및-부하-테스트)
5. [🛠️ MySQL 성능 및 부하 테스트](#%EF%B8%8F-mysql-성능-및-부하-테스트)

---

## 🙆🏻‍♂️ 팀원

#### 팀명 : Puppynetes

|<img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/103871252?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/179544856?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/98368034?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|:-:|
|김민성<br/>[@minsung159357](https://github.com/minsung159357)|김우현<br/>[@woody6624](https://github.com/woody6624)|이은준<br/>[@2EunJun](https://github.com/2EunJun)|장수현<br/>[@Aunsxm](https://github.com/Aunsxm)|

---

## 🖥️ 프로젝트 개요: Spring Boot & MySQL 성능 테스트를 위한 Prometheus-Grafana 통합 모니터링 <br>

**본 프로젝트의 목표는 Prometheus와 Grafana를 활용하여 시스템 및 애플리케이션의 모니터링 환경을 구축하고, 실시간 상태 시각화와 부하 테스트 대응 능력을 확인하는 것입니다.**

📌 주요 내용
- Prometheus + Grafana 모니터링 스택 구성

- Node, MySQL, Spring Boot의 메트릭 수집 및 시각화

- stress를 이용한 부하 테스트 수행

- Spring Boot 애플리케이션과 /actuator/prometheus 엔드포인트 연동

- 시스템 재부팅 후 대시보드 상태 유지 여부 확인 및 예외 상황 처리



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

---



## 🛠️ Spring Boot 애플리케이션 성능 및 부하 테스트

### 1. build.gradle에 Prometheus와 Swagger 관련 의존성 설정

 **`springdoc-openapi-starter-webmvc-ui`**

- Swagger UI를 자동 설정하여 API 문서를 제공

 **`micrometer-registry-prometheus`**

- 애플리케이션 메트릭을 Prometheus 형식으로 노출해 Grafana에서 모니터링 가능

**`spring-boot-starter-actuator`**

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

### 7. Swagger 기반 API 요청을 활용한 부하 테스트

주소 : `http://IP주소:8080/swagger-ui/index.html#/`
![image (8)](https://github.com/user-attachments/assets/74d70a75-b16b-48f0-8cf6-a7cc34fe506f)

**⚠️ 주의사항**

객체를 생성만 하고 삭제하지 않을 경우, Spring Boot 애플리케이션에서 OutOfMemoryError(OOM) 가 발생해 자동으로 종료될 수 있음

<details>
<summary>오류 로그</summary>
	
```bash	
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
---
</details>


### 8. Grafana 대시보드를 통한 모니터링 확인

1. [localhost:3000](http://localhost:3000) 으로 그라파나 접속
2. Dashboards → Import → 4701(스프링 부트 어플리케이션)입력 후 Load 버튼 클릭
3. 과부화 테스트의 결과 확인 
![image (9)](https://github.com/user-attachments/assets/891fd9d1-0875-4b24-abf7-f641845f383d)

## 🛠️ MySQL 성능 및 부하 테스트


### :compass: MySQL 모니터링 및 부하 테스트 전체 구성 흐름

```
sysbench (부하 생성)
    ↓
MySQL (부하 받음)
    ↓
mysqld_exporter (메트릭 수집, 예: 쿼리 수, 커넥션 수 등)
    ↓
Prometheus (수집)
    ↓
Grafana (시각화)
```
---

## 1. `mysqld_exporter` 정상 실행 확인

```bash
sudo mysqld_exporter --config.my-cnf="/etc/.my.cnf" --web.listen-address=":9104"
```

- 실행 후 브라우저에서 다음 주소에 접속하여 메트릭 확인  
  👉 [http://localhost:9104/metrics](http://localhost:9104/metrics)

---

## 2. Prometheus에서 `mysqld_exporter` 추가

`prometheus.yml` 파일에 다음 블록 추가

```yaml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']
```

- Prometheus는 이 설정을 통해 `mysqld_exporter`가 수집한 MySQL 메트릭을 가져옴옴

---

## 3. Grafana에서 MySQL 대시보드 가져오기

1. Grafana에 로그인
2. `Dashboards > Import` 메뉴로 이동
3. 다음 대시보드 ID 입력: **7362 (MySQL Overview)**
4. 데이터 소스로 **Prometheus** 선택

---

## 4. `sysbench` 설치

```bash
sudo apt update
sudo apt install sysbench -y
```

- `sysbench`는 MySQL 안에서 실행하는 게 아니라, **리눅스 쉘(터미널)**에서 실행하는 도구
- MySQL 서버에 외부에서 접속하여 부하를 주는 프로그램

---

## 5. 사전 준비: 테이블 생성

```bash
sysbench \
  --db-driver=mysql \
  --mysql-user=user01 \
  --mysql-password=user01 \
  --mysql-db=fisa \
  --tables=10 \
  --table-size=100000 \
  oltp_read_write prepare
```

- `--tables=10`: 총 10개 테이블 생성  
- `--table-size=100000`: 각 테이블에 10만 개의 row → 총 100만개 row

---

## 6. 부하 테스트 실행

```bash
sysbench \
  --db-driver=mysql \
  --mysql-user=user01 \
  --mysql-password=user01 \
  --mysql-db=fisa \
  --tables=10 \
  --table-size=100000 \
  --threads=32 \
  --time=300 \
  oltp_read_write run
```

- 32개의 스레드로 300초 동안 테스트  
- 결과에는 **TPS (Transactions Per Second)**, **응답 시간**, **에러율** 등이 출력

---

## 7. Grafana Dashboard 확인

- Grafana에서 대시보드로 실시간 메트릭 확인
- TPS, 커넥션 수, 쿼리 속도, 캐시 히트율 등 시각화 확인

![image](https://github.com/user-attachments/assets/7fc2dc05-f29e-4b17-808b-741f51b1f1b3)


---

## 8. 테스트 후 정리

```bash
sysbench \
  --db-driver=mysql \
  --mysql-user=user01 \
  --mysql-password=user01 \
  --mysql-db=fisa \
  --tables=10 \
  oltp_read_write cleanup
```

- 테스트가 끝난 뒤 생성된 데이터 삭제
```bash
sysbench \
  --db-driver=mysql \
  --mysql-user=user01 \
  --mysql-password=user01 \
  --mysql-db=fisa \
  --tables=10 \
  oltp_read_write cleanup
```

---
