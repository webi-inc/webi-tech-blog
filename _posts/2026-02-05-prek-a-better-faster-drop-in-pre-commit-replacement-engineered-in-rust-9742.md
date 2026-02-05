---
title: "Rust로 재설계된 pre-commit 대체 도구 Prek: 더 빠르고 효율적인 Git 훅 관리"
description: "Prek은 Rust로 재설계된 pre-commit의 drop-in 대체 도구입니다. Python 런타임 없이 단일 바이너리로 동작하며, 기존 대비 수배 빠른 성능과 절반의 디스크 공간 사용량을 자랑합니다. 워크스페이스 지원, uv 통합 등 다양한 기능을 제공합니다."
date: 2026-02-05 17:00:16 +0900
categories: ['Development', 'DevOps']
tags: [prek, pre-commit, rust, git-hooks, devops, ci-cd, code-quality, tooling]
image:
  path: "/assets/img/2026-02-05-prek-a-better-faster-drop-in-pre-commit-replacement-engineered-in-rust-9742.jpg"
  alt: "Prek 로고 - Rust로 재설계된 빠른 pre-commit 대체 도구"
published: true
---

## 3줄 요약
- Prek은 Rust로 재설계된 pre-commit 대체 도구로, 단일 바이너리로 동작하며 Python 런타임이 필요 없습니다
- 기존 pre-commit 대비 수배 빠른 성능과 절반의 디스크 공간 사용량으로 효율성이 크게 향상되었습니다
- 워크스페이스(모노레포) 지원, uv 통합, 병렬 실행, 내장 Rust 구현 훅 등 다양한 개선 사항을 제공합니다

## 📌 주요 내용

### Prek이란 무엇인가

Prek은 여러 언어로 작성된 훅을 실행하는 프레임워크인 pre-commit의 재설계 버전입니다. Rust로 작성되어 더 빠르고, 의존성이 없으며, drop-in 대체가 가능하도록 설계되었습니다. 기존 pre-commit과 완전히 호환되면서도 오랫동안 요청되어 온 추가 기능들을 제공합니다.

Prek은 이미 CPython, Apache Airflow, FastAPI 등 실제 프로젝트에서 사용되고 있으며, 점점 더 많은 프로젝트가 채택하고 있습니다. pre-commit의 대안을 찾고 있다면 Prek을 시도해보고 피드백을 공유해 주시기 바랍니다.

### 주요 기능

#### 성능 및 효율성

- **단일 바이너리**: Python이나 다른 런타임이 필요 없는 독립 실행형 바이너리입니다
- **빠른 속도**: pre-commit보다 수배 빠르며 디스크 공간 사용량은 절반입니다
- **완전한 호환성**: 기존 pre-commit 설정 파일과 훅과 완벽하게 호환됩니다
- **병렬 처리**: 우선순위가 같은 훅들은 동시에 실행되어 전체 실행 시간을 단축합니다

#### 고급 기능

- **모노레포 지원**: 워크스페이스 모드를 내장 지원하여 각 하위 프로젝트가 자체 설정 파일을 가질 수 있습니다
- **uv 통합**: Python 가상 환경 및 의존성 관리에 uv를 사용합니다
- **개선된 툴체인 관리**: Python, Node.js, Bun, Go, Rust, Ruby의 툴체인 설치가 개선되어 훅 간 공유됩니다
- **내장 훅**: 일반적인 훅들의 Rust 네이티브 구현을 내장하여 더 빠른 실행이 가능합니다

### 설치 방법

Prek은 다양한 방법으로 설치할 수 있습니다:

#### 독립형 설치 프로그램

Linux 및 macOS:
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/j178/prek/releases/download/v0.3.1/prek-installer.sh | sh
```

Windows:
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://github.com/j178/prek/releases/download/v0.3.1/prek-installer.ps1 | iex"
```

#### PyPI

```bash
# uv 사용 (권장)
uv tool install prek

# uvx 사용 (설치와 실행을 한 번에)
uvx prek

# pip 사용
pip install prek

# pipx 사용
pipx install prek
```

#### 패키지 매니저

```bash
# Homebrew
brew install prek

# mise (v2025.8.11 이상)
mise use prek

# Cargo
cargo install --locked prek

# npm
npm add -D @j178/prek

# Conda
conda install conda-forge::prek
```

### 빠른 시작 가이드

#### 기존 pre-commit 사용자

기존에 pre-commit을 사용 중이라면 짧은 마이그레이션 체크리스트를 따라 안전하게 prek으로 전환할 수 있습니다.

#### 새로운 사용자

pre-commit 스타일 도구를 처음 사용한다면 설정 파일 생성, 훅 실행, git 훅 설치 등의 기본 사항을 초보자용 빠른 시작 가이드에서 배울 수 있습니다.

### Prek을 선택해야 하는 이유

#### 성능 개선

- pre-commit보다 수배 빠르며 디스크 공간은 절반만 사용합니다
- 훅 환경과 툴체인 관리 방식을 재설계하여 모든 것을 훅 간에 공유합니다
- 저장소는 병렬로 복제되고 훅은 의존성이 분리되어 있으면 병렬로 설치됩니다
- Python 가상 환경 생성과 의존성 설치에 속도와 효율성으로 유명한 uv를 사용합니다
- 일부 일반적인 훅을 Rust로 구현하여 Python 버전보다 빠릅니다

#### 개선된 사용자 경험

- Python이나 다른 런타임을 설치할 필요 없이 단일 바이너리만 다운로드하면 됩니다
- Python 버전이나 가상 환경에 대한 걱정 없이 prek이 자동으로 필요한 Python 버전을 설치합니다
- 워크스페이스(모노레포)에 대한 내장 지원으로 각 하위 프로젝트가 자체 설정 파일을 가질 수 있습니다
- `prek run` 명령이 다양한 개선 사항을 제공합니다:
  - `prek run --directory <dir>`: 지정된 디렉토리의 파일에 대해 훅 실행
  - `prek run --last-commit`: 마지막 커밋에서 변경된 파일에 대해 훅 실행
  - `prek run [HOOK] [HOOK]`: 여러 훅 선택 및 실행
- `prek list` 명령으로 사용 가능한 모든 훅과 ID, 설명 확인 가능
- 셸 자동 완성 기능 제공

### 실제 사용 사례

Prek은 비교적 새로운 도구이지만 이미 다음과 같은 프로젝트와 조직에서 사용되거나 권장되고 있습니다:

- apache/airflow
- python/cpython
- fastapi/fastapi
- astral-sh/ruff
- home-assistant/core
- django/djangoproject.com
- python-attrs/attrs
- simple-icons/simple-icons
- commitizen-tools/commitizen
- 그 외 다수

## 👨‍💻 개발자에게 미치는 영향

### 개발 워크플로우 개선

Prek은 pre-commit의 모든 기능을 제공하면서도 성능과 사용자 경험을 크게 개선했습니다. 기존 pre-commit 설정 파일을 그대로 사용할 수 있어 마이그레이션 비용이 낮으며, 더 빠른 실행 속도로 개발자의 생산성을 향상시킵니다.

### CI/CD 파이프라인 최적화

GitHub Actions에서 prek을 사용하려면 j178/prek-action을 활용할 수 있습니다. 병렬 실행과 개선된 성능 덕분에 CI/CD 파이프라인의 실행 시간을 단축할 수 있습니다.

### 모노레포 환경 지원

워크스페이스 모드를 통해 모노레포 환경에서 각 프로젝트별로 독립적인 pre-commit 설정을 관리할 수 있어, 복잡한 프로젝트 구조에서도 효율적인 코드 품질 관리가 가능합니다.

> <a href="https://github.com/j178/prek" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>