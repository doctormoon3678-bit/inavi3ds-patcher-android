# iNavi 3DS Patcher - Android 앱 (Termux 연동)

아이나비 3DS APK(2.5 / 3.0 / 4.0)를 폰에서 패치하는 앱입니다.
화면(APK·패치 옵션·항목·파일·서명 선택)은 이 앱이 맡고, 실제 패치(apktool 디컴파일 → 패치 → 빌드 → 서명)는
같은 폰의 Termux에서 실행됩니다. Create by DoctorMoon

> 왜 Termux를 쓰나: apktool은 안드로이드 앱 안에서 그대로 실행되지 않는 라이브러리에 의존하는 것으로 알고 있어,
> 완전 독립형은 만들지 않았습니다. Termux는 일반 자바 환경이라 PC용과 같은 방식으로 돌아갑니다.

## 1. APK 빌드 (직접)
### 방법 A. Android Studio
1. Android Studio(Koala 이상) 설치, JDK 17
2. `File > Open` 으로 이 폴더(`inavi3ds_patcher_android`) 열기 → Gradle Sync
3. `Build > Build Bundle(s) / APK(s) > Build APK(s)`
4. 결과: `app/build/outputs/apk/debug/app-debug.apk` → 폰에 설치

터미널: `gradle assembleDebug` (Gradle 8.7, JDK 17)

### 방법 B. GitHub Actions (설치 없이)
이 폴더를 GitHub 저장소에 올리면 `.github/workflows/build-apk.yml` 이 APK를 빌드해
Actions의 Artifacts 로 내려받을 수 있게 해줍니다.

## 2. 폰 준비 (한 번만)
1. Termux 설치 (F-Droid 또는 GitHub 릴리스 버전 — Play 스토어 버전은 오래되어 권장하지 않음)
2. Termux에서 실행 후 Termux를 완전히 종료했다가 다시 열기:
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

## 구조
```
app/src/main/java/.../MainActivity.kt   화면과 동작
app/src/main/java/.../Termux.kt         Termux RUN_COMMAND 호출
app/src/main/java/.../Packs.kt          패치팩 항목 읽기
app/src/main/java/.../Ui.kt             다크모드 UI 도우미
app/src/main/assets/engine/             파이썬 엔진 + 패치팩 (2.5, 3.0, 4.0)
```
새 패치팩은 `build_patchpack.py` 로 만든 폴더를 `assets/engine/patches/<버전>/` 에 넣으면 목록에 자동으로 나타납니다.

## 알려진 사항
- 이 프로젝트는 컴파일/실행 검증을 하지 못했습니다. 빌드 오류가 나면 오류 메시지를 알려 주세요.
- 앱의 targetSdk 가 28 입니다(저장공간 접근을 단순하게 하려는 것). 설치할 때 "구버전용 앱" 경고가 뜰 수 있습니다.
- 4.0 패치는 시간이 오래 걸리고 메모리를 많이 씁니다. 다른 앱을 닫고 실행하세요.
- Termux 쪽 동작(apktool 빌드, aapt, zipalign)도 폰에서 직접 확인하지 못했습니다.
