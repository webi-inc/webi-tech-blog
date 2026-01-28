---
title: "ChatGPT Containers, 이제 Bash 실행과 pip/npm 패키지 설치 지원"
description: "ChatGPT Containers가 대폭 업그레이드되어 Bash 명령 실행, Node.js 지원, pip/npm 패키지 설치, 파일 다운로드 기능을 제공합니다. 10개 이상의 프로그래밍 언어를 지원하는 새로운 기능을 살펴보세요."
date: 2026-01-28 17:00:38 +0900
categories: ['Development', 'Backend']
tags: [ChatGPT, Containers, Code Interpreter, Bash, Python, Node.js, pip, npm, OpenAI, GPT-5.2]
image:
  path: "/assets/img/2026-01-28-chatgpt-containers-can-now-run-bash-pipnpm-install-packages-and-download-files-7381.jpg"
  alt: "ChatGPT Containers의 새로운 기능을 보여주는 인터페이스 화면"
published: true
---

## 3줄 요약
- ChatGPT Containers가 Bash 명령 직접 실행과 Node.js를 포함한 10개 이상의 프로그래밍 언어를 지원하게 되었습니다.
- pip install과 npm install을 통한 패키지 설치가 가능해져, 개발자들이 더 다양한 라이브러리를 활용할 수 있습니다.
- container.download 도구를 통해 웹에서 파일을 다운로드하여 샌드박스 환경에서 처리할 수 있는 기능이 추가되었습니다.

## 📌 주요 내용

### ChatGPT Containers의 혁신적인 업그레이드

ChatGPT의 가장 유용한 기능 중 하나인 컨테이너 내 코드 작성 및 실행 기능이 대폭 강화되었습니다. 이 기능은 약 3년 전 ChatGPT Code Interpreter로 출시되었으며, 한때 "Advanced Data Analysis"로 리브랜딩되었습니다. 흥미롭게도 지난 몇 달 동안 대규모 업그레이드가 이루어졌지만, 공식 문서에서는 이에 대한 상세한 내용을 찾기 어려운 상황입니다.

### Bash 명령 직접 실행 지원

ChatGPT Containers는 이제 **Bash 명령을 직접 실행**할 수 있습니다. 이전에는 Python 코드만 실행할 수 있었으며, 셸 명령은 Python의 `subprocess` 모듈을 통해서만 가능했습니다. Bash 지원은 Claude Code나 Codex CLI와 같은 코딩 에이전트에서 입증된 것처럼, 거의 모든 작업을 수행할 수 있는 강력한 기능입니다.

사용자가 "npm install a fun package and demonstrate using it"라는 프롬프트를 입력하면, ChatGPT는 다음과 같이 Bash 명령을 실행합니다:

```bash
mkdir -p /mnt/data/npmfun
cd /mnt/data/npmfun
npm init -y
npm install cowsay --no-fund --no-audit
node -e "const cowsay=require('cowsay'); console.log(cowsay.say({text:'Hello from npm inside the container!'}))"
```

### 다양한 프로그래밍 언어 지원

**Node.js**가 추가되어 Python 외에도 JavaScript를 직접 실행할 수 있게 되었습니다. 더 나아가 ChatGPT는 **Ruby, Perl, PHP, Go, Java, Swift, Kotlin, C, C++**에서 "Hello World"를 실행하는 데 성공했습니다. 현재 Rust는 아직 지원되지 않지만, 환경 변수에서 Cargo 레지스트리 참조가 발견되어 향후 추가될 가능성이 있습니다.

### pip와 npm을 통한 패키지 설치

컨테이너는 여전히 아웃바운드 네트워크 요청을 수행할 수 없지만, **`pip install package`와 `npm install package`가 커스텀 프록시 메커니즘을 통해 작동**합니다. 

다음과 같은 환경 변수가 `pip`와 `uv`가 프록시를 통해 패키지를 설치하도록 설정되어 있습니다:

```
PIP_INDEX_URL=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../pypi-public/simple
UV_INDEX_URL=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../pypi-public/simple
```

npm의 경우:

```
NPM_CONFIG_REGISTRY=https://reader:****@packages.applied-caas-gateway1.internal.api.openai.org/.../npm-public
```

이러한 설정을 통해 `applied-caas-gateway1.internal.api.openai.org` 프록시가 컨테이너 내에서 사용 가능하며, 다양한 패키징 도구들이 이를 활용하도록 구성되어 있습니다.

### container.download 도구의 등장

새롭게 추가된 `container.download` 도구는 **공개적으로 접근 가능한 URL에서 파일을 다운로드하여 컨테이너 파일시스템에 저장**할 수 있습니다. 

도구의 시그니처는 다음과 같습니다:

```json
{
  "url": "https://example.com/somefile.zip",
  "filepath": "/mnt/data/somefile.zip"
}
```

사용자가 로스앤젤레스의 대기 질에 대해 질문했을 때, ChatGPT GPT-5.2 Thinking 세션은 자동으로 관련 Excel 파일을 찾아 `container.download`를 사용하여 다운로드하고 분석했습니다.

다운로드 요청 시 사용되는 User-Agent는 다음과 같습니다:

```
User-Agent: Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; ChatGPT-User/1.0; +https://openai.com/bot
```

### 보안 메커니즘

`container.download`는 데이터 유출 취약점이 될 수 있을까요? OpenAI는 이를 방지하기 위한 안전장치를 마련했습니다. URL 접근은 사용자가 직접 입력했거나 프롬프트 인젝션의 영향을 받지 않은 검색 결과에서 나온 경우에만 허용됩니다.

ChatGPT가 쿼리 문자열이 포함된 URL을 조합하여 접근하려고 하면 다음과 같은 오류가 발생합니다:

```
ERROR: download failed because url not viewed in conversation before. open the file or url using web.run first.
```

이는 Claude의 Web Fetch 도구에서 사용된 것과 유사한 안전 메커니즘입니다.

### 환경 분석 결과

ChatGPT가 자체적으로 환경을 탐색한 결과, 다음과 같은 흥미로운 환경 변수들이 발견되었습니다:

```
CAAS_ARTIFACTORY_BASE_URL=packages.applied-caas-gateway1.internal.api.openai.org
CAAS_ARTIFACTORY_CARGO_REGISTRY=.../artifactory/api/cargo/cargo-public/index
CAAS_ARTIFACTORY_DOCKER_REGISTRY=.../dockerhub-public
CAAS_ARTIFACTORY_GO_REGISTRY=.../artifactory/api/go/golang-main
CAAS_ARTIFACTORY_MAVEN_REGISTRY=.../artifactory/maven-public
CAAS_ARTIFACTORY_GRADLE_REGISTRY=.../artifactory/gradle-public
```

Rust나 Docker는 현재 컨테이너 환경에 설치되어 있지 않지만, 이러한 레지스트리 참조는 향후 추가될 기능에 대한 힌트일 수 있습니다.

### 사용 가능한 전체 도구 목록

ChatGPT에게 사용 가능한 모든 도구를 나열하도록 요청한 결과, 다음과 같은 주요 도구들이 확인되었습니다:

- **python.exec**: Jupyter 환경에서 Python 코드 실행
- **web.run**: 인터넷 접근 및 검색, 페이지 열기/클릭, 스크린샷 등
- **container.exec**: 컨테이너 내부에서 명령 실행
- **container.download**: URL에서 파일 다운로드
- **container.open_image**: 컨테이너 내 이미지 표시
- **image_gen.text2im**: 텍스트 지시사항으로부터 이미지 생성
- **automations.create/update/list**: 미래/반복 작업 예약
- **bio.update**: 대화 간 사용자 정보 저장 (메모리)

Gmail, Google Calendar, Google Contacts에 대한 읽기 전용 도구들도 포함되어 있습니다.

## 👨‍💻 개발자에게 미치는 영향

### 프로토타이핑과 테스팅의 혁신

개발자들은 이제 ChatGPT를 통해 Python뿐만 아니라 JavaScript, Go, Java 등 다양한 언어로 코드를 작성하고 즉시 테스트할 수 있습니다. pip와 npm을 통한 패키지 설치가 가능해져, 실제 프로젝트에서 사용하는 라이브러리를 활용한 프로토타이핑이 가능합니다.

### 데이터 분석 워크플로우 개선

`container.download` 기능을 통해 공개 데이터셋을 직접 다운로드하고 분석할 수 있게 되어, 데이터 분석 워크플로우가 크게 개선되었습니다. 사용자가 업로드한 파일뿐만 아니라 웹에서 찾은 파일도 즉시 처리할 수 있습니다.

### 공식 문서의 필요성

이러한 강력한 기능들에도 불구하고, 가장 큰 문제는 **공식 문서의 부재**입니다. OpenAI의 릴리스 노트는 이러한 주요 업데이트를 반영하지 않고 있으며, 기능의 제한사항과 사용 방법에 대한 상세한 정보가 필요합니다. 개발자 커뮤니티는 이러한 기능들이 어떻게 작동하는지, 그리고 실제 프로젝트에서 어떻게 활용할 수 있는지에 대한 공식 가이드를 필요로 합니다.

### 무료 계정에서도 사용 가능

흥미롭게도 이러한 새로운 기능들은 무료 ChatGPT 계정에서도 사용할 수 있는 것으로 확인되었습니다. 이는 더 많은 개발자들이 이 기능을 활용할 수 있음을 의미합니다.

> <a href="https://simonwillison.net/2026/Jan/26/chatgpt-containers/" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>