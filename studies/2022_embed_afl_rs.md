---
layout: default
title: Fuzzing JavaScript Engines in Rust with AFL++
---

# Fuzzing JavaScript Engines in Rust
### Using AFL++ and LLVM Instrumentation

## 개요

이 문서는 **Rust 기반 러너(runner)** 를 사용해 주요 JavaScript 엔진(V8, SpiderMonkey, JavaScriptCore)을  
**AFL++ LLVM 모드**로 퍼징(fuzzing)하기 위한 실험 환경 구성 및 빌드 과정을 정리한 실전 가이드이다.

목표는 다음과 같다.

- Rust로 작성한 퍼저 러너에서 JS 엔진을 직접 구동
- AFL++의 LLVM 계측(instrumentation)을 엔진 내부까지 적용
- 연구 실험에 재현 가능한 환경을 제공

---

## 0. 지원 환경

- **x86_64 Ubuntu 22.04**
  - WSL2 가능
  - Docker / Container 환경 가능
- 기타 환경은 연구 실험 종료 후 지원 예정

---

## 1. AFL++ 및 LLVM 환경 구성

### 1.1 LLVM 15 설치

AFL++ LLVM 모드를 사용하기 위해 LLVM 15 및 관련 도구를 설치한다.

```bash
sudo apt-get install -y   llvm-15 llvm-15-tools   clang-15 lld-15   build-essential
```

**왜 LLVM 15인가?**

- Ubuntu 22.04 기본 LLVM 버전
- AFL++ LLVM 계측 기능과 가장 안정적으로 호환
- 이후 버전 출시 시 교체 가능

---

### 1.2 AFL++ 클론 및 빌드

```bash
cd path_to_clone
git clone https://github.com/AFLplusplus/AFLplusplus.git
```

LLVM 15를 사용하도록 환경 변수 설정 후 빌드한다.

```bash
cd AFLplusplus
export LLVM_CONFIG=llvm-config-15
make
sudo make install
```

설치 확인:

```bash
which afl-clang-fast
afl-clang-fast -hh
```

---

### 1.3 AFL++ 런타임 라이브러리 준비

정적 링크를 위해 런타임 라이브러리를 아카이브로 생성한다.

```bash
ar rcus libafl-rt.a afl-compiler-rt.o
```

---

### 1.4 Clang / LLVM 도구 후킹

AFL 계측을 강제 적용하기 위해 clang 및 llvm-tools를 후킹한다.

```bash
sudo ln -s /usr/local/bin/afl-clang-fast /usr/local/bin/clang
sudo ln -s /usr/local/bin/afl-clang-fast++ /usr/local/bin/clang++
```

LLVM 도구 복사:

```bash
sudo cp /usr/bin/llvm-ar-15 /usr/local/bin/llvm-ar
sudo cp /usr/bin/llvm-readelf-15 /usr/local/bin/llvm-readelf
sudo cp /usr/bin/llvm-nm-15 /usr/local/bin/llvm-nm
```

---

## 2. JavaScript 엔진 빌드 환경 구성

### 2.1 V8

#### rusty_v8 클론

```bash
cd path_to_clone
git clone https://github.com/denoland/rusty_v8.git
cd rusty_v8
git submodule update --init --recursive
```

#### 의존성 설치

```bash
sudo apt install -y libglib2.0-dev
```

---

### 2.2 SpiderMonkey

#### mozjs 클론

```bash
cd path_to_clone
git clone https://github.com/servo/mozjs.git
```

#### 의존성 설치

```bash
sudo apt install -y autoconf pkg-config zlib1g-dev
```

#### 빌드 스크립트 수정 (계측 제외)

Rust ↔ C++ glue 코드에 AFL 계측이 들어가지 않도록 컴파일러를 고정한다.

- `mozjs/rust-mozjs/build.rs`
  - 15번째 줄 이후:
    ```rust
    .compiler("clang++-15")
    ```

- `mozjs/mozjs/build.rs`
  - 235번째 줄 이후:
    ```rust
    .compiler("clang++-15")
    ```

---

### 2.3 JavaScriptCore (JSC)

#### jsc-sys 클론

```bash
cd path_to_clone
git clone https://github.com/drtychai/jsc-sys.git
cd jsc-sys
git clone https://github.com/WebKit/WebKit.git
```

#### 의존성 설치

```bash
sudo apt install -y ruby3.0
```

#### 빌드 스크립트 수정

- `makefile.cargo` (38번째 줄 이후):

```makefile
CMAKE_ARGS += " -DPython_EXECUTABLE=path_to_python"
```

- `build.rs` (103번째 줄 이후):

```rust
println!("cargo:rustc-link-lib=atomic");
```

---

## 3. 퍼징 러너 빌드

### 3.1 공통 사항

- 각 러너의 `Cargo.toml`에서 **커스텀 빌드된 JS 엔진 경로**를 명시
- AFL++ 계측을 위해 환경 변수 설정 필수

---

### 3.2 V8 러너

```bash
git clone https://github.com/UsQuake/v8_integ.git
cd v8_integ
```

`Cargo.toml` 수정:

```toml
[dependencies]
v8 = { path = "path_to_clone_rusty_v8" }
```

환경 변수 설정:

```bash
export AFL_PATH="path_to_clone_AFL++"
export AFL_LLVM_INSTRUMENT="CLASSIC"
export RUSTFLAGS="-Ccodegen-units=1 -Clink-arg=-fuse-ld=gold -lafl-rt -Lpath_to_clone_AFL++"
export CLANG_BASE_PATH="/usr/local"
export V8_FROM_SOURCE=1
export PYTHON="path_to_python3"
```

빌드:

```bash
cargo build
```

---

### 3.3 SpiderMonkey 러너

```bash
git clone https://github.com/UsQuake/mozjs_integ.git
cd mozjs_integ
```

`Cargo.toml` 수정:

```toml
[dependencies]
mozjs = { path = "path_to_clone_mozjs/rust-mozjs" }

[patch."https://github.com/servo/mozjs"]
mozjs_sys = { path = "path_to_clone_mozjs/mozjs" }
```

환경 변수:

```bash
export AFL_PATH="path_to_clone_AFL++"
export AFL_LLVM_INSTRUMENT="CLASSIC"
export RUSTFLAGS="-Ccodegen-units=1 -Clink-arg=-fuse-ld=gold -lafl-rt -Lpath_to_clone_AFL++"
export CC="afl-clang-fast"
export CXX="afl-clang-fast++"
export LIBCLANG_PATH=/usr/lib/x86_64-linux-gnu
```

빌드:

```bash
cargo build
```

---

### 3.4 JavaScriptCore 러너

```bash
git clone https://github.com/UsQuake/jsc_integ.git
cd jsc_integ
```

`Cargo.toml` 수정:

```toml
[dependencies]
jscjs_sys = { path = "path_to_clone_jsc_sys" }
url = "2.4.0"
```

환경 변수:

```bash
export AFL_PATH="path_to_clone_AFL++"
export AFL_LLVM_INSTRUMENT="CLASSIC"
export RUSTFLAGS="-Ccodegen-units=1 -Clink-arg=-fuse-ld=gold -lafl-rt -Lpath_to_clone_AFL++"
```

빌드:

```bash
cargo build
```

---

## 마무리

이 문서는 **실험 재현성**과 **계측 일관성**을 우선한 설정을 기반으로 한다.

향후 확장 가능 주제:

- Sanitizer (ASan / UBSan) 병행 퍼징
- Coverage metric 분석
- IR 기반 JS 엔진 퍼징 파이프라인
- Differential fuzzing across engines
