---
title: "Beautiful Mermaid: AI 코딩 시대를 위한 아름다운 다이어그램 렌더러"
description: "Beautiful Mermaid는 SVG와 ASCII 출력을 모두 지원하는 초고속 다이어그램 렌더링 라이브러리입니다. 15가지 테마, Shiki 호환성, 그리고 터미널 출력까지 지원합니다."
date: 2026-01-31 05:00:26 +0900
categories: ['Tools', 'AI Coding']
tags: [Beautiful Mermaid, Mermaid, 다이어그램, ASCII 렌더링, SVG, 테마, Shiki, 시각화]
image:
  path: "/assets/img/2026-01-30-beautiful-mermaid-3838.jpg"
  alt: "Beautiful Mermaid 라이브러리 로고 및 다이어그램 예시"
published: true
---

## 3줄 요약
- Beautiful Mermaid는 AI 코딩 환경을 위해 개발된 고속 다이어그램 렌더링 라이브러리로, SVG와 ASCII 출력을 모두 지원합니다.
- 단 2가지 색상만으로 아름다운 다이어그램을 생성하는 Mono Mode와 15가지 내장 테마, Shiki 호환성을 제공합니다.
- Flowchart, State, Sequence, Class, ER 다이어그램 등 5가지 유형을 지원하며, 100개 이상의 다이어그램을 500ms 이내에 렌더링합니다.

## 📌 주요 내용

### Beautiful Mermaid 개발 배경

Craft팀은 AI 보조 프로그래밍 환경에서 다이어그램의 중요성을 인식하고 Beautiful Mermaid를 개발했습니다. Mermaid는 텍스트 기반 다이어그램의 사실상 표준이지만, 기존 렌더러는 몇 가지 문제점을 가지고 있었습니다.

기존 Mermaid 렌더러의 한계는 다음과 같습니다:
- **미적 완성도** - 더욱 전문적인 외관 필요
- **복잡한 테마 커스터마이징** - CSS 클래스와의 씨름 필요
- **터미널 출력 미지원** - CLI 도구용 ASCII 렌더링 불가
- **무거운 의존성** - 단순 다이어그램에도 많은 코드 필요

Beautiful Mermaid는 Craft Agents를 지원하기 위해 개발되었으며, 리치 UI부터 일반 터미널까지 모든 환경에서 빠르고 아름답게 작동합니다.

### 핵심 기능

Beautiful Mermaid는 다음과 같은 강력한 기능을 제공합니다:

- **5가지 다이어그램 유형** - Flowchart, State, Sequence, Class, ER 다이어그램
- **이중 출력 모드** - 리치 UI용 SVG, 터미널용 ASCII/Unicode
- **15가지 내장 테마** - 간단하게 커스텀 테마 추가 가능
- **완전한 Shiki 호환성** - VS Code 테마 직접 사용 가능
- **라이브 테마 전환** - CSS 커스텀 속성으로 재렌더링 없이 전환
- **Mono Mode** - 단 2가지 색상으로 아름다운 다이어그램 생성
- **제로 DOM 의존성** - 순수 TypeScript, 어디서나 작동
- **초고속** - 100개 이상의 다이어그램을 500ms 이내에 렌더링

### 설치 및 기본 사용법

Beautiful Mermaid는 npm, bun, pnpm을 통해 설치할 수 있습니다:

```bash
npm install beautiful-mermaid
# 또는
bun add beautiful-mermaid
# 또는
pnpm add beautiful-mermaid
```

SVG 출력 예시:

```typescript
import { renderMermaid } from 'beautiful-mermaid'

const svg = await renderMermaid(`
  graph TD
  A[Start] --> B{Decision}
  B -->|Yes| C[Action]
  B -->|No| D[End]
`)
```

ASCII 출력 예시:

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'

const ascii = renderMermaidAscii(`graph LR; A --> B --> C`)
```

브라우저에서는 CDN을 통해 스크립트 태그로 직접 사용할 수 있습니다:

```html
<script src="https://unpkg.com/beautiful-mermaid/dist/beautiful-mermaid.browser.global.js"></script>
<script>
  const { renderMermaid, THEMES } = beautifulMermaid;
  renderMermaid('graph TD; A-->B').then(svg => { ... });
</script>
```

### 혁신적인 테마 시스템

Beautiful Mermaid의 테마 시스템은 강력하면서도 매우 간단합니다. 모든 다이어그램은 단 2가지 색상만 필요합니다: 배경(`bg`)과 전경(`fg`)입니다. 이 두 색상으로부터 `color-mix()`를 사용하여 전체 다이어그램이 파생됩니다.

**Mono Mode** 사용 예시:

```typescript
const svg = await renderMermaid(diagram, {
  bg: '#1a1b26',  // 배경
  fg: '#a9b1d6',  // 전경
})
```

시스템은 다음과 같이 자동으로 색상을 파생합니다:

| 요소 | 파생 방식 |
| --- | --- |
| 텍스트 | `--fg` 100% |
| 보조 텍스트 | `--fg` 60%를 `--bg`에 믹스 |
| 엣지 레이블 | `--fg` 40%를 `--bg`에 믹스 |
| 커넥터 | `--fg` 30%를 `--bg`에 믹스 |
| 화살표 | `--fg` 50%를 `--bg`에 믹스 |
| 노드 배경 | `--fg` 3%를 `--bg`에 믹스 |
| 노드 테두리 | `--fg` 20%를 `--bg`에 믹스 |

더욱 풍부한 테마를 원한다면 선택적 "enrichment" 색상을 제공할 수 있습니다:

```typescript
const svg = await renderMermaid(diagram, {
  bg: '#1a1b26',
  fg: '#a9b1d6',
  // 선택적 enrichment:
  line: '#3d59a1',    // 엣지/커넥터 색상
  accent: '#7aa2f7',  // 화살표, 강조
  muted: '#565f89',   // 보조 텍스트, 레이블
  surface: '#292e42', // 노드 배경 틴트
  border: '#3d59a1',  // 노드 테두리
})
```

### 15가지 내장 테마

Beautiful Mermaid는 세심하게 큐레이션된 15가지 테마를 제공합니다:

- **Light 테마**: zinc-light, tokyo-night-light, catppuccin-latte, nord-light, github-light, solarized-light
- **Dark 테마**: zinc-dark, tokyo-night, tokyo-night-storm, catppuccin-mocha, nord, dracula, github-dark, solarized-dark, one-dark

테마 사용 예시:

```typescript
import { renderMermaid, THEMES } from 'beautiful-mermaid'

const svg = await renderMermaid(diagram, THEMES['tokyo-night'])
```

### Shiki와의 완벽한 통합

VS Code 테마를 Shiki를 통해 직접 사용할 수 있습니다:

```typescript
import { getSingletonHighlighter } from 'shiki'
import { renderMermaid, fromShikiTheme } from 'beautiful-mermaid'

const highlighter = await getSingletonHighlighter({
  themes: ['vitesse-dark', 'rose-pine', 'material-theme-darker']
})

const colors = fromShikiTheme(highlighter.getTheme('vitesse-dark'))
const svg = await renderMermaid(diagram, colors)
```

### 지원하는 다이어그램 유형

**Flowchart** - 모든 방향 지원(TD, LR, BT, RL):
```
graph TD
  A[Start] --> B{Decision}
  B -->|Yes| C[Process]
  B -->|No| D[End]
```

**State Diagram**:
```
stateDiagram-v2
  [*] --> Idle
  Idle --> Processing: start
  Processing --> Complete: done
```

**Sequence Diagram**:
```
sequenceDiagram
  Alice->>Bob: Hello Bob!
  Bob-->>Alice: Hi Alice!
```

**Class Diagram**:
```
classDiagram
  Animal <|-- Duck
  Animal: +int age
  Duck: +swim()
```

**ER Diagram**:
```
erDiagram
  CUSTOMER ||--o{ ORDER : places
  ORDER ||--|{ LINE_ITEM : contains
```

### 터미널을 위한 ASCII 출력

CLI 도구나 터미널 환경을 위해 ASCII 또는 Unicode 박스 드로잉 문자로 렌더링할 수 있습니다:

```typescript
import { renderMermaidAscii } from 'beautiful-mermaid'

// Unicode 모드 (기본값) - 더 예쁜 박스 드로잉
const unicode = renderMermaidAscii(`graph LR; A --> B`)

// 순수 ASCII 모드 - 최대 호환성
const ascii = renderMermaidAscii(`graph LR; A --> B`, { useAscii: true })
```

ASCII 렌더링 엔진은 Alexander Grooff의 mermaid-ascii를 기반으로 하며, Go에서 TypeScript로 포팅되었습니다.

## 👨‍💻 개발자에게 미치는 영향

### AI 코딩 환경의 시각화 혁신

Beautiful Mermaid는 AI 보조 프로그래밍 환경에서 다이어그램 시각화의 새로운 표준을 제시합니다. LLM과 대화하면서 실시간으로 시스템 아키텍처나 데이터 플로우를 시각화할 수 있어, 복잡한 개념을 즉시 이해할 수 있습니다.

### 성능과 경량화

기존 Mermaid 렌더러의 무거운 의존성 문제를 해결하고, 100개 이상의 다이어그램을 500ms 이내에 렌더링하는 초고속 성능을 제공합니다. 순수 TypeScript로 구현되어 제로 DOM 의존성을 달성했으며, 이는 다양한 환경에서의 유연한 활용을 가능하게 합니다.

### 개발자 경험 향상

단 2가지 색상으로 시작하는 Mono Mode부터 Shiki 테마 통합까지, 개발자 친화적인 API 설계가 돋보입니다. CSS 커스텀 속성을 활용한 라이브 테마 전환은 재렌더링 없이 즉각적인 스타일 변경을 가능하게 하여, 개발 생산성을 크게 향상시킵니다.

### 터미널 환경 지원

ASCII/Unicode 출력 지원은 CLI 도구 개발자들에게 특히 유용합니다. 리치 UI가 없는 환경에서도 다이어그램을 효과적으로 표현할 수 있어, 서버 관리, 디버깅, 로그 분석 등 다양한 시나리오에서 활용 가능합니다.

> <a href="https://github.com/lukilabs/beautiful-mermaid" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>