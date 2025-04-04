<img src="https://capsule-render.vercel.app/api?type=waving&color=00C3FF&height=150&section=header" width="1000" />

<div align="center">
<h1 style="font-size: 36px;">🌱 Docker & K8s를 활용한<br> SpringBoot App 배포</h1>
</div>
</br>

## 목차
1. [🙆🏻‍♂️ 팀원](#%EF%B8%8F-팀원)
2. [🌱 프로젝트 개요: Docker & K8s를 활용한 SpringBoot App 배포](#-프로젝트-개요-docker--k8s를-활용한-springboot-app-배포)
3. [🛠️ 실습 과정](#%EF%B8%8F-실습-과정)
4. [📖 배운 점](#-배운-점)
5. [💜 회고](#-회고)

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

