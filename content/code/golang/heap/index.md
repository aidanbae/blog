---
title: "golang과 자료구조 힙(heap)"
date: 2018-11-17T09:31:27+09:00

categories: ['Code', 'Golang']
tags: ['golang', 'heap', '우선순위 큐']
author: "Aidan.bae"
draft : false
noSummary: false
---

마크다운문서가 테마때문에 잘 적용되지 않아 수정할 예정입니다. 다음번 업데이트에 테마를 고칠예정이니 불편해도 양해부탁드려요

# 자료구조 힙

그래프의 트리 구조중 하나로 '우선순위 큐(priority queue)'를 구현할 때 사용됩니다. 우선순위 큐는 데이터 구조의 하나로서 데이터를 자유롭게 추가할 수 있습니다. 반면 데이터를 추출할 때는 최솟값부터 순서대로 선택됩니다. 추가는 자유롭게하고 추출할 때는 작은 값부터 꺼내는 것이 우선순위 큐입니다.



### 특징

- 힙을 표현하는 트리 구조에서는 각 정점을 '노드'라고 부릅니다.

- 자식 노드의 숫자는 반드시 부모의 숫자보다 커야한다는 규칙이 있습니다.

따라서 가장 높은 루트노드에는 가장 작은 숫자가 들어있습니다. 데이터를 추가할 때는 이 규칙을 지키기 위해 가장 아래에 있는 왼쪽 노드부터 값을 채웁니다. 가장 아래층이 모두 채워지면 새로운 층을 만들어 가장 왼쪽에 채웁니다.



## 삽입(Push)

가장 아래층의 왼쪽부터 채웁니다.

rule : 만약 **삽입된 데이터가 부모보다 작다면 부모와 자식을 교환**합니다.

이런처리를 교환이 발생하지 않을 때까지 반복합니다.



## 추출(Pop)

가장 위에 있는 숫자가 추출됩니다.

가장 위의 숫자가 없어졌으므로 트리를 다시 정리해야합니다.

가장 후미에 있는 숫자를 가장 높은 루트노드로 이동시킵니다.

다음 규칙에 의거해 트리를 재정리합니다.

rule : **부모숫자보다 자식 숫자가 작은 경우는 자식의 좌우 숫자중 더 작은 쪽과 교환**



## 시간복잡도

힙은 항상 가장 위에 최솟값이 있으므로 추출시 O(1)의 시간으로 데이터를 추출합니다.

추출한 후 힙을 재구축 할 때에는 가장 후미에 있는 데이터를 위로 가져온 뒤 자식과 비교해가며 아래방향으로 진행됩니다.  이때문에 계산 시간은 트리의 높이와 비례하게 됩니다. 데이터 수를 n이라고 하면 힙 형성 조건에 따라 트리의 높이는 log2n이므로 재구축의 계산시간은 O(logn)이 됩니다.



## 활용

관리하고 있는 데이터중 최소값을 자주 추출해야하는 경우 힙을 사용하면 편하다!



## Golang과 함께 공부해보는 힙(heap)

golang에서 사용하기 쉽도록 구현되어 있는 힙과 그 활용법을 알아봅니다. 추가로 더 깊게 들어가서 golang 힙 내부 구조가 어떤지 파고들어가보겠습니다. 따라오세요.

```golang
package main

import (
	"container/heap"
	"fmt"
)

type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

// This example inserts several ints into an IntHeap, checks the minimum,
// and removes them in order of priority.
func main() {
	h := &IntHeap{2, 1, 5}
	heap.Init(h)
	heap.Push(h, 3)
	fmt.Printf("minimum: %d\n", (*h)[0])
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
}
```



golang의 container/heap이라는 내장 라이브러리를 활용한 minHeap구현입니다.

해당 예제는 https://golang.org/pkg/container/heap/ 공식문서를 참고했습니다.

예제를 천천히 살펴볼까요. 제시한 IntHeap은 Len(), Less(), Swap()메서드를 리시버함수로 달고있어서 sort.Interface를 만족하고있고 추가로 Push와 Pop매서드를 들고있습니다.

즉, container/heap의 heap.Interface는 다음과 같습니다.

```golang
package heap

import "sort"

type Interface interface {
	sort.Interface
	Push(x interface{}) // add x as element Len()
	Pop() interface{}   // remove and return element Len() - 1.
}

```

근데 공식예제에서 특이한 점이있습니다. 리시버함수 Push와 Pop에서 포인터를 사용하고있네요?

```golang
func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}


```

이는 해당 작업들이 slice 안에 있는 값 뿐만아니라 '길이'도 변경하기때문입니다.



push는 단순히 값을 slice에 삽입해주는 작업만 들어있네요.

pop은 slice의 마지막녀석을 반환하고 그녀석을 제외한 나머지로 현재힙을 구성합니다.

이건 단순히 말그대로 스택의 push pop..인데? 라이브러리 내부에서 특수한 작업을 하고 있음이 분명합니다.

내부를 살펴보러 출발해보기전에 main에서 호출한 순서를 봐봅시다.

```golang
func main() {
	h := &IntHeap{2, 1, 5}
	heap.Init(h) // <- 초기화
	heap.Push(h, 3) // <- 3을 삽입
	fmt.Printf("minimum: %d\n", (*h)[0])
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
}
```



heap의 Init함수를 호출해서 첫 힙구성을 시작합니다. 내부를 좀 구경해보겠습니다:

```golang
package heap

...
// A heap must be initialized before any of the heap operations
// can be used. Init is idempotent with respect to the heap invariants
// and may be called whenever the heap invariants may have been invalidated.
// Its complexity is O(n) where n = h.Len().
//
func Init(h Interface) {
	// heapify
	n := h.Len()
	for i := n/2 - 1; i >= 0; i-- {
		down(h, i, n)
	}
}
```

down이라는 메서드(소문자시작 private메서드)가 초반 힙구성을 하는데 핵심적인 역할을 하네요. 반복문의 조건식은 n/2 -1 의 값부터 시작해서 i를 줄여나갑니다. i가 0보다 작아지면 down을 호출하는 반복문이 종료됩니다. 초기화의 복잡도는 O(n)입니다.

down의 내부를 살펴보겠습니다:

```golang
func down(h Interface, i0, n int) bool {
	i := i0
	for {
		j1 := 2*i + 1
		if j1 >= n || j1 < 0 { // j1 < 0 after int overflow
			break
		}
		j := j1 // left child
		if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
			j = j2 // = 2*i + 2  // right child
		}
		if !h.Less(j, i) {
			break
		}
		h.Swap(i, j)
		i = j
	}
	return i > i0
}
```

i0은 무엇이며 n은 무엇일까...일단 첫눈에 감이 오지않네요.

j1이 `left child`이고 j2가 `right child`... 뭔가 비교를 하네요. (위에서 봤던 힙의 특징들을 토대로 느낌만 우선 가져갑시다.)



힌트를 얻기위해 3을 삽입하는 heap.Push(h, 3)의 내부부터 한번 봐보겠습니다.

```golang
package heap
// Push pushes the element x onto the heap. The complexity is
// O(log(n)) where n = h.Len().
//
func Push(h Interface, x interface{}) {
	h.Push(x)
	up(h, h.Len()-1)
}

func up(h Interface, j int) {
	for {
		i := (j - 1) / 2 // parent
		if i == j || !h.Less(j, i) {
			break
		}
		h.Swap(i, j)
		j = i
	}
}
```

사용자가 정의한 Push호출뒤 `up`이라는 heap패키지의 private메서드가 실행됩니다. up은 또뭘하는지 heap슬라이스를 첫번째인자로 그리고 해당 슬라이스의 마지막인덱스를 인자로 가져가네요. up메서드 안에서 i는 `마지막 녀석의 인덱스 j에서 1을 뺀뒤 2로 나눈 몫`입니다. 즉, 트리 구조에서 부모의 인덱스를 뜻합니다.

부모노드를 탐색하면서 자식보다 부모노드가 크다면 교환하는 반복문입니다. 루트노드이거나 부모노드가 자식보다 작다면 반복문을 중지합니다. 이는 위에서 설명한 힙의 삽입할 때 룰입니다.

Push방식이 이해되었으니 Pop 방식을 이해해볼까요:

```golang
package heap
// Pop removes the minimum element (according to Less) from the heap
// and returns it. The complexity is O(log(n)) where n = h.Len().
// It is equivalent to Remove(h, 0).
//
func Pop(h Interface) interface{} {
	n := h.Len() - 1
	h.Swap(0, n)
	down(h, 0, n)
	return h.Pop()
}
```

heap패키지의 Pop메서드입니다. 앗!

먼저 **마지막녀석을 첫번째 녀석과 Swap하네요.** 단순히 첫번째 녀석을 빼내는게 아니고 마지막녀석을 첫번째 녀석과 스왑한 후  down이라는 메서드를 호출해 트리를 정리 어찌저찌 정리한뒤. h.Pop()으로 사용자가 구현한 메서드(slice의 마지막녀석을 제거해 리턴하는 메서드였죠)를 실행하네요.

결과적으로 반환되는 값은 Slice의 가장 앞단에 있었던 첫번째 녀석입니다.

`down(h, 0, n)`

이녀석만 이제 이해하면 golang이 구현한 heap을 정복할 수 있습니다. 다시 보겠습니다:

```golang
func down(h Interface, i0, n int) bool {
	i := i0
	for {
		j1 := 2*i + 1
		if j1 >= n || j1 < 0 { // j1 < 0 after int overflow
			break
		}
		j := j1 // left child
		if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
			j = j2 // = 2*i + 2  // right child
		}
		if !h.Less(j, i) {
			break
		}
		h.Swap(i, j)
		i = j
	}
	return i > i0
}
```

이제 좀 감이 오시나요?

해당 반복문은 추출(pop)의 룰 특성인 왼쪽자식, 오른쪽자식, 부모노드를 비교하며 정렬하는 메서드입니다. 두번째 인자인 i0은 첫번째 타겟노드로 해당 노드가 부모노드가되어 비교를 시작하는 메서드입니다.

부모노드의 index인 i에 *2+1연산을 통해 왼쪽 자식 노드의 인덱스를 구합니다.

오른쪽 자식노드는 왼쪽 자식노드의 인덱스 +1이겠죠?

**부모숫자보다 자식 숫자가 작은 경우는 자식의 좌우 숫자중 더 작은 쪽과 교환**이라는 룰에 따라 down 방식으로 내려가면서 더이상 부모보다 작은 자식숫자가 없을때까지 반복문을 수행합니다. 마지막 bool값 반환(i > i0)은 변화가 일어났다는 더티체크네요.

이제 Pop방식도 이해되었고 Init에서 사용한 down 메서드의 순차 호출이 이해가갑니다.

```golang
func Init(h Interface) {
	// heapify
	n := h.Len()
	for i := n/2 - 1; i >= 0; i-- {
		down(h, i, n)
	}
}
```

---



### 마무리

지금까지 자료구조 힙(heap)과 golang(1.10버전)의 내장 자료구조 heap의 내부를 살펴보았습니다.

자료구조의 특성을 확실히 알고 내부코드를 보니 어려운 코드들도 차근차근 이해할 수 있습니다.

p.s 자료구조 설명을 위한 이미지파일은 아이패드를 사면...그려서 추후 업데이트하겠습니다.
