### You-tility 프로젝트 인수인계서

#### 1. 프로젝트 개요
* **프로젝트명:** You-tility (웹 유틸리티 도구 모음)
* **배포 주소:** https://ddp-doryeonnim.github.io/you-tility/
* **저장소:** https://github.com/DDP-Doryeonnim/you-tility

#### 2. 기술 및 파일 구조
* 별도의 프레임워크나 빌드 도구 없는 **순수 HTML / CSS / JavaScript 단일 파일(Single-File)** 구조입니다.
* 루트 폴더 안에 총 11개의 `.html` 파일과 `memo.md`가 평면(Flat) 구조로 배치되어 있으며, 각 `.html` 파일 내부에 스타일과 스크립트가 통합되어 있습니다.
  * `index.html`, `screen-laser.html`, `pdf-editor.html`, `snapdrop-share.html`, `doc-scanner.html`, `ringtone-maker.html`, `ocr-scanner.html`, `data-decoder.html`, `image-diff.html`, `text-diff.html`, `image-converter.html`, `memo.md`

#### 3. Gemini 작업 절대 규칙
1. **기존 코드 및 디자인 보존:** 요청받지 않은 기존 레이아웃, CSS 스타일, 정상 작동하는 기능을 임의로 삭제하거나 변경하지 않습니다.
2. **검증 완료 경로 코드 잠금 (변경 불가):** 아래 4번 항목에서 **정상 작동 검증 완료**로 표기된 `screen-laser.html`의 6개 송수신 경로 로직(`DIRECTION_ROUTES`, WebRTC 및 WSS 바이너리 캔버스 엔진)은 사용자의 별도 명시적 지시가 있을 때까지 절대 변경하지 않습니다.
3. **명확한 수정 안내:** 코드를 수정할 때는 대상 파일명과 수정할 위치(또는 전체 코드)를 명확히 구분해 바로 적용할 수 있도록 제공합니다.
4. **단계별 진행:** 한 번에 너무 많은 작업을 몰아서 제안하지 않고, 단계별로 확인하며 진행합니다.

#### 4. 현재 진행 상태 및 다음 할 일
* **완료 (기본 구축):** VS Code ↔ GitHub ↔ Gemini 노트북 연동 및 배포 테스트 완료
* **완료 (2026-11-19 / `screen-laser.html` v4.3 전면 개편·실기기 검증·공유 버튼 단일화):**
  * **듀얼 엔진 & 방향별 독립 라우팅(`DIRECTION_ROUTES`) 구축:**
    * 송출·수신 OS 조합별로 독립 라우팅하는 듀얼 엔진(WebRTC / WSS 바이너리 캔버스) 구조 구축
    * **공통 제어·주석 채널:** PIN/QR 매칭, 시그널링, 레이저·펜·동그라미, 간편 메시지, 사진 칠판을 MQTT WSS(EMQX/HiveMQ 자동 예비 전환 + PIN 기반 SHA-256 토픽 + AES-GCM 종단간 암호화)로 통합
    * **WebRTC 채널:** PC·iOS·Android 전 기기 PeerJS 초기화(Google 다중 STUN 풀, `readyState >= 2` 빠른 판정, 6초 미수신 시 WSS 자동 폴백)
  * **실기기 테스트 검증 완료 6개 경로 (별도 지시 전까지 코드 변경 불가):**
    1. `PC → PC` : PeerJS WebRTC (60fps) — **✅ 정상 작동**
    2. `PC → 아이폰(iOS)` : WSS 바이너리 캔버스 (`hiddenVideo` 캡처, 64×36 변화 감지 Diff, 0.45초 정지 시 1280px HD 선명 키프레임, 수신 ACK 흐름 제어) — **✅ 정상 작동**
    3. `아이폰(iOS 27+) → PC` : PeerJS WebRTC (백그라운드 화면 미러링 및 카메라 중계 유지) — **✅ 정상 작동**
    4. `PC → 안드로이드` : PeerJS WebRTC — **✅ 정상 작동**
    5. `아이폰(iOS 27+) → 안드로이드` : PeerJS WebRTC — **✅ 정상 작동**
    6. `아이폰(iOS 27+) → 아이폰` : WSS 바이너리 캔버스 (`visibleVideo` 캡처, 구버전 iOS 수신 포함) — **✅ 정상 작동**
  * **OS·브라우저 제약 확정 사항 (코드 오류 아님 — 자체 화면 공유 포기 확정):**
    * `안드로이드 → PC / 아이폰`: 모든 안드로이드 모바일 브라우저의 `getDisplayMedia` 미지원으로 자체 화면 미러링 포기 → `[📷 내 카메라 중계]`, `[🖼️ 폰 캡처·사진 공유]`로 대체
    * `구버전 아이폰(iOS 26 이하) → 전체`: 구버전 Safari의 `getDisplayMedia` 미지원으로 자체 화면 송출 불가(수신은 정상) → 동일하게 카메라·사진 칠판으로 대체
    * `canSupportScreenShare()`로 미지원 기기를 자동 감지해 팝업 내 화면 공유 버튼 숨김 및 카메라·사진 안내 문구로 자동 분기
  * **1차 UI 정리:** 좌측 대기 화면(4개), 우측 QR 하단(3개), PIN 입력 탭 하단(1개)의 중복 공유 버튼을 모두 제거하고 연결 모달(`#inviteeModal`) 1곳으로 단일화, `<video id="senderHiddenVideo">` 닫는 태그 버그 수정, iOS Safari CSS 전체화면 폴백 적용
* **완료 (2026-11-23 / `screen-laser.html` v4.5 역할 분리 · 자동 스크롤 · 불필요/중복 버튼 최종 정리):**
  * **송출자 vs 수신·안내자 역할 UI 명확화 (검증 완료 6개 전송 엔진 100% 무변경 보존):**
    * **요청자·송출자 (`viewer`, QR 스캔·PIN 입력자):** 연결 즉시 `📤 송출자 (요청자)` 배지가 붙은 공유 선택 팝업(`#inviteeModal`) 자동 오픈 (내 화면·카메라·사진 선택 메인, `상대방 화면 먼저 보기`는 하단 보조 버튼으로 축소 / 공유 요청 수신 시 숨김)
    * **수신·안내자 (`host`, QR·PIN 표시자):** 연결 시 공유 선택 팝업을 띄우지 않고, 스테이지 중앙의 혼동 버튼(초록 `상대방에게 공유 요청`, 검정 `내 화면 대신 공유하기`)을 모두 숨겨 순수 수신 대기 화면만 깔끔하게 표시
  * **수신·안내자 화면 자동 스크롤 포커싱 (`focusStageArea`):**
    * QR 패널을 띄워둔 수신자 화면에서 연결 완료 시(1차) 및 첫 영상·사진 프레임 수신 시(2차, 최초 전환 1회) 공유 화면 스테이지(`#stageFullscreenTarget`)로 부드럽게 자동 이동
  * **2차 중복·불필요 버튼 완전 정리:**
    * `전체화면 보기` 통합 (3곳 → **스테이지 우측 상단 1곳** `#stageFullscreenBtn`)
    * `끝내기`/`종료` 통합 (2곳 → **상단 헤더 1곳** `[■ 끝내기]`, 하단 플로팅 독은 순수 주석 도구 `레이저·펜·동그라미·지우기`만 유지)
    * 불필요한 노란색 자동재생 잠금 해제 배너(`#iosForcePlayBanner`) 및 `forceUnlockVideoPlayback()` 관련 HTML·JS 코드 완전 삭제
  * **안정성 디버깅 반영:** `updateEmptyStageUI()` 단일화로 문구 충돌 방지, `updateFullscreenLabels()` null 참조 제거, `stopAllSharingAndDisconnect()` 종료 상태 초기화 순서 보정
* **다음 작업:**
  * `screen-laser.html` v4.5 최종 배포 후 실기기 점검 (송출자 팝업 자동 오픈 / 수신자 자동 스크롤 및 중앙 버튼 미표시 / 노란 배너 미노출 / 안드로이드 카메라·사진 송출 정상 동작 확인)
  * (선택) 다른 유틸리티 도구 기능 개선 또는 신규 도구 추가

#### 5. 대화 마무리 규칙
* 사용자가 "인수인계서 업데이트해 줘"라고 요청하면, 오늘 수정한 파일과 변경된 기능 내역을 위 **4번 항목**에 바로 덮어쓸 수 있도록 정리해 줍니다.
