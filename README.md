# 🏠 My Smart Home Dashboard

ESP32와 웹 브라우저 간의 **블루투스 BLE NUS(Nordic UART Service)** 통신을 활용하여 스마트홈 기기를 무선으로 제어하고 센서 데이터를 실시간으로 모니터링하는 **스마트홈 웹 대시보드 프로젝트**입니다.

---

## 📌 0. 프로젝트 정보

- **프로젝트명**: `[내 이름]`의 스마트홈 대시보드
- **제작자**: `[내 이름]`
- **BLE 기기명**: `ESP_[내 기기명]` (웹 Bluetooth 스캔 필터 `ESP_` 매칭)
- **OLED 비트맵 이미지**: `/image/choonsik3.pbm` (`choonsik.pbm`)
- **메인 테마 컬러**: 노란색 계열

---

## 🔩 1. 하드웨어 시스템 사양 (Hardware Specifications)

ESP32 DEVKIT V1(30핀) 마이크로컨트롤러를 기반으로 아래와 같이 센서 및 액추에이터가 연결되어 있습니다.

### 1.1. 표준 핀 결선 맵 (Pin Mapping)

| 하드웨어 / 센서 | 모델 | ESP32 GPIO 핀 | 입출력 구분 | 비고 |
| :--- | :--- | :--- | :--- | :--- |
| **터치 센서 1번** | TTP223 | `D17` | 디지털 입력 | RGB LED 빨강 점등 |
| **터치 센서 2번** | TTP223 | `D5` | 디지털 입력 | RGB LED 노랑 점등 |
| **터치 센서 3번** | TTP223 | `D18` | 디지털 입력 | RGB LED 보라 점등 |
| **터치 센서 4번** | TTP223 | `D19` | 디지털 입력 | RGB LED 소등 |
| **RGB LED (빨강)** | 공통 캐소드 RGB | `D25` | 디지털 출력 | |
| **RGB LED (초록)** | 공통 캐소드 RGB | `D26` | 디지털 출력 | |
| **RGB LED (파랑)** | 공통 캐소드 RGB | `D27` | 디지털 출력 | |
| **조도 센서** | CdS 광도전 셀 | `D36` (ADC) | 아날로그 입력 | 어두워지면 서보모터 차단막 동작 |
| **서보 모터** | SG90 | `D13` | PWM 출력 | 블라인드/문 제어 |
| **온습도 센서** | DHT11 | `D4` | 디지털 입력 | (동작에 따라 `D14` 등으로 조율 가능) |
| **피에조 부저** | 패시브 부저 | `D23` | PWM 주파수 출력 | 멜로디 연주 |
| **I2C LCD (SDA)** | HD44780 I2C (0x27) | `D21` | I2C 통신 | 온습도 및 조도 텍스트 출력 |
| **I2C LCD (SCL)** | HD44780 I2C (0x27) | `D22` | I2C 통신 | |
| **OLED (SDA)** | SSD1306 (0x3C) | `D21` | I2C 통신 | 춘식이 비트맵 캐릭터 출력 |
| **OLED (SCL)** | SSD1306 (0x3C) | `D22` | I2C 통신 | |

---

## 📡 2. 블루투스 통신 및 제어 규격 (BLE & Control Spec)

웹 대시보드와 ESP32는 **NUS (Nordic UART Service)** 프로토콜을 사용해 양방향 통신을 처리합니다.

### 2.1. NUS UUID 정보

- **NUS 서비스 UUID**: `6e400001-b5a3-f393-e0a9-e50e24dcca9e`
- **RX (Write) UUID**: `6e400002-b5a3-f393-e0a9-e50e24dcca9e` (웹 ➡️ ESP32)
- **TX (Notify) UUID**: `6e400003-b5a3-f393-e0a9-e50e24dcca9e` (ESP32 ➡️ 웹)

### 2.2. 웹 ➡️ ESP32 제어 명령어 (ASCII 1글자)

| 명령어 | 기능 | ESP32 하드웨어 동작 |
| :---: | :--- | :--- |
| `'1'` | 온습도 조회 | DHT11 측정 ➡️ LCD 출력 + 웹으로 온도/습도 데이터 송신 |
| `'2'` | 조도 조회 | CdS ADC 측정 ➡️ LCD 출력 + 웹으로 조도값 송신 |
| `'3'` | LCD 켜기 | LCD 백라이트 ON |
| `'4'` | LCD 끄기 | LCD 백라이트 OFF |
| `'5'` | 멜로디 1 재생 | 부저로 "학교종이 땡땡땡" 주파수 출력 |
| `'6'` | 멜로디 2 재생 | 부저로 "반짝반짝 작은별" 주파수 출력 |
| `'7'` | 전등 켜기 | RGB LED 전체 HIGH (백색 점등) |
| `'8'` | 전등 끄기 | RGB LED 전체 LOW (소등) |
| `'9'` | OLED 캐릭터 출력 | OLED 디스플레이에 춘식이 이미지 (`/image/choonsik3.pbm`) 드로잉 |

### 2.3. ESP32 ➡️ 웹 센서 피드백 패킷 포맷

- **온습도 데이터 (`'1'` 요청 시)**
  ```text
  temp : [온도값]\n
  humi : [습도값]\n
  (예: "temp : 24\n", "humi : 52\n")
  ```
- **조도 데이터 (`'2'` 요청 시)**
  ```text
  [조도값]\n
  (예: "3850\n")
  ```

---

## 🚀 3. 설치 및 실행 가이드 (Quick Start)

### 3.1. ESP32 마이크로파이썬 설정

1. ESP32에 [MicroPython 펌웨어](https://micropython.org/)를 플래싱합니다.
2. `smartHome.py` 코드를 `main.py`로 이름을 바꾸거나 툴(Thonny 등)을 사용하여 ESP32에 업로드합니다.
3. 필요한 라이브러리들(`ble_library.py`, `lcd_api.py`, `i2c_lcd.py`, `ssd1306.py` 등)과 OLED 이미지 파일(`choonsik.pbm` 또는 `/image/choonsik3.pbm`)이 ESP32 내부 스토리지에 포함되어 있는지 확인합니다.

### 3.2. 웹 대시보드 로컬 테스트 방법

Web Bluetooth API는 **보안 컨텍스트(HTTPS)** 또는 **localhost** 환경에서만 정상적으로 동작합니다. `file://` 경로로 단순 더블클릭 실행 시 블루투스 기능이 동작하지 않습니다.

1. 프로젝트 폴더의 터미널을 실행합니다.
2. 아래 명령어로 로컬 파이썬 웹 서버를 구동합니다.
   ```bash
   python -m http.server 8000
   ```
3. 크롬(Chrome) 또는 엣지(Edge) 브라우저를 열고 아래 주소로 접속합니다.
   - `http://localhost:8000`
4. **[기기 연결]** 버튼을 클릭하여 `ESP_[내 기기명]` 디바이스를 검색하고 페어링합니다.

### 3.3. 모바일 접속 및 Vercel 무료 배포 방법

무선 환경(모바일 스마트폰)에서 제어하기 위해, 보안 HTTPS 접속을 지원하는 **Vercel** 클라우드로 무중단 배포하여 테스트할 수 있습니다.

1. 본인 GitHub 계정에 원격 저장소를 생성하고 프로젝트를 업로드합니다.
   ```bash
   git init
   git add .
   git commit -m "feat: 스마트홈 대시보드 구성 완료"
   git branch -M main
   git remote add origin https://github.com/[GitHub-ID]/[Repository-Name].git
   git push -u origin main
   ```
2. [Vercel](https://vercel.com/) 가입 후 해당 GitHub 저장소를 연동하여 클릭 한 번으로 배포합니다.
3. **모바일 연결**:
   - 🤖 **안드로이드**: 크롬(Chrome) 브라우저로 배포된 HTTPS 주소 접속 후 연결 및 제어
   - 🍏 **아이폰(iOS)**: App Store에서 **Bluefy** (Web BLE 지원 브라우저) 앱을 설치하고, 해당 앱의 주소창에 배포된 HTTPS 주소를 입력하여 연결 및 제어

---

## 📁 4. 프로젝트 폴더 구조

```text
s_SmartHome/
├── index.html                  # 웹 블루투스 기반 스마트홈 대시보드
├── smartHome.py                # ESP32 마이크로파이썬 메인 컨트롤러 코드
├── PRD_s.md                    # 제품 요구사항 정의서 (기본 사양 설계도)
├── student_web_dashboard_guide.md # 웹 대시보드 제작 및 배포 가이드 문서
├── choonsik.pbm                # OLED 출력용 단색 흑백 비트맵 이미지
├── 스마트홈_회로연결.txt        # 하드웨어 회로 배선 설명서
└── img/                        # 웹 디자인 리소스 및 기타 이미지 폴더
```
