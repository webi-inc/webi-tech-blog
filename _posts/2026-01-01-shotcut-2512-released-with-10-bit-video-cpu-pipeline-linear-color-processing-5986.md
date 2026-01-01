---
title: "Shotcut 25.12 출시: 10비트 비디오 CPU 파이프라인과 선형 색상 처리 지원"
description: "오픈소스 비디오 편집기 Shotcut 25.12가 10비트 비디오 CPU 파이프라인, 선형 색상 처리, NVIDIA NVENC 가속 화면 녹화 등 주요 기능을 추가하며 출시되었습니다."
date: 2026-01-01 11:26:08 +0900
categories: ['Software', 'Multimedia']
tags: ['Shotcut', '비디오편집', '오픈소스', '10비트비디오', 'MLT', 'FFmpeg', 'NVENC']
image:
  path: "/assets/img/2026-01-01-shotcut-2512-released-with-10-bit-video-cpu-pipeline-linear-color-processing-5986.jpg"
  alt: "Shotcut 비디오 편집기 인터페이스"
published: true
---

## 3줄 요약
- Shotcut 25.12가 10비트/12비트 비디오 소스를 지원하는 CPU 파이프라인과 선형 색상 처리 기능을 추가하며 출시되었습니다.
- NVIDIA NVENC 가속 화면 녹화가 Linux X.Org 세션에서 지원되며, 두 개의 새로운 HTML 프리셋이 추가되었습니다.
- Flowblade 2.24, OpenShot 3.4, Kdenlive 25.12에 이어 12월 오픈소스 비디오 편집기 릴리스 열풍을 이어갑니다.

## 📌 주요 내용

### Shotcut 25.12의 핵심 업데이트

오픈소스 비디오 편집 소프트웨어 **Shotcut 25.12**가 2025년 12월 30일 공식 출시되었습니다. 이번 버전은 MLT Multimedia Framework와 FFmpeg 기반으로 구축된 크로스 플랫폼 비디오 편집기로서, 10비트 비디오 처리 능력을 대폭 강화했습니다.

### 10비트 비디오 CPU 파이프라인 지원

Shotcut은 이전 버전에서 GPU 효과를 위한 부분적인 10비트 색상 지원을 제공했지만, CPU 기반 필터는 제한적이었습니다. Shotcut 25.12 버전에서는 많은 CPU 비디오 효과가 10비트 및 12비트 비디오 소스를 지원하도록 개선되었습니다. 일부 CPU 기반 필터는 여전히 변환 작업이 필요하지만, 전반적인 10비트 비디오 CPU 파이프라인이 훨씬 향상된 상태입니다.

### 선형 색상 처리로 향상된 이미지 품질

이번 업데이트의 또 다른 하이라이트는 선형 색상 처리(Linear Color Processing) 지원입니다. CPU 경로에서 선형 10비트 처리를 지원하며, 실험적으로 GPU/CPU 혼합 환경에서도 선형 10비트 색상 처리 모드를 제공합니다. 이는 색상 정확도와 전반적인 이미지 품질 향상으로 이어집니다.

### NVIDIA NVENC 화면 녹화 지원

Linux 사용자를 위한 개선 사항으로, X.Org 데스크톱 세션에서 NVIDIA NVENC 가속 화면 녹화 기능이 추가되었습니다. 이를 통해 하드웨어 가속을 활용한 효율적인 화면 캡처가 가능해졌습니다.

### 추가 기능 및 버그 수정

- 두 개의 새로운 HTML 프리셋 추가
- 다양한 버그 수정 및 안정성 개선
- 사용자 경험 향상을 위한 인터페이스 개선

## 👨‍💻 개발자에게 미치는 영향

### 비디오 처리 워크플로우 개선

Shotcut 25.12의 10비트 CPU 파이프라인 지원은 고품질 비디오 콘텐츠를 제작하는 개발자와 크리에이터에게 중요한 발전입니다. GPU가 제한적이거나 사용할 수 없는 환경에서도 높은 색심도의 비디오를 처리할 수 있게 되어 작업 환경의 유연성이 크게 향상되었습니다.

### 오픈소스 비디오 편집 생태계 강화

12월 한 달 동안 Flowblade 2.24, OpenShot 3.4, Kdenlive 25.12, Shotcut 25.12 등 주요 오픈소스 비디오 편집기들이 연이어 업데이트를 발표했습니다. 이는 오픈소스 멀티미디어 소프트웨어 생태계가 활발히 발전하고 있음을 보여주는 긍정적인 신호입니다.

### MLT와 FFmpeg 기반 개발

Shotcut은 MLT Multimedia Framework와 FFmpeg을 기반으로 구축되어 있어, 이러한 라이브러리를 활용하는 개발자들에게 실용적인 참고 사례가 됩니다. 특히 10비트 색상 처리와 하드웨어 가속 통합 방법론은 유사한 프로젝트에 적용 가능한 인사이트를 제공합니다.

> <a href="https://www.phoronix.com/news/Shotcut-25.12-Released" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>