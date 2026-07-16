# ssdam 포크 패치 노트

쓰담(ssdam) 앱용 CamerAwesome 포크. base = upstream `master` (pubspec 2.5.0 시점).
앱에서 `dependency_overrides`로 이 브랜치/커밋을 참조한다.

## 패치 (iOS, `SingleCameraPreview.m`)

두 곳 모두 `// PATCH(ssdam)` 주석으로 표시. 이유·복원 근거는 인라인 주석 참조.

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

## upstream 반영(선택)

두 수정 모두 모든 사용자에게 유효한 버그 픽스. upstream PR로 올리면 머지 시 이 포크가 불필요해진다.

## upstream 동기화

`git fetch upstream && git merge upstream/master` 후 충돌 해소(패치는 iOS 파일 2곳뿐).
