---
title: "WebGL 셰이더로 구현하는 실시간 ASCII와 디더링 효과: Efecto 프로젝트 분석"
description: "WebGL 셰이더를 활용하여 Floyd-Steinberg, Atkinson 등 클래식 디더링 알고리즘과 ASCII 아트를 실시간으로 구현하는 방법을 상세히 알아봅니다. 3D 모델과 이미지에 레트로 CRT 효과를 적용하는 기술을 소개합니다."
date: 2026-01-06 17:00:55 +0900
categories: ['Development', 'Web']
tags: ['WebGL', 'Shader', 'Dithering', 'ASCII', 'Graphics', 'Three.js', 'GLSL', 'Floyd-Steinberg']
image:
  path: "/assets/img/2026-01-06-efecto-building-real-time-ascii-and-dithering-effects-with-webgl-shaders-2916.jpg"
  alt: "실시간 디더링 효과가 적용된 3D 모델 렌더링"
published: true
---

## 3줄 요약
- Floyd-Steinberg, Atkinson 등 8가지 클래식 디더링 알고리즘을 WebGL로 구현하여 실시간 이미지 처리를 실현했습니다.
- ASCII 아트는 5×7 그리드에서 문자를 수학적으로 생성하는 GLSL 셰이더로 구현되며, GPU에서 완전히 병렬 처리됩니다.
- 스캔라인, 화면 곡률, 색수차 등 CRT 효과를 추가하여 레트로 컴퓨팅 시대의 비주얼을 재현합니다.

## 📌 주요 내용

### WebGL 셰이더와 디더링의 만남

디자이너로서 셰이더를 작성하지 않았던 개발자가 실시간 디더링 효과를 구현하기 위해 시작한 프로젝트 Efecto는 WebGL의 가능성을 보여주는 흥미로운 사례입니다. 디더링(Dithering)은 제한된 색상 팔레트로 더 많은 색상의 착시를 만드는 기술로, 1976년 Robert Floyd와 Louis Steinberg의 논문에서 시작되었습니다.

프로젝트는 Lummi를 위한 효과 도구에서 시작되었으며, v0를 사용한 프로토타이핑을 거쳐 본격적인 WebGL 셰이더 기반 애플리케이션으로 발전했습니다. Shadertoy, The Book of Shaders, React Three Fiber 등의 오픈소스 자료를 활용하여 구축되었습니다.

### Floyd-Steinberg 알고리즘의 핵심 원리

디더링의 기본 개념은 픽셀을 가장 가까운 색상으로 반올림할 때 발생하는 "오차"를 버리지 않고 주변 픽셀에 분산시키는 것입니다. 이를 통해 거친 밴딩 대신 유기적인 패턴을 생성합니다.

```javascript
// 각 픽셀마다...
const [r, g, b] = getPixel(x, y)

// 팔레트에서 가장 가까운 색상 찾기
const [qR, qG, qB] = findNearestColor(r, g, b, palette)

// 오차 계산
const errR = r - qR
const errG = g - qG
const errB = b - qB

// Floyd-Steinberg 가중치로 오차를 이웃 픽셀에 분산
addError(x + 1, y,     errR * 7/16, errG * 7/16, errB * 7/16)
addError(x - 1, y + 1, errR * 3/16, errG * 3/16, errB * 3/16)
addError(x,     y + 1, errR * 5/16, errG * 5/16, errB * 5/16)
addError(x + 1, y + 1, errR * 1/16, errG * 1/16, errB * 1/16)
```

가중치(7/16, 3/16, 5/16, 1/16)의 합이 1이 되어 오차의 100%를 재분배하며, 비대칭 분포는 대각선 패턴을 방지합니다.

### 다양한 디더링 알고리즘 구현

**Atkinson 알고리즘**은 1984년 Bill Atkinson이 오리지널 매킨토시를 위해 개발했으며, 오차의 75%만 분산시켜 더 높은 대비와 "크런치한" 질감을 만듭니다.

```javascript
const atkinson = {
  kernel: [
    [1, 0, 1],   // 오른쪽
    [2, 0, 1],   // 오른쪽 2칸
    [-1, 1, 1],  // 왼쪽 아래
    [0, 1, 1],   // 아래
    [1, 1, 1],   // 오른쪽 아래
    [0, 2, 1],   // 아래 2칸
  ],
  divisor: 8,  // 6개 이웃 × 1 = 6, 하지만 제수는 8
}
```

6/8만 분산되어 "손실된" 25%가 Atkinson만의 독특한 외관을 만듭니다.

**Jarvis-Judice-Ninke** 알고리즘은 3개 행의 12개 이웃에 오차를 분산하여 더 부드러운 그라디언트를 생성합니다. 속도는 느리지만 결과물의 품질이 뛰어납니다.

Efecto는 총 8가지 디더링 알고리즘을 구현했으며, 각 알고리즘은 고유한 시각적 특성을 가집니다.

### 컬러 팔레트와 루미넌스 매핑

흑백 디더링은 클래식하지만, 멀티 컬러 팔레트는 더 많은 가능성을 열어줍니다. Efecto는 터미널 컬러, 웜/쿨 톤, 네온/신스웨이브, 어스 톤, 모노크롬 등 5개 카테고리의 31개 프리셋 팔레트를 제공합니다.

루미넌스 기반 컬러 매핑 방식은 다음과 같습니다:

```javascript
// 픽셀의 밝기 계산
const luminance = 0.299 * r + 0.587 * g + 0.114 * b

// 밝기를 팔레트 인덱스로 매핑
const index = Math.floor(luminance * palette.length)
const color = palette[Math.min(index, palette.length - 1)]
```

계수 0.299, 0.587, 0.114는 인간 시각의 색상 민감도를 반영합니다. 녹색에 가장 민감하고, 그 다음 빨강, 파랑 순입니다.

게임보이는 4가지 녹색 음영만 가졌지만, 아티스트들은 이 제약 안에서 기억에 남는 게임을 만들었습니다. 제한된 팔레트는 창의성을 강제합니다.

### 블룸 효과로 CRT 모니터 재현

블룸(Bloom) 효과는 CRT 모니터의 외관을 시뮬레이션하는 핵심 기술입니다. 디더링은 고대비 픽셀 패턴을 생성하고, 블룸은 밝은 픽셀이 어두운 픽셀로 번지게 하여 거친 가장자리를 부드럽게 만들면서도 디더링된 텍스처를 유지합니다.

블룸 효과는 WebGPU로 가속되며, WebGPU를 사용할 수 없는 Firefox와 같은 환경에서는 Canvas 2D 폴백을 제공합니다.

### ASCII 아트의 수학적 구현

디더링 구현 후 ASCII 아트로 관심을 확장했습니다. 같은 기본 개념(패턴으로 밝기 표현)이지만 픽셀 배열 대신 텍스트 문자를 사용합니다.

셰이더에는 폰트가 없어 `drawText()`를 호출할 수 없습니다. 모든 것을 수학으로 처리해야 합니다. 해결책은 5×7 픽셀 그리드에서 문자를 절차적으로 그리는 것입니다.

```glsl
// 콜론: 수직으로 중앙 정렬된 두 점
if (grid.x == 2.0 && (grid.y == 2.0 || grid.y == 4.0)) {
  return 1.0;
}
return 0.0;

// 별표: 중심 + 팔 + 대각선
bool center = (grid.x == 2.0 && grid.y == 3.0);
bool vert = (grid.x == 2.0 && (grid.y >= 2.0 && grid.y <= 4.0));
bool horiz = (grid.y == 3.0 && (grid.x >= 1.0 && grid.x <= 3.0));
bool diag1 = ((grid.x == 1.0 && grid.y == 2.0) || (grid.x == 3.0 && grid.y == 4.0));
bool diag2 = ((grid.x == 1.0 && grid.y == 4.0) || (grid.x == 3.0 && grid.y == 2.0));

return (center || vert || horiz || diag1 || diag2) ? 1.0 : 0.0;
```

셰이더는 화면을 셀 그리드로 나누고 각 셀마다:
1. 셀 중심의 색상을 샘플링
2. 밝기 계산
3. 밝기에 따라 문자 선택

```glsl
float brightness = dot(cellColor.rgb, vec3(0.299, 0.587, 0.114));
```

어두운 영역은 밀도 높은 문자(`@`, `#`, `8`)를, 밝은 영역은 희소한 문자(`.`, `:`, 공백)를 사용합니다.

Efecto는 standard, dense, minimal, blocks, braille, technical, matrix, hatching 등 8가지 ASCII 스타일을 제공합니다.

### CRT 효과의 완성

**스캔라인**은 CRT 인광체 행을 시뮬레이션하는 수평 어두운 밴드입니다.

**화면 곡률**은 오래된 모니터의 곡면 유리를 흉내냅니다:

```glsl
vec2 centered = uv * 2.0 - 1.0;
float dist = dot(centered, centered);
centered *= 1.0 + curvature * dist;
uv = centered * 0.5 + 0.5;
```

이 수식은 중심에서 픽셀을 바깥쪽으로 밀어내며, 가장자리에서 더 강하게 작용합니다.

**색수차(Chromatic Aberration)**는 RGB 채널을 약간 분리하여 저렴한 광학 장치처럼 보이게 합니다.

**비네트(Vignette)**는 가장자리를 어둡게 하여 중심에 초점을 맞춥니다.

녹색 인광체나 앰버 팔레트와 결합하면 전체가 오래된 터미널처럼 느껴집니다.

### 아키텍처 및 성능 고려사항

**디더링은 CPU에서 실행됩니다.** 오차 확산은 각 픽셀이 이전에 처리된 픽셀에 의존하기 때문에 본질적으로 순차적입니다. 실제 디더링 알고리즘은 JavaScript에서 실행되며 메모리에서 픽셀 데이터를 처리합니다.

**ASCII는 WebGL 셰이더로 실행됩니다.** 디더링과 달리 각 셀이 독립적이므로 완전히 GPU에서 실행할 수 있습니다. 셰이더는 Three.js와 postprocessing 라이브러리로 구축됩니다.

**일부 효과는 리소스 집약적입니다.** 복잡한 셰이더와 많은 후처리는 특히 오래된 하드웨어에서 프레임 레이트를 크게 떨어뜨릴 수 있습니다. 이는 시각적 복잡성과 성능 사이의 트레이드오프입니다.

## 👨‍💻 개발자에게 미치는 영향

### 역사적 알고리즘의 현대적 재해석

1976년의 Floyd-Steinberg 알고리즘이 여전히 최고 수준이라는 사실은 개발자에게 중요한 교훈을 줍니다. 최신 기술만이 답이 아니며, 잘 설계된 클래식 알고리즘은 시대를 초월합니다. 원본 논문을 읽고 이해하는 것이 여전히 가치 있습니다.

### 제약이 만드는 창의성

셰이더에서 폰트를 사용할 수 없다는 제약은 문자를 수학적으로 그리는 창의적 해결책을 낳았습니다. 오차 확산이 쉽게 병렬화되지 않는다는 제약은 CPU에서 디더링을 실행하고 GPU에서 블룸을 실행하는 하이브리드 아키텍처로 이어졌습니다. 기술적 한계는 다른 솔루션을 강제합니다.

### 디테일의 중요성

루미넌스 가중치(0.299, 0.587, 0.114)는 인간 시각 연구의 결과입니다. Floyd-Steinberg의 비대칭 오차 분포는 대각선 아티팩트를 발견한 누군가의 통찰입니다. 이러한 작은 결정들이 복합적으로 작용합니다.

### 학습 리소스

**논문:**
- Floyd & Steinberg (1976): "An adaptive algorithm for spatial grey scale"
- Jarvis, Judice & Ninke (1976): "A survey of techniques for the display of continuous tone pictures on bilevel displays"
- Donald Knuth (1987): "Digital Halftones by Dot Diffusion"

**학습 자료:**
- The Book of Shaders by Patricio Gonzalez Vivo
- Shadertoy (영감과 학습)
- Inigo Quilez의 셰이더 기법 아티클

**라이브러리:**
- postprocessing (Three.js 후처리 효과)
- React Three Fiber (React + Three.js 통합)

이 프로젝트는 클래식 알고리즘과 현대 웹 기술의 만남이 어떻게 독특한 비주얼 경험을 만들어내는지 보여줍니다. WebGL 셰이더의 성능과 역사적 이미지 처리 기법의 미학이 결합되어 실시간 레트로 효과를 구현한 사례로, 프론트엔드 개발자와 그래픽 프로그래머 모두에게 영감을 줄 수 있습니다.

> <a href="https://tympanus.net/codrops/2026/01/04/efecto-building-real-time-ascii-and-dithering-effects-with-webgl-shaders/" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>