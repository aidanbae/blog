---
title: "Golang 동시성을 활용한 알고리즘 문제풀이 - 1"
date: 2018-08-20T09:31:27+01:00

categories: ['Algorithm', 'Golang']
tags: ['golang', 'algorithm', '알고리즘', '구글코드잼']
author: "Aidan.bae"
draft : false
noSummary: false
---

파이프라인 패턴을 활용해서 알고리즘을 풀어보기로 했다.  
알고리즘 문제: [Google Code Jam 2016 qualification round A](https://code.google.com/codejam/contest/6254486/dashboard#s=p0)  

**궁금했던 점**  
 동시성 프로그래밍으로 기존 알고리즘 모범답안보다 얼마나 퍼포먼스를 낼 수 있을까

많은 프로그래밍대회가 싱글쓰레드기반으로 동작해 정답이 순차적으로 나와야한다. 하지만 멀티쓰레드 프로그래밍은 Case #1번이 아니라 Case #5번이 먼저 풀릴 수 있다.

---

(사실 구글 코드잼의 가장 큰 난적은 문제가 영어라는 점이다)

#### 문제요약  
Bleatrix Trotter 이라는 여자분은 불면증에 걸린거 같다.  
이 여자분은 잠에 골아떨어지기위해 특별한 전략이 있다.  
첫째, 여자가 N을 뽑는다.
그러고나서 여자는  N, 2 × N, 3 × N, ... 쭉 수를 말한다.
여자는 해당 수를 부를때 해당수에 포함된 하나하나의 숫자들을 전부 생각한다.
0,1,2,3,4,...9,0까지 다보면 잠든다.  
(별에별.. -_-)

For example)  

- N = 1692. 여자가 본 숫자 ( 1, 2, 6, and 9)
- 2N = 3384. 여자가 본 숫자 (1, 2, 3, 4, 6, 8, and 9)
- 3N = 5076. 이제 모든숫자를 다보앗기때문에 잠든다.

그녀가 잠들기전에 본 마지막 숫자가 무엇인가를 구하는 문제  
만약 이분이 평생 카운팅해야된다면 INSOMNIA를 출력하시면됩니다.

---
#### 문제접근

딱히 어려운 문제가 아니었기때문에 파이프라인을 설계하기 편했다.  
우선 해야할 Task를 쪼개보았다.
```
1) mutiple: 들어온 케이스 input값에 따라 순차적으로 1,2,3...을 곱해주는 Task
2) beApart: 곱해진 값을 digit으로 분리하는 작업 (몇자리 수인지까지 포함)
3) check: 0에서부터 9까지 여자가 본 숫자들을 저장하는 slice check
```

저번에 공부해본 파이프라인을 적용해볼려고 일부러 분리해본 것이다.
느낌이 오셨다면 훌륭하다.

multiple함수 부터 만들어보자:
```golang
// 특정수가 들어오면 out채널로 곱한 값들을 차례로 쏴주는 함수
func multiple (input int, ctx context.Context) (out chan int){
	// 들어온 수가 있다면 계속 배수해준다. 브레이크될때까지
	out = make(chan int)
	go func() {
		if input == 0 {
			no, _ := ctx.Value("caseNo").(int)
			fmt.Printf("Case #%d: INSOMNIA \n", no)
			close(out)
			return
		}
		i:= 1
		for  {
			select {
			case out <- input * i:
				i++
				if i > 500 {
					fmt.Println("too many loop")
					break
				}
			case <-ctx.Done():
				return
			}
		}
		close(out)
	}()
	return out
}
```
`인자값: 특정 케이스의 input값, context`  

context는 Go 1.7버전부터 내장 라이브러리로 삽입된 goroutine제어용 라이브러리이다. 사실 quit채널로 고루틴을 종료시켜도 문제없다. 공부용으로 사용해보았다. 그래도 고루틴의 생애주기를 좀 더 쉽게 제어할수있도록 구조화해놓았기때문에 공부해보길 추천한다.

여기서 핵심은 input값의 i곱을 계속해서 out으로 보내주는 부분이다.  
케이스문을 통해 함수로 값을 전달하는 것도 가능하다.  

ctx.Done()은 종료신호를 늘 기다린다.
혹시 다른 고루틴에서 정답을 찾는다면 이쪽으로 신호가 날라오고 우린 multiple 고루틴을 깔끔하게 정리할 수 있게된다.

정리) input이 4가 들어오면 4, 8, 12, 16, ... 곱한값을 out채널로 전달한다.

```golang
// 전달 받은 값들을 자릿수별로 분리한다. 분
func beApartAndCheck (in chan int, ctx context.Context, cancel context.CancelFunc) (out chan int) {
	out = make(chan int)
	digitSlice := make([]bool, 10)
	go func () {
		for inputData := range in {
			// 1. 몇자리인지 구한다
			// 2. dataSlice[분리된 숫자] = true
			// 3. dataSlice가 올 true인지 확인한다.
			apart(inputData, digitSlice)
			if checkAllClear(digitSlice) {
				caseNo, _ := ctx.Value("caseNo").(int)
				fmt.Printf("Case #%d: %d \n", caseNo, inputData)
				cancel()
				break;
			}
			out <- inputData
		}
		close(out)
	}()
	return out
}


func apart (i int, dataSlice []bool)  {
	for {
		dataSlice[i % 10] = true
		i /= 10
		if i < 10 {
			dataSlice[i] = true
			break
		}
	}
}

func checkAllClear(dataSlice []bool) bool {
	for _, bool := range dataSlice {
		if bool == false {
			return false
		}
	}
	return true
}
```

(정리중 추가업데이트하겠습니다.)
몇자리수인걸 어떻게구하지?

1. 문자열변환
2. 그냥 10으로 나누어본다. 몇번 나누어지는지
3. 상용로그활용

```golang
// 작업 파이프라인 함수
func pipeline(caseNo int, input int, wg *sync.WaitGroup) {
	ctx, cancel := context.WithCancel(context.Background())
	ctx = context.WithValue(ctx, "caseNo", caseNo)
	outMulti := multiple(input, ctx)
	outApart := beApartAndCheck(outMulti, ctx, cancel)
	for range outApart{ }
	wg.Done()
}
```
```golang
func main () {
	f, _ := os.Open("A-large-practice.in")
	mi := MyInput{rdr: f}

	t := mi.readInt()
	var wg sync.WaitGroup

	for caseNo := 1; caseNo <= t; caseNo++ {
		wg.Add(1)
		testCase := mi.readInt()
		go pipeline(caseNo, testCase, &wg)
	}

	wg.Wait()
}
```

#### 결과
```
// Google CodeJam 모범답안
Case #95: 4997286
Case #96: 1097403
Case #97: 1219887
Case #98: 90
Case #99: 7999984
Case #100: 90
    2000       1056109 ns/op
PASS
ok      _/Users/aidan/go/src/reborn/googlecodejamhis    2.235s
```

```
// My logic
Case #93: 3129210
Case #96: 1097403
Case #97: 1219887
Case #95: 4997286
Case #98: 90
Case #99: 7999984
Case #100: 90
Case #80: 9000
    2000        685255 ns/op
PASS
ok      _/Users/aidan/go/src/reborn/googlecodejam    1.456s
```

정답테스트는 간단하게 통과했고 원래 목적이었던 퍼포먼스 향상이 궁금해서
벤치마크 테스트를 돌려보았다.
2000번의 실행 결과 평균적으로 68만 나노초가 걸린 것이 확인되었다.
모범답안에 비해 37만 나노초 정도 퍼포먼스 이득을 벌었다.
퍼센트로 따지면 꽤 훌륭하다. 만족!

#### PostScript
알고리즘은 문제 하나를 풀때마다 다양한 것들을 찾아보고 배우게된다.  
아울러 내가 설계한 디자인패턴으로 문제를 풀면 또다른 즐거움을 맛볼 수 있다.  

아직은 알고리즘대회에서는 동시성프로그래밍보다 순차프로그래밍이 기본 베이스이기 때문에 Case 1번이 무조건 첫번째 정답 Output으로 나와야 정답처리가 되는 쪽이 많다. 보시다시피 동시성프로그래밍에서는 어떤 문제가 먼저풀릴지 예측할 수 없기 때문에 마지막에 정렬해주고 정답을 출력해야하는 번거러움이 있다.
