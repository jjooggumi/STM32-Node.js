# 🌐 STM32 + Node.js + Web 기반 LED 제어 프로젝트


## 📌 프로젝트 개요

- **목적:** STM32 보드의 LED를 웹 브라우저 버튼 또는 음성 명령(한국어) 으로 켜고 끄기

- **구성:**
  - STM32 보드 (USART2를 통해 명령 수신)
  - Node.js 서버 (Serial 통신 → STM32 제어)
  - Web UI (버튼 클릭 + annyang 라이브러리를 이용한 음성 인식)

---
 
## 🏗 시스템 아키텍처
```txt
    flowchart LR
    User["웹 브라우저 / 음성 명령"] -->|HTTP 요청| NodeJS["Node.js (Express 서버)"]
    NodeJS -->|UART 전송| STM32["STM32 보드"]
    STM32 -->|GPIO 출력| LED["LD2 LED ON/OFF"]
```

---

## 🖥 STM32 펌웨어 (main.c)

- UART2 초기화 및 GPIO LD2 핀 설정
- 무한 루프에서 UART 데이터를 1바이트씩 수신 → 조건문으로 LED 제어
- **명령:**
  - '1' → LD2 ON
  - '0' → LD2 OFF
```c
while (1)
{
    HAL_UART_Receive(&huart2, &ch, 1, HAL_MAX_DELAY);

    if(ch == '1') {
        HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, 1);
        printf("LED ON\n");
    }
    else if(ch == '0') {
        HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, 0);
        printf("LED OFF\n");
    }
}
```

---

## 🌐 Node.js 서버 (app.js)

- Express.js 웹 서버와 serialport 라이브러리 사용
- 브라우저에서 /led_on, /led_off 요청 → 시리얼 포트로 문자 전송
```js
const sp = new SerialPort({ path: 'COM3', baudRate: 115200 });

app.get('/led_on', function(req, res) {
  sp.write('1\n\r', function(err){
    console.log('send: led on');
    res.end('LED ON\n');
  });
});

app.get('/led_off', function(req, res) {
  sp.write('0\n\r', function(err){
    console.log('send: led off');
    res.end('LED OFF\n');
  });
});

app.listen(3000, function() {
    console.log('listening on *:3000');
});
```

---

## 💻 웹 UI (index.html)

- 브라우저에서 LED On/Off 버튼 클릭 시 Node.js 서버로 HTTP 요청
- annyang 라이브러리를 이용해 음성 명령(불 켜, 불 꺼) 인식
- LED 상태 이미지를 동적으로 변경 (onRed.gif, offRed.gif)
```html
<a href="/led_on" target="response"><div onClick="led_on()">Turn On</div></a>
<a href="/led_off" target="response"><div onClick="led_off()">Turn Off</div></a>

<script>
  if (annyang) {
    var commands = {
      '불 켜': function() {
        response.location.href = ("http://localhost:3000/led_on");
        led_on();
      },
      '불 꺼': function() {
        response.location.href = ("http://localhost:3000/led_off");
        led_off();
      }
    };
    annyang.addCommands(commands);
    annyang.setLanguage('ko');
    annyang.start();
  }
</script>
```

---

## ⚡ 실행 방법

1. STM32 펌웨어 업로드
  - CubeIDE에서 main.c 빌드 및 플래싱
  - 시리얼 모니터로 RESET 메시지 확인
2. Node.js 서버 실행
```bash
node app.js
```
  - 콘솔에 `listening on *:3000` 확인
3. 브라우저 접속
  - `http://localhost:3000/index.html` 접속
  - 버튼 클릭 or 음성 명령 → STM32 LD2 LED 제어
