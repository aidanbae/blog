---
title: "Golang Design Pattern - Command"
date: 2017-11-01T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'design pattern']
author: "Aidan.bae"
noSummary: false
---
>그림판이나 Photoshop을 사용해본 경험이 있다면!  
**Ctrl+z키라는 마법의 주문** 으로 우리는 우리가 했던 액션들을 취소해본 경험 유!
요구사항에 대한 캡슐화와  
요구사항을 큐에 저장하거나 기록,취소 할수 있다는 장점!

---

## 명령패턴

게임프로그래밍을 접하면서 첫 디자인패턴을 공부했을때 만난 패턴
그땐 왜이렇게 어렵게 느껴졌었는지..

- 유저의 요청사항(명령) 캡슐화!
- 요청사항과 큐를 이용해서 작업내역을 관리!

그냥 가볍게 위 두가지만 생각하면 편하다.
우선 `Command` Interface를 구현해보자.
스터디한 책에는 ConsoleCommand와 Excute만 있지만 그대로하면 재미없다. 
실수가 많은 나는 `Ctrl+z`의 광팬, 도전해보자.
ConsolePrint를 취소하는 것은 와닿지 않으니
간단하게 `계산기명령`으로 해봐야겠다.

플러스와 마이너스 Command를 구현해보자:

```golang

type Command interface {
	Execute(*Calculator)
	Undo(*Calculator)
}

type PlusCommand struct {
	beforeVal int
	num       int
}

func (a *PlusCommand) Execute(calculator *Calculator) {
	a.beforeVal = calculator.val
	calculator.Add(a.num)
	fmt.Println(calculator.val)
}

func (a *PlusCommand) Undo(calculator *Calculator) {
	calculator.val = a.beforeVal
	fmt.Println(calculator.val)
}
```

Command interface는 `Execute`와 `Undo` 두가지 메소드로 이루어져있습니다.  
두개의 함수가 리시버함수로 달려있기만하면 인터페이스에 부합하는 것이죠  
인자로 `명령을 실행하는 주체가 될 Actor(계산기)`를 받습니다.


**PlusCommand(더하기 요청사항)가 하는 건은 간단합니다.**  
```aidl
Execute - 계산기에 있는 이전 값을 저장해두고(기록) 계산기의 Value에 특정값을 더하는 것  
Undo - 이전 값으로 계산기의 Value를 변경
```



마이너스는 더 쉽겠죠 똑같습니다
```golang
type MinusCommand struct {
	beforeVal int
	num       int
}

func (m *MinusCommand) Execute(calculator *Calculator) {
	m.beforeVal = calculator.val
	calculator.Minus(m.num)
	fmt.Println(calculator.val)
}

func (m *MinusCommand) Undo(calculator *Calculator) {
	calculator.val = m.beforeVal
	fmt.Println(calculator.val)
}
```

읽다보니 계산기는 왜 설명안해주세요 하는 분들이 있을까바~
제가 그냥 임의로 add와 minus로 구조화한겁니다 Simple하죠

```golang
type Calculator struct {
	val int
}
func (c *Calculator) Add(num int) {
	c.val += num
}
func (c *Calculator) Minus(num int) {
	c.val -= num
}
```

이제 명령패턴을 정복할 거의 모든 준비가 끝났습니다.
그나마 필요한게 요청사항(작업내역)을 관리할 큐정도겠네요.

작업내역을 저장할 큐슬라이스와 Actor를 품고 있는 `CommandQueue구조체`를 만들어봅시다:
```go
type CommandQueue struct {
	queue []Command
	actor *Calculator
}

func (p *CommandQueue) AddCommand(c Command) {
	// 명령을 큐에 저장하고
	p.queue = append(p.queue, c)
	// 실행합니다
	c.Execute(p.actor)
	// 길이가 10이 될경우 작업내역을 지웁니다
	if len(p.queue) == 10 {
		p.queue = make([]Command, 10)
	}
}

func (p *CommandQueue) RemoveCommand() {
	// 마지막 명령을 꺼내서 Undo를 호출합니다.
	lastCommand := p.queue[len(p.queue)- 1]
	lastCommand.Undo(p.actor)
	// 마지막 명령을 큐슬라이스에서 제거합니다.
	p.queue = p.queue[:len(p.queue)-1]
}

func main() {
	calculator := &Calculator{val:0} // 0으로 시작하는 계산기
	queue := CommandQueue{actor: calculator} // 계산기를 액터로 지정후 커맨드큐생성

	queue.AddCommand(CreatePlusCommand(3)) // +3
	queue.AddCommand(CreatePlusCommand(3)) // +3
	queue.AddCommand(CreatePlusCommand(3)) // +3
	queue.AddCommand(CreateMinusCommand(3)) // -3
	queue.AddCommand(CreateMinusCommand(3)) // -3
	queue.RemoveCommand() // 돌려돌려 되돌려줘
	queue.RemoveCommand() // 돌려돌려 되돌려줘...
}

```
AddCommand는 명령을 축적!
RemoveCommand 명령을 제거!

결과는!!!?
![result](/code/golang-design/command/screenshot.png)
작업취소까지 잘 작동합니다
여러분의 프로젝트에 녹여보세요~
