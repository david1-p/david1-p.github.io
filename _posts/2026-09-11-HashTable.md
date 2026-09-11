---
title: "해시 테이블 탐색법 – 해시 충돌 대처 (Java)"
date: 2026-09-11 09:00:00 +0900
categories: [Algorithm, Java]
tags: [hash-table, java, algorithm]
---

## 해시 충돌에 대처하도록 수정한 자바 프로그램

해시값이 같은 데이터가 들어오면(해시 충돌) 다음 배열 요소로 한 칸씩 이동하면서 빈자리를 찾는다. 배열 끝에 닿으면 첫 번째 요소로 돌아가고, 해시값 위치까지 한 바퀴 돌아오면 테이블이 가득 찬 것으로 본다.

```java
import java.util.Scanner;

public class HashTableSearchSyn {
    // 해시 테이블 실체가 되는 배열(요소 수를 10개로 함)
    public static int[] hashTable = { -1, -1, -1, -1, -1, -1, -1, -1, -1, -1 };

    // 해시 함수 역할을 하는 메소드
    public static int hashFunc(int data) {
        return data % 10;
    }

    // 프로그램 실행의 시작점인 main 메소드
    public static void main(String[] args) {
        int data, hashValue;
        int pos; // 저장 위치, 검색 위치

        // 키보드로 데이터를 입력하여 해시 테이블에 저장
        Scanner scn = new Scanner(System.in);
        do {
            // 저장할 데이터 입력
            System.out.printf("\n저장할 데이터 = ");
            data = scn.nextInt();

            // 음숫값이 입력되면 데이터의 저장을 종료
            if (data < 0) {
                break;
            }

            // 해시값을 구함
            hashValue = hashFunc(data);

            // 데이터의 저장 위치를 정함
            pos = hashValue;
            while (hashTable[pos] != -1) {
                // 다음 배열 요소에서 데이터를 저장할 수 있는지 확인
                pos++;

                // 배열 마지막 요소까지 데이터 저장 가능 여부를 확인하면 배열 첫 번째 요소를 지정
                if (pos >= hashTable.length) {
                    pos = 0;
                }

                // 해시값의 배열 요소 위치까지 돌아오면,
                // 해시 테이블에 데이터가 가득 찬 것이므로 반복을 종료
                if (pos == hashValue) {
                    break;
                }
            }

            if (hashTable[pos] == -1) {
                // 해시 테이블에 데이터가 가득 차지 않았다면 데이터를 저장
                hashTable[pos] = data;
            } else {
                // '해시 테이블이 가득 찼습니다'를 표시
                System.out.printf("해시 테이블이 가득 찼습니다.\n");
            }
        } while (true);

        // 해시 테이블에서 데이터를 탐색
        do {
            // 키보드로 탐색할 데이터를 입력
            System.out.printf("\n검색할 데이터 = ");
            data = scn.nextInt();

            // 음숫값이 입력되면 데이터 탐색을 종료
            if (data < 0) {
                break;
            }

            // 해시값을 구함
            hashValue = hashFunc(data);

            // 데이터를 탐색
            pos = hashValue;
            while (hashTable[pos] != -1 && hashTable[pos] != data) {
                // 다음 배열 요소로 탐색 위치를 이동
                pos++;

                // 배열 마지막 요소까지 탐색하면 배열 첫 번째 요소를 지정
                if (pos >= hashTable.length) {
                    pos = 0;
                }

                // -1을 찾았거나, 해시값의 인덱스 위치로 돌아오면,
                // 데이터를 찾을 수 없는 것이므로 반복을 종료
                if (hashTable[pos] == -1 || pos == hashValue) {
                    break;
                }
            }

            // 탐색한 결과를 표시
            if (hashTable[pos] == data) {
                System.out.printf("%d번째에서 발견되었습니다.\n", pos);
            } else {
                System.out.printf("찾을 수 없습니다.\n");
            }
        } while (true);
        scn.close();
    }
}
```

### 코드 설명

데이터의 저장 위치를 찾는 부분에는 기존 `do~while` 문 안에 데이터 저장 위치를 정하는 `while` 문이 추가되었다.

`while` 문은 `hashTable[pos] != -1`이라는 조건일 때 다음 배열 요소에 데이터를 저장 가능한지 확인한다. 이를 위해 배열 마지막 요소보다 큰 인덕스 숫자일 때 배열 첫 번째 요소를 가리키도록 하는 `if` 문, 현재 확인 중인 인덱스와 해시값이 가리키는 인덱스가 같을 때 반복을 종료하는 `if` 문이 있다.

## 해시 충돌을 해결하는 해시 테이블의 추적 코드

위 프로그램에 저장 위치·탐색 위치가 어떻게 변하는지 출력하는 추적 코드를 추가한 것이다. 추가된 줄은 모두 `System.out.printf`로, 로직은 그대로다.

```java
import java.util.Scanner;

public class HashTableSearchSynTrace {
    // 해시 테이블의 실체가 되는 배열(요소 수를 10개로 함)
    public static int[] hashTable = { -1, -1, -1, -1, -1, -1, -1, -1, -1, -1 };

    // 해시 함수 역할을 하는 메소드
    public static int hashFunc(int data) {
        return data % 10;
    }

    // 프로그램 실행의 시작점인 main 메소드
    public static void main(String[] args) {
        int data, hashValue;
        int pos;    // 저장 위치, 검색 위치

        // 키보드로 데이터를 입력하여 해시 테이블에 저장
        Scanner scn = new Scanner(System.in);
        do {
            // 저장할 데이터 입력
            System.out.printf("\n저장할 데이터 = ");
            data = scn.nextInt();

            // 음숫값이 입력되면 데이터 저장을 종료
            if (data < 0) {
                break;
            }

            // 해시값을 구함
            hashValue = hashFunc(data);
            System.out.printf("해시값 = %d %% 10 = %d\n", data, hashValue);

            // 데이터의 저장 위치를 정함
            pos = hashValue;
            System.out.printf("저장 위치 pos = %d\n", pos);
            while (hashTable[pos] != -1) {
                // 다음 배열 요소에서 데이터를 저장할 수 있는지 확인
                pos++;

                // 배열 마지막 요소까지 데이터 저장 가능 여부를
                // 확인하면 배열 첫 번째 요소를 지정
                if (pos >= hashTable.length) {
                    pos = 0;
                }
                System.out.printf("저장 위치 pos = %d\n", pos);

                // 해시값의 배열 요소 위치까지 돌아오면,
                // 해시 테이블에 데이터가 가득 찬 것이므로 반복을 종료
                if (pos == hashValue) {
                    break;
                }
            }

            if (hashTable[pos] == -1) {
                // 해시 테이블이 가득 차지 않았다면 데이터를 저장
                hashTable[pos] = data;
                System.out.printf("hashTable[%d]에 %d을(를) 저장합니다.\n", pos, data);
            } else {
                // '해시 테이블이 가득 찼습니다'를 표시
                System.out.printf("해시 테이블이 가득 찼습니다.\n");
            }
        } while (true);

        // 해시 테이블에서 데이터를 탐색
        do {
            // 키보드로 탐색할 데이터를 입력
            System.out.printf("\n탐색할 데이터 = ");
            data = scn.nextInt();

            // 음숫값이 입력되면 데이터 탐색을 종료
            if (data < 0) {
                break;
            }

            // 해시값을 구함
            hashValue = hashFunc(data);
            System.out.printf("해시값 = %d %% 10 = %d\n", data, hashValue);

            // 데이터를 탐색
            pos = hashValue;
            System.out.printf("탐색 위치 pos = %d\n", pos);
            while (hashTable[pos] != -1 && hashTable[pos] != data) {
                // 다음 배열 요소로 탐색 위치를 이동
                pos++;

                // 배열 마지막 요소까지 탐색하면 배열 첫 번째 요소를 지정
                if (pos >= hashTable.length) {
                    pos = 0;
                }
                System.out.printf("탐색 위치 pos = %d\n", pos);

                // -1을 찾았거나, 해시값의 인덱스 위치로 돌아오면,
                // 데이터를 찾을 수 없는 것이므로 반복을 종료
                if (hashTable[pos] == -1 || pos == hashValue) {
                    break;
                }
            }

            // 탐색한 결과를 표시
            if (hashTable[pos] == data) {
                System.out.printf("hashTable[%d]값은 %d이므로, 발견한 위치를 표시합니다.\n",
                                  pos, data);
                System.out.printf("%d번째에서 발견되었습니다.\n", pos);
            } else {
                System.out.printf(
                    "hashTable[%d]값은 %d이므로, '찾을 수 없습니다.'를 표시합니다.\n",
                                  pos, hashTable[pos]);
                System.out.printf("찾을 수 없습니다.\n");
            }
        } while (true);
        scn.close();
    }
}
```

### 코드 설명

- `"저장 위치 pos = %d\n", pos`와 `"탐색 위치 pos = %d\n", pos`는 데이터의 저장 위치와 탐색 위치가 어떻게 변하는지 추적하는 코드다. 바로 저장될 수도 있고 `while` 문을 돌며 여러 번 위치를 확인할 수도 있으므로, 시작 위치 직후와 `while` 문 안 두 곳에 두었다.
- `"hashTable[%d]에 %d을(를) 저장합니다.\n", pos, data`는 실제로 저장하는 위치를 확인하는 코드이므로 `while` 문 밖 `if~else`의 `if` 쪽에 두었다.
- 데이터 탐색에서는 현재 해시값을 확인하려고 `"해시값 = %d %% 10 = %d\n", data, hashValue`를 사용한다. (`printf`에서 `%`를 그대로 출력하려면 `%%`로 써야 한다.)
- 최종 탐색 결과를 표시할 때는 데이터를 찾았을 때와 못 찾았을 때 각각의 이유를 알리기 위해 `hashTable[pos]` 값을 함께 출력한다.
