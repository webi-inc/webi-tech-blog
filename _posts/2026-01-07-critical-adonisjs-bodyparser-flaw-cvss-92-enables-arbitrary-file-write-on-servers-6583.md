---
title: "AdonisJS Bodyparser 취약점 CVE-2026-21440, CVSS 9.2 점수로 임의 파일 쓰기 공격 가능"
description: "AdonisJS Bodyparser 패키지에서 발견된 치명적인 경로 순회 취약점 CVE-2026-21440은 CVSS 9.2 점수를 기록하며, 공격자가 서버에 임의의 파일을 작성할 수 있는 심각한 보안 위협을 초래합니다."
date: 2026-01-07 17:01:21 +0900
categories: ['Development', 'Backend']
tags: ['AdonisJS', 'CVE-2026-21440', 'Path Traversal', 'npm', 'Node.js', 'Bodyparser', '보안취약점', 'MultipartFile']
image:
  path: "/assets/img/2026-01-07-critical-adonisjs-bodyparser-flaw-cvss-92-enables-arbitrary-file-write-on-servers-6583.jpg"
  alt: "AdonisJS 보안 취약점을 나타내는 이미지"
published: true
---

## 3줄 요약
- AdonisJS의 @adonisjs/bodyparser npm 패키지에서 CVSS 9.2 점수의 치명적인 경로 순회 취약점(CVE-2026-21440)이 발견되었습니다.
- MultipartFile.move() 함수의 파일명 검증 부재로 인해 공격자가 서버의 임의 위치에 파일을 작성할 수 있으며, RCE(원격 코드 실행)로 이어질 수 있습니다.
- 버전 10.1.2 및 11.0.0-next.6으로 즉시 업데이트가 권장되며, jsPDF 라이브러리에서도 유사한 취약점이 발견되었습니다.

## 📌 주요 내용

### CVE-2026-21440: AdonisJS Bodyparser의 치명적인 취약점

"@adonisjs/bodyparser" npm 패키지 사용자들은 심각한 보안 취약점이 공개됨에 따라 최신 버전으로 즉시 업데이트해야 합니다. **CVE-2026-21440**로 추적되는 이 취약점은 CVSS 9.2 점수를 기록하며, 원격 공격자가 서버에 임의의 파일을 작성할 수 있는 경로 순회(Path Traversal) 문제입니다.

AdonisJS는 TypeScript로 웹 애플리케이션과 API 서버를 개발하기 위한 Node.js 프레임워크이며, @adonisjs/bodyparser 라이브러리는 AdonisJS HTTP 요청 본문을 처리하는 데 사용됩니다.

### 취약점 발생 원리와 공격 시나리오

AdonisJS 프로젝트 관리자들은 공식 보안 권고에서 다음과 같이 설명했습니다: "개발자가 두 번째 옵션 인자 없이 MultipartFile.move()를 사용하거나 파일명을 명시적으로 검증하지 않는 경우, 공격자는 순회 시퀀스를 포함한 조작된 파일명 값을 제공하여 의도된 업로드 디렉토리 외부의 대상 경로에 파일을 작성할 수 있습니다."

핵심 문제는 **MultipartFile.move(location, options)** 함수에 있습니다. 이 함수는 파일을 지정된 위치로 이동시키는 역할을 하며, options 매개변수에는 파일명과 덮어쓰기 플래그가 포함됩니다.

```javascript
// 취약한 코드 예시
const file = request.file('document')
await file.move(Helpers.tmpPath('uploads'))
// 파일명 검증 없이 기본 클라이언트 파일명 사용
```

취약점은 name 매개변수가 입력으로 전달되지 않을 때 발생합니다. 이 경우 애플리케이션은 검증되지 않은 클라이언트 파일명을 기본값으로 사용하게 되어 경로 순회 공격에 노출됩니다.

### 원격 코드 실행(RCE) 가능성

AdonisJS는 "공격자가 애플리케이션 코드, 시작 스크립트 또는 나중에 실행/로드되는 구성 파일을 덮어쓸 수 있다면 RCE가 가능합니다"라고 경고했습니다. 다만 RCE는 파일 시스템 권한, 배포 레이아웃, 애플리케이션/런타임 동작에 따라 달라지며 항상 보장되는 것은 아닙니다.

성공적인 공격을 위해서는 접근 가능한 업로드 엔드포인트가 필요합니다. 공격자는 다음과 같은 조작된 파일명을 사용할 수 있습니다:

```
../../../../../../etc/cron.d/malicious
../../../app/start/routes.js
```

### 영향받는 버전과 패치

다음 버전이 취약점의 영향을 받습니다:

- **10.1.1 이하 버전** (10.1.2에서 수정됨)
- **11.0.0-next.5 이하 버전** (11.0.0-next.6에서 수정됨)

이 취약점은 보안 연구원 Hunter Wodzenski(@wodzen)에 의해 발견되고 보고되었습니다.

### jsPDF 라이브러리의 유사한 취약점

비슷한 시기에 jsPDF라는 npm 패키지에서도 경로 순회 취약점(**CVE-2025-68428**, CVSS 9.2)이 공개되었습니다. 이 취약점은 검증되지 않은 경로를 전달하여 Node.js 프로세스가 실행 중인 로컬 파일 시스템의 임의 파일 내용을 검색할 수 있게 합니다.

```javascript
// jsPDF 취약점 예시
const doc = new jsPDF();
doc.addImage('../../../etc/passwd', 'PNG', 10, 10);
// 임의 파일 내용이 PDF에 포함됨
```

jsPDF 개발자인 Parallax는 "파일 내용이 생성된 PDF에 그대로 포함됩니다. 라이브러리의 Node.js 빌드만 영향을 받으며, 구체적으로 dist/jspdf.node.js 및 dist/jspdf.node.min.js 파일입니다"라고 설명했습니다.

이 취약점은 2026년 1월 3일 릴리스된 jsPDF 버전 4.0.0에서 패치되었습니다. 임시 해결책으로는 Node.js의 --permission 플래그를 사용하여 파일 시스템 접근을 제한할 수 있습니다.

### 보안 강화 권장사항

개발자들은 다음과 같은 조치를 취해야 합니다:

```javascript
// 안전한 코드 예시
const file = request.file('document')
await file.move(Helpers.tmpPath('uploads'), {
  name: `${new Date().getTime()}.${file.subtype}`,
  overwrite: false
})
// 명시적인 파일명 지정과 검증
```

또한 업로드되는 파일명에 대한 화이트리스트 검증과 경로 순회 문자열(../, ..\) 필터링을 구현해야 합니다.

## 👨‍💻 개발자에게 미치는 영향

### 즉각적인 보안 업데이트 필요성

AdonisJS를 사용하는 모든 개발자는 즉시 영향받는 패키지를 최신 버전으로 업데이트해야 합니다. 특히 파일 업로드 기능을 구현한 애플리케이션은 높은 위험에 노출되어 있습니다. CVSS 9.2라는 매우 높은 점수는 이 취약점의 심각성을 나타내며, 즉각적인 조치가 필요함을 의미합니다.

### 보안 코드 리뷰 필요성

기존 코드베이스에서 MultipartFile.move() 함수 사용을 검토하고, 파일명 검증 로직이 적절히 구현되어 있는지 확인해야 합니다. 외부 입력값을 신뢰하지 않고, 항상 서버 측에서 파일명을 생성하거나 엄격하게 검증하는 것이 중요합니다.

### npm 패키지 의존성 관리의 중요성

이번 사례는 오픈소스 패키지 의존성 관리의 중요성을 다시 한번 강조합니다. 정기적인 의존성 감사와 보안 업데이트 모니터링이 필수적입니다. npm audit 명령어나 Snyk, Dependabot과 같은 도구를 활용하여 취약점을 조기에 발견하고 대응해야 합니다.

> <a href="https://thehackernews.com/2026/01/critical-adonisjs-bodyparser-flaw-cvss.html" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>