---
title: 백준 10989. Collection.sort() vs Arrays.sort() vs. Counting sort 성능 비교     
date: 2024-11-23 00:10:18
categories: Algorithm       
published: true 
tags:
- JAVA   
- Algorithm       
---  

### Collection.sort() vs Arrays.sort() vs. Counting sort 성능을 비교해보자  

아래의 백준 10989번 코드는 문제 없이 작동한다. 하지만 제출을 했을 때 메모리 초과가 떴다. 
    
```java
import java.io.*;
import java.util.*;

public class solve10989 {
    /*
    - 문제 분석 :
    - N개의 수가 주어졌을 때 이를 오름차순으로 정렬하는 프로그램을 작성하시오.
    - 풀이 전략 :
    -
        */

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        int count = Integer.parseInt(br.readLine());
        List<Integer> list = new ArrayList<>();
        //리스트 초기화
        for(int i=0;i<count;i++){
            list.add(Integer.parseInt(br.readLine()));
        }
        Collections.sort(list);
        for (Integer integer : list) {
            bw.write(integer+" ");
            bw.newLine();
        }
        bw.flush();
        bw.close();

    }
}  
```

이유는 ArrayList로 수를 저장하고, 그 후에 정렬하는 과정에서 메모리 사용량이 많아졌기 때문이다. 어떻게 하면 메모리 사용량을 더 줄일 수 있을까?  
이 문제에서는 데이터의 크기가 정확하게 주어지기 때문에, 고정 크기일 때 리스트보다 효율적인 배열을 사용할 수 있다(배열은 고정 크기의 메모리만 사용한다).

그래서 `Collections.sort()` 대신 `Arrays.sort()`를 사용하여 배열을 정렬하면 더 적은 메모리로도 결과값을 얻을 수 있다. 

### Collection.sort() vs. Arrays.sort() 메모리 사용량 및 속도 비교

1. **메모리 사용량** 
    - Collections.sort() : Collections.sort()는 **내부적으로 List를 배열로 변환한 후, Arrays.sort()를 사용해서 정렬**한다. 이 과정에서 추가적인 Boxing 및 메모리 할당이 필요하기 때문에 배열보다 많은 메모리를 소모한다.
    - Arrays.sort() : 배열을 직접 정렬하는 메서드로, 이미 배열이 메모리에 존재하고 있기 때문에 추가적인 메모리 할당이 필요하지 않다.
2. **속도** 
    - Collections.sort() : 메모리 사용량과 마찬가지로 List를 배열로 변환한 후, Arrays.sort()를 사용해서 정렬하는 과정에서 바로 Arrays.sort()를 호출하는 것보다 **추가 시간이 걸린다.**

---

### Counting Sort를 이용한다면  

Arrays.sort()로도 백준 문제는 통과가 가능하지만 라이브러리를 사용하지 않고 풀어보고 싶었다. 처음에는 퀵 정렬을 이용해서 풀어보려 했는데, 퀵 정렬로 풀지 않고 Counting sort를 이용해서 풀도록 유도하는 문제라는 조언을 보고 Counting sort를 사용해보기로 했다.  

- **Counting sort란? :** 
숫자의 크기 대신 빈도(카운트)에 기반하여 정렬하는 알고리즘이다. 특정 범위 내의 정수 데이터를 정렬할 때 효과적이고 빠르다.  
**비교 기반 정렬 알고리즘이 아니며, O(N)의 시간 복잡도**를 가진다.  
- 사용 조건 :
    - 정수 값만 존재해야 한다(소수점, 문자열 X)
    - 범위가 제한적이어야 한다. (ex. 0~10,000 같은 작은 범위에서 효율적이다.)
    - **데이터가 많지만 값의 범위가 적을 때** 적합하다. (ex. 0~10,000범위 내의 수가 10,000,000개 있을 때)
    - 주어진 데이터의 최대값과 최소값 범위를 알아야 한다.  
- 사용법 :  
    1. **카운트 배열 생성 :** 각 숫자의 빈도수를 저장하는 배열을 만든다. 배열의 크기는 데이터의 최댓값 + 1로 설정한다. 
    2. **출력 배열에 값 저장** : 입력 데이터를 순회하면서 숫자의 위치를 계산하여 정렬된 결과 배열에 저장한다. 
    3. **결과 반환** : 정렬된 데이터를 출력하거나 반환한다.  
    
    ```java
    import java.io.*;
    import java.util.Arrays;
    
    public class solve10989_re {
        /*
        - 문제 분석 :
        - N개의 수가 주어졌을 때 이를 오름차순으로 정렬하는 프로그램을 작성하시오.
        - 풀이 전략 :
        - 원래 ArrayList와 Collection.sort()를 이용해서 구현했었는데, 메모리 초과가 떴다.
        - 이 문제에서는 데이터의 크기가 미리 주어지므로 배열을 통해 구현이 가능하고,
        - 배열은 고정 크기의 메모리만 사용하므로 메모리 사용량을 줄일 수 있다.
        - 그래서 리스트를 배열로 바꾸고, Collection 대신 Arrays에서 sort()했더니 통과함.
        - Array.sort()보다 효과적인 Counting sort 방식으로도 풀어보았다.
            */
    
        public static void main(String[] args) throws IOException {
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
            // 여기서는 Counting sort를 사용해서 풀어본다.
            int count = Integer.parseInt(br.readLine());
            int[] list = new int[10001]; //숫자의 범위는 1~10,000
    
            //1. 입력 숫자를 카운트 배열에 기록
            for(int i=0;i<count;i++){
                int num = Integer.parseInt(br.readLine());
                list[num]++;
            }
    
            // 2. 카운트 배열을 순회하며 정렬된 결과 출력
            for(int i=1;i<list.length;i++){
                while (list[i]>0){
                    bw.write(i+" ");
                    list[i]--;
                }
            }
    
            bw.flush();
            bw.close();
    
        }
    }
    
    ```
    

### Arrays.sort() vs Counting Sort 비교  

1. 시간 복잡도  
    - Arrays.sort() :  
        - 내부적으로 Dual-Pivot Quicksort를 사용하고, 평균 O(N log N)의 시간 복잡도를 가진다.  
        - 최악의 경우(모든 데이터가 정렬된 상태)에도 O(N log N).  
        - 모든 정렬 가능한 데이터(정수, 실수, 객체)에 적합하다.  
    - Counting sort :  
        - 숫자의 범위가 K일 때 O(N+K)의 시간 복잡도를 가진다  
        - 데이터의 범위가 작을수록 더 빠르게 동작한다  
        - 비교 기반이 아니라, 정수 데이터에만 사용 가능    
    1. 메모리 사용  
        - Arrays.sort() :  
            - 입력 데이터를 정렬하기 위해 추가 메모리를 거의 사용하지 않음  
            - 단, 배열 크기가 크다면 메모리 사용량이 데이터 크기에 직접 비례  
        - Counting sort :  
            - 추가적으로 K 크기의 배열(카운트 배열)을 사용함  
            - 데이터 값의 범위가 클수록 메모리 사용량이 급격히 증가  
    
    ---
    
    ### 이 문제에서 Arrays.sort()와 Counting Sort 비교  
    
    1. 시간 복잡도  
        - Arrays.sort() : 시간 복잡도가 O(N log N)이므로, N이 10,000,000이면 약 2억 번 이상의 연산이 필요하다.  
        - Counting sort : 시간복잡도가 O(N+K)이므로 연산 횟수는 (10,000,000 + 10,000 = 10,010,000)으로 Arrays.sort()보다 훨씬 적다.  
    2. 메모리 사용  
        - Arrays.sort() : 입력 데이터를 직접 정렬하므로 데이터 크기(10,000,000 정수)에 비례하는 메모리를 사용  
        - Counting sort : 숫자의 범위(1~10,000)에 해당하는 카운트 배열 크기(10,001)만 추가로 사용한다.  

풀이 난이도 자체는 어렵지 않았지만, 알고리즘 문제를 해결 할 때 단순히 결과값이 일치하는 것뿐만 아니라 시간 복잡도와 메모리 사용량을 고려해서 최적의 방법을 찾는 게 중요하다는 걸 새삼 생각하게 되는 문제였다.  