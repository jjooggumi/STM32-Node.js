# 🌐 STM32 + Node.js + Web 기반 LED 제어 프로젝트

## 📌 프로젝트 개요

- **목적:** STM32 보드의 LED를 웹 브라우저 버튼 또는 음성 명령(한국어) 으로 켜고 끄기

- **구성:**
  - STM32 보드 (USART2를 통해 명령 수신)
  - Node.js 서버 (Serial 통신 → STM32 제어)
  - Web UI (버튼 클릭 + annyang 라이브러리를 이용한 음성 인식)
 
## 🏗 시스템 아키텍처
```txt
    flowchart LR
    User["웹 브라우저 / 음성 명령"] -->|HTTP 요청| NodeJS["Node.js (Express 서버)"]
    NodeJS -->|UART 전송| STM32["STM32 보드"]
    STM32 -->|GPIO 출력| LED["LD2 LED ON/OFF"]
```

## 🖥 STM32 펌웨어 (main.c)

- UART2 초기화 및 GPIO LD2 핀 설정
- 무한 루프에서 UART 데이터를 1바이트씩 수신 → 조건문으로 LED 제어
- **명령:**
  - '1' → LD2 ON
  - '0' → LD2 OFF

