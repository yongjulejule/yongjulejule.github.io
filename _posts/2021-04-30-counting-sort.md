---
layout: post
title: "Sort 01. Counting Sort, Radix Sort, Lower Bounds for Sorting (계수 정렬, 기수 정렬, 정렬 알고리즘의 Lower bound)"
comments: true
excerpt_separator: "<!--more-->"
categories: 
- Algorithms
tags:
- Algorithms
- MIT OCW
- Decision Tree
- Sort
order: 2
date: '2021-04-16 10:47:00 +0900'
last_modified_at: '2021-04-18 17:55:00 +0900'


---

> _MIT introduction to Algorithm (6.006) Lecture 7을 기반으로 정리한 내용입니다._
>
> 데이터를 비교 및 정렬하는 알고리즘을 위한 ADT를 살펴보고, 각 알고리즘의 Lower Bound를 살펴보자. 그리고 적은 양의 정수로 이루어진 데이터에 대하여, $O(n)$ 의 시간복잡도를 지니는 계수 정렬과 기수 정렬에 대해 살펴보자. 

<!--more-->

힙 정렬, 병합 정렬이나 퀵 정렬은 흔히 사용되며 $\Omega(nlgn)$ 의 시간복잡도를 가진다. 이런 정렬들은 공통적인 특징을 가지는데, 바로 입력된 두 데이터를 비교하면서 정렬을 한다는 것이다. 이런 정렬 알고리즘을 *Comparison Sorts*라고 하자. 이 게시글에서 어떤 Comprision Sorts라도 최악의 경우 $\Omega(nlgn)$ 의 시간복잡도를 지닌다는 것을  증명할 것이다.  

# Decision Tree(의사 결정 트리)

Comprision Sort 알고리즘은 의사 결정 트리로 나타낼 수 있으며, 의사 결정 트리는 모든 데이터와 특정한 값 n을 비교하여 결과를 낼 수 있다.  예를 들어, 3개의 원소에 대한$(n = 3)$ 의사 결정 트리는 다음과 같이 구성된다.

![Decision Tree]()

`x`는 새로운 데이터이고, `A[]`는 입력 되어 있던 데이터라고 하자. 그림과 같이 트리의 구조로 탐색을 할 수 있으며 각 `leaf`는 모든 데이터와 대소 비교 한 결과가 된다.

## Lower Bound

**탐색의 Lower Bound**

n개의 데이터가 이미 정렬되어 있을 때 어떤 데이터 x를 찾는 알고리즘은 $\Omega(lgn)$ 이 된다.(Lower Bound = $lgn$)

> 간단한 증명(?)
>
> n개의 데이터가 정렬되어 있으면, n개의 노드를 가진 의사 결정 트리가 되고, 트리의 leaf를 찾아가야 하므로 트리의 높이만큼의 시간이 걸릴 것이다. 의사 결정 트리는 이진 트리 이므로 높이는 $lgn$ 이 된다. 

**정렬의 Lower Bound**

세개의 데이터를 정렬하는 의사 결정 트리는 조금 더 복잡하게 생겼는데, 3가지 원소{1, 2, 3}을 정렬하는 경우 다음과 같은 트리를 이용하여 경우의 수 들을 찾을 수 있다.

![Decision Tree Sort]()

이와 같이 모든 경우의 수를 고려한다면 하나의 순열이 되므로 $n!$ 이 되며 여기서는 3개의 원소가 있으므로 $3!$ 개가 나온다. 

결국 $n!$개의 데이터를 갖는 하나의 트리로 생각할 수 있으며 $height \ge lg(n!)$ 가 된다. 이를 로그의 성질에 따라 잘 분해한 뒤 계산하면 $\Omega(nlgn)$ (=Lower Bound)가 된다.

> 증명) 
>
> *스털링 근사 $n! \sim \sqrt{2\pi n}(n/e)^n$ 을 이용하면 더 깔끔하지만, 이에 대해 제대로 배운 적이 없으니 다른 방법으로 증명한다...*
>
> $lg(n!)$
>
> $=lg1+lg2+...+lg(n-1)+lg(n)$ (로그의 성질)
>
> $=\sum_\limits{i=1}^\limits{n}lg(i) ~ (where ~i = integer)$ 
>
> (시그마 표현으로 전환)
>
> $\ge \sum_\limits{i=n/2}^\limits{n}lg(i)$ 
>
> (로그이므로 모든 수는 양수이므로 $i=1 \sim n$ 까지 더하는 것 보다 $n/2 \sim n$ 까지 더하는게 작다)
>
> $\ge \sum_\limits{i=n/2}^\limits{n}lg(n/2) = \sum_\limits{i=n/2}^\limits{n}(lg(n)-1)$  
>
> (로그는 단조증가함수 이므로 $lg(n/2) + lg(n/2 + 1) + ... + lg(n) \ge lg(n/2) + lg(n/2) + ... +lg(n/2)$) 이고, $lg(n/2)$ 을 로그의 성질에 따라 풀어서 쓰면 우변의 식이 나온다)
>
> $\sum_\limits{i=n/2}^\limits{n}(lg(n)-1) = n/2 * lg(n) - n/2$  이고, $lg(n!)$ 가 항상 이 값보다 크므로 
>
> $\Omega(nlgn) \blacksquare$ 

이렇게 정렬은 최소한 $nlgn$ 의 시간복잡도를 갖지만, 데이터의 수가 너무 많지 않고 (not too big) 모두 $integer$ 라는 가정 하에 $O(n)$  시간 내에 정렬을 수행 할 수 있는 알고리즘이 있다. 이에 대해 알아보자.

# Linear Time Sort 

## Counting Sort (계수 정렬)

계수 정렬은 세개의 배열을 필요로 하는데, 원본 데이터를 갖고 있는 `A[1...n]` 와 정렬된 결과를 나타낼 `B[1...n]` 그리고 정렬을 위해 임시로 값을 저장할 `C[0...k]` 가 필요하다. 이때 원본 데이터는 0부터 k사이의 숫자여야 한다. (계수 정렬의 가정!!) 이러한 성질을 갖는 데이터가 있다면, 원본 데이터를 `C[]` 의 인덱스로 하고 `C[]` 의 값에 각 인덱스에 해당하는 데이터의 개수를 저장한 후 `B[]`에 순서대로 넣으면 성공적으로 정렬 된다는 논리로 진행된다. Pseudo Code와 그림을 통해 이해해보자.

## Pesudo Code

```c
Counting-Sort(A, B, k)
  C[k] //C를 정의
  for i = 0 to k //C 초기화
    C[i] = 0 
  for j = 1 to A.length //C의 A[j]번째 인덱스마다 한번씩 ++ 해줌
    C[A[j]]++ //C[i]는 i값을 가지는 데이터가 몇 개 있는지 저장함
  for i = 1 to k
    C[i] = C[i] + C[i-1] // C[i]는 이제 i보다 같거나 작은 데이터들의 수를 저장함
  for j = A.length downto 1 //마지막 반복문은 밑에서 설명한다.
    B[C[A[j]]] = A[j]
    C[A[j]] = C[A[j]] - 1
```

Line 9 ~ 11의 반복문은 `A[]` 의 각 원소들을 `B[]` 에 올바르게 정렬하여 넣어준다. 만약 모든n(=`A.length`)개의 원소들이 중복되지 않는다면(distinct 하다면), 각 `A[j]`에 대하여 `C[A[j]]`의 값은 output 배열 `A[j]`의 올바른 자리이다. 왜냐하면 `A[j]`보다 같거나 작은 원소들이 `C[A[j]]`에 있기 때문이다. 이 원소들이 중복될 수 있기 때문에 C[A[j]]을 매번 감소시키며 A[j]의 값을 B[j]에 넣는다. C[A[j]]를 감소시키는것은 A[j]와 같은 값을 가진 다음 인풋 원소가, 만약 존재한다면 바로 그 위치로 간다. A[j]가 인풋 배열이 되기 전에.

Input 배열 `A[1...8]` 에 대해 계수 정렬을 하는 과정이다. A의 원소는 모두 5보다 같거나 작은(k = 5) 양의 정수이다. 

(a) Line 6 이후 배열 A와  C의 모습이다. 0이 2개 있어서 C[0] = 2, 1이 0개 있으니 C[1] = 0, 3이 3개 있으니 C[3] = 3이 되는 구조이다.

(b) Line 8 이후 배열 C의 모습이다. 피보나치 수열처럼 현재 값 + 이전 값이 다음 위치에 들어간다.

(c) ~ (e) Line 9 ~ 11을 3번 반복하는 B와 C의 모습이다. 진한 회색으로 표시된 부분은 아직 값이 삽입되지 않은 부분이다. 

(f)는 반복문을 마치고 정렬된 결과인 B를 나타낸다. 








