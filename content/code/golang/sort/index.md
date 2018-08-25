---
title: "Golang Sort 정렬"
date: 2018-08-14T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'sort', '정렬']
author: "Aidan.bae"
draft : true
noSummary: false
---

golang에서는 정렬 (sort) 라이브러리를 제공하고 있습니다.  

다음은 내장 함수들을 활용한 예제코드입니다:
```golang
func main() {
	str := []string{"c", "a", "b"}
	sort.Strings(str)
	fmt.Println("Strings: ", str)

	ints := []int{70, 2, 4}
	sort.Ints(ints)
	fmt.Println("Ints:   ", ints)

	s := sort.IntsAreSorted(ints)
	fmt.Println("Sorted: ", s)
}
```

결과:
```
Strings:  [a b c]
Ints:    [2 4 70]
Sorted:  true

Process finished with exit code 0
```

##### sort.Sort()

정렬을 꼭 이런식으로 하고싶은게 아닐 수 있습니다.
**커스터마이징**을 할 수 있도록 sort는 Sort라는 메소드를 가지고있습니다. Sort에 인자로 들어가는 struct가 sort의 Interface에 정의된 메소드들만 들고있으면 됩니다.

```golang
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}

	fmt.Println(people)
	// There are two ways to sort a slice. First, one can define
	// a set of methods for the slice type, as with ByAge, and
	// call sort.Sort. In this first example we use that technique.
	sort.Sort(ByAge(people))
	fmt.Println(people)

	// The other way is to use sort.Slice with a custom Less
	// function, which can be provided as a closure. In this
	// case no methods are needed. (And if they exist, they
	// are ignored.) Here we re-sort in reverse order: compare
	// the closure with ByAge.Less.
	sort.Slice(people, func(i, j int) bool {
		return people[i].Age > people[j].Age
	})
	fmt.Println(people)

}
```
