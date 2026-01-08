---
title: "C++23 고성능 B+Tree 구현체 fast-containers: Abseil 대비 최대 5배 빠른 성능"
description: "C++23 기반 헤더 온리 라이브러리 fast-containers의 B+Tree 구현체를 소개합니다. SIMD 검색과 hugepage 할당자를 통해 Abseil B+tree 대비 2-5배 빠른 성능을 제공합니다."
date: 2026-01-08 17:01:01 +0900
categories: ['Development', 'Language']
tags: ['C++23', 'B+Tree', 'SIMD', 'performance', 'hugepage', 'x86-64', 'header-only', 'fast-containers']
image:
  path: "/assets/img/2026-01-08-high-performance-header-only-container-library-for-c23-on-x86-64-0842.jpg"
  alt: "fast-containers 고성능 C++23 B+Tree 라이브러리 아키텍처"
published: true
---

## 3줄 요약
- C++23 기반 헤더 온리 컨테이너 라이브러리로 SIMD 검색과 hugepage 할당자를 활용한 B+Tree 구현체를 제공합니다.
- 1천만 개 요소 규모에서 Abseil B+tree 대비 2-5배, std::map 대비 2-5배 빠른 성능을 보여줍니다.
- AVX2 명령어 세트와 캐시 친화적인 노드 구조를 통해 대규모 트리에서 탁월한 성능을 발휘합니다.

## 📌 주요 내용

### fast-containers 라이브러리 개요

**fast-containers**는 x86-64 아키텍처의 C++23을 위한 고성능 헤더 온리 컨테이너 라이브러리입니다. 현재 B+Tree 구현체를 중심으로 개발되었으며, 캐시 친화적인 설계와 SIMD 검색, hugepage 지원을 통해 업계 표준 대비 상당한 성능 개선을 제공합니다.

라이브러리에 포함된 주요 컴포넌트는 다음과 같습니다:

- **B+Tree** (`kressler::fast_containers::btree`): SIMD 검색과 hugepage 지원을 갖춘 캐시 친화적인 B+Tree
- **Dense Map** (`kressler::fast_containers::dense_map`): B+Tree 노드 내부에서 사용되는 고정 크기 정렬 배열
- **Hugepage Allocators**: TLB 미스와 할당 오버헤드를 줄이는 풀링 할당자 제공

### 성능 벤치마크 결과

대규모 트리(1천만 개 요소)에서 측정한 성능 결과는 다음과 같습니다:

**8바이트 키, 32바이트 값 기준 P99.9 레이턴시:**

- **INSERT**: 1,023 ns (hugepage 사용 시)
- **FIND**: 864 ns (hugepage 사용 시)
- **ERASE**: 1,086 ns (hugepage 사용 시)

**Abseil B+tree 대비 성능:**

- Abseil (hugepage 사용): INSERT 27% 빠름, FIND 38% 빠름, ERASE 20% 빠름
- Abseil (표준 할당자): INSERT 3.2배 빠름, FIND 55% 빠름, ERASE 55% 빠름

**std::map 대비 성능:**

- INSERT 3.5배 빠름
- FIND 2.7배 빠름
- ERASE 2.1배 빠름

### SIMD 가속 검색과 Hugepage 할당자

fast-containers의 핵심 성능 최적화 기술은 두 가지입니다:

**1. SIMD 가속 검색**

AVX2 명령어를 활용하여 노드 내부 검색을 3-10% 가속합니다. int32_t, uint32_t, int64_t, uint64_t, float, double 타입의 키를 지원합니다.

```cpp
// SIMD 검색 활성화 예제
using Tree = kressler::fast_containers::btree<
  int64_t,                                      // Key type
  int32_t,                                      // Value type
  96,                                           // Leaf node size
  128,                                          // Internal node size
  std::less<int64_t>,                           // Comparator
  kressler::fast_containers::SearchMode::SIMD,  // SIMD search
  std::allocator<std::pair<int64_t, int32_t>>   // Allocator
>;
```

**2. Hugepage 할당자**

TLB 미스를 줄이고 할당 비용을 최소화하여 3-5배의 성능 향상을 제공합니다:

```cpp
// Hugepage 할당자 사용 예제
using Allocator = kressler::fast_containers::HugePageAllocator<
    std::pair<int64_t, int32_t>>;

using Tree = kressler::fast_containers::btree<
  int64_t, int32_t, 96, 128, std::less<int64_t>,
  kressler::fast_containers::SearchMode::SIMD,
  Allocator
>;

Tree tree;  // 기본값: 256MB 초기 풀, 64MB 증가량
```

### 튜닝 가능한 노드 크기

fast-containers는 키/값 크기에 따라 최적화할 수 있는 노드 크기를 제공합니다:

**LeafNodeSize**: 리프 노드당 키-값 쌍의 개수
- 기본값: 약 2KB(32 캐시 라인)를 목표로 자동 계산
- 작은 값: 64-96 추천
- 큰 값: 8-16 추천

**InternalNodeSize**: 내부 노드당 자식 포인터 개수
- 기본값: 약 1KB(16 캐시 라인)를 목표로 자동 계산
- 일반적으로 기본값 사용 권장

```cpp
template <
  typename Key,
  typename Value,
  std::size_t LeafNodeSize = default_leaf_node_size<Key, Value>(),
  std::size_t InternalNodeSize = default_internal_node_size<Key>(),
  typename Compare = std::less<Key>,
  SearchMode SearchModeT = SearchMode::Linear,
  typename Allocator = std::allocator<std::pair<Key, Value>>
>
class btree;
```

### 다양한 할당자 옵션

**HugePageAllocator**: 단순하고 자동으로 타입별 별도 풀 생성 (권장)

```cpp
using Allocator = kressler::fast_containers::HugePageAllocator<
    std::pair<int64_t, int32_t>>;
```

**MultiSizeHugePageAllocator**: 가변 크기 할당을 위한 할당자 (Abseil B+tree에 적합)

```cpp
auto alloc = kressler::fast_containers::make_multi_size_hugepage_allocator<
    std::pair<const int64_t, ValueType>>();

absl::btree_map<int64_t, ValueType, std::less<int64_t>, Allocator> tree(alloc);
```

**PolicyBasedHugePageAllocator**: 세밀한 제어와 공유 풀 지원

```cpp
auto leaf_pool = std::make_shared<kressler::fast_containers::HugePagePool>(
    512 * 1024 * 1024, true);
auto internal_pool = std::make_shared<kressler::fast_containers::HugePagePool>(
    256 * 1024 * 1024, true);

kressler::fast_containers::TwoPoolPolicy policy{leaf_pool, internal_pool};
using Allocator = kressler::fast_containers::PolicyBasedHugePageAllocator<
    std::pair<int64_t, int32_t>,
    kressler::fast_containers::TwoPoolPolicy>;
```

### 설치 및 사용법

**필수 요구사항:**
- C++23 컴파일러 (GCC 14+, Clang 19+)
- CMake 3.30+
- AVX2 지원 CPU (Intel Haswell 2013+, AMD Excavator 2015+)

**프로젝트에 통합:**

```bash
# Git 서브모듈로 추가
git submodule add https://github.com/kressler/fast-containers.git third_party/fast-containers
```

```cmake
# CMakeLists.txt에 링크
add_subdirectory(third_party/fast-containers)
target_link_libraries(your_target PRIVATE fast_containers::fast_containers)
```

**기본 사용 예제:**

```cpp
#include <fast_containers/btree.hpp>
#include <iostream>

int main() {
  using Tree = kressler::fast_containers::btree<int64_t, int32_t>;
  Tree tree;

  tree.insert(42, 100);
  tree.insert(17, 200);
  tree.insert(99, 300);

  auto it = tree.find(42);
  if (it != tree.end()) {
    std::cout << "Found: " << it->second << std::endl;
  }

  for (const auto& [key, value] : tree) {
    std::cout << key << " -> " << value << std::endl;
  }

  tree.erase(17);
  std::cout << "Size: " << tree.size() << std::endl;
}
```

### API 개요

std::map과 유사한 API를 제공합니다:

- **삽입**: `insert()`, `emplace()`, `operator[]`
- **검색**: `find()`, `lower_bound()`, `upper_bound()`, `equal_range()`
- **삭제**: `erase()`
- **순회**: `begin()`, `end()`, 범위 기반 for 루프 지원
- **용량**: `size()`, `empty()`, `clear()`
- **기타**: `swap()`, `key_comp()`, `value_comp()`

## 👨‍💻 개발자에게 미치는 영향

### 대규모 데이터 처리 성능 개선

1천만 개 이상의 요소를 다루는 애플리케이션에서 hugepage 할당자를 사용하면 2-5배의 성능 향상을 기대할 수 있습니다. 특히 INSERT 작업에서 TLB 미스 감소와 풀링 할당의 이점이 크게 나타납니다.

### 헤더 온리 라이브러리의 장점

별도의 빌드나 링크 과정 없이 헤더 파일만 포함하면 사용할 수 있어 통합이 간편합니다. CMake 기반 프로젝트에서는 서브모듈로 추가하여 즉시 활용할 수 있습니다.

### 플랫폼 제약사항 고려

현재 Linux x86-64 환경에서만 빌드 및 테스트되었으며, AVX2 지원 CPU가 필요합니다. SIMD 구현체가 x86-64 전용이므로 다른 아키텍처에서는 Linear 또는 Binary 검색 모드를 사용해야 합니다.

### Hugepage 설정 필요

최대 성능을 위해서는 시스템에 hugepage를 구성해야 합니다:

```bash
sudo sysctl -w vm.nr_hugepages=<num_pages>
```

Hugepage를 사용할 수 없는 경우 자동으로 일반 페이지로 폴백되지만 성능 이점은 감소합니다.

### 프로덕션 사용 시 주의사항

이 라이브러리는 현재 진행 중인 프로젝트이며, 소규모 트리(1만 개 이하 요소)에서는 종합적인 테스트가 이루어지지 않았습니다. 프로덕션 환경에서 사용하기 전에 실제 워크로드로 충분한 테스트를 수행할 것을 권장합니다.

> <a href="https://github.com/kressler/fast-containers" target="_blank" rel="noopener noreferrer">원문 기사 보기</a>