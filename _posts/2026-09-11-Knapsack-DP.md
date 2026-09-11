---
title: "동적 계획법으로 배낭 문제 풀기 (Java)"
date: 2026-09-11 09:20:00 +0900
categories: [Algorithm, Java]
tags: [dynamic-programming, knapsack, java, algorithm]
---

## 문제

최대 6kg까지 담을 수 있는 배낭에 물건 A~E(무게 1~5kg, 가치 100~650원) 중 어떤 것을 넣어야 가치의 합계가 최대가 되는가. 동적 계획법(DP)으로 "i번째 물건까지 고려했을 때 k kg 배낭의 최대 가치"라는 부분 문제를 채워 나가며 푼다.

## 자바

```java
public class KnapsackDP {
    // 배낭의 최대 무게
    public static final int KNAP_MAX = 6;

    // 물건의 종류
    public static final int ITEM_NUM = 5;

    // 물건의 명칭
    public static char[] name = { 'A', 'B', 'C', 'D', 'E' };

    // 물건의 무게
    public static int[] weight = { 1, 2, 3, 4, 5 };

    // 물건의 가치
    public static int[] value = { 100, 300, 350, 500, 650 };

    // 물건을 넣을지 판단한 직후의 최대 가치
    public static int[][] maxValue = new int[ITEM_NUM][KNAP_MAX + 1];

    // 마지막에 넣은 물건
    public static int[] lastItem = new int[KNAP_MAX + 1];

    // item번째 물건을 넣을지 판단한 직후 배낭의 내용을 표시하는 메소드
    public static void showKnap(int item) {
        int knap;   // 0~6kg의 배낭을 가리킴

        // 넣을지 말지 판단할 물건의 정보를 표시
        System.out.printf("<%c, %dkg, %d원을 고려한 결과>\n",
                          name[item], weight[item], value[item]);

        // 각 배낭의 무게를 표시
        for (knap = 0; knap <= KNAP_MAX; knap++) {
            System.out.printf("%dkg\t", knap);
        }
        System.out.printf("\n");

        // 배낭에 담긴 상품 가치의 합계를 표시
        for (knap = 0; knap <= KNAP_MAX; knap++) {
            System.out.printf("%d원\t", maxValue[item][knap]);
        }
        System.out.printf("\n");

        // 배낭에 마지막으로 넣은 물건을 표시
        for (knap = 0; knap <= KNAP_MAX; knap++) {
            if (lastItem[knap] != -1) {
                System.out.printf("%c\t", name[lastItem[knap]]);
            } else {
                System.out.printf("없음\t");
            }
        }
        System.out.printf("\n\n");
    }

    // 프로그램 실행의 시작점인 main 메소드
    public static void main(String[] args) {
        int item;           // 물건 번호
        int knap;           // 0~6kg의 배낭을 가리킴
        int selVal;         // 임시로 물건을 선택한 경우의 가치 합계
        int totalWeight;    // 중량의 합계

        // 0번째 물건을 넣을지 판단
        item = 0;
        // 0~KNAP_MAX kg의 배낭을 고려
        for (knap = 0; knap <= KNAP_MAX; knap++) {
            // 최대 무게 이하면 선택
            if (weight[item] <= knap) {
                maxValue[item][knap] = value[item];
                lastItem[knap] = item;
            }
            // 최대 무게 이하가 아니면 선택하지 않음
            else {
                maxValue[0][knap] = 0;
                lastItem[knap] = -1;
            }
        }
        showKnap(item);

        // 1번째~ITEM_NUM-1번째 물건을 고려
        for (item = 1; item < ITEM_NUM; item++) {
            // 0kg~KNAP_MAX kg의 배낭을 고려
            for (knap = 0; knap <= KNAP_MAX; knap++) {
                // 최대 무게 이하의 경우
                if (weight[item] <= knap) {
                    // 선택한 경우의 가치를 구함
                    selVal = maxValue[item - 1][knap - weight[item]] + value[item];
                    // 가치가 크면 선택
                    if (selVal > maxValue[item - 1][knap]) {
                        maxValue[item][knap] = selVal;
                        lastItem[knap] = item;
                    }
                    // 가치가 크지 않으면 선택하지 않음
                    else {
                        maxValue[item][knap] = maxValue[item - 1][knap];
                    }
                }
                // 최대 무게 이하가 아니면 선택하지 않음
                else {
                    maxValue[item][knap] = maxValue[item - 1][knap];
                }
            }
            showKnap(item);
        }

        // 배낭에 들어 있는 물건을 조사하여 정답을 표시
        System.out.printf("<배낭에 들어 있는 물건을 조사>\n");
        totalWeight = 0;
        for (knap = KNAP_MAX; knap > 0; knap -= weight[item]) {
            item = lastItem[knap];
            System.out.printf("%dkg의 배낭에 마지막으로 넣은 물건은 %c입니다.\n",
                              knap, name[item]);
            System.out.printf(" %dkg - %dkg = %dkg입니다.\n",
                              knap, weight[item], knap - weight[item]);
            totalWeight += weight[item];
        }
        System.out.printf("\n<정답을 표시>\n");
        System.out.printf("무게의 합계 = %dkg\n", totalWeight);
        System.out.printf("가치의 최댓값 = %d원\n", maxValue[ITEM_NUM - 1][KNAP_MAX]);
    }
}
```

## 코드 설명

프로그램은 배낭의 최대 무게, 물건의 종류·명칭·무게·가치 등을 정의한 필드, `item`번째 물건을 넣을지 판단한 직후 배낭의 내용을 표시하는 `showKnap` 메소드, 그리고 `main` 메소드로 구성된다. 배낭 문제를 푸는 구조를 설명하기 위한 최소한의 프로그램이다.

- **부분 문제의 정답**은 2차원 배열 `maxValue[item][knap]`에 저장한다. 배낭 각각에 마지막으로 넣은 물건은 `lastItem[knap]`에 저장한다. `showKnap`은 물건을 넣을지 판단한 직후의 `maxValue[][]` 내용을 표시한다.
- `showKnap`은 `for` 문 3개로 배낭 모두(0~6kg)를 훑는다. 배낭 안의 물건은 숫자가 아니므로 `if~else`로 출력할 값을 정하고, 물건이 없을 때(0kg 배낭)는 `없음`을 출력한다.
- `main`은 먼저 **0번째 물건 A**를 배낭마다 넣는 처리를 따로 구성한다. 비교할 이전 결과가 없기 때문이다.
- 그 뒤 다중 반복문으로 물건을 하나씩 늘리면서, 배낭 최대 무게 이하인지(`weight[item] <= knap`) 확인한다. 이하이면 이 물건을 넣은 가치 `selVal = maxValue[item - 1][knap - weight[item]] + value[item]`과 이전 물건까지의 최댓값 `maxValue[item - 1][knap]`을 비교해 더 큰 쪽을 `maxValue[item][knap]`에 저장한다. 최대 무게를 넘으면 이전 결과를 그대로 가져온다.
- 마지막에는 이 문제에서 구하려는 6kg 배낭의 가치 최댓값 `maxValue[ITEM_NUM - 1][KNAP_MAX]`을 표시한다. 배낭에 여러 물건이 들어 있을 수 있으므로 `for` 문으로 배낭에 든 물건을 모두 확인한다. 마지막으로 넣은 물건이 무엇인지 `lastItem[knap]`으로 찾고, 현재 배낭 무게에서 그 물건의 무게를 빼 남은 배낭(예: 6kg − 4kg = 2kg)을 다시 조사한다. 뺀 값이 0이 되면 종료하고, 각 물건 무게의 합계와 가치의 최댓값을 표시한다.
