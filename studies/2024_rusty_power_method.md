---
layout: default
title: Rust로 고유값·고유벡터 찾기 (Power Method & Inverse Power Method)
---

# Rust로 고유값·고유벡터 찾기  
### Power Method와 Inverse Power Method의 구현

## 개요

이 문서는 Rust로 **고유값(Eigenvalue)** 과 **고유벡터(Eigenvector)** 를 수치적으로 계산하는 과정을 정리한 연구·스터디 노트이다.  
대표적인 반복 알고리즘인 **Power Method** 와 **Inverse Power Method** 를 직접 구현하며,  
선형대수 이론과 실제 시스템 코드 사이의 연결을 명확히 하는 것을 목표로 한다.

구현은 외부 수치해석 라이브러리에 의존하지 않고,  
행렬 연산과 반복 수렴 과정을 **명시적으로 코드로 표현**한다.

## 선행 지식

- 선형대수학
- 수치해석 기초 (반복법, 수렴)
- Rust 또는 C/C++ 시스템 프로그래밍 경험

## 개발 환경

- Rust 1.74.0
- 표준 라이브러리만 사용
- 참고 문서  
  https://rinthel.github.io/rust-lang-book-ko/ch01-01-installation.html

## 이론적 배경

### Power Method

반복식:

x_{k+1} = Ax_k / ||Ax_k||

절댓값이 가장 큰 고유값과 해당 고유벡터로 수렴한다.

### Inverse Power Method

(A - αI)^{-1} 를 사용하여  
α에 가장 가까운 고유값을 탐색한다.

## 구현 개요

정적 크기 행렬을 사용하여  
수식과 코드의 대응 관계를 명확히 유지한다.

## 행렬 정의

```rust
#[derive(Clone)]
struct Matrix<const ROW: usize, const COL: usize> {
    elements: [[f64; COL]; ROW],
}
```

## 기본 연산

### 벡터 절댓값 최대값

```rust
fn get_abs_max(&self) -> f64 {
    let mut max = 0.0;
    for v in self.elements {
        if v[0].abs() > max.abs() {
            max = v[0];
        }
    }
    max
}
```

## Power Method

```rust
fn power_method(
    &self,
    mut x: Matrix<N, 1>,
    iterations: usize,
) -> (Matrix<N, 1>, f64) {
    for _ in 0..iterations {
        let y = self.clone() * x;
        let lambda = y.get_abs_max();
        x = y / lambda;
    }
    let eigenvalue = (self.clone() * x.clone()).get_abs_max();
    (x, eigenvalue)
}
```

## Inverse Power Method

```rust
fn inverse_power_method(
    &self,
    mut x: Matrix<N, 1>,
    iterations: usize,
    alpha: f64,
) -> (Matrix<N, 1>, f64) {
    let shifted = self.clone() - identity().mul(alpha);
    let inv = shifted.inverse();

    for _ in 0..iterations {
        let y = inv.clone() * x;
        let mu = y.get_abs_max();
        x = y / mu;
    }

    let eigenvalue = alpha + 1.0 / (inv * x.clone()).get_abs_max();
    (x, eigenvalue)
}
```

## 정리

- Power Method는 지배 고유값에 수렴
- Inverse Power Method는 특정 고유값 근처 탐색 가능
- 반복법의 수렴성은 초기값과 정규화에 민감

## 확장 아이디어

- Rayleigh Quotient
- 수렴 조건 도입
- IR / DAG 기반 연산 표현
