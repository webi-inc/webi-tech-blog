---
title: "pi-coding-agent: 최소주의 코딩 에이전트 구축에서 배운 것들"
description: "pi-coding-agent 개발 과정에서 얻은 통찰력을 공유합니다. 통합 LLM API, 터미널 UI 프레임워크, 그리고 최소한의 도구로 효과적인 코딩 에이전트를 만드는 방법을 알아봅니다."
date: 2026-02-03 05:00:36 +0900
categories: ['Development', 'Web']
tags: [코딩에이전트, LLM, AI, pi-coding-agent, TUI, TypeScript]
image:
  path: "/assets/img/2026-02-02-what-i-learned-building-an-opinionated-and-minimal-coding-agent-2465.png"
  alt: "pi-coding-agent 프로젝트 헤더 이미지"
published: true
---

## 3줄 요약
- pi-coding-agent는 최소주의 철학으로 구축된 코딩 에이전트 하네스로, 완전한 제어와 관찰 가능성을 제공합니다.
- 통합 LLM API(pi-ai), 터미널 UI 프레임워크(pi-tui), 에이전트 코어(pi-agent-core)를 포함한 완전한 스택을 직접 구현했습니다.
- Terminal-Bench 2.0 벤치마크에서 Claude Opus 4.5와 함께 상위권 성능을 달성하며 최소주의 접근의 효과를 입증했습니다.

## 📌 주요 내용

### pi-coding-agent를 만들게 된 배경

지난 3년간 LLM을 활용한 코드 작성 도구의 진화를 경험했습니다. ChatGPT에 코드를 복사-붙여넣기하던 시절부터 Copilot 자동완성, Cursor, 그리고 2025년의 새로운 코딩 에이전트 하네스인 Claude Code, Codex, Amp, Droid, opencode에 이르기까지 많은 변화가 있었습니다.

Claude Code를 주로 사용했지만 몇 가지 문제점이 있었습니다. 시스템 프롬프트와 도구가 릴리스마다 변경되어 워크플로우가 깨지고, 백그라운드에서 주입되는 컨텍스트를 정확히 제어할 수 없었으며, 모델과의 상호작용을 완전히 검사할 수 없었습니다. 이러한 한계를 극복하고자 직접 코딩 에이전트 하네스를 구축하기로 결정했습니다.

### pi 프로젝트의 핵심 컴포넌트

pi 프로젝트는 네 개의 핵심 패키지로 구성됩니다:

**pi-ai**: Anthropic, OpenAI, Google, xAI, Groq, Cerebras, OpenRouter 및 OpenAI 호환 엔드포인트를 지원하는 통합 LLM API입니다. 스트리밍, TypeBox 스키마를 사용한 도구 호출, 사고/추론 지원, 프로바이더 간 원활한 컨텍스트 전환, 토큰 및 비용 추적 기능을 제공합니다.

**pi-agent-core**: 도구 실행, 검증, 이벤트 스트리밍을 처리하는 에이전트 루프입니다. 상태 관리, 간소화된 이벤트 구독, 메시지 큐잉, 첨부파일 처리 기능을 제공합니다.

**pi-tui**: 차등 렌더링, 동기화된 출력을 통한 거의 깜빡임 없는 업데이트, 자동완성 및 마크다운 렌더링을 지원하는 에디터와 같은 컴포넌트를 갖춘 최소 터미널 UI 프레임워크입니다.

**pi-coding-agent**: 세션 관리, 커스텀 도구, 테마, 프로젝트 컨텍스트 파일을 통해 모든 것을 연결하는 실제 CLI입니다.

### 통합 LLM API 구축의 과제

#### 네 가지 핵심 API

실질적으로 거의 모든 LLM 프로바이더와 통신하기 위해서는 네 가지 API만 필요합니다: OpenAI의 Completions API, 새로운 Responses API, Anthropic의 Messages API, Google의 Generative AI API입니다.

이들은 기능면에서 매우 유사하지만 프로바이더별 특이사항이 있습니다. 예를 들어:
- Cerebras, xAI, Mistral, Chutes는 `store` 필드를 허용하지 않습니다
- Mistral과 Chutes는 `max_completion_tokens` 대신 `max_tokens`를 사용합니다
- Cerebras, xAI, Mistral, Chutes는 시스템 프롬프트에 대한 `developer` 역할을 지원하지 않습니다
- Grok 모델은 `reasoning_effort`를 허용하지 않습니다

#### 프로바이더 간 컨텍스트 전환

pi-ai는 처음부터 프로바이더 간 컨텍스트 전환을 염두에 두고 설계되었습니다. 각 프로바이더는 도구 호출과 사고 추적을 추적하는 고유한 방식이 있기 때문에 이는 최선의 노력만 가능합니다. 예를 들어, 세션 중간에 Anthropic에서 OpenAI로 전환하면 Anthropic 사고 추적이 `<thinking></thinking>` 태그로 구분된 어시스턴트 메시지 내 콘텐츠 블록으로 변환됩니다.

```typescript
import { getModel, complete, Context } from '@mariozechner/pi-ai';

// Claude로 시작
const claude = getModel('anthropic', 'claude-sonnet-4-5');
const context: Context = {
  messages: []
};

context.messages.push({ role: 'user', content: '25 곱하기 18은?' });
const claudeResponse = await complete(claude, context, {
  thinkingEnabled: true
});
context.messages.push(claudeResponse);

// GPT로 전환
const gpt = getModel('openai', 'gpt-5.1-codex');
context.messages.push({ role: 'user', content: '그게 맞나요?' });
const gptResponse = await complete(gpt, context);
context.messages.push(gptResponse);
```

#### 요청 중단 지원

많은 통합 LLM API가 요청 중단 기능을 완전히 무시하는데, 이는 프로덕션 시스템에 LLM을 통합하려는 경우 완전히 받아들일 수 없는 일입니다. pi-ai는 도구 호출을 포함한 전체 파이프라인에서 중단을 지원하도록 처음부터 설계되었습니다.

```typescript
import { getModel, stream } from '@mariozechner/pi-ai';

const model = getModel('openai', 'gpt-5.1-codex');
const controller = new AbortController();

// 2초 후 중단
setTimeout(() => controller.abort(), 2000);

const s = stream(model, {
  messages: [{ role: 'user', content: '긴 이야기를 써주세요' }]
}, {
  signal: controller.signal
});

for await (const event of s) {
  if (event.type === 'text_delta') {
    process.stdout.write(event.delta);
  } else if (event.type === 'error') {
    console.log(`${event.reason === 'aborted' ? '중단됨' : '오류'}:`, event.error.errorMessage);
  }
}
```

### 터미널 UI 프레임워크 구현

#### 두 가지 유형의 TUI

터미널 사용자 인터페이스를 작성하는 방법은 기본적으로 두 가지입니다. 하나는 터미널 뷰포트의 소유권을 가져와 픽셀 버퍼처럼 취급하는 것입니다(풀스크린 TUI). Amp와 opencode가 이 방식을 사용합니다.

두 번째 접근 방식은 일반 CLI 프로그램처럼 터미널에 작성하여 스크롤백 버퍼에 콘텐츠를 추가하고, 애니메이션 스피너나 텍스트 편집 필드와 같은 항목을 다시 그리기 위해 가시 뷰포트 내에서 가끔 "렌더링 커서"를 조금 위로 이동하는 것입니다. Claude Code, Codex, Droid가 이 방식을 사용합니다.

코딩 에이전트는 기본적으로 채팅 인터페이스이기 때문에 "네이티브" 터미널 에뮬레이터와 함께 작동하는 것이 적합합니다. pi-tui는 이 방향을 선택했습니다.

#### 보존 모드 UI와 차등 렌더링

pi-tui는 간단한 보존 모드 접근 방식을 사용합니다. `Component`는 `render(width)` 메서드를 가진 객체로, 문자열 배열(ANSI 이스케이프 코드가 포함된 뷰포트에 수평으로 맞는 라인)을 반환하고 키보드 입력을 위한 선택적 `handleInput(data)` 메서드를 가집니다.

차등 렌더링 알고리즘은 간단합니다:
1. **첫 렌더링**: 모든 라인을 터미널에 출력
2. **너비 변경**: 화면을 완전히 지우고 모든 것을 다시 렌더링
3. **일반 업데이트**: 화면에 표시된 것과 다른 첫 번째 라인을 찾아 커서를 해당 라인으로 이동하고 끝까지 다시 렌더링

깜빡임을 방지하기 위해 pi-tui는 모든 렌더링을 동기화된 출력 이스케이프 시퀀스(`CSI ?2026h` 및 `CSI ?2026l`)로 감쌉니다. 이는 터미널에 모든 출력을 버퍼링하고 원자적으로 표시하도록 지시합니다. 대부분의 최신 터미널이 이를 지원합니다.

### pi-coding-agent의 철학적 차별점

#### 최소한의 시스템 프롬프트

pi의 시스템 프롬프트는 매우 간결합니다:

```markdown
You are an expert coding assistant. You help users with coding tasks by reading files, executing commands, editing code, and writing new files.

Available tools:
- read: Read file contents
- bash: Execute bash commands
- edit: Make surgical edits to files
- write: Create or overwrite files

Guidelines:
- Use bash for file operations like ls, grep, find
- Use read to examine files before editing
- Use edit for precise changes (old text must match exactly)
- Use write only for new files or complete rewrites
- When summarizing your actions, output plain text directly - do NOT use cat or bash to display what you did
- Be concise in your responses
- Show file paths clearly when working with files
```

Claude Code의 시스템 프롬프트, Codex의 시스템 프롬프트, 또는 opencode의 모델별 프롬프트와 비교하면 극히 간단합니다. 하지만 모든 프론티어 모델이 강화학습으로 충분히 훈련되어 있어 코딩 에이전트가 무엇인지 본질적으로 이해하기 때문에 10,000 토큰의 시스템 프롬프트가 필요하지 않은 것으로 나타났습니다.

#### 최소한의 도구셋

pi는 네 가지 핵심 도구만 제공합니다:
- **read**: 파일 내용 읽기 (텍스트 파일 및 이미지 지원)
- **write**: 파일에 콘텐츠 작성
- **edit**: 정확한 텍스트를 대체하여 파일 편집
- **bash**: 현재 작업 디렉토리에서 bash 명령 실행

pi의 시스템 프롬프트와 도구 정의를 합치면 1000 토큰 미만입니다.

#### 기본적으로 YOLO 모드

pi는 완전한 YOLO 모드로 실행되며 사용자가 무엇을 하는지 알고 있다고 가정합니다. 파일 시스템에 대한 무제한 액세스 권한이 있으며 권한 확인이나 안전 장치 없이 모든 명령을 실행할 수 있습니다.

에이전트가 코드를 작성하고 실행할 수 있게 되면 사실상 게임이 끝난 것과 같습니다. 데이터 유출을 방지할 수 있는 유일한 방법은 에이전트가 실행되는 실행 환경에 대한 모든 네트워크 액세스를 차단하는 것인데, 이는 에이전트를 대부분 쓸모없게 만듭니다.

#### 내장 할 일 목록 없음

pi는 내장 할 일 목록을 지원하지 않으며 앞으로도 지원하지 않을 것입니다. 경험상 할 일 목록은 일반적으로 모델을 돕기보다 더 혼란스럽게 만듭니다.

작업 추적이 필요한 경우 파일에 작성하여 외부적으로 상태를 유지할 수 있습니다:

```markdown
# TODO.md

- [x] 사용자 인증 구현
- [x] 데이터베이스 마이그레이션 추가
- [ ] API 문서 작성
- [ ] 속도 제한 추가
```

#### 계획 모드 없음

pi는 내장 계획 모드가 없으며 앞으로도 없을 것입니다. 파일을 수정하거나 명령을 실행하지 않고 에이전트에게 문제를 함께 생각하도록 지시하는 것으로 일반적으로 충분합니다.

세션 간 지속적인 계획이 필요한 경우 파일에 작성할 수 있습니다:

```markdown
# PLAN.md

## 목표
OAuth를 지원하도록 인증 시스템 리팩토링

## 접근 방식
1. OAuth 2.0 플로우 연구
2. 토큰 저장 스키마 설계
3. 인증 서버 엔드포인트 구현
4. 클라이언트 측 로그인 플로우 업데이트
5. 테스트 추가

## 현재 단계
3단계 작업 중 - 인증 엔드포인트
```

#### MCP 지원 없음

pi는 MCP를 지원하지 않으며 앞으로도 지원하지 않을 것입니다. MCP 서버는 대부분의 사용 사례에서 과도하며 상당한 컨텍스트 오버헤드가 발생합니다.

Playwright MCP(21개 도구, 13.7k 토큰) 또는 Chrome DevTools MCP(26개 도구, 18k 토큰)와 같은 인기 있는 MCP 서버는 모든 세션에서 전체 도구 설명을 컨텍스트에 덤프합니다. 작업을 시작하기도 전에 컨텍스트 윈도우의 7-9%가 사라집니다.

대안은 간단합니다: README 파일이 있는 CLI 도구를 구축하는 것입니다. 에이전트는 도구가 필요할 때 README를 읽고, 필요할 때만 토큰 비용을 지불하며(점진적 공개), bash를 사용하여 도구를 호출할 수 있습니다.

#### 백그라운드 bash 없음

pi의 bash 도구는 명령을 동기적으로 실행합니다. 개발 서버를 시작하거나 백그라운드에서 테스트를 실행하거나 명령이 여전히 실행 중인 동안 REPL과 상호작용할 수 있는 내장 방법이 없습니다.

대신 tmux를 사용하세요. tmux는 또한 모든 활성 세션을 나열하는 CLI 인수를 제공합니다. 백그라운드 bash는 필요하지 않습니다.

#### 서브 에이전트 없음

pi에는 전용 서브 에이전트 도구가 없습니다. Claude Code는 복잡한 작업을 수행해야 할 때 종종 서브 에이전트를 생성하여 작업의 일부를 처리합니다. 하지만 해당 서브 에이전트가 무엇을 하는지 전혀 볼 수 없습니다.

pi가 자신을 생성해야 하는 경우 bash를 통해 실행하도록 요청하면 됩니다. 심지어 완전한 관찰 가능성과 해당 서브 에이전트와 직접 상호작용할 수 있는 능력을 위해 tmux 세션 내에서 자신을 생성하도록 할 수도 있습니다.

### 벤치마크 결과

Terminal-Bench 2.0에서 pi를 Claude Opus 4.5와 함께 테스트했으며 Codex, Cursor, Windsurf 및 기타 코딩 하네스와 경쟁시켰습니다. 작업당 5번의 시도로 완전한 실행을 수행한 결과, pi는 리더보드에서 상위권에 위치했습니다.

2025년 12월 2일 기준 현재 리더보드에서 pi의 순위는 최상위권을 기록하고 있으며, 이는 최소주의 접근 방식이 훨씬 더 정교한 도구를 갖춘 에이전트만큼 잘 수행할 수 있다는 증거입니다.

흥미롭게도 Terminal-Bench 팀의 자체 최소 에이전트인 Terminus 2도 리더보드에서 좋은 성과를 보이고 있습니다. Terminus 2는 모델에게 tmux 세션만 제공합니다. 모델은 명령을 텍스트로 tmux에 보내고 터미널 출력 자체를 파싱합니다. 멋진 도구도 없고 파일 작업도 없이 원시 터미널 상호작용만으로 훨씬 더 정교한 도구를 갖춘 에이전트와 대등한 성능을 보이고 있습니다.

## 👨‍💻 개발자에게 미치는 영향

### 컨텍스트 엔지니어링의 중요성

pi 프로젝트는 컨텍스트 엔지니어링이 무엇보다 중요하다는 것을 보여줍니다. 모델의 컨텍스트에 들어가는 것을 정확히 제어하면 특히 코드를 작성할 때 더 나은 출력을 얻을 수 있습니다. 기존 하네스는 UI에 표시되지 않는 내용을 백그라운드에서 주입하여 이를 극도로 어렵게 만들거나 불가능하게 만듭니다.

### 최소주의의 힘

Terminal-Bench 결과는 최소주의 접근 방식이 실제로 효과가 있음을 증명합니다. 1000 토큰 미만의 시스템 프롬프트와 네 가지 핵심 도구만으로도 10,000 토큰 이상의 복잡한 시스템 프롬프트를 사용하는 다른 하네스와 경쟁할 수 있습니다.

### 관찰 가능성과 제어

pi는 모델과의 모든 상호작용을 검사할 수 있는 완전한 관찰 가능성을 제공합니다. 에이전트가 실제로 어떤 소스를 살펴봤는지, 어떤 것을 완전히 놓쳤는지 확인할 수 있습니다. 이러한 투명성은 디버깅과 워크플로우 최적화에 필수적입니다.

### 멀티 모델 세계

pi-ai의 통합 API는 세션 중간에 프로바이더와 모델을 전환할 수 있게 해줍니다. 이는 각 모델의 강점을 활용하고, 비용을 최적화하며, 셀프 호스팅 모델을 실험할 수 있는 유연성을 제공합니다.

### 오픈소스 기여

pi는 완전히 오픈소스이며 GitHub에서 사용할 수 있습니다. 개발자는 자신의 필요에 맞게 포크하고 수정할 수 있으며, 더 나은 솔루션을 만들 경우 커뮤니티와 공유할 수 있습니다. 이는 AI 도구 생태계의 발전에 기여할 수 있는 좋은 기회입니다.

> <a href="https://mariozechner.at/posts/2025-11-30-pi-coding-agent/" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>