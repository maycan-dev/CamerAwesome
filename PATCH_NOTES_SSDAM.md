# ssdam 포크 패치 노트

쓰담(ssdam) 앱용 CamerAwesome 포크. base = upstream `master` (pubspec 2.5.0 시점).
앱에서 `dependency_overrides`로 이 브랜치/커밋을 참조한다.

## 패치 (iOS)

모두 `// PATCH(ssdam)` 주석으로 표시. 이유·복원 근거는 인라인 주석 참조.
파일: `SingleCameraPreview.m`(1·2·4), `Controllers/Picture/CameraPictureController.m`(3).

### 1. `setCameraPreset` — 프리뷰 FOV를 캡처와 일치 (4:3 full sensor)

CamerAwesome iOS는 프리뷰 세션에 최고 비디오 프리셋(16:9)을 골라, 세로에서 좌우 FOV가
네이티브 카메라(4:3)보다 좁아진다. `aspectRatio` 설정이 iOS 프리뷰엔 무효한 upstream 오픈 버그:
- https://github.com/Apparence-io/CamerAwesome/issues/624

사진 모드에서 세션을 `AVCaptureSessionPresetPhoto`(4:3 full sensor)로 강제해 프리뷰 = 캡처
FOV를 일치시킨다(WYSIWYG). 프리뷰 크기는 `activeFormat`(프리셋 무관 항상 4:3) 실측에서 산출.
비디오 녹화/이미지 스트리밍 모드는 원 로직 유지.

### 2. `focusOnPoint` — portrait 초점/노출 좌표 회전 + 노출점

`focusPointOfInterest`는 센서 landscape 정규화 좌표를 요구하는데 원본은 정규화 좌표를 변환 없이
넘겨 portrait 앱에서 초점 지점이 어긋났다. portrait 90° 회전(후면 `(y,1-x)`, 전면 `(y,x)`)을
적용하고, `exposurePointOfInterest`도 같은 지점으로 설정(탭-포커스 시 노출도 함께, iOS 결).

### 3. `CameraPictureController.m` — 촬영본 크롭 생략 (full 4:3 유지)

`imageByCroppingImage`의 `Ratio4_3` 분기가 portrait에서 오리엔테이션 혼동으로 정사각(1:1)을
잘라낸다 — 4:3 센서가 1440×1440으로 저장되는 버그. 패치 1로 Photo 프리셋이 이미 full 4:3
(=portrait 3:4)이라 크롭 자체가 불필요 → 크롭 호출을 건너뛰고 원본을 그대로 저장한다.
프리뷰(4:3)와 WYSIWYG, 앱의 3:4 박스에서 잘림 없음.

### 4. `SingleCameraPreview.m` `takePictureAtPath` — 캡처 지연 최소화 (`.speed` 우선순위)

`AVCapturePhotoSettings`가 `photoQualityPrioritization`을 지정하지 않아 기본 `.balanced` — iPhone
계산사진(Deep Fusion·멀티프레임 융합)을 태워 셔터~반환 지연이 ~1s. 그동안 앱이 셔터 화이트를
홀드해 "촬영 애니가 너무 길다"는 UX 문제. 라이브 캔디드 1컷은 어차피 다운샘플하므로 계산사진
품질이 낭비 → `AVCapturePhotoQualityPrioritizationSpeed`(단일프레임 즉시 반환)로 지연을 급감시킨다.
`@available(iOS 13.0, *)` 가드. (이 패치는 앱 정책 성격이 강함 — upstream PR보다는 옵션 노출이 맞을 수 있음.)

## upstream 반영(선택)

패치 1·2·3은 모든 사용자에게 유효한 버그 픽스. upstream PR로 올리면 머지 시 불필요해진다.
패치 4(.speed)는 앱 정책 성향이라 upstream엔 "옵션"으로 제안하는 게 적절.

## upstream 동기화

`git fetch upstream && git merge upstream/master` 후 충돌 해소(패치는 iOS 파일 3곳뿐).
