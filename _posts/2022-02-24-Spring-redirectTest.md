---
title: git 사용법을 쉽게 정리해보자   
date: 2022-08-11 17:54:00
categories: JAVA, Algorithm 
published: true 
tags:
- Spring  
---


## 내용  

텍스트  

## 버블정렬, 선택정렬, 삽입정렬 간단 요약       
버블정렬 : 숫자 A를 바로 오른쪽에 있는 숫자 B와 비교했을 때 B가 더 작으면 서로 자리를 바꾼다. 
선택정렬 : 숫자 A의 자리를 배열에서 가장 작은 수와 교환한다. 
삽입정렬 : 배열의 왼쪽에서 두번째 숫자 B(a[1])를 먼저 선택 후, A(B의 왼쪽 숫자)와 비교했을 때 B가 더 작다면 위치를 바꿔준다.  
<br /> 
버블정렬은 성능이 좋지 않으므로 사용을 피한다. 초보자들이 처음으로 배우기 좋은 쉬운 정렬법이라 같이 알아두는 것. 삽입정렬은 셋 중에서 가장 빠르지만, 배열이 길어질수록 효율성이 떨어진다는 점을 알아두어야 한다. 


**1. 버블정렬**

```JAVA    
public class bubbleSort {

	public static void main(String[] args) {
		// 버블정렬
		// 숫자 A를 바로 오른쪽에 있는 숫자 B와 크기를 비교해서, A>B이면 서로 자리를 바꾼다. 
		
		int[] a = {3,5,1,2,4};
		int temp; 
		for(int i=a.length-1;i>0;i--) {
			System.out.println(a.length-i + "번째");
			for(int j=0; j<i;j++) {
				
				if(a[j]>a[j+1]) { //a[1]을 a[2](바로 오른쪽 숫자)와 비교하여 크기가 클 경우
					temp = a[j]; //temp에 a[1] 값을 저장해두고 
					a[j] = a[j+1]; //a[1]에는 a[2]의 값을 넣고 
					a[j+1] = temp; //a[2]에 temp에 저장해 둔 a[1]의 값을 넣으면 
					// a[1]에는 a[2]의 값이, a[2]에는 a[1]의 값이 들어가게 되므로 서로 자리가 바뀌는 것. 
				}
				
				for(int v:a) {
					System.out.printf("%3d",v);
				}
				 System.out.println("");
			}
			
		}
		
	}

}
```

---
