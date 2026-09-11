---
title: "재귀 호출과 퀵 정렬 (Java)"
date: 2026-09-11 09:10:00 +0900
categories: [Algorithm, Java]
tags: [recursion, quick-sort, java, algorithm]
---

## 재귀 호출

재귀 호출은 함수가 자기 자신을 호출하는 것이다. 6장 이진 탐색 트리에서 이미 사용했고, 이 장 후반의 퀵 정렬도 재귀 없이는 작성이 어렵다.

재귀 구조를 확인하는 예제로 인수 n의 계승(factorial)을 구하는 함수를 만든다. 계승은 보통 반복문으로도 작성할 수 있으므로 **재귀 호출을 사용하는 것이 결코 스마트하지만은 않다**. 다만 재귀의 구조를 설명하기 좋아 자주 예로 쓰인다.

### 의사코드

```text
/* 인수 n의 계승을 구하는 함수(여기부터) */
○ 정수형: factorial(정수형: n)
▲ n = 0
    /* 0의 계승은 1이므로 1을 반환하여 재귀 호출을 종료한다 */
    · return 1
─
    /* n의 계승은 n * (n - 1)의 계승이므로, */
    /* 재귀 호출로 (n - 1)의 계승을 구한다 */
    · return n * factorial(n - 1)
▼
/* 인수 n의 계승을 구하는 함수(여기까지) */

/* 프로그램 실행의 시작점인 main 함수(여기부터) */
○ main
○ 정수형: ans
/* 5의 계승을 구한다 */
· ans ← factorial(5)
· ans값을 표시한다
/* 프로그램 실행의 시작점인 main 함수(여기까지) */
```

### 자바

```java
public class RecursiveCall {
    // 인수 n의 계승을 구하는 메소드
    public static int factorial(int n) {
        if (n == 0) {
            // 0의 계승은 1이므로 1을 반환하여 재귀 호출을 종료
            return 1;
        } else {
            // n의 계승은 n * (n - 1)의 계승이므로, 재귀 호출로 (n - 1)의 계승을 구함
            return n * factorial(n - 1);
        }
    }

    // 프로그램 실행의 시작점인 main 메소드
    public static void main(String[] args) {
        int ans;

        // 5의 계승을 구함
        ans = factorial(5);
        System.out.printf("%d\n", ans);
    }
}
```

### 코드 설명

`factorial` 메소드에서는 "0의 계승은 1"이라는 수학적 조건 이외에는 다른 조건이 없다. 따라서 `if` 문의 조건으로 `n == 0`을 설정해 0의 계승을 처리하고, `else` 문에서 `n * factorial(n - 1)`이라는 계승 계산을 반환하면서 일반적인 계승 연산을 하도록 했다.

## 퀵 정렬 알고리즘

퀵 정렬(quick sort)은 대량의 데이터를 효율적으로 정렬하는 알고리즘이다. **기준값(pivot)이 되는 요소를 하나 선택해, 나머지 요소들을 기준값보다 작은 값과 큰 값으로 그룹 나누기를 반복하여 전체를 정렬**한다. 그룹으로 나누어진 데이터가 하나가 될 때까지 반복하며(2개 이상이면 계속), 데이터가 하나면 위치가 확정된다.

1. 기준값이 되는 요소를 하나 선택한다.
2. 나머지 요소를 기준값보다 작은 값과 큰 값으로 그룹 나누기를 반복한다.
3. 그룹으로 나누어진 데이터가 하나가 될 때까지 1과 2를 반복한다.

그룹 나누기 함수 `divideArray`와 그 함수로 정렬을 수행하는 함수 `sortArray`를 준비하면 작성이 매우 간단해진다. `sortArray` 안에서 `divideArray`를 사용하고, `sortArray` 처리 내에서 재귀 호출을 사용한다. 오름차순으로 정렬한다.

### 의사코드 – sortArray

```text
/* 배열 a[start]~a[end]를 오름차순으로 정렬하는 함수(여기부터) */
○ sortArray(정수형: a[], 정수형: start, 정수형: end)
○ 정수형: pivot      /* 배열을 그룹으로 나누는 기준값의 위치 */
/* 배열의 요소가 2개 이상인 경우 처리한다 */
▲ start < end
    /* 기준값과의 대소 관계에 따라 그룹 나누기 */
    · pivot ← divideArray(a, start, end)
    /* 기준값보다 작은 앞쪽 그룹에 동일한 처리를 적용한다(재귀 호출) */
    · sortArray(a, start, pivot - 1)
    /* 기준값보다 큰 뒤쪽 그룹에 동일한 처리를 적용한다(재귀 호출) */
    · sortArray(a, pivot + 1, end)
▼
/* 배열 a[start]~a[end]를 오름차순으로 정렬하는 함수(여기까지) */
```

인수로 지정된 `a[start]~a[end]` 범위를 정렬한다. 요소 수가 2개 이상이라는 조건에서 그룹 나누기를 수행한 뒤, 앞쪽 그룹과 뒤쪽 그룹을 각각 인수로 지정해 `sortArray`를 재귀 호출한다. 처리 내용은 이것뿐이다.

### 의사코드 – divideArray

```text
/* 배열 a[head]~a[tail]을 그룹으로 나누는 함수(여기부터) */
○ 정수형: divideArray(정수형: a[], 정수형: head, 정수형: tail)
○ 정수형: left, right, temp
· left ← head + 1     /* 선두 +1부터 훑어 가는 위치 */
· right ← tail        /* 끝부터 훑어 가는 위치 */
/* 기준값 a[head]보다 작은 요소는 앞쪽으로, 큰 요소는 뒤쪽으로 이동한다 */
■ true
    /* 배열을 선두 +1부터 뒤쪽으로 훑어 기준값보다 큰 요소를 찾아낸다 */
    ■ left < tail and a[head] > a[left]
        · left ← left + 1
    ■
    /* 배열을 끝에서 앞으로 훑어 기준값보다 작은 요소를 찾아낸다 */
    ■ a[head] < a[right]
        · right ← right - 1
    ■
    /* 확인할 요소가 없어지면 종료한다 */
    ▲ left >= right
        · break
    ▼
    /* 기준값보다 큰 a[left]와 기준값보다 작은 a[right]를 교환한다 */
    · temp ← a[left]
    · a[left] ← a[right]
    · a[right] ← temp
    /* 다음 요소를 체크해 간다 */
    · left ← left + 1
    · right ← right - 1
■
/* 기준값 a[head]와 a[right]를 교환한다 */
· temp ← a[head]
· a[head] ← a[right]
· a[right] ← temp
/* 기준값 a[right]의 위치를 반환한다 */
· return right
/* 배열 a[head]~a[tail]을 그룹으로 나누는 함수(여기까지) */
```

`a[head]~a[tail]` 범위를 두 그룹으로 나누어 기준값의 인덱스를 반환한다. 기준값보다 앞쪽에는 작은 요소, 뒤쪽에는 큰 요소가 있다. 예를 들어 `a[0]~a[6]`을 나누어 3이 반환되면 `a[3]`이 기준값이고, `a[0]~a[2]`에는 `a[3]`보다 작은 요소, `a[4]~a[6]`에는 큰 요소가 있다.

`■ true` 부분만으로는 무한 반복이 되므로 처리 속에 `break`를 넣어 해당 시점에서 반복이 종료된다.

### 자바

```java
public class QuickSort {
    // 배열 내용을 표시하는 메소드
    public static void printArray(int[] a) {
        for (int i = 0; i < a.length; i++) {
            System.out.printf("[" + a[i] + "]");
        }
        System.out.printf("\n");
    }

    // 배열 a[head]~a[tail]을 그룹으로 나누는 메소드
    public static int divideArray(int[] a, int head, int tail) {
        int left, right, temp;
        left = head + 1;    // 배열 첫 요소 + 1부터 뒷 요소로 훑어 가는 위치
        right = tail;       // 배열 끝 요소부터 앞 요소로 훑어 가는 위치

        // 기준값 a[head]보다 작은 요소를 앞쪽으로, 큰 요소를 뒤쪽으로 이동
        while (true) {
            // 배열을 첫 요소 + 1부터 뒤쪽으로 훑어가,
            // 기준값보다 큰 요소를 찾음
            while (left < tail && a[head] > a[left]) {
                left++;
            }

            // 배열 끝 요소에서 앞으로 훑어 기준값보다 작은 요소를 찾음
            while (a[head] < a[right]) {
                right--;
            }

            // 확인할 요소가 없어지면 종료
            if (left >= right) {
                break;
            }

            // 기준값보다 큰 a[left]와 기준값보다 작은 a[right]를 교환
            temp = a[left];
            a[left] = a[right];
            a[right] = temp;

            // 다음 요소를 확인함
            left++;
            right--;
        }

        // 기준값 a[head]와 a[right]를 교환
        temp = a[head];
        a[head] = a[right];
        a[right] = temp;

        // 기준값 a[right]의 위치를 반환
        return right;
    }

    // 배열 a[start]~a[end]를 오름차순으로 정렬하는 메소드
    public static void sortArray(int[] a, int start, int end) {
        int pivot;    // 배열을 그룹으로 나누는 기준값의 인덱스 위치

        // 배열 요소가 2개 이상인 경우 정렬 처리 진행
        if (start < end) {
            // 기준값과의 대소 관계에 따라 그룹 나누기
            pivot = divideArray(a, start, end);

            // 기준값보다 작은 앞쪽 그룹에 동일한 처리를 적용(재귀 호출)
            sortArray(a, start, pivot - 1);

            // 기준값보다 큰 뒤쪽 그룹에 동일한 처리를 적용(재귀 호출)
            sortArray(a, pivot + 1, end);
        }
    }

    // 프로그램 실행의 시작점인 main 메소드
    public static void main(String[] args) {
        int[] a = { 4, 7, 1, 6, 2, 5, 3 };

        // 정렬 전 배열을 표시
        printArray(a);

        // 퀵 정렬 실행
        sortArray(a, 0, a.length - 1);

        // 정렬된 배열을 표시
        printArray(a);
    }
}
```

### 코드 설명

의사코드로 설명한 것과 동일한 기능의 `divideArray`, `sortArray`, 배열을 표시하는 `printArray`, 그리고 `main` 메소드가 있다. 정렬 대상은 `main`에서 선언된 요소 7개의 배열 `[4][7][1][6][2][5][3]`이다.

`divideArray`는 배열 `a`, 기준값의 인덱스 `head`, 배열 끝 요소의 인덱스 `tail`을 인수로 받는다. 안에서는 배열 첫 요소 + 1부터 뒷 요소를 훑는 `left`, 뒷 요소부터 앞 요소를 훑는 `right`, 임시 저장용 `temp`를 선언한다.

`while (true)` 안에 `while` 문 2개와 `if` 문을 구성한다. 첫 번째 `while`은 `left < tail && a[head] > a[left]`(뒷 요소 인덕스가 끝 요소보다 작고, 기준값이 앞 요솟값보다 큼)일 때 `left`를 1 증가시킨다. 두 번째 `while`은 `a[head] < a[right]`(기준값이 뒷 요솟값보다 작음)일 때 `right`를 1 감소시킨다. `if` 문은 `left >= right`(확인할 요소가 없음)일 때 반복을 중단한다. 그 외에는 매 반복마다 기준값보다 큰 `a[left]`와 작은 `a[right]`를 교환한다.

`sortArray`는 배열 `a`, 정렬 기준 위치 `start`, 정렬할 끝 요소 인덕스 `end`를 인수로 받고, 기준값 인덕스 `pivot`을 선언한다. `start < end`일 때 `divideArray`로 그룹을 나눈 뒤, 앞쪽 그룹에 `sortArray(a, start, pivot - 1)`, 뒤쪽 그룹에 `sortArray(a, pivot + 1, end)`를 실행한다. 재귀 호출이므로 `while` 문 없이도 반복 처리가 가능하다.
