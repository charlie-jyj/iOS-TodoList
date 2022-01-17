## To-Do List 앱 만들기

### 1) 사용 기능
- TableView 에 할 일을 추가
- TableView 할 일 삭제
- TableView 할 일 재정렬
- 할 일들을 데이터 저장소에 저장하여 앱을 재실행하여도 데이터 유지

### 2) 기본 개념
#### (1) UITableView
데이터를 목록 형태로 보여줄 수 있는 가장 기본적인 UI 컴포넌트
UIScrollView 를 상속받고 있기 때문에 스크롤 통해 목록 보여주기 가능

##### (1) 특징
- 여러개의 cell 을 가지고 있다 => 하나의 열과 여러 행을 지니고 있으며 수직 스크롤만 가능
- 섹션을 이용해 행을 그룹화하여 콘텐츠를 좀더 쉽게 탐색
- 섹션의 헤더와 푸터에 view 를 구성하여 추가 정보 표시 가능

<img src="https://docs-assets.developer.apple.com/published/6da9972ff1/content-only~dark@2x.png">

##### (2) 구현 방법
- Delegate 프로토콜 : tableview 의 동작과 외관 담당, view 는 delegate에 의존하여 업데이트된다 (행의 높이, 행 클릭 시 액션 정의)
- DataSource : 데이터를 받아 뷰를 그려준다 (섹션은 몇개? 행은 몇개? 내용은 무엇?을 정의)

##### (3) UITableViewDataSource
테이블 뷰를 생성하고 수정하는데 필요한 정보를 테이블 뷰 객체에 제공
무엇을 보여줄 것인가

- dequeueReusableCell
스토리보드 위의 셀을 가져올 수 있는 메서드
가져온 셀을 리턴하면 스토리보드에 셀이 출력된다.
셀을 재사용하기 위한 메서드 (queue 사용) => 메모리 낭비 방지
스크롤하면서 새로운 셀이 보이면 이전의 셀은 reuse pool로 보내지고 
다시 그 셀을 보게 되면 dequeue  


##### (4) UITableViewDelegate
행의 액션 관리
액세서리 뷰 지원
테이블 뷰의 개별 행 편집 


#### (2) UIAlertController
    - preferredStyle: .alert / .actionsheet
    - Alert 스타일은 가운데에 뜨고 actionsheet는 밑에서 위로 올라오는 스타일

UIAlertAction handler 
버튼 클릭 시 action 을 클로저로 정의한다

```swift

let alert = UIAlertController(title: "My Alert", message: "This is an alert.", preferredStyle: .alert) 
alert.addAction(UIAlertAction(title: NSLocalizedString("OK", comment: "Default action"), style: .default, handler: { _ in 
NSLog("The \"OK\" alert occured.")
}))
self.present(alert, animated: true, completion: nil)

func addTextField(configurationHandler: ((UITextField) -> Void)? = nil)

```
https://developer.apple.com/documentation/uikit/uialertcontroller

#### (3) 재사용 메커니즘
> Reuse Mechanism

1. 테이블 뷰가 화면에 나타날 셀 객체를 데이터 소스에 요청
2. 데이터 소스는 테이블 뷰의 재사용 큐 (Reuse Queue)에서 사용가능한 셀이 있는지 확인, 있으면 하나를 꺼내어 전달하고 없으면 새로 생성
3. tableView(_:cellForRowAt:) 가 셀의 contents를 구성한 다음 반환하면 테이블 뷰는 이 셀을 받아 화면에 표시
4. 사용자가 테이블 뷰를 스크롤 함에 따라 화면을 벗어난 셀은 테이블 뷰에서 제거되지만 완전히 삭제하지 않고 재사용 큐에 추가
5. 스크롤에 따라 앞의 과정 반복

- 재사용 식별자 
    - 속성값
    - 스토리 보드에서 프로토 타입 셀 설계 시 Identifier 항목에 입력
- 셀 자체는 재사용하지만 콘텐츠는 cellForRowAt 메서드를 통해 매번 새롭게 구성
    - 반복적으로 호출되는 메서드의 내부에는 네트워크 통신 등 처리 시간이 긴 로직을 포함하지 않는다
    - 네트워크 통신을 통해 읽어온 데이터는 재사용할 수 있도록 캐싱(Caching) 처리하여 통신 횟수를 줄이는 것이 좋다. (메모이제이션)
    - 네트워크 통신이나 시간이 오래 걸리는 코드를 사용할 때 비동기로 처리한다. (blocking 해결)
    

```swift

let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
```

#### (4) UserDefaults
Data 를 local에 저장한다.
Runtime environment 위에서 동작
앱이 실행되는 동안 기본 저장소에 저장 
Key:value 로 저장되고, singleton 패턴이므로 애플리케이션 전체에 하나의 인스턴스가 존재한다.

```swift
UserDefaults.standard
userDefaults.set(value, key)
userDefaults.object(key)
```


#### 3) 새로 배운 것

- 테이블뷰의 셀이 배열 prop 을 기반으로 그려지는데, 
배열에 새로운 아이템이 append 되어 갱신될 때 마다 
tableView를 갱신해주기 위해 => reloadData() 메서드를 사용한다.

- UIAlertController
Alert view 를 화면에 표시하기 위해서는 다른 view 처럼 
현재 컨트롤러에서 present 한다.
After configuring the alert controller with the actions and style you want,
present it using the present(_:animated:completion:) method.
UIKit displays alerts and action sheets modally over your app's content

- done button 

Edit button 을 IBOutlet 으로 빼낼 때 strong 으로 선언하는 이유는
weak으로 선언할 경우 edit => done으로 바뀌었을 때 edit button 이 메모리에서 해제 되어 재사용할 수 없기 때문

- viewDidLoad에서 
doneButton = UIBarButtonItem(barButtonSystemItem, target, action)

Target : the object that receives the action message
Action : #selector
Object-c 에서 class 이름 가르키는 데에 사용되었던 것 (동적 호출 위해) 
Swift 에서는 구조체 형식으로 정의
#selector() 통해 생성

- @objc method
Object-c 와의 호환을 위해 사용되는 어노테이션




