---
title: 백준 9184. 메모이제이션Memoization에 관해 알아보자(DP, 동적계획법)    
date: 2024-11-14 14:42:18
categories: Algorithm       
published: true 
tags:
- JAVA   
- Algorithm       
---  

### 메모이제이션Memoization에 관해 알아보자(DP, 동적계획법) 

백준 9184번 문제를 해결할 방법에 대해 찾다가 메모이제이션Memoization 배열을 사용하면 수행 시간을 줄일 수 있다는 걸 알게 되었다.  

- **Memoization이란 뭘까?**  
동적 계획법(DP)의 한 기법으로, 계산된 결과를 배열이나 맵 같은 자료구조에 저장해두고, 나중에 같은 값에 대해 다시 계산할 필요 없이 저장된 값을 바로 사용하는 방식이다. 
대표적인 메모이제이션의 용례로 피보나치 수열 계산법이 있다.
- Memoization 없이 피보나치를 기본적인 재귀 방식으로 계산한다고 생각해 보자.
    
    ```java
    public class Fibonacci {
        public static int fibonacci(int n) {
            if (n <= 1) return n;
            return fibonacci(n - 1) + fibonacci(n - 2);  // 같은 값을 여러 번 계산함
        }
        
        public static void main(String[] args) {
            System.out.println(fibonacci(5));  // 5를 계산
        }
    }
    ```
    
    이 코드는 fibonacci(5)를 계산할 때 fibonacci(3)와 fibonacci(4)를 계산하고, 그 안에서 또 fibonacci(2)와 fibonacci(1)을 계산하는데 그러다보면 fibonacci(2)와 fibonacci(1)은 여러 번 중복으로 계산하게 된다. 
    fibonacci(519431)를 계산해야 하는 상황이라면 중복으로 계산하느라 낭비되는 시간이 무척 클 것이다. 
    
- **Memoization을 적용한 계산**
    
    ```java
    public class Fibonacci {
        // 메모이제이션 배열
        static int[] memo = new int[100];  // 충분히 큰 크기
        
        public static int fibonacci(int n) {
            if (n <= 1) return n;
            
            // 이미 계산된 값이 있으면 그 값을 반환
            if (memo[n] != 0) {
                return memo[n];
            }
            
            // 계산되지 않았다면 계산하고 메모이제이션 배열에 저장
            memo[n] = fibonacci(n - 1) + fibonacci(n - 2);
            
            return memo[n];
        }
        
        public static void main(String[] args) {
            System.out.println(fibonacci(5));  // 5를 계산
        }
    }
    
    ```
    
    memo[n] 배열의 값이 0 아니라면, 이미 계산되어 값을 넣어놓은 것이므로 그 값을 바로 반환하면 된다. 
    → 피보나치 수열의 경우 중복 계산을 방지함으로써 O(2^n)에서 O(n)으로 성능이 개선된다. 
    
- **백준 9184번 문제의 경우** 
링크 : https://www.acmicpc.net/problem/9184  
이 문제도 w(a,b,c)를 계산할 때 같은 계산을 여러 번 반복할 수 있기 때문에, 이를 배열에 저장해서 중복 계산을 방지하면 시간복잡도를 줄일 수 있다.
    
    ```java
    static int[][][] memo = new int[21][21][21];  // 0부터 20까지의 값에 대해 저장 
    ```
    
    `memo[a][b][c]` 배열에 `w(a, b, c)` 값을 저장해두고, 같은 값이 필요할 때 저장된 값을 바로 사용하면 된다.