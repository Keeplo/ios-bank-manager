# BankManager
## Information
* 프로젝트 기간 : 2021.07.26. ~ 2021.08.06.
* 프로젝트 인원 : 2명 Marco(@Keeplo), Luyan(@KimWanki)
* 프로젝트 소개 
    > 은행 업무를 보려는 고객의 대기줄을 여러 은행창구에서 처리하는 멀티태스킹 앱
* Pull Requests
    * [Step 1](https://github.com/yagom-academy/ios-bank-manager/pull/69)
    * [Step 2](https://github.com/yagom-academy/ios-bank-manager/pull/77)
    * [Step 3](https://github.com/yagom-academy/ios-bank-manager/pull/88)
### Tech Stack
* Swift 5.4
* Xcode 12.5
* iOS 14.0
### Demo
<details><summary>Demo</summary><div markdown="1">


</div></details>


## Troubleshootings
<details><summary>ARC를 고려한 LinkedList Node의 메모리해제 고민 UnitTest 과정</summary><div markdown="1">

**UnitTest 중 모든 리스트를 삭제하는 `clear()` 메서드의 오류 발견**

```swift
// 최초 clear() 아이디어
mutating func clear() {
    head = nil
}
```

메서드 동작 아이디어 : 다음 Node를 참조하는 `next` 프로퍼티의 `strong` 참조로 연결된 Node들이 최초 `head` 프로퍼티의 참조가 끊어지면 `strong` Reference Count가 0이 되며 연쇄적으로 Node가 메모리에서 해제됨

```swift
func test_성공케이스_링크드리스트의_clear을하면_노드의개수가_0이_나온다() {
    // given
    var dummyNodes = [
		        DummyNode(value: 1, weakPointer: nil),
            DummyNode(value: 2, weakPointer: nil),
            DummyNode(value: 3, weakPointer: nil),
            DummyNode(value: 4, weakPointer: nil)
        ]
    dummyNodes.enumerated().forEach({ value in
        sutLinkedList.append(value.element.value)
        dummyNodes[value.offset].weakPointer = sutLinkedList.peekLastNode()
    })
    // when
    sutLinkedList.clear()
    let result = dummyNodes.filter({ $0.weakPointer != nil }).count
    // then
    XCTAssertEqual(result, 0)
}
```

`weak` 참조로 각 Node를 연결시킨 `dummyNodes` 포인터 배열을 이용해서 메모리해제 여부를 테스트함
	
![IMG_F491CDE2439A-1](https://user-images.githubusercontent.com/24707229/149658218-a9f06f18-a71b-40de-be5b-31b2628f6fd3.jpeg)
UnitTest를 진행하던 중 마지막 Node가 해제 되지 않는 점을 발견

![IMG_CE8AC305F722-1](https://user-images.githubusercontent.com/24707229/149658177-d3391e92-067e-4a2d-8c67-4911dbac9529.jpeg)
마지막 노드를 가리키는 `tail` 프로퍼티가 strong 참조를 유지해서 해제되지 않는 점을 인지함

```swift
mutating func clear() {
     head = nil
     tail = nil
} 
```

`clear()` 메서드에서 내부에 `tail` 프로퍼티의 참조를 끊어주어서 해결

추가 고민사항) `tail` 프로퍼티를 `weak` 키워드를 추가하는 것을 고민함
</div></details>
<details><summary>`함수명은 모호하면 안된다` / `함수는 하나의 기능만 동작한다` 의미를 좀 더 고민해보기</summary><div markdown="1">

UnitTest의 Test Case를 성공케이스와 실패케이스로 나누어 기능/메서드 검증을 하는 과정에서 메서드의 성공과 실패라는 관점이 어떤 것이어야 하는 가에 대한 고민을 함

1. 변수/함수명이 중요한 이유에 대해 고민함 `함수명은 모호하면 안된다`
    
    ```swift
    mutating func popFirst() -> T? {
         defer {
             head = head?.next
             if head == nil {
                 tail = nil
             }
         }
         return head?.value
    }
    // popFirst 라는 함수명의 목적에는 first item을 pop 하지 못하는 상황은 포함되지 않으므로
    // 성공 : Item 반환
    // 실패 : nil 반환
    ```
    
    코드에 함수명과 함께 명시적으로 코드를 짜보면 어떨까 생각해봄
    
    ```swift
    mutating func popFirst() -> T? {
         defer {
             head = head?.next
             if head == nil {
                 tail = nil
             }
         }
    		 guard let head = head else { return nil } // 함수의 임무 실패
         return head.value // 함수의 임무 성공
    }
    ```
    
    다음과 같은 기준을 잡고 UnitTest의 성공/실패케이스를 구현해봄
    
    > 함수명에 담긴 목적 = 기능의 성공케이스 
    함수명에 담기지 않은 모든 결과물 (에러, nil 등) = 기능의 실패케이스
    > 
    
    ```swift
    // popFirst UnitTest
    func test_실패케이스_링크드리스트의_popFirst메서드를_호출하면_head노드를_pop한다() {
        // given
    
        // when
        let popFirst = sutLinkedList.popFirst()
        // then
        XCTAssertEqual(popFirst, nil)
    }
        
    func test_성공케이스_링크드리스트의_popFirst메서드를_호출하면_head노드를_pop한다() {
        // given
        let inputNumber = 1
        sutLinkedList.push(inputNumber)
        // when
        let popFirst = sutLinkedList.popFirst()!
        // then
        XCTAssertEqual(popFirst, inputNumber)
    }
    ```
    
2. 함수의 기능에 대한 고민 `함수는 하나의 기능만 동작한다`
    
    ```swift
    // 함수의 임무 -> 주스 만들기
    // 성공 : try 재고검사 성공해서 재고 변경하기
    // 실패 : 재고감사 함수에서 던진 에러 catch 
    func makeJuice(_ juice: Juice, _ completion: (Result<String, JuiceMakerError>) -> Void) {
        do {
            try checkStock(juice.ingredients)
            // ...
            //completion(.success(juice.name))
        } catch {
            //completion(.failure(.outOfStock))
        }
    }
    ```
    
    분기처리보다 do-catch로 목적 - 에러 반환을 두고 함수는 결과적으로 하나의 기능만을 목표로함
    
     
    
    ```swift
    // 함수의 임무 -> edit Mode 인지 detail Mode 인지 구분해서 해당 테이블을 보여주기
    // 성공 : dataSource 선택해서 테이블 보여주기
    // 실패 : reminder 데이터 없음 (dataSource 인스턴스 생성 에러?!, 테이블뷰 못 보여줌?!)
    override func setEditing(_ editing: Bool, animated: Bool) {
        super.setEditing(editing, animated: animated)
    	  // ,,,
        if editing {
            transitionToEditMode(reminder)
            tableView.backgroundColor = UIColor(named: "EDIT_Background")
        } else {
            transitionToViewMode(reminder)
            tableView.backgroundColor = UIColor(named: "VIEW_Background")
        }
        tableView.dataSource = dataSource
        tableView.reloadData()
    }
    ```
    
    if-else 분기는 결과의 구분이라기 보다 과정이며 setEditing 이라는 함수의 목적은 DataSource 변경과 테이블뷰 `reloadData()`
    
    반환 경우가 다른 경우, 함수명에 드러나지 않는 목적은 올바른 반환으로 함수명에 드러나지 않는 결과는 에러 또는 실패 결과를 전달
    

함수명과 함수가 하나의 기능을 해야하는 것, 기능 분리를 세분화 해야하는 것 등 모두 연관되어 있다고 생각함.
</div></details>

<details><summary>Xcode를 이용해서 Thread 생성 체크와 DispatchSemaphore의 올바른 동작 이해</summary><div markdown="1">
    
Xcode의 CPU Report 기능을 이용해서 스레드의 생성을 체크함

기대 아이디어는 2개의 예금업무 스레드, 1개의 대출업무 스레드가 메인스레드와 별도로 추가되어 해당 태스크를 동작할꺼라고 예상함

<img width="1031" alt="128197047-e464e30d-418f-40bf-896f-fe8df157806e" src="https://user-images.githubusercontent.com/24707229/149658285-4ca291b1-7437-424e-8e0c-528410ee7225.png">
다음과 같이 3개 초과된 스레드가 생성하는 것을 인지하고 의도와 다른 동작에서 비동기처리 구현에 오류가 있음을 발견함

<img width="874" alt="Screen Shot 2022-01-16 at 5 44 53 PM" src="https://user-images.githubusercontent.com/24707229/149658237-bad127c8-0144-41e5-86da-66924f7e843e.png">
위 코드처럼 DispatchSemaphore는 하나의 태스크를 스레드로 처리하는걸 제한하기 보다는 스레드 추가는 이미 되고 해당 태스크가 동작가능한 갯수만 제한한다는 점을 깨달음

때문에 DispatchQueue는 스레드의 갯수를 조절하는 기능이 없고 OperationQueue 기능이 있다는 개념적인 이야기를 이해함

GCD를 계속 사용한다면 제어 용 스레드를 생성하고 해당 스레드의 DispatchSemaphore에서 태스크 생성을 제한해서 스레드의 갯 수를 제한하는 방향을 고민함
</div></details>
<details><summary>Model 타입과 Data 타입 고민</summary><div markdown="1">

Node는 내부 `next`프로퍼티를 이용해서 다음 Node를 가리키는 특징때문에 해당 타입을 인스턴스 참조가 가능한 class 타입으로 구현함
	
<img width="682" alt="Screen Shot 2022-01-16 at 8 28 46 PM" src="https://user-images.githubusercontent.com/24707229/149658332-06b5135a-a1d9-4c21-b401-87116da612a9.png">
다음과 같이 해당 타입에 대한 프로퍼티는 private 접근제어가 필요한지 고민함

get set 동작을 따로 구현하는 것에 대한 고민을 할때 해당 타입이 비즈니스 로직으로 중요한 Model이 아니라 Int/Double 타입처럼 Model 내부의 Data로 사용되는 타입이기 때문에 내부 프로퍼티에 직접 접근이 가능한 internal(default) 제어접근자로 결정함
</div></details>
<br>
