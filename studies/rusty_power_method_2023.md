---
layout: default
title: Rust로 고유값/고유벡터 찾기
---

# Rust로 고유값/고유벡터 찾기

  ## 소개
  
   - Rust로 간단한 iterative power method 구현해 고유값/고유벡터 찾기.
     
  ## 선행 지식

   - 선형대수학에 대한 이해.
   - Rust/C++에 대한 이해.
     
  ## 요구 사항
   
   - Rust 버전 ***1.74.0***
   - 참고: *Rust Installation*[https://rinthel.github.io/rust-lang-book-ko/ch01-01-installation.html]

  ## 구현 과정
  
  ### 1.행렬/열벡터를 정의합니다.

  - 인덱스를 통해 행렬에[행][열]꼴로 접근할 수 있게 다음과 같이 정의합니다.
    ```Rust
    struct Matrix<const ROW_COUNT: usize, const COLUMN_COUNT: usize>{
        elements: [[f64;COLUMN_COUNT];ROW_COUNT]
    }
    ```
    
  ### 2.벡터/행렬에 대한 기본적인 연산들(정규화, 내적, 행렬곱)을 구현합니다.

  - 행렬-열벡터 곱 구현.
    ```Rust
    fn mul(self, other_vector: Vec3) -> Vec3{
       Matrix{
        elements :[
                  [ self.elements[0][0] * other_vector.elements[0][0] + self.elements[0][1] * other_vector.elements[1][0] + self.elements[0][2] * other_vector.elements[2][0] ],
                  [ self.elements[1][0] * other_vector.elements[0][0] + self.elements[1][1] * other_vector.elements[1][0] + self.elements[1][2] * other_vector.elements[2][0] ],
                  [ self.elements[2][0] * other_vector.elements[0][0] + self.elements[2][1] * other_vector.elements[1][0] + self.elements[2][2] * other_vector.elements[2][0] ]
               ]
         }
    }
    ```
  - (A - I α) 식 해석을 위해 행렬-행렬 뺄셈 구현.
    ```Rust
    fn sub(self, other_matrix: Self) -> Matrix<ROW_COUNT,COLUMN_COUNT>{
        let mut result = Matrix{elements:[[0.0;COLUMN_COUNT];ROW_COUNT]};
        for i in 0..ROW_COUNT{
            for j in 0..COLUMN_COUNT{
                result.elements[i][j] = self.elements[i][j] - other_matrix.elements[i][j];
            }
        }
        result
    }
    ```
  - (A - I α) 식 해석을 위해 행렬-스칼라 곱 구현.
    ```Rust
    fn mul(self, other_scala: f64) -> Matrix<ROW_COUNT,COLUMN_COUNT>{
        let mut result = Matrix{elements: [[0.0;COLUMN_COUNT];ROW_COUNT]};
        for row in 0..ROW_COUNT{
            for col in 0..COLUMN_COUNT{
                result.elements[row][col] = self.elements[row][col] * other_scala;
            }
        }
        result
    }
    ```
  - Power method 구현을 위해 행렬/벡터-스칼라 나눗셈 구현.
    ```Rust
    fn div(self, other_scala:f64) -> Matrix<ROW_COUNT,COLUMN_COUNT>{
        let mut result = Matrix{elements:[[0.0;COLUMN_COUNT];ROW_COUNT]};
        for i in 0..ROW_COUNT{
            for j in 0..COLUMN_COUNT{
                result.elements[i][j] = self.elements[i][j] / other_scala;
            }
        }
        result
    }
    ```
  - Power method 구현을 위해 벡터 절대값 최대값 구현.
    ```Rust
    pub fn get_abs_max(&self) -> f64{
        let mut max:f64 = 0.0;
        for i in self.elements{
            if max.abs() < i[0].abs() {
                max = i[0];
            }
        }
        max
    }
    ```

  - Inverse power method 구현을 위해 역행렬을 구하는 가우스-조르단 소거법을 구현.
    ```Rust
    pub fn get_inverse_matrix(&self) -> Self
    {
       let mut self_copy = self.clone(); 
       let mut result =  Self::get_identity_matrix();
       for i in 0..N{
        let pivot = self_copy.elements[i][i];
        for j  in 0..N{
            self_copy.elements[i][j] /= pivot;
            result.elements[i][j] /= pivot;
        }


        for j in 0..i{   
            let ratio = self_copy.elements[j][i];

            for k in 0..N{   
                    result.elements[j][k] -= ratio * result.elements[i][k];
                    self_copy.elements[j][k] -= ratio * self_copy.elements[i][k];
            }
        }
        for j in i + 1..N{
            let ratio = self_copy.elements[j][i];

            for k in 0..N{
                result.elements[j][k] -= ratio * result.elements[i][k];
                self_copy.elements[j][k] -= ratio *  self_copy.elements[i][k];
            }
        }
       }
       result
    }
    ```

  ### 4.inverse power method & power method를 구현합니다.

  - power method로 dominant eigen을 탐색.
      ```Rust
      pub fn get_dominant_eigen(&self, x: Matrix<N,1>, try_count: usize) -> (Matrix<N,1>, Matrix<N,1>, f64)
      {
        let mut try_count = try_count;
        let mut x= x.clone();
        let mat_a = self.clone(); 
        loop{
            let a_mul_x = mat_a * x;
            if try_count == 0 
            {
                return (x, a_mul_x, a_mul_x.get_abs_max());
            }
            x = a_mul_x / a_mul_x.get_abs_max();
            try_count -= 1;
        }

      }
      ```
 - inverse power method로 근삿값과 가장 가까운 eigen을 탐색.
   ```Rust
   pub fn get_alpha_nearest_eigen(&self, x: Matrix<N,1>, try_count: usize, alpha:f64)-> (Matrix<N,1>, Matrix<N,1>, f64, f64){
        let mut try_count = try_count;
        let mut x= x.clone();
        let solution_with_matrix: Matrix<N, N> = (*self -  Self::get_identity_matrix() * alpha).get_inverse_matrix(); 
        loop{
            let y = solution_with_matrix * x;
            let mu =  y.get_abs_max();
            let v = alpha + (1.0 / mu);
            if try_count == 0 
            {
                return (x, y, mu, v);
            }
            x = y / mu;
            try_count -= 1;
        }
   }
   ```
