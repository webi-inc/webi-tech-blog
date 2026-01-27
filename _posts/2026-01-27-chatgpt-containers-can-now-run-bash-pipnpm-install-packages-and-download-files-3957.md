---
title: "ChatGPT Containers: Bash 실행, pip/npm 패키지 설치, 파일 다운로드 기능 대폭 강화"
description: "ChatGPT Containers가 대규모 업그레이드를 통해 Bash 명령어 실행, pip/npm 패키지 설치, container.download 도구를 이용한 파일 다운로드 기능을 지원합니다. Python 외에도 10개 이상의 프로그래밍 언어를 직접 실행할 수 있게 되었습니다."
date: 2026-01-28 05:00:34 +0900
categories: ['Infrastructure', 'Cloud']
tags: ['ChatGPT', 'Containers', 'Code Interpreter', 'OpenAI', 'Bash', 'pip', 'npm', 'container.download', 'GPT-5.2']
image:
  path: "/assets/img/2026-01-27-chatgpt-containers-can-now-run-bash-pipnpm-install-packages-and-download-files-3957.jpg"
  alt: "ChatGPT Containers의 새로운 기능을 보여주는 인터페이스 화면"
published: true
---

## 3줄 요약
- ChatGPT Containers가 Bash 명령어 직접 실행, Node.js를 포함한 10개 이상의 프로그래밍 언어 지원으로 대폭 업그레이드되었습니다.
- container.download 도구를 통해 웹에서 파일을 다운로드하고, pip/npm 패키지를 설치할 수 있는 기능이 추가되었습니다.
- 공식 문서나 릴리스 노트 없이 조용히 배포된 이 기능은 무료 계정에서도 사용 가능합니다.

## 📌 주요 내용

### ChatGPT Containers의 혁신적인 업그레이드

ChatGPT의 코드 실행 기능이 지난 몇 달 사이 대규모 업그레이드를 거쳤습니다. 약 3년 전 ChatGPT Code Interpreter로 처음 출시된 이 기능은 "Advanced Data Analysis"로 이름이 변경되었다가, 이제 **ChatGPT Containers**라는 명칭으로 불리며 훨씬 강력한 기능을 제공합니다.

가장 주목할만한 변화는 다음과 같습니다:

1. **Bash 명령어 직접 실행**: 이전에는 Python의 `subprocess` 모듈을 통해서만 셸 명령어를 실행할 수 있었지만, 이제 Bash를 직접 실행할 수 있습니다.

2. **다중 언어 지원**: Node.js를 포함하여 JavaScript, Ruby, Perl, PHP, Go, Java, Swift, Kotlin, C, C++ 등 10개 이상의 프로그래밍 언어를 직접 실행할 수 있습니다. (Rust는 아직 지원되지 않습니다)

3. **패키지 관리**: 컨테이너가 외부 네트워크 요청을 직접 할 수 없음에도 불구하고, 커스텀 프록시 메커니즘을 통해 `pip install package`와 `npm install package`가 정상 작동합니다.

4. **파일 다운로드**: `container.download` 도구를 사용하여 웹에서 파일을 찾아 샌드박스 컨테이너 내부로 다운로드할 수 있습니다.

### container.download 도구의 작동 원리

`container.download`는 ChatGPT가 공개적으로 접근 가능한 URL에서 파일을 가져와 컨테이너 파일시스템에 저장하는 내장 도구입니다. 이 도구의 시그니처는 다음과 같습니다:

```json
{
  "url": "https://example.com/somefile.zip",
  "filepath": "/mnt/data/somefile.zip"
}
```

실제 테스트 결과, 이 도구는 다음과 같은 HTTP 헤더를 사용합니다:

```
User-Agent: Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; ChatGPT-User/1.0; +https://openai.com/bot
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, image/avif, image/webp, image/apng, */*;q=0.8, application/signed-exchange;v=b3;q=0.9
Cf-Connecting-Ip: 52.230.164.178
```

IP 주소는 미국 아이오와주 디모인의 Microsoft Azure Cloud(centralus)로 확인되었습니다.

### 데이터 유출 취약점은 없을까?

`container.download` 기능이 prompt injection 공격을 통해 민감한 데이터를 유출하는 데 악용될 수 있을지에 대한 우려가 있을 수 있습니다. 하지만 테스트 결과, 다음과 같은 안전장치가 작동합니다:

```
ERROR: download failed because url not viewed in conversation before. open the file or url using web.run first.
```

이는 Claude의 Web Fetch 도구와 유사한 보안 메커니즘으로, 사용자가 직접 입력했거나 prompt injection의 영향을 받지 않은 검색 결과에서 나온 URL만 접근할 수 있도록 제한합니다.

### Bash와 다양한 프로그래밍 언어 지원

Anthropic이 지난 2025년 9월 Claude에 코드 인터프리터 기능을 추가할 때 Bash를 중심으로 구축한 것처럼, OpenAI도 ChatGPT에 동일한 접근 방식을 적용한 것으로 보입니다.

실제 테스트에서 ChatGPT는 다음과 같은 명령어를 성공적으로 실행했습니다:

```bash
npm install cowsay
node -e "const cowsay=require('cowsay'); console.log(cowsay.say({text:'Hello from npm inside the container!'}))"
```

결과적으로 ASCII 아트로 그려진 소가 메시지를 말하는 출력을 확인할 수 있었습니다.

### pip과 npm 패키지 설치 메커니즘

컨테이너가 외부 네트워크 요청을 할 수 없음에도 패키지 설치가 가능한 이유는 `applied-caas-gateway1.internal.api.openai.org` 프록시 덕분입니다.

다음 환경 변수들이 `pip`와 `uv`가 이 프록시를 통해 패키지를 설치하도록 설정되어 있습니다:

```bash
PIP_INDEX_URL=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../pypi-public/simple
PIP_TRUSTED_HOST=packages.applied-caas-gateway1.internal.api.openai.org
UV_INDEX_URL=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../pypi-public/simple
UV_INSECURE_HOST=https://packages.applied-caas-gateway1.internal.api.openai.org
```

`npm`의 경우 다음 변수가 사용됩니다:

```bash
NPM_CONFIG_REGISTRY=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../npm-public
```

환경 변수에서 Cargo(Rust), Docker, Maven, Gradle 레지스트리에 대한 참조도 발견되어, 향후 더 많은 언어와 도구가 지원될 가능성을 시사합니다.

### GPT-5.2 Thinking에서 사용 가능한 전체 도구 목록

ChatGPT에게 직접 질문한 결과, 2026년 1월 26일 기준 GPT-5.2 Thinking 모델에서 사용 가능한 주요 도구는 다음과 같습니다:

- **python**: Jupyter 환경에서 Python 코드 실행
- **web.run**: 인터넷 접근, 검색, 페이지 열기/클릭/스크린샷 등
- **container.exec**: 컨테이너 내부에서 명령어 실행
- **container.download**: URL에서 파일 다운로드
- **container.open_image**: 컨테이너의 이미지 파일 표시
- **file_search.msearch**: 내부 소스 검색
- **gmail** (읽기 전용): Gmail 메시지 검색 및 읽기
- **gcal** (읽기 전용): Google Calendar 이벤트 검색
- **canmore**: Canvas 문서 생성 및 편집
- **image_gen.text2im**: 텍스트 프롬프트로 이미지 생성
- **bio.update**: 대화 간 사용자 정보 기억(메모리)

## 👨‍💻 개발자에게 미치는 영향

### 프로토타이핑과 테스트의 혁신

ChatGPT Containers는 개발자들에게 즉석에서 다양한 프로그래밍 언어로 코드를 작성하고 테스트할 수 있는 강력한 환경을 제공합니다. Python뿐만 아니라 JavaScript, Go, Java 등 10개 이상의 언어를 지원하므로, 언어별 구문이나 라이브러리 동작을 빠르게 확인할 수 있습니다.

### 패키지 생태계 활용

`pip`와 `npm` 패키지를 자유롭게 설치할 수 있다는 점은 매우 중요합니다. 이제 ChatGPT에게 특정 라이브러리를 사용한 코드를 요청하면, 직접 해당 패키지를 설치하고 실제로 작동하는 코드를 생성할 수 있습니다. 데이터 분석, 웹 스크래핑, 이미지 처리 등 복잡한 작업도 외부 라이브러리를 활용하여 수행할 수 있습니다.

### 문서 부재의 문제

가장 큰 아쉬움은 OpenAI가 이러한 중요한 기능 업데이트에 대한 공식 문서나 릴리스 노트를 제공하지 않았다는 점입니다. 개발자들이 이 기능의 한계와 가능성을 명확히 이해하려면, 커뮤니티가 직접 실험하고 문서화해야 하는 상황입니다.

### 보안 고려사항

`container.download` 기능의 보안 메커니즘은 잘 설계된 것으로 보이지만, 개발자들은 이 기능을 사용할 때 여전히 주의가 필요합니다. 특히 민감한 데이터를 다루는 경우, prompt injection 공격의 가능성을 항상 염두에 두어야 합니다.

### 무료 계정 접근성

이 모든 기능이 무료 ChatGPT 계정에서도 사용 가능하다는 점은 특히 주목할 만합니다. 개발 학습 단계에 있는 개발자들이나 스타트업 팀이 추가 비용 없이 강력한 코드 실행 환경을 활용할 수 있게 되었습니다.

> <a href="https://simonwillison.net/2026/Jan/26/chatgpt-containers/" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>