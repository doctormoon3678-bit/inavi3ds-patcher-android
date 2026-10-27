# iNavi 3DS Patcher - Android 앱 (Termux 연동)

아이나비 3DS APK(2.5 / 3.0 / 4.0)를 폰에서 패치하는 앱입니다.
화면(APK·패치 옵션·항목·파일·서명 선택)은 이 앱이 맡고, 실제 패치(apktool 디컴파일 → 패치 → 빌드 → 서명)는
같은 폰의 Termux에서 실행됩니다. Create by DoctorMoon

> 왜 Termux를 쓰나: apktool은 안드로이드 앱 안에서 그대로 실행되지 않는 라이브러리에 의존하는 것으로 알고 있어,
> 완전 독립형은 만들지 않았습니다. Termux는 일반 자바 환경이라 PC용과 같은 방식으로 돌아갑니다.

## 저장소 구성

이 저장소에는 앱 소스 전체가 **`project.zip`** 하나로 압축되어 올라가 있습니다.

```
(저장소 루트)
├── .github/workflows/build-apk.yml   GitHub Actions 빌드 설정
├── project.zip                       앱 소스 전체 (압축 상태)
└── README.md                         이 문서
```

`project.zip` 안의 구조는 다음과 같습니다.

```
build.gradle.kts, settings.gradle.kts, gradle.properties
gradle/wrapper/                         Gradle 래퍼 설정
app/src/main/java/.../MainActivity.kt   화면과 동작
app/src/main/java/.../Termux.kt         Termux RUN_COMMAND 호출
app/src/main/java/.../Packs.kt          패치팩 항목 읽기
app/src/main/java/.../Ui.kt             다크모드 UI 도우미
app/src/main/assets/engine/             파이썬 엔진 + 패치팩 (2.5, 3.0, 4.0)
```

## 1. APK 빌드

### 방법 A. GitHub Actions (설치 없이, 권장)

`.github/workflows/build-apk.yml` 이 아래 순서로 자동 빌드합니다.

1. 저장소 루트의 `project.zip` 을 `build_src/` 에 압축 해제
2. 압축 안에서 `settings.gradle.kts` 위치를 찾아 그 폴더를 프로젝트 루트로 사용
3. JDK 17 + Gradle 8.7 로 `gradle assembleDebug --no-daemon` 실행
4. 결과 APK를 Artifacts(`iNavi3DS-Patcher-debug-apk`)로 업로드

실행 방법
- `project.zip` 을 저장소 루트에 올리고 push 하면 자동으로 빌드됩니다.
- 또는 저장소의 `Actions` 탭 → `Build APK` → `Run workflow` 로 수동 실행할 수 있습니다.
- 빌드가 끝나면 해당 실행 화면 아래쪽 `Artifacts` 에서 APK를 내려받습니다.

참고
- 워크플로 파일은 반드시 저장소 루트의 `.github/workflows/` 에 있어야 실행됩니다. `project.zip` 안에 들어 있는 워크플로 파일은 실행되지 않습니다.
- 파일 이름은 `project.zip` 이며, 대소문자(`Project.zip`)는 구분하지 않습니다.
- 소스를 수정했다면 `project.zip` 을 다시 만들어 같은 이름으로 덮어써서 올립니다.

### 방법 B. 내 PC에서 직접 빌드

1. JDK 17, Gradle 8.7 (또는 Android Studio Koala 이상) 준비
2. `project.zip` 을 원하는 폴더에 압축 해제
3. 압축 푼 폴더에서 터미널 실행
   ```
   gradle assembleDebug
   ```
   Android Studio를 쓴다면 압축 푼 폴더를 `File > Open` 으로 열고 Gradle Sync 후
   `Build > Build Bundle(s) / APK(s) > Build APK(s)` 를 실행합니다.
4. 결과: `app/build/outputs/apk/debug/app-debug.apk` → 폰에 설치

## 2. 폰 준비 (한 번만)

1. Termux 설치 (F-Droid 또는 GitHub 릴리스 버전 — Play 스토어 버전은 오래되어 권장하지 않음)
2. Termux에서 아래 명령을 실행한 뒤 Termux를 완전히 종료했다가 다시 열기:
   ```
   mkdir -p ~/.termux && echo "allow-external-apps=true" >> ~/.termux/termux.properties
   ```
3. 이 앱을 열고 `권한 허용` → 저장공간 / Termux 실행 / (필요 시) 모든 파일 접근 허용
4. `엔진 설치` → `/sdcard/Download/inavi3ds_patcher_termux/` 에 엔진과 패치팩이 복사됨
5. `apktool.jar` 와 `signer.jar`(uber-apk-signer) 선택 → 같은 폴더의 `tools/` 에 저장됨
6. `Termux 환경 설치` → Termux 창에서 python / openjdk-17 / aapt 설치 (인터넷 필요)

## 3. 사용

1. `APK 찾아보기` 로 APK 선택 (버전이 감지되면 패치 옵션이 자동 선택됨)
2. 패치 옵션(2.5/3.0/4.0)과 항목 선택, `파일 선택` 으로 파일 단위 제외 가능
3. 서명: 기본은 자동 디버그 키. 내 키스토어는 스위치를 켜고 입력
4. `패치 시작` → Termux 창이 열려 진행 상황이 표시됨
5. 완료되면 `/sdcard/Download/<원본이름>_patched.apk`

## 새 패치팩 추가

`build_patchpack.py` 로 만든 폴더를 `app/src/main/assets/engine/patches/<버전>/` 에 넣으면
앱의 목록에 자동으로 나타납니다. (`project.zip` 안의 같은 경로에 넣은 뒤 다시 압축해서 올립니다.)

## 알려진 사항

- 이 프로젝트는 컴파일/실행 검증을 하지 못했습니다. 빌드 오류가 나면 Actions 로그의 오류 메시지를 알려 주세요.
- 앱의 targetSdk 가 28 입니다(저장공간 접근을 단순하게 하려는 것). 설치할 때 "구버전용 앱" 경고가 뜰 수 있습니다.
- 4.0 패치는 시간이 오래 걸리고 메모리를 많이 씁니다. 다른 앱을 닫고 실행하세요.
- Termux 쪽 동작(apktool 빌드, aapt, zipalign)도 폰에서 직접 확인하지 못했습니다.
