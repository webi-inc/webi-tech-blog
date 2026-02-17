---
title: "Gwtar: 단일 파일 HTML 아카이브의 효율성을 실현한 혁신적 포맷"
description: "Gwtar는 정적이면서도 단일 파일로 구성되며 효율적인 지연 로딩을 지원하는 새로운 HTML 아카이브 포맷입니다. HTTP Range 요청을 활용한 JavaScript 헤더로 대용량 HTML 파일을 효율적으로 제공합니다."
date: 2026-02-17 17:00:16 +0900
categories: ['Development', 'Web']
tags: ['Gwtar', 'HTML', '아카이브', '웹 표준', 'JavaScript', 'HTTP Range', 'SingleFile', '파일 포맷']
image:
  path: "/assets/img/2026-02-17-gwtar-a-static-efficient-single-file-html-format-1810.png"
  alt: "Gwtar 로고 - 기타 모양의 HTML 아카이브 포맷 심볼"
published: true
---

## 3줄 요약
- Gwtar는 정적(static), 단일 파일(single-file), 효율적(efficient) 속성을 모두 만족하는 새로운 HTML 아카이브 포맷입니다
- `window.stop()` 트릭과 HTTP Range 요청을 활용하여 필요한 자산만 지연 로딩하면서도 하나의 자체 완결형 HTML 파일로 제공됩니다
- 특별한 서버 설정이나 미래의 소프트웨어 지원 없이도 표준 웹 브라우저에서 바로 작동하는 호환성을 제공합니다

## 📌 주요 내용

### Gwtar의 탄생 배경

웹 페이지 아카이빙은 오랫동안 세 가지 중요한 속성 중 두 가지만 선택해야 하는 트릴레마에 직면해 왔습니다. **Gwtar**는 이 문제를 해결하기 위해 2026년 1월 Gwern.net에서 개발한 실험적 포맷으로, "기타(guitar)"처럼 발음되며 `.gwtar.html` 확장자를 사용합니다.

기존 HTML 아카이브 포맷들은 다음과 같은 한계가 있었습니다:
- **MHTML, MAFF, SingleFile**: 정적이고 단일 파일이지만, 전체 파일을 다운로드해야 하므로 비효율적
- **WARC/WACZ**: 정적이고 효율적이지만, 복잡한 소프트웨어(WebRecorder/Replay Webpage) 설치 필요
- **기본 브라우저 저장**: 단일 파일과 효율성은 제공하지만 정적이지 않음

### Gwtar의 핵심 기술: window.stop() 트릭

Gwtar의 혁신은 `window.stop()` JavaScript 명령어를 활용한다는 점입니다. 이 명령어는 브라우저에서 현재 로딩 중인 추가 리소스를 중단시키는 기능을 제공합니다(96% 이상의 브라우저 지원률).

**Gwtar의 구조:**
1. HTML + JavaScript 헤더
2. 원본 HTML과 자산들을 포함한 tarball
3. 선택적으로 PAR2 FEC(순방향 오류 정정) 데이터

JavaScript 헤더는 웹 브라우저가 파일의 나머지 부분을 로딩하는 것을 중단시키고, 원본 HTML만 먼저 로드한 후, 필요한 자산들을 HTTP Range 요청을 통해 tarball 내에서 선택적으로 가져옵니다.

### 실제 작동 방식

Gwtar 파일이 브라우저에서 열리면 다음과 같은 과정이 진행됩니다:

1. 브라우저가 일반 HTML 파일로 인식하고 로딩 시작
2. JavaScript가 실행되어 `window.stop()`으로 추가 다운로드 중단
3. 헤더의 JS가 원본 HTML을 로드
4. 페이지에 필요한 자산 요청을 가로채어 HTTP Range 요청으로 변환
5. tarball 내부의 해당 바이트 범위만 선택적으로 다운로드

예를 들어, Brian Moriarty의 강연 "The Secret of Psalm 46"는 원본 SingleFile 아카이브가 286MB였지만, Gwtar로 변환하면 사용자는 실제로 보는 부분만 다운로드하게 됩니다.

### 생성 및 사용 방법

Gwtar 파일은 PHP 참조 스크립트 `deconstruct_singlefile.php`를 통해 SingleFile HTML 스냅샷에서 생성할 수 있습니다:

```bash
php ./static/build/deconstruct_singlefile.php --create-gwtar --add-fec-data \
    original-file.html
```

이 명령은 JPG/PNG/GIF를 재압축하고 PAR2 FEC 데이터를 추가하여 파일 손상에 대비합니다.

### 압축 및 최적화

Gwtar는 직접적인 압축을 지원하지 않지만, 여러 최적화 방법을 활용합니다:

- **중복 제거**: 데이터 중복 제거 파일시스템에서 동일한 자산의 중복을 자동으로 제거
- **미디어 압축**: 빌드 단계에서 MP3, JPEG 등 미디어 자산을 사전 압축
- **연결 수준 압축**: gzip이나 Brotli 같은 전송 압축 활용 가능

### 제한사항과 해결책

**로컬 파일 보기 문제:**
보안상의 이유(CORS, 샌드박싱)로 로컬 HTML 페이지에서는 특정 요청이 차단되어 Gwtar가 작동하지 않을 수 있습니다. 이 경우 파일을 일반 HTML 포맷으로 쉽게 변환할 수 있습니다:

```bash
cat foo.gwtar.html | perl -ne'print $_ if $x; $x=1 if /<!-- GWTAR END/' | tar xf -
```

**Cloudflare 호환성:**
Cloudflare는 `text/html` 응답에 대해 Range 요청 헤더를 제거하는 비문서화된 동작을 보입니다. 이를 우회하기 위해 Gwern.net에서는 `x-gwtar` MIME 타입을 사용하며, Firefox와 Chromium은 `<html>` 태그를 감지하여 정상적으로 렌더링합니다.

### 추가 기능: FEC와 서명

Gwtar는 tarball 뒤에 추가 데이터를 붙일 수 있습니다:

**PAR2 순방향 오류 정정:**
파일 손상이나 부분 손실에 대비하여 25% PAR2 FEC를 추가할 수 있습니다:

```bash
par2create -r25 -n1 foo.gwtar.html && \
tar cf. - foo.gwtar.html.par2 foo.gwtar.html.vol*.par2 >> foo.gwtar.html && \
rm foo.gwtar.html*.par2
```

손상된 파일 복구:
```bash
ln --symbolic broken.gwtar.html broken.gwtar.html.par2 && \
par2repair broken.gwtar.html.par2 broken.gwtar.html
```

**GPG 서명:**
```bash
gpg --detach-sign --armor foo.gwtar.html
tar cf. - foo.gwtar.html.sig >> foo.gwtar.html
rm foo.gwtar.html.sig
```

### 미래 발전 방향

**Gwtar v1 개선사항:**
- 검증 도구 개발
- 렌더링 시 해시섬 체크
- 더 적극적인 자산 프리페칭
- SingleFile과의 통합

**Gwtar v2 가능성:**
- HTML 및 자산의 엄격한 검증
- 표준화된 추가 포맷
- 필수 FEC
- Brotli/gzip 내장 압축
- 다중 페이지 지원
- 콘텐츠 주소 지정 자산 이름으로 중복 제거 개선

## 👨‍💻 개발자에게 미치는 영향

### 아카이빙 워크플로우의 혁신

Gwtar는 대용량 HTML 아카이브를 다루는 개발자들에게 실질적인 솔루션을 제공합니다. 특히 문서 보존, 웹 아카이빙, 재현 가능한 과학 연구 등의 분야에서 유용합니다. 286MB짜리 웹 페이지를 효율적으로 제공하면서도 단일 파일로 관리할 수 있다는 것은 운영 복잡도를 크게 줄여줍니다.

### 표준 기술만 활용한 호환성

Gwtar의 가장 큰 장점은 기존 웹 표준만 사용한다는 점입니다. HTTP Range 요청은 오래된 표준이며, `window.stop()`은 96% 이상의 브라우저에서 지원됩니다. 특별한 서버 설정이나 플러그인 없이도 작동하므로, 미래의 사용자나 호스트가 특별한 조치를 취할 필요가 없습니다.

### 실무 적용 고려사항

개발자들은 다음 사항을 고려해야 합니다:

1. **Range 요청 지원 확인**: 서버가 HTTP 206 응답을 반환하는지 확인
```bash
curl --head --header "Range: bytes=0-99" 'https://example.com/file.gwtar.html'
```

2. **CDN 호환성**: Cloudflare와 같은 CDN을 사용하는 경우 MIME 타입 조정 필요
3. **로컬 개발 환경**: JavaScript 실행 제한으로 인해 로컬 파일 테스트 시 주의

### 재현 가능한 연구와 데이터 과학

Gwtar는 Sqlite3 데이터베이스 같은 대용량 바이너리 자산을 포함할 수 있어, 재현 가능한 과학 연구에 이상적입니다. `<object>` 태그와 load 이벤트 리스너를 활용하여 데이터셋을 효율적으로 로드할 수 있습니다.

### 장기 보존 전략

PAR2 FEC와 GPG 서명 지원으로 Gwtar는 단순한 아카이브 포맷을 넘어 장기 디지털 보존 솔루션으로 활용될 수 있습니다. 25% 중복도로 파일 손상에 대비하면서도, 중복 제거 파일시스템에서는 실제 저장 공간 낭비를 최소화할 수 있습니다.

> <a href="https://gwern.net/gwtar" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>