---
title: "AV Chaos Monkey: WebRTC 부하 테스트를 위한 분산 카오스 엔지니어링 플랫폼"
description: "WebRTC와 UDP 환경에서 1500+ 동시 참가자를 시뮬레이션하는 AV Chaos Monkey. H.264/Opus 스트림 생성과 네트워크 카오스 주입으로 영상 회의 시스템의 복원력을 검증하는 오픈소스 도구를 소개합니다."
date: 2026-02-25 05:00:10 +0900
categories: ['Infrastructure', 'Cloud']
tags: ['WebRTC', 'Chaos Engineering', 'Load Testing', 'Kubernetes', 'Video Conferencing', 'UDP', 'RTP', 'H.264', 'Opus', 'AV Chaos Monkey']
image:
  path: "/assets/img/2026-02-24-show-hn-chaos-monkey-but-for-audio-video-testing-webrtc-and-udp-2367.jpg"
  alt: "AV Chaos Monkey 아키텍처 다이어그램"
published: true
---

## 3줄 요약
- AV Chaos Monkey는 WebRTC 및 UDP 기반 영상 회의 시스템의 부하 테스트를 위한 분산 카오스 엔지니어링 플랫폼입니다.
- H.264/Opus 스트림을 사용해 1500명 이상의 가상 참가자를 시뮬레이션하고, 패킷 손실·지터·비트레이트 감소 등의 네트워크 카오스를 주입합니다.
- Kubernetes 기반 수평 확장을 지원하며, Prometheus/Grafana 모니터링 스택과 통합되어 실시간 성능 메트릭을 제공합니다.

## 📌 주요 내용

### AV Chaos Monkey란 무엇인가

AV Chaos Monkey는 영상 회의 시스템의 복원력을 검증하기 위한 오픈소스 카오스 엔지니어링 도구입니다. Netflix의 Chaos Monkey에서 영감을 받아 WebRTC와 UDP 환경에 특화되어 개발되었으며, 실제 미디어 스트림(H.264 비디오, Opus 오디오)을 생성하여 대규모 참가자를 시뮬레이션합니다.

이 도구는 1500명 이상의 동시 참가자를 지원하며, 네트워크 장애 상황(패킷 손실, 지터, 대역폭 제한)을 의도적으로 주입하여 시스템이 열악한 조건에서도 정상 작동하는지 검증할 수 있습니다.

### 핵심 아키텍처 구성 요소

#### 미디어 처리 파이프라인

AV Chaos Monkey의 미디어 처리 파이프라인은 FFmpeg를 활용해 입력 비디오를 H.264 Annex-B와 Ogg/Opus 포맷으로 변환합니다. NAL Reader는 H.264 스트림을 파싱하여 SPS/PPS/IDR/Slices를 추출하고, Opus Reader는 Ogg 컨테이너에서 20ms 오디오 프레임을 추출합니다.

변환된 프레임은 메모리에 캐싱되어 모든 참가자가 공유하며, 제로 카피(zero-copy) 방식으로 참가자별 인코딩 대비 CPU 사용량을 약 90% 절감합니다.

#### 제어 평면과 참가자 풀

HTTP 서버(:8080)는 REST API를 통해 테스트 생명주기를 관리하며, Spike Scheduler는 카오스 이벤트를 even/random/front/back/legacy 전략에 따라 분산시킵니다. Network Degrader는 패킷 손실(1-25%), 지터(10-50ms), 비트레이트 감소(30-80%), 프레임 드롭(10-60%)을 적용합니다.

참가자 풀은 `participant_id % total_partitions = partition_id` 공식으로 파드 간에 자동 분할됩니다. 각 참가자는 RTP 스트림(PT=96 비디오, PT=111 오디오)을 생성하며, 참가자 ID는 RTP 확장 헤더(ID=1)에 포함됩니다.

#### Kubernetes 자동 구성과 UDP 릴레이 체인

Kubernetes 환경에서는 파드가 이름에서 파티션 ID를 자동 감지합니다(예: `orchestrator-3` → `PARTITION_ID=3`). 포트 할당은 `base_port + (partition_id × 10000) + participant_index` 방식으로 이루어지며, 파티션 0은 5000-14999, 파티션 1은 15000-24999 포트를 사용합니다.

kubectl port-forward는 TCP만 지원하므로, UDP 스트림을 전달하기 위한 릴레이 체인이 구축됩니다:

```
Orchestrator Pods (10×) → UDP :5000 → udp-relay Pod (Python)
→ Length-Prefixed TCP :5001 → kubectl port-forward 15001:5001
→ tools/udp-relay (Go) → UDP :5002 → 수신자
```

이 아키텍처는 1500개 참가자 스트림을 단일 연결로 집계하여 효율적으로 전달합니다.

### 카오스 주입 메커니즘

#### 5가지 스파이크 타입

AV Chaos Monkey는 실제 네트워크 상황을 시뮬레이션하는 5가지 카오스 타입을 제공합니다:

- **RTP 패킷 손실**: 애플리케이션 레이어에서 RTP 패킷을 1-100% 비율로 드롭합니다.
- **네트워크 지터**: 기본 지연시간에 가우시안 지터를 추가합니다.
- **비트레이트 감소**: 비디오 인코딩을 30-80% 제한합니다.
- **프레임 드롭**: 비디오 프레임을 10-60% 비율로 건너뜁니다.
- **대역폭 제한**: 전체 처리량에 상한선을 설정합니다.

#### 분산 전략

스파이크는 테스트 기간 동안 구성 가능한 전략에 따라 분산됩니다:

- **Even**: 지터를 포함한 균등 간격 (예측 가능한 부하)
- **Random**: 예측 불가능한 타이밍 (현실적인 카오스)
- **Front-loaded**: 초반에 집중된 스파이크 (복구 테스트)
- **Back-loaded**: 베이스라인 후 카오스 (비교 테스트)
- **Legacy**: 고정 간격 티커 (런타임 주입)

### 배포 및 실행 방법

#### 로컬 개발 환경 (1-100 참가자)

로컬 개발 환경에서는 단일 오케스트레이터 프로세스로 소규모 테스트를 수행할 수 있습니다:

```bash
# 오케스트레이터 시작
go run cmd/main.go

# UDP 수신자 시작
go run examples/go/udp_receiver.go 5002

# config/config.json에서 num_participants: 10 설정
# 카오스 테스트 실행
go run tools/chaos-test/main.go -config config/config.json
```

#### Docker Compose 환경 (100-500 참가자)

격리된 테스트와 CI/CD 파이프라인에 적합한 Docker Compose 환경:

```bash
# 컨테이너 빌드 및 시작
./scripts/start_everything.sh build

# 수신자 시작
go run examples/go/udp_receiver.go 5002

# 카오스 테스트 실행
go run tools/chaos-test/main.go -config config/config.json
```

Docker 메모리 할당에 따른 최대 참가자 수:
- 8GB: ~100명 (4 코어)
- 16GB: ~250명 (8 코어)
- 24GB: ~400명 (12 코어)
- 32GB: ~500명 (14 코어)

#### Kubernetes 프로덕션 환경 (500-1500 참가자)

Nix 개발 환경을 활용한 대규모 테스트:

```bash
# Nix 환경 진입 (Go, Docker, kubectl, kind, ffmpeg 제공)
nix develop

# 자동 배포 (시스템 리소스 자동 감지)
./scripts/start_everything.sh run -config config/config.json

# UDP 수신자 시작
go run ./examples/go/udp_receiver.go 5002
```

StatefulSet으로 10개 오케스트레이터 파드가 배포되며, 각 파드는 약 150명의 참가자를 처리합니다. 전체 1500명의 참가자 스트림은 UDP 릴레이를 통해 집계되어 단일 수신자로 전달됩니다.

### WebRTC 인프라 구성

#### Coturn TURN 서버

Kubernetes 환경에서는 Coturn StatefulSet이 3개 초기 레플리카로 배포되며, HPA(Horizontal Pod Autoscaler)가 부하에 따라 1-10개로 자동 스케일링합니다. 각 레플리카는 약 500명의 참가자를 처리할 수 있습니다.

`coturn-lb` 서비스는 레플리카 간 TURN 트래픽을 로드밸런싱하며, 포트 3478(TURN)과 49152-65535(릴레이 범위)를 사용합니다. 기본 인증 정보는 `webrtc/webrtc123`입니다.

#### WebRTC 수신자 사용법

WebRTC 연결은 1:1 방식이므로, 최대 150명의 참가자와 동시 연결할 수 있습니다:

```bash
# 단일 참가자 연결
go run ./examples/go/webrtc_receiver.go http://localhost:8080 <test_id>

# 다중 참가자 연결 (최대 150명)
go run ./examples/go/webrtc_receiver.go http://localhost:8080 chaos_test_1770831684 150
```

Kubernetes 환경에서 대규모 테스트 시에는 UDP 수신자를 사용하는 것이 권장됩니다.

### 성능 메트릭과 모니터링

#### 리소스 요구사항

참가자 수에 따른 시스템 리소스 요구사항:

| 참가자 수 | 메모리 | CPU | 대역폭 |
|---------|--------|-----|--------|
| 100명 | 2GB | 2 코어 | 250 Mbps |
| 500명 | 6GB | 8 코어 | 1.2 Gbps |
| 1000명 | 12GB | 16 코어 | 2.5 Gbps |
| 1500명 | 18GB | 24 코어 | 3.7 Gbps |

참가자당 처리량(1280x720@30fps + Opus):
- 비디오: ~2.5 Mbps (H.264)
- 오디오: ~128 Kbps (Opus)
- 총합: ~2.6 Mbps
- 패킷: ~90 비디오 + 50 오디오 = 140 pkt/s

#### Prometheus와 Grafana 통합

Kubernetes 모드에서는 Prometheus가 모든 오케스트레이터 파드의 `/metrics` 엔드포인트를 5초마다 자동으로 스크랩합니다:

```bash
# 모니터링 스택 배포
kubectl apply -f k8s/monitoring/prometheus-rbac.yaml
kubectl apply -f k8s/monitoring/prometheus.yaml
kubectl apply -f k8s/monitoring/grafana.yaml

# Grafana 접속: http://localhost:30030 (admin/admin)
# Prometheus 접속: http://localhost:30090
```

노출되는 주요 메트릭:
- `av_chaos_monkey_participants_total`: 총 참가자 수
- `av_chaos_monkey_packets_sent_total`: 전송된 패킷 수
- `av_chaos_monkey_bytes_sent_total`: 전송된 바이트 수
- `av_chaos_monkey_spikes_active`: 활성 스파이크 수
- `av_chaos_monkey_packet_loss_percent`: 패킷 손실률
- `av_chaos_monkey_jitter_ms`: 지터 시간

### API 레퍼런스

#### 테스트 생명주기 관리

```bash
# 테스트 생성
POST /api/v1/test/create
{
  "test_id": "optional_id",
  "num_participants": 100,
  "duration_seconds": 600,
  "spike_distribution": {
    "strategy": "even",
    "min_spacing_seconds": 5,
    "jitter_percent": 15
  }
}

# 테스트 시작
POST /api/v1/test/{test_id}/start

# 메트릭 조회
GET /api/v1/test/{test_id}/metrics

# 테스트 중지
POST /api/v1/test/{test_id}/stop
```

#### 카오스 스파이크 주입

```bash
POST /api/v1/test/{test_id}/spike
{
  "spike_id": "unique_id",
  "type": "rtp_packet_loss",
  "duration_seconds": 30,
  "participant_ids": [1001, 1002],
  "params": {"loss_percentage": "15"}
}
```

### RTP 패킷 포맷과 클라이언트 통합

AV Chaos Monkey가 생성하는 RTP 패킷은 RFC 3550 표준을 준수하며, 다음과 같은 구조를 가집니다:

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       sequence number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           synchronization source (SSRC) identifier            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Extension ID=1 | Length=4    |    Participant ID (uint32)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         H.264/Opus Payload                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

페이로드 타입:
- `96`: H.264 비디오 (RFC 6184)
- `111`: Opus 오디오 (RFC 7587)

참가자 ID는 RTP 확장 헤더에 리틀 엔디안 uint32 형식으로 인코딩되어 있으며, Go 언어로 다음과 같이 추출할 수 있습니다:

```go
// 확장 비트 체크
if (packet[0] & 0x10) != 0 {
    offset := 12 + int(packet[0]&0x0F)*4  // CSRC 건너뛰기
    extID := binary.BigEndian.Uint16(packet[offset:])
    if extID == 1 {
        participantID := binary.LittleEndian.Uint32(packet[offset+4:])
    }
}
```

## 👨‍💻 개발자에게 미치는 영향

### 실전 카오스 엔지니어링의 필요성

WebRTC 기반 영상 회의 시스템을 개발할 때, 프로덕션 환경에서 발생할 수 있는 네트워크 장애 상황을 사전에 테스트하는 것은 매우 중요합니다. 패킷 손실, 지터, 대역폭 제한 등의 문제는 사용자 경험에 직접적인 영향을 미치지만, 개발 환경에서 이를 재현하기는 쉽지 않습니다.

AV Chaos Monkey는 이러한 문제를 해결하기 위한 실용적인 도구를 제공합니다. 1500명 이상의 동시 참가자를 시뮬레이션하고, 다양한 네트워크 카오스를 주입함으로써 시스템의 한계점을 미리 파악하고 개선할 수 있습니다.

### Kubernetes 네이티브 설계의 장점

이 도구는 Kubernetes StatefulSet과 HPA를 활용한 수평 확장을 지원하여, 클라우드 네이티브 환경에서 대규모 부하 테스트를 효율적으로 수행할 수 있습니다. 파티셔닝 메커니즘을 통해 참가자가 자동으로 파드 간에 분산되며, 각 파드는 독립적으로 150명의 참가자를 처리할 수 있습니다.

또한 Prometheus/Grafana 통합을 통해 테스트 중 실시간 메트릭을 모니터링할 수 있으며, 이는 시스템 성능 분석과 병목 지점 파악에 큰 도움이 됩니다.

### 표준 준수와 확장 가능성

AV Chaos Monkey는 RFC 3550(RTP), RFC 6184(H.264), RFC 7587(Opus), WebRTC 표준을 준수하여 실제 미디어 스트림을 생성합니다. 이는 단순한 더미 데이터가 아닌, 프로덕션 환경과 동일한 조건에서 테스트할 수 있음을 의미합니다.

개발자는 제공되는 Go 언어 예제를 참고하여 자신의 시스템에 통합할 수 있으며, REST API를 통해 테스트 시나리오를 자동화할 수 있습니다. Nix를 활용한 크로스 플랫폼 빌드 지원은 다양한 배포 환경에서의 활용도를 높여줍니다.

> <a href="https://github.com/MdSadiqMd/AV-Chaos-Monkey" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>