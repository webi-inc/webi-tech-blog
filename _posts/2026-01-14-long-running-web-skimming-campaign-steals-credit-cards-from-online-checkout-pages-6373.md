---
title: "Magecart 웹 스키밍 공격, 4년간 주요 결제 네트워크 신용카드 정보 탈취"
description: "2022년부터 현재까지 지속된 Magecart 웹 스키밍 캠페인이 American Express, Mastercard 등 주요 결제 네트워크를 대상으로 신용카드 정보를 탈취하고 있습니다. WordPress 관리자 감지 회피 및 가짜 Stripe 결제 폼을 이용한 정교한 공격 기법을 분석합니다."
date: 2026-01-15 05:00:40 +0900
categories: ['Security', 'Infosec']
tags: ['Magecart', '웹스키밍', 'WordPress', 'Stripe', '결제보안', 'JavaScript', '사이버보안']
image:
  path: "/assets/img/2026-01-14-long-running-web-skimming-campaign-steals-credit-cards-from-online-checkout-pages-6373.png"
  alt: "온라인 결제 페이지를 겨냥한 웹 스키밍 공격 개념도"
published: true
---

## 3줄 요약
- 2022년 1월부터 활동 중인 Magecart 웹 스키밍 캠페인이 American Express, Mastercard 등 주요 결제 네트워크를 표적으로 신용카드 정보를 탈취하고 있습니다.
- 공격자는 WordPress 관리자 감지 회피 기능과 가짜 Stripe 결제 폼을 통해 사용자의 신용카드 번호, CVC, 개인정보를 탈취합니다.
- 악성 스크립트는 자체 삭제 기능을 갖추고 있어 탐지를 회피하며, 탈취된 데이터는 외부 서버로 전송됩니다.

## 📌 주요 내용

### Magecart 웹 스키밍 캠페인의 전모

사이버보안 연구진이 2022년 1월부터 지속되고 있는 대규모 Magecart 웹 스키밍 캠페인을 발견했습니다. 이 공격은 American Express, Diners Club, Discover, JCB Co., Ltd., Mastercard, UnionPay 등 주요 결제 네트워크를 대상으로 하고 있습니다.

Silent Push의 보고서에 따르면, "이러한 결제 서비스를 이용하는 엔터프라이즈 조직이 가장 큰 영향을 받을 가능성이 높습니다." 디지털 스키밍 공격은 공격자가 정상적인 전자상거래 사이트와 결제 포털을 침해하여, 사용자가 결제 페이지에서 결제를 시도할 때 신용카드 정보와 개인정보를 몰래 탈취할 수 있는 악성 JavaScript 코드를 주입하는 클라이언트 측 공격입니다.

### 제재 회피 호스팅 인프라 활용

연구진은 현재 제재를 받은 불릿프루프 호스팅 제공업체인 Stark Industries(모회사 PQ.Hosting)와 연결된 의심스러운 도메인을 분석하는 과정에서 이 캠페인을 발견했습니다. 해당 업체는 제재 회피를 위해 네덜란드 법인 WorkTitans B.V. 소유의 THE[.]Hosting으로 리브랜딩했습니다.

문제의 도메인인 cdn-cookie[.]com은 고도로 난독화된 JavaScript 페이로드(예: "recorder.js" 또는 "tab-gtm.js")를 호스팅하며, 이는 웹 쇼핑몰에 로드되어 신용카드 스키밍을 수행합니다.

### WordPress 관리자 감지 및 회피 메커니즘

스키머는 사이트 관리자의 탐지를 회피하기 위한 정교한 기능을 갖추고 있습니다. 특히 DOM(Document Object Model) 트리에서 "wpadminbar"라는 요소를 확인하는데, 이는 WordPress 웹사이트에서 로그인한 관리자나 적절한 권한을 가진 사용자가 사이트를 볼 때 나타나는 툴바를 의미합니다.

"wpadminbar" 요소가 존재하는 경우, 스키머는 자체 삭제 시퀀스를 시작하여 웹 페이지에서 자신의 존재를 제거합니다. 사용자가 페이지와 상호작용할 때 발생하는 표준 동작인 웹 페이지의 DOM이 수정될 때마다 스키머 실행이 시도됩니다.

### 가짜 Stripe 결제 폼을 통한 정보 탈취

스키머는 결제 옵션으로 Stripe가 선택되었는지 확인하고, 브라우저의 localStorage에 "wc_cart_hash"라는 요소가 존재하는지 검사합니다. 이 플래그는 피해자가 이미 성공적으로 스키밍되었음을 나타내기 위해 스키머가 생성하고 "true"로 설정합니다.

이 플래그가 없으면 스키머는 사용자 인터페이스 조작을 통해 정상 결제 폼을 대체하는 가짜 Stripe 결제 폼을 렌더링합니다. 이를 통해 피해자는 신용카드 번호, 유효기간, CVC 번호를 입력하도록 속입니다.

Silent Push는 "피해자가 실제 Stripe 결제 폼 대신 가짜 폼에 신용카드 정보를 입력했기 때문에, 처음 입력할 때 스키머가 숨긴 실제 결제 폼이 오류를 표시하게 됩니다"라고 설명했습니다. "이는 피해자가 단순히 결제 정보를 잘못 입력한 것처럼 보이게 만듭니다."

### 광범위한 데이터 수집 및 전송

스키머가 탈취하는 데이터는 결제 정보뿐만 아니라 이름, 전화번호, 이메일 주소, 배송 주소까지 확장됩니다. 수집된 정보는 최종적으로 "lasorie[.]com" 서버로 HTTP POST 요청을 통해 유출됩니다.

데이터 전송이 완료되면 스키머는 결제 페이지에서 자신의 흔적을 지우고, 생성했던 가짜 결제 폼을 제거하며 정상 Stripe 입력 폼을 복원합니다. 그런 다음 "wc_cart_hash"를 "true"로 설정하여 동일한 피해자에게 스키머가 두 번 실행되는 것을 방지합니다.

### 고도화된 WordPress 지식 활용

Silent Push는 "이 공격자는 WordPress의 내부 작동에 대한 고급 지식을 보유하고 있으며, 잘 알려지지 않은 기능까지도 공격 체인에 통합하고 있습니다"라고 평가했습니다.

## 👨‍💻 개발자에게 미치는 영향

### 전자상거래 플랫폼 보안 강화 필요성

전자상거래 플랫폼을 운영하는 개발자는 클라이언트 측 보안에 특별한 주의를 기울여야 합니다. 특히 타사 JavaScript 라이브러리와 결제 통합 시 다음 사항을 고려해야 합니다:

- **CSP(Content Security Policy) 구현**: 승인되지 않은 외부 스크립트 로딩을 차단
- **SRI(Subresource Integrity) 적용**: 로드되는 외부 리소스의 무결성 검증
- **정기적인 보안 감사**: 주입된 악성 스크립트 탐지를 위한 정기적인 코드 리뷰

### WordPress 기반 쇼핑몰 관리자 대응 방안

WordPress를 사용하는 개발자는 다음과 같은 보안 조치를 취해야 합니다:

- DOM 요소를 통한 관리자 감지 회피 기법을 인지하고, 추가적인 보안 레이어 구현
- localStorage 사용 패턴을 모니터링하여 비정상적인 플래그 생성 탐지
- 결제 폼의 렌더링 과정을 모니터링하여 UI 조작 시도 감지

### 결제 통합 보안 베스트 프랙티스

Stripe와 같은 결제 서비스를 통합할 때는 다음을 권장합니다:

- 공식 SDK와 라이브러리만 사용
- 결제 프로세스의 각 단계에서 데이터 흐름 검증
- 클라이언트 측과 서버 측 양쪽에서 결제 데이터 검증 수행
- 민감한 결제 정보가 브라우저의 localStorage나 sessionStorage에 저장되지 않도록 보장

> <a href="https://thehackernews.com/2026/01/long-running-web-skimming-campaign.html" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>