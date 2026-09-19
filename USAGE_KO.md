# Google ARTEMIS 사용 가이드 (로컬 설치본)

이 문서는 **이 PC에 설치된 ARTEMIS** 기준으로 작성되었습니다.  
공식 README(`README.md`)를 보완하는 **실무용 빠른 사용법**입니다.

---

## 1. 이 PC에서의 설치 정보

| 항목 | 값 |
|------|-----|
| 설치 경로 | `D:\Workspace\artemis` |
| Python venv | `D:\Workspace\artemis\.venv` |
| 전역 CLI | `artemis`, `artemis-admin` (`uv tool install -e .`) |
| Cursor MCP 설정 | `C:\Users\Developer\.cursor\mcp.json` (`artemis` 서버) |
| Cursor 규칙 | `C:\Users\Developer\.cursor\rules\artemis.mdc` |
| 환경 설정 파일 | `D:\Workspace\artemis\.env` |
| Showcase UI | `http://localhost:8000` (기본 포트) |

### 이미 설치된 도구체인

- `adb` — Android SDK platform-tools
- `scrcpy` — 화면 미러링
- `ffmpeg` — 녹화/재생
- `uv` — Python 패키지/런타임 관리
- Angular Showcase UI — 빌드 완료

---

## 2. ARTEMIS가 하는 일

자연어로 Android 기기(또는 에뮬레이터)를 조작하는 **자율 모바일 에이전트**입니다.

예시:

> “설정 앱을 열고 배터리 잔량을 알려줘”

흐름:

1. 화면을 보고 (스크린샷 / UI 계층)
2. 다음 액션을 결정
3. ADB로 탭·스와이프·입력 실행
4. 결과를 확인한 뒤 다음 단계로 진행

**Flash** (빠르고 가벼움) / **Pro** (계획·검증·진단 포함) 두 프로필을 지원합니다.

---

## 3. 사전 준비 (필수)

### 3.1 API 키 설정

`.env`에 **최소 1개** LLM 키를 넣어야 합니다.

```powershell
cd D:\Workspace\artemis
notepad .env
```

지원 키 (예시):

```env
GEMINI_API_KEY=...
# 또는
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
```

대화형 마법사로 설정하려면:

```powershell
uv run artemis init
```

### 3.2 Android 기기 또는 에뮬레이터

**실기기**

1. 개발자 옵션 → **USB 디버깅** ON
2. USB 연결 후 PC에서 RSA 허용
3. 연결 확인:

```powershell
# PATH에 adb가 없으면 SDK 경로를 사용
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" devices
```

`device`로 보이면 OK. `unauthorized`면 폰에서 허용하세요.

**에뮬레이터**

Android Studio AVD를 실행한 뒤 동일하게 `adb devices`로 확인합니다.

### 3.3 (권장) Accessibility Helper

첫 태스크 시 자동 설치됩니다. 미리 설치하면 첫 실행이 빨라집니다.

```powershell
cd D:\Workspace\artemis
uv run artemis helper install
uv run artemis helper status
```

제거:

```powershell
uv run artemis helper uninstall
```

---

## 4. 상태 점검 (`doctor`)

문제 발생 시 **가장 먼저** 실행하세요.

```powershell
cd D:\Workspace\artemis
uv run artemis doctor
```

확인 항목: Python, `.env`, LLM 키, ADB, 연결된 기기, scrcpy/ffmpeg 등.

Cursor에서 MCP가 안 되면 채팅으로 다음과 같이 요청해도 됩니다.

> Artemis `mobile_diagnose`로 환경 진단해줘

---

## 5. 사용 방법 3가지

### 방법 A — 웹 콘솔 (초보자 추천)

```powershell
cd D:\Workspace\artemis
.\start.bat
```

또는 UI만:

```powershell
uv run artemis ui --port 8000 --open
```

브라우저에서 `http://localhost:8000` 이 열립니다.

| 기능 | 설명 |
|------|------|
| 기기 연결 마법사 | 디바이스 선택 / 연결 상태 |
| 실시간 미러링 | 화면 보기 |
| Prompt Dock | 자연어로 태스크 실행 |
| Replay | 이전 실행 재생 |

서버 제어:

```powershell
uv run artemis status
uv run artemis restart
uv run artemis stop
```

### 방법 B — CLI (터미널에서 바로 실행)

```powershell
cd D:\Workspace\artemis

# Flash (기본, 빠른 반응형)
uv run artemis run "설정 앱을 열고 배터리 메뉴로 들어가서 잔량을 알려줘" --profile flash

# Pro (계획 + 검증, 긴 시나리오)
uv run artemis run "앱을 설치하고 로그인 화면까지 이동한 뒤 팝업이 있는지 확인해줘" --profile pro
```

특정 기기 지정:

```powershell
uv run artemis run "홈 화면으로 이동해줘" --device-serial emulator-5554
```

### 방법 C — Cursor IDE (MCP)

이미 이 PC에는 Cursor용 MCP가 설치되어 있습니다.

1. Cursor에서 **Reload Window** (`Developer: Reload Window`)
2. 채팅에서 자연어로 요청:

> 연결된 안드로이드에 PitchTypes APK를 설치하고 앱을 열어줘  
> 로그인 화면 스크린샷을 찍고 예상치 못한 팝업이 있는지 확인해줘

사용 가능한 MCP 도구:

| 도구 | 용도 |
|------|------|
| `mobile_run_task` | 자연어 태스크를 백그라운드로 실행 (flash/pro) |
| `mobile_manage_task` | 상태 조회 / 중지 / 지시 주입 |
| `mobile_get_device_state` | 스크린샷 또는 UI hierarchy |
| `mobile_inspect_trace` | 실행 트레이스·추론 과정 검사 |
| `mobile_diagnose` | 환경 진단 + 복구 제안 |

MCP 재설치 (설정이 깨졌을 때):

```powershell
cd D:\Workspace\artemis
uv run artemis mcp --install cursor
```

---

## 6. Flash vs Pro 선택 가이드

| | **Flash** | **Pro** |
|---|-----------|---------|
| 속도 | 단계당 약 3–5초 | 단계당 약 15–40초 |
| 적합한 작업 | 단순 UI 이동, 설정 확인, 탭·입력 | 긴 E2E, 계획/검증, 크래시 진단 |
| 계획/체크포인트 | 없음 | Planner + Checker |
| ADB 진단 | 제한적 | 가능 |
| 기본값 | `.env`의 `ARTEMIS_DEFAULT_PROFILE=flash` | `--profile pro`로 지정 |

**추천 규칙**

- “버튼 눌러 / 화면 열어 / 텍스트 읽어” → **Flash**
- “설치→로그인→결제까지 검증하고 리포트” → **Pro**

---

## 7. PitchTypes(Flutter)와 함께 쓰는 예시

이 워크스페이스(`PitchTypes`)와 조합하는 전형적인 흐름입니다.

```powershell
# 1) Flutter APK 빌드
cd D:\Workspace\flutter\PitchTypes
flutter build apk --debug

# 2) 기기에 설치 (adb)
$apk = "D:\Workspace\flutter\PitchTypes\build\app\outputs\flutter-apk\app-debug.apk"
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" install -r $apk

# 3) ARTEMIS로 UI 검증
cd D:\Workspace\artemis
uv run artemis run "PitchTypes 앱을 실행하고 홈에서 게임 모드를 선택한 뒤 투구 화면이 보이는지 확인해줘" --profile flash
```

Cursor 채팅 예시:

> PitchTypes 최신 debug APK를 빌드·설치한 뒤, ARTEMIS로 홈 → 게임 모드 진입까지 검증하고 스크린샷을 남겨줘

---

## 8. 자주 쓰는 명령 치트시트

```powershell
cd D:\Workspace\artemis

# 환경
uv run artemis doctor
uv run artemis init
uv run artemis helper install

# UI
.\start.bat
uv run artemis ui --port 8000 --open
uv run artemis status
uv run artemis stop

# 실행
uv run artemis run "할 일 설명" --profile flash
uv run artemis run "할 일 설명" --profile pro

# MCP
uv run artemis mcp --install cursor
uv run artemis mcp --install all

# 트레이스
uv run artemis trace --help
```

전역 명령이 PATH에 있으면:

```powershell
artemis doctor
artemis run "설정을 열어줘" --profile flash
```

---

## 9. 문제 해결

| 증상 | 조치 |
|------|------|
| `adb devices`에 기기 없음 | USB 디버깅 ON, 케이블/드라이버 확인, RSA 허용 |
| `unauthorized` | 폰에서 “이 컴퓨터 허용” |
| LLM 오류 / 키 없음 | `.env`에 API 키 입력 후 `artemis doctor` |
| Cursor에 MCP 도구 안 보임 | Reload Window → `uv run artemis mcp --install cursor` |
| 첫 태스크가 느림 | `artemis helper install` 미리 실행 |
| Helper와 다른 도구 충돌 | `.env`에 `ARTEMIS_HIERARCHY_BACKEND=uiautomator` |
| scrcpy/ffmpeg 누락 | `.\scripts\install_deps.ps1` 재실행 |
| UI가 안 열림 | `uv run artemis status` 후 `restart`, 포트 8000 충돌 확인 |

진단 한 줄:

```powershell
uv run artemis doctor
```

---

## 10. 디렉터리 구조 (알아야 할 것만)

```
D:\Workspace\artemis\
├── start.bat                 # Windows 원클릭 실행
├── .env                      # API 키 / 로컬 설정 (비밀 — 커밋 금지)
├── .env.example              # 템플릿
├── USAGE_KO.md               # ← 이 문서
├── README.md                 # 공식 영문 README
├── artemis\                  # 코어 패키지
├── mcp_server\               # Cursor 등 IDE용 MCP
│   └── rules.md              # AI 테스트 마인드셋 규칙
├── apps\showcase_ui\         # 웹 콘솔 (Angular)
├── config\                   # artemis.jsonc 등
└── scripts\                  # install_deps.ps1, start.ps1
```

---

## 11. 보안 주의

- `.env`의 API 키는 **절대 Git에 올리지 마세요**.
- 개인 폰을 쓸 경우 `.env`에서 다음을 검토하세요:
  - `ARTEMIS_KEEP_DEVICE_AWAKE=false` — 화면 항상 켜기 비활성
  - `ARTEMIS_HELPER_AUTO_INSTALL=false` — Helper 자동 설치 비활성
- Accessibility Helper는 **기기 로컬**에서 UI 계층만 읽습니다. 원격으로 화면을 전송하지 않습니다.

---

## 12. 업데이트

```powershell
cd D:\Workspace\artemis
git pull
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\install_deps.ps1
uv run artemis mcp --install cursor
```

---

## 13. 참고 링크

- 공식 저장소: https://github.com/google/artemis
- 영문 README: `README.md`
- MCP 상세: `mcp_server/README.md`
- Discord: https://discord.gg/wF2FN4WHGY

---

*작성 기준: 이 PC의 ARTEMIS 설치본 (`D:\Workspace\artemis`) + Cursor MCP 연동 완료 상태*
