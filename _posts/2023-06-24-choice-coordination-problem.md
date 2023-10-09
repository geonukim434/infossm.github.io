---
layout: post
title: 'Choice Coordination Problem'
author: junis3
date: 2023-06-24
tags: [random]
---

# 개요

## 문제 상황

$n$개의 프로세서가 존재한다. 각 프로세서는 $m$개의 선택지 중 하나를 선택해야 한다. 그런데, 프로세서들이 완전히 비동기적으로 작동한다. 프로세서가 공유하는 일관적인 시계 (global clock)이 존재하지도 않을 뿐더러, 프로세서의 응답 속도에 대한 가정을 할 수도 없다. 이러한 상황에서 모든 프로세서가 동일한 선택으로 합의하도록 하는 알고리즘이 필요하다.

인격을 부여해 이러한 비유 상황을 만들어낼 수 있다: $n$명의 관광객이 존재한다. 그들이 머무는 여행지는 $m$개의 방으로 이루어져 있다. 이 때 관광객 모두가 방 하나에 모일 수 있도록 행동 규약을 잘 정해야 한다.

만약 문제에서 프로세서가 하나 뿐이거나, 여러 개더라도 주종관계가 존재한다면 문제가 쉬워졌을 것이다. 주 프로세서 하나가 선택지에 번호를 붙인 다음 그 중 아무거나 하나를 고르면 나머지 모든 프로세서가 이를 따르면 된다. 그러나 애석하게도 n개의 프로세서는 서로 구분할 수 없으며, m개의 선택지도 서로 구분할 수 없다. 불가능할 것 같은 문제이지만, 이를 효율적으로 푸는 알고리즘이 존재한다.

## 재료

먼저 용어를 정의하자. $n$개의 프로세서를 $P _ 0, \cdots, P _ {n-1}$라고 부르자. 또한 $m$개의 선택지를 $C _ 0, \cdots, C _ {m-1}$라고 부르자. 물론, 이것은 문제를 푸는 우리가 각 프로세서와 선택지에 임의로 붙인 번호이다.

문제를 풀기 위해 $m$개의 레지스터 (전역 변수)를 선언할 것이다. 즉, 선택지의 개수만큼의 메모리를 사용할 것이다. 레지스터를 $R _ 0, \cdots, R _ {m-1}$이라고 부를 것이며, 레지스터 $R _ i$는 선택지 $C _ i$에 대응된다.

이 글에서 $m$개의 레지스터 가운데 정확히 하나에 $\star$ 표시를 하는 분산 알고리즘을 구성할 것이다. 알고리즘이 성공적으로 실행되었다면, 정확히 하나의 레지스터가 $\star$ 값을 가지고 나머지 모든 레지스터는 다른 값을 가지고 있을 것이다. 그 뒤, $n$개의 프로세서는 $\star$ 표시된 레지스터에 대응되는 선택지로 합의할 수 있다.

여러 개의 프로세서가 비동기적으로 작동하므로, 레지스터에 동시에 접근 및 수정을 시도할 가능성이 있다. 이 글에서 다루는 알고리즘에서는 lock 메커니즘을 이용해 충돌이 발생하지 않도록 제어한다.

# 알고리즘

이 문제를 푸는 어떤 결정론적 알고리즘이든 최소 $\Omega(n^{1/3})$의 연산이 필요하다는 것이 증명되어 있다.

반면, 랜덤의 힘을 빌리면, 임의의 상수 $c > 0$에 대해 $c$번 이하의 연산으로 합의를 이끌어낼 수 있는 확률이 $1 - 2^{-\Omega(c)}$인 알고리즘을 구성할 수 있다. 먼저 $n = m = 2$인 경우만 고려하고 문제를 풀어 보자. 이를 임의의 $m$과 $n$으로 확장하는 것은 직관적으로 할 수 있다.

## 동기적인 경우

먼저 두 프로세서가 동기적으로 작동하는 경우를 다루어 보자. 이 때의 **알고리즘 1**은 비동기적인 경우의 **알고리즘 2**의 원형이 된다.

두 프로세서 $P _ 0$과 $P _ 1$이 각각 $B _ 0$과 $B _ 1$이라는 이름의 지역 변수를 가지고 있다고 하자. 이 알고리즘에서 두 프로세서는 두 레지스터를 서로 바꾸어가면서 바라본다.

---

**알고리즘 1**

- 입력: 0으로 초기화되어 있는 두 레지스터 $R _ 0$과 $R _ 1$
- 출력: 두 레지스터 $R _ 0$과 $R _ 1$ 중 정확히 하나에 $\star$ 표시가 되어 있음

0. 프로세서 $P _ i$는 현재 레지스터 $R _ i$를 보고 있으며, 지역 변수 $B _ i$는 0으로 초기화되어 있다.

1. 레지스터 $R _ i$를 읽는다. $\star$ 표시 또는 하나의 비트 (0 또는 1)이 쓰여 있을 것이다.

2. 아래 세 가지 중 하나를 실행한다.

   2-1. $R _ i = \star$이면, 알고리즘을 종료한다.

   2-2. $R _ i = 0$, $B _ i = 1$이면, 레지스터 $R _ i$에 $\star$ 표시를 하고 알고리즘을 종료한다.

   2-3. 둘 모두 해당되지 않으면, 로컬 변수 $B _ i$에 무작위 비트를 쓰고, 이 값을 레지스터 $R _ i$에 쓴다.

3. 반대편 레지스터로 이동한다. ($R _ 0$과 $R _ 1$를 교환한다.) **반대편 프로세서가 아직 반대편 레지스터를 바라보고 있다면, 끝날 때까지 기다린다.** 1.로 돌아간다.

---

두 프로세서가 동기적이므로 "반대편 프로세서를 기다리는" 작업을 할 수 있으며, 덕분에 "지금이 몇 번째 반복인지" 정의할 수 있다는 점에 주목하라.

이 알고리즘이 올바르게 작동하였다면, 알고리즘이 종료된 뒤 $\star$ 표시가 된 레지스터가 정확히 하나 존재할 것이다. 알고리즘이 올바르게 작동함을 보인다.

알고리즘이 종료된 뒤, 두 레지스터 모두 $\star$ 표시가 되어 있다고 가정하자. 이 때 두 $\star$는 같은 반복에서 쓰였어야 한다. 그렇지 않다면, 첫 $\star$가 쓰여진 직후의 반복에서 반대편 프로세서가 레지스터에 쓰인 $\star$을 보고 종료되었을 것이다.

두 $\star$가 $t$번째 반복에서 쓰였다고 가정하자. $t$번째 반복이 끝난 시점에서 로컬 변수 $B _ i$의 값을 $B _ i(t)$, 레지스터 $R _ i$의 값을 $R _ i(t)$로 쓰자. 2-3.에 의해, $R _ 1(t) = B _ 0(t)$과 $R _ 0(t) = B _ 1(t)$이다.

$R _ i = 0$이고 $B _ i = 1$인 경우에 $P _ i$가 $\star$ 표시를 할 것이다. 이 때 반대편 프로세서 $P _ {1-i}$의 로컬 변수 $B _ {1-i} = 0$이고, 반대편 레지스터 $R _ {1-i} = 1$이다. 따라서 $P _ {1-i}$는 2-2.로 진입할 수 없고, 레지스터에 $\star$ 표시를 할 수 없다. 따라서 알고리즘은 두 레지스터 모두에 $\star$ 표시가 되어 있는 상태로 종료할 수 없다.

따라서 알고리즘은 항상 두 레지스터 중 정확히 하나에 $\star$ 표시가 되어 있는 상태로 종료한다.

위 문단에서 이 알고리즘이 종료한 뒤에는 우리가 원하는 상태가 됨을 보였다. 그렇다면, 이 알고리즘이 종료하는 데에는 몇 번의 연산이 필요할까? 무작위로 고른 두 비트 $B _ 0$과 $B _ 1$이 같을 확률은 $\frac{1}{2}$이다. (알고리즘에서 무작위 비트를 고르는 과정은 모두 서로 독립적이다!) 그리고 두 비트가 같은 값을 가지기만 한다면 이 알고리즘은 2번의 반복 안에 종료한다. 따라서, 알고리즘의 반복 횟수가 $t$번을 초과할 확률은 $O\left(\frac{1}{2^t}\right)$이다. 각 반복의 연산량은 상수이므로, 이 알고리즘은 $1 - O\left(\frac{1}{2^t}\right)$의 확률로 $O(t)$의 시간 안에 실행된다.

## 완전히 비동기적인 경우

두 프로세서가 완전히 비동기적으로 작동하는 경우에는 어떻게 할 수 있을까? 이 때에는 각 반복을 끝내고 두 레지스터를 교환하는 행위를 할 수 없다. 각 프로세서가 서로 다른 레지스터에서 시작한다는 가정 자체를 할 수 없을 것이다. 이 가정 자체가 Choice coordination problem이기 때문이다. 대신, 각 프로세서가 무작위 레지스터에서 알고리즘을 시작한다고 가정할 수밖에 없다. 이렇게 하면 필연적으로 충돌이 발생할 가능성이 생기는데, 이는 lock 메커니즘을 이용해 해결한다. 이에 대한 세부사항은 생략한다.

기본적인 아이디어는, 각 프로세서의 지역 변수 $B _ i$가 마지막으로 수정된 시각 $T _ i$와 각 레지스터 $R _ j$가 마지막으로 수정된 시각 $t _ j$를 기록하는 것이다. 즉, $R _ j$를 읽으면 마지막 수정 시각과 레지스터의 값으로 이루어진 쌍 $\left< t _ j, R _ j \right>$를 얻을 수 있다. **알고리즘 1**과 같이 동기적으로 실행하는 상황이었다면, $t _ i$의 값은 "몇 번 반복했는지"를 나타내는 값 $t$와 같을 것이다.

---

**알고리즘 2**

- 입력: 0으로 초기화되어 있는 두 레지스터 $R _ 0$과 $R _ 1$
- 출력: 두 레지스터 $R _ 0$과 $R _ 1$ 중 정확히 하나에 $\star$ 표시가 되어 있음

0. 프로세서 $P _ i$는 현재 무작위로 고른 레지스터 $R _ j$를 읽고 있다. 아래 과정을 수행하고 난 뒤 $P _ i$는 $R _ j$에 새로운 값을 쓸 것이다. 지역 변수 $T _ i$와 $B _ i$는 0으로 초기화되어 있다.

1. $P _ i$가 레지스터 $R _ j$를 읽기 시작한다. 먼저 레지스터를 lock한 다음, 값 $\left<t _ j, R _ j \right>$를 읽는다.

2. 아래 다섯 가지 중 하나를 실행한다.

   2-1. $R _ j = \star$이면, 알고리즘을 종료한다.

   2-2. $T _ i < t _ j$이면, 두 변수 $T _ i$와 $B _ i$에 값 $t _ j$와 $R _ j$를 대입한다. ($T _ i \leftarrow t _ j$, $B _ i \leftarrow R _ j$)

   2-3. $T _ i > t _ j$이면, $R _ j$에 $\star$ 표시를 하고 알고리즘을 종료한다.

   2-4. $T _ i = t _ j, R _ j = 0, B _ i = 1$이면, $R _ j$에 $\star$ 표시를 하고 알고리즘을 종료한다.

   2-5. 넷 모두 해당되지 않으면, $T _ i \leftarrow T _ i + 1$, $t _ j \leftarrow t _ j + 1$을 수행한 다음, 무작위 비트를 골라 $B _ i$에 쓰고, 이 값을 레지스터 $R _ j$에 쓴다($R _ j \leftarrow \left<t _ j, B _ j\right>$).

3. 레지스터 $R _ j$를 unlock한다. 다시 무작위로 레지스터를 골라 해당 레지스터로 이동한다. 이 레지스터가 lock되어 있으면 레지스터를 다시 고른다. 1.로 돌아간다.

---

이제, **알고리즘 2**도 올바르게 작동한다는 것, 그리고 **알고리즘 1**과 같이 $1 - O\left(\frac{1}{2^t}\right)$의 확률로 $O(t)$의 시간에 작동함을 보일 것이다.

실질적으로 두 알고리즘의 차이는 2-2.과 2-3.이다. 2-2.는 다른 프로세서의 작업을 따라잡는 일이고, 2-3.은 다른 프로세서보다 자신이 앞서있다는 것을 깨닫고 마음대로 선택하는 것이다. 알고리즘이 올바르게 작동한다는 것을 보이기 위해, 프로세서가 $\star$ 표시를 할 수 있는 두 가지 경우인 2-3.과 2-4.을 생각한다. 반복이 끝날 때, 프로세서의 타임스탬프 $T _ i$는 레지스터의 타임스탬프 $t _ i$와 같다. 그리고 두 프로세서 모두 첫 반복에서는 $\star$ 표시를 할 수 없다.

$P _ i$가 타임스탬프 $T _ i^ * $인 상태로 2-3.에 진입했다고 가정해보자. 이 때 레지스터 $C _ j$의 타임스탬프 $t _ j^ * $는 $t _ j^ *  < T _ i^ * $를 만족할 것이다. 알고리즘에 문제가 생길 수 있는 유일한 경우는 반대편 프로세서 $P _ {1-i}$가 동시에 실행되면서 레지스터 $C _ {1-j}$에 $\star$ 표시를 하는 경우이다.

지금 $P _ i$가 타임스탬프 $T _ i^ * $를 가지고 레지스터 $C _ j$를 보고 있으므로, 반대편 레지스터 $C _ {1-j}$의 타임스탬프 값은 $P _ {1-i}$가 $\star$ 표시를 하기 전의 값을 가지고 있을 것이다. 타임스탬프는 항상 단조증가하므로, $t _ {1-j}^ \* \ge T _ i^ \* $이다.

그리고, 이 시점에서 반대편 프로세서 $P _ {1-i}$의 타임스탬프 $T _ {1-i}^ * $는 $t _ j^ * $를 초과할 수 없다. 왜냐하면 $P _ {1-i}$는 $C _ j$에서 $C _ {1-j}$로 가야 하고, 이 레지스터의 타임스탬프는 항상 $t _ j$ 이하이기 때문이다.

따라서 $T _ {1-i}^ *  \le t _ j^ *  < T _ j^ *  \le t _ {1-i}^ * $가 성립한다. 이 시점에서 프로세서 $P _ {1-i}$가 $C _ {1-j}$에 $\star$ 표시를 했다고 가정해 보자. 직전에 $P _ {1-i}$는 2-2.를 들러야 한다. 이 때 $T _ {1-i}^ *  < t _ {1-j}^ * $인 상태였을 수밖에 없는데, 이는 앞서 보인 내용과 모순이다. 따라서, $P _ i$가 $\star$를 쓰는 동안 $P _ {1-i}$도 $\star$를 쓰는 일은 일어날 수 없으며, 알고리즘은 올바르게 작동한다.

이제 알고리즘의 시간 복잡도를 보이는 것은 쉽다. 수행 시간은 알고리즘이 실행되는 동안 기록된 타임스탬프의 최댓값에 비례한다. 레지스터의 타임스탬프는 2-5.에서만 증가한다. 그리고 2-5.는 2-4.가 실행되지 못했을 때에만 실행된다. 프로세서 $P _ i$의 타임스탬프가 증가한 다음에는 무작위 비트를 $B _ i$에 쓰고 다른 레지스터를 찾아 떠나게 된다. 따라서, **알고리즘 1**에서 했던 분석을 그대로 써먹을 수 있다. 물론 최악의 경우에는 무한대의 시간이 소요될 수 있지만, 소요 시간의 기댓값이 $O(1)$인 Las Vegas 알고리즘이 된다.

# 결론

두 단계에 거쳐, Choice Coordination Problem을 푸는 Las Vegas Algorithm을 구성하였다. 이는 임의의 $n$과 $m$에 대해 풀 수 있도록 확장 가능하다. $n$이 증가할 때에는 시간 복잡도와 모든 분석이 유지되는데, $m$이 증가할 때에는 조금 더 복잡하다. 이러한 분산 합의 알고리즘에는 다양한 응용이 나오고 있으며, 일부 프로세서의 실패 가능성을 고려한 응용이 포함된다.

# 참고문헌

1. Rajeev Motwani and Prabhakar Raghavan, Randomized Algorithms, 1995
2. M. Rabin., The Choice Coordination Problem., 1982