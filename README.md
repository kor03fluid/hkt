# hkt — HoloHand STL Viewer

웹캠 손 인식으로 STL 어셈블리(Turret, hailo)를 **분해 · 회전 · 확대/축소**하고, **부품을 하나씩 집어서 옮기거나 돌려 볼 수 있는** 웹앱입니다.
[HoloHand-Gesture-Agent-3D](https://github.com/tubakhxn/HoloHand-Gesture-Agent-3D)(MIT)의 손 인식 · 제스처 엔진을 그대로 쓰고, 렌더러를 STL 메시용으로 새로 만들었습니다.

모든 처리는 브라우저 안에서 합니다(MediaPipe Hand Landmarker + Three.js). 서버, API 키가 필요 없고 카메라 영상도 밖으로 나가지 않습니다.

## 실행

Node.js 18+ 와 Chrome / Edge 가 필요합니다.

```bash
npm install
npm run dev
```

Vite가 띄워 주는 주소(보통 http://localhost:5173)에서 **카메라 시작**을 누르세요. 카메라는 `http://localhost` 또는 `https` 에서만 동작합니다.

## STL 넣는 법

저장소 루트에 모델별 폴더를 만들고 부품별 STL을 넣으면 됩니다. **폴더 하나 = 모델 하나, STL 하나 = 부품 하나**입니다.

```
Turret/
  base.stl
  pan_motor.stl
  ...
hailo/
  board.stl
  heatsink.stl
  ...
```

- STL은 **어셈블리 좌표 그대로** 내보낸 파일이어야 합니다. 불러온 위치가 곧 조립 상태입니다.
- 분해 방향은 자동 계산합니다(어셈블리 중심 → 부품 중심 방향).
- 기본은 **Z축이 위**(CAD 기본)입니다. 다르면 아래 `parts.json`에서 `"up": "y"` 로 바꾸세요.
- 저장소에 올리지 않고 보려면, 실행 후 **📂 폴더 열기**를 누르거나 STL 폴더를 화면에 끌어다 놓으면 됩니다.
- Git LFS로 올린 STL은 `git lfs pull` 로 실제 파일을 받은 뒤 실행하세요(포인터 파일은 읽을 수 없습니다).

### parts.json (선택)

STL 폴더 안에 두면 이름 · 설명 · 색 · 분해 방향을 직접 정할 수 있습니다. 적지 않은 항목은 자동값을 씁니다.

```json
{
  "name": "Turret",
  "subtitle": "팬/틸트 터렛",
  "up": "z",
  "parts": {
    "base.stl": { "name": "베이스", "desc": "전체를 받치는 하판", "color": "#f2c94c", "explode": [0, 0, -1] },
    "barrel.stl": { "explode": [1.2, 0, 0.3] }
  }
}
```

`explode` 는 STL 좌표계 기준 방향이며, 크기 1 = 어셈블리 반지름의 70% 만큼 이동합니다.

## 제스처

| 제스처 | 동작 |
|---|---|
| 손 천천히 펴기 | 분해 (손바닥 링의 흰 호 = 분해 정도) |
| 주먹 | 조립. 0.6초 이상 쥐고 있으면 옮긴 부품도 원위치 |
| 손 이동 · 비틀기 | 모델 회전 · 기울이기 |
| 검지로 가리키기 | 손끝 아래 부품 강조, 0.7초 멈추면 선택 |
| 집기(핀치) | 부품을 잡아 끌어서 이동, 집은 채로 손목을 비틀면 부품 회전. 놓은 자리에 그대로 둠 |
| 양손 벌리기 / 모으기 | 확대 / 축소 |
| 브이(약 0.3초 유지) | 다음 모델 |

패널의 **손으로 분해** 버튼을 끄면, 손을 펴도 분해되지 않고 회전만 합니다.

## 마우스 · 키보드

드래그 = 회전 · 휠 = 확대 · 클릭 = 부품 선택 · **Shift+드래그** = 부품 이동 · **Alt(또는 Ctrl)+드래그** = 부품 회전

`Space` 분해/조립 · `←` `→` / `1`–`9` 모델 전환 · `R` 시점 초기화 · `P` 부품 원위치 · `X` X-ray · `I` 선택 부품만 보기 · `H` 선택 부품 숨기기 · `E` 모서리 선 · `Esc` 선택 해제

## 파일

```
src/main.js          앱 루프, 손 오버레이, 제스처 → 동작, 이름표, UI
src/scene.js         Three.js STL 렌더러: 분해, 부품 이동/회전, 선택(BVH 레이캐스트), X-ray, 단독 보기
src/library.js       저장소 STL 폴더 자동 탐색, parts.json, 폴더 열기 / 드래그 앤 드롭
src/handTracking.js  MediaPipe 래퍼 (HoloHand 그대로)
src/gestureEngine.js 랜드마크 → 제스처, 손 펴짐 정도, 손바닥/손끝/핀치 위치, 손목 회전 (HoloHand 그대로)
scripts/setup.mjs    npm install 시 MediaPipe WASM 복사 + 손 모델 다운로드 (실패하면 실행 시 CDN 사용)
```

HoloHand 원본 라이선스는 `LICENSE-HoloHand` 에 있습니다.
