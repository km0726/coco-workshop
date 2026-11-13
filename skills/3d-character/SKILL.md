---
name: 3d-character
description: 사용자가 설명한 3D 캐릭터를 Three.js로 만들어, 브라우저에서 바로 돌려 볼 수 있는 HTML 파일 하나로 내놓는다. 캐릭터는 기본 도형으로 조립한 로우폴리 스타일이며 대기 애니메이션이 들어간다. "3D 캐릭터 만들어줘", "3D 마스코트", "귀여운 3D 동물", "로우폴리 캐릭터", "three.js character", "3D avatar" 같은 요청이나, 캐릭터를 3D로 보고·돌려 보고·움직이게 하고 싶다는 요청에 사용한다.
---

# 3D 캐릭터 만들기 (Three.js)

사용자의 설명을 바탕으로 **로우폴리 스타일 3D 캐릭터**를 만든다. 결과물은 브라우저에서 바로 열리는 HTML 파일 하나다.
캐릭터는 Three.js 기본 도형(구, 캡슐, 원기둥, 원뿔, 박스, 토러스)을 조립해 만든다. 외부 모델 파일이나 이미지는 쓰지 않는다.

## 1. 요구사항 정리

설명이 짧아도 되묻지 말고 합리적인 기본값으로 바로 만든다. 다음 항목만 머릿속으로 정한다.

| 항목 | 정할 내용 | 기본값 |
|---|---|---|
| 종류 | 사람, 동물, 로봇, 몬스터, 사물 의인화 | 설명에서 추론 |
| 비율 | 치비(머리 큼) / 일반 / 길쭉함 | 치비 (머리:몸 ≈ 1:1) |
| 색상 | 주색, 보조색, 포인트색 3~4개 | 파스텔 톤 |
| 특징 | 귀, 뿔, 꼬리, 모자, 안경, 소품 등 2~3개 | 종류에 맞게 |
| 동작 | 숨쉬기, 통통 튀기, 손 흔들기, 걷기 | 숨쉬기 + 눈 깜빡임 |

## 2. 템플릿에서 시작

`assets/template.html`을 복사해 시작한다. 템플릿에는 다음이 이미 들어 있다.

- 장면, 카메라, 조명(키·필·림), 바닥 그림자, `OrbitControls`
- 이름 붙은 부위를 담는 계층 구조 (`root → body → head`, `armL/armR`, `legL/legR`)
- 툰 셰이딩 재질 헬퍼 `mat(color)`와 외곽선 헬퍼 `outline(mesh)`
- 애니메이션 루프와 눈 깜빡임
- 화면 크기 대응, 다크/라이트 배경

**`buildCharacter()` 함수와 `CONFIG` 객체만 바꾸면 된다.** 장면 설정 코드는 건드리지 않는다.

## 3. 모델링 규칙

1. **피벗을 관절에 둔다.** 팔과 다리는 `Group`을 어깨·엉덩이 위치에 놓고, 메시는 그 아래로 내려 단다. 그래야 `rotation.x/z`로 자연스럽게 흔들린다.
2. **단위:** 캐릭터 전체 키는 약 2 유닛으로 하고, 발바닥을 y=0에 맞춘다.
3. **좌우 대칭:** 왼쪽 부위를 만드는 함수를 하나 쓰고, `side = ±1`로 x 위치와 회전 방향만 바꿔 양쪽을 만든다.
4. **얼굴이 생명이다.** 눈은 흰자 없이 검은 구(살짝 납작하게)에 작은 흰 하이라이트 구를 붙인다. 볼터치는 반투명 분홍 원판으로 한다. 입은 얇은 토러스 반쪽(`arc = Math.PI`)으로 만든다. 얼굴 부품은 머리 표면에서 살짝 앞으로 띄워 겹침 깜빡임(z-fighting)을 막는다.
5. **세그먼트 수:** 구는 24~32 정도로 부드럽게 한다. 로우폴리 느낌을 원하면 `flatShading: true`에 8~12로 한다.
6. **부위 이름:** 애니메이션이나 확장을 위해 `parts` 객체에 주요 부위를 등록한다 (`parts.head`, `parts.armL` 등).

## 4. 애니메이션

`animate(t)` 안에서 `parts`를 움직인다. 모든 움직임은 `Math.sin(t * 속도) * 크기` 꼴로 쓰고 크기는 작게 잡는다.

- 숨쉬기: `body.scale.y = 1 + sin(t*2)*0.03`
- 통통 튀기: `root.position.y = abs(sin(t*3))*0.15`
- 손 흔들기: `armR.rotation.z = -2.2 + sin(t*8)*0.4`
- 걷기: 팔과 다리를 반대 위상으로 `rotation.x = ±sin(t*6)*0.5`
- 고개: `head.rotation.y = sin(t*0.7)*0.2`

사용자가 동작을 여러 개 원하면 화면 아래에 버튼을 두고 `CONFIG.action`을 바꾸게 한다. 템플릿에 버튼 영역이 이미 있다.

## 5. 검증

파일을 만든 뒤 반드시 확인한다.

1. Playwright로 열어 콘솔 오류가 없는지 본다.
2. 1~2초 기다린 뒤 스크린샷을 찍어 직접 본다. 부위가 떠 있거나 파묻혔는지, 얼굴이 정면을 보는지, 그림자가 발밑에 있는지 확인한다.
3. 문제가 있으면 고친 뒤 다시 찍는다.

샌드박스에서 CDN이 막혀 있으면(`ERR_TUNNEL_CONNECTION_FAILED`, 화면이 비어 있음) `npm i three@0.160.0`로 받은 뒤, Playwright에서 CDN 요청을 로컬 파일로 바꿔 준다.

```js
const ROOT = require('path').resolve('node_modules/three');
await p.route('https://cdn.jsdelivr.net/npm/three@0.160.0/**', r => r.fulfill({
  path: require('path').join(ROOT, r.request().url().split('three@0.160.0/')[1]),
  contentType: 'application/javascript',
}));
```

```js
// node verify.js <html 경로> <png 경로>
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch({ args: ['--use-gl=swiftshader', '--enable-unsafe-swiftshader'] });
  const p = await b.newPage({ viewport: { width: 900, height: 900 } });
  p.on('console', m => m.type() === 'error' && console.log('ERR', m.text()));
  p.on('pageerror', e => console.log('PAGEERR', e.message));
  await p.goto('file://' + require('path').resolve(process.argv[2]));
  await p.waitForTimeout(1500);
  await p.screenshot({ path: process.argv[3] });
  await b.close();
})();
```

## 6. 결과 전달

- HTML 파일을 사용자가 열 수 있는 위치에 저장하고, 가능하면 Artifact로 게시하거나 파일로 보내 준다.
- 한두 줄로 캐릭터 특징과 조작법(드래그로 회전, 휠로 확대, 버튼으로 동작 변경)을 알려 준다.
- 바꾸고 싶은 점(색, 부위, 동작)이 있으면 말해 달라고 짧게 덧붙인다.

## 하지 말 것

- 기존 게임·애니메이션·브랜드 캐릭터를 그대로 재현하지 않는다. 비슷한 느낌의 오리지널 캐릭터로 만든다.
- 외부 GLB/텍스처를 받아 오지 않는다. 모든 형태는 코드로 만든다.
- Three.js는 버전을 고정한 CDN 주소(`three@0.160.0`)로만 불러온다.
