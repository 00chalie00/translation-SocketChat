170323

# socket.IO 번역
<br>
Preface
---
iOS앱은 이미 수백만 개가 존재하며 댑분 서버와 통신하여 데이터를 교환한다. 대다수의 경우, 서버는 앱이 통신에 사용할 수있는 RESTful API를 구현하고 제공한다. 앱이 서버에 데이터를 보내거나 서버에서 데이터를 가져와야하는 경우 적절한 요청을하고 잠시 후 데이터가 반환된다. 앱 런타임 기간 동안 여러 번 발생한다.

위 내용은 대부분 사용 사례를 다루지만 전부는 아니다. 예를 들어 실시간으로 업데이트 해야 하는 일종의 뉴스 피드가 앱에 표시되어야 한다면 어떻게 해야할까? 혹은 사용자간 실시간 대화가 앱 기능으로 지원되어야 한다면 어떻게 될까? 해결책은 앱이 새로운 데이터를 서버에 자주 요청하도록 하는 것이므로 가능한한 빨리 새로운 기능을 추가해야 한다. 그러나 이는 두 가지 기본적인 이유 때문에 최선의 방법은 아니다. 첫째, 끝없이 반복되는 요청을 수행하면 말할 것도 없이 불필요한 자원 낭비가 발생하고, 이는 모바일 장치에 관해서도 중요하다. 둘째, 요청간 짧은 간격이 설정 되더라도 곧 가져올 데이터가 실제로 가져올 시간을 보장하지는 않는다.

고맙게도 서버에서 즉시 데이터를 수신할 필요가 있을 때(이런 데이터가 사용가능해질 때마다) 앱이 서버에 요청을 보내지 않고도 이를 가능케하는 더 좋은 솔루션이 있다. 이 솔루션은 웹 소켓을 사용하는 것을 기반으로 하며, 앞서 언급한 두 가지 문제를 완전히 없애버린다. 여기에서 웹소켓에 관한 몇 가지 흥미로운 주제를 찾을 수 있지만 구글링해서 추가적인 정보를 얻을 수 있으니까 생략하겠다. 간단한 말 속에 큰 그림을 줄거니까 나중에 나오는 것을 더 잘 이해할 수 있다.

웹소켓 통신은 서버와 클라간의 지속적인 연결이 항상 존재하는 클라-서버 로직을 사용한다. 정확하게 말하자면, 서버는 클라가 연결되는 전용 포트를 열어둔다. 연결된 모든 앱이 해당 포트로 메시지를 보내고 (보내는 메시지) 수신 메시지 또한 listen할 수 있다. 소켓에 연결할 때의 기본 동작이므로 서버에 연결된 모든 클라는 추가 요청없이 메시지를 자동으로 받는다. 가장 중요한 것은 메시지가 서버에 의해 해당 포트로 전송되면 받는 사람의 클라이언트가 즉시 해당 메시지를 수신하므로 추가작업(예: 뉴스피드 업데이트)을 즉시 수행할 수 있다는 것이다.

주의사항 : 지금부터 간단히 하기 위해 "websocket (s)"대신 "socket (s)"이라는 용어를 주로 사용한다. 그 외에도 "클라이언트"라는 용어는 웹, 모바일 또는 데스크톱 응용 프로그램과 같이 서버에 연결할 수 있는 모든 응용 프로그램을 의미한다. 그러나 이 텍스트에서 "고객"이라고 말하면 구체적으로 iOS 응용 프로그램과 데모 응용 프로그램을 지칭합니다.

소켓에 대한 연결을 구현하고 이를 서버와 통신하는 것이 세상에서 가장 쉬운 작업은 아니다. 비록 전체적인 아이디어가 매혹적으로 들릴지라도, 적절한 의사소통을 달성할 수 있을 때까지 몇 가지 문제가 존재할 수 있다. 고맙게도 (다시 한번 말하지만), 여기서는 배후의 모든 연결 문제를 담당하는 아주 편한 프레임 워크를 제공하고, 소켓 기반의 통신을 쉽게 만든다. 그리고 이를 Socket.IO라고 한다.

최근 큰 프로젝트에서 Socket.IO와 함께 일하게 된 것은 사실이다. 서버와 양방향 통신을 하고 앱에 즉시 메시지를 보내는 것이 얼마나 쉬운지를 볼 기회가 있었다. 공식 웹 사이트를 들어가서 무엇이 무엇이며 어떻게 작동하는지 이해하기를 바란다. Socket.IO는 주로 웹 응용 프로그램 용으로 설계 되었지만 이 프로젝트에 바로 사용될 수 있는 iOS 용 라이브러리도 제공한다.

위 모든 내용을 염두에 두고 이 튜토리얼에서 Socket.IO의 맛보기를 소개하고자 한다. 서버와 메시지를 교환할 수 있도록 메시지를 통합하고 사용하는 방법을 보여 주고 예제를 통해 클라-서버 통신의 작동 방식을 설명하고자 한다. 소개가 너무 길어졌으니 나중에 세부 정보를 남기겠다. 마지막으로 서버 측 부분은 localhost에서 작동한다. 그렇게하기 위해 필요한 모든 세부 사항과 코드를 제공 할 것이다.


<br>
Demo App overview
---
iOS 용 Socket.IO 라이브러리 (클라이언트)를 사용하는 방법과 서버와 실시간 통신을 수행하는 방법을 보여주기 위해 데모 앱으로 채팅 응용 프로그램을 만드는 것이 가장 좋았다. 이 자습서는 물론 중요한 부분에만 초점을 맞추기 때문에 처음부터 모든 것을 만들지는 않을 것이다. 따라서 일단 작업 할 시작 프로젝트를 다운로드 하자.

다음 부분에서 모든 것을 자세하게 볼 수 있지만 여기에 우리의 목표에 대한 간략한 설명을 드리겠다. 첫째, 앱은 두 개의 ViewController로 분리된다. 첫 번째 경우 사용자는 닉네임만 제공하여 앱에 '로그인'할 수 있다(확실한 이유때문에 더 복잡한 로그인 메커니즘을 피할 것이다). 이렇게 되면 기존의 모든 사용자가 테이블 뷰에 나열되고 이름 옆에 연결 상태 표시 (온라인 또는 오프라인)가 표시된다. tableview 바로 아래에 채팅을 수행하기 위한 다음 View Controller로 이동하는 "Join Chat"버튼이 있다.

두 번째 ViewController는 채팅 기능 전용이다. 메시지를 보내고 받는것 외에도 앱에 몇 가지 흥미로운 기능을 추가할 것이다. 새로운 사용자가 채팅에 연결되거나 기존 사용자가 채팅을 떠날 때 팝업 라벨이 표시된다. 또한 한 명 이상의 유저가 메시지를 입력하면 다른 사용자에게 메시지 쓰기에 사용된 텍스트 뷰 바로 위에 나타나는 레이블로 다른 사용자에게 이를 알린다.

<br>
Connecting a user to the chat room
---
내가 데모 앱의 오버뷰에서 이미 언급한 바에 따르면, 우리는 하나의 채팅방만을 만들 것이다. 이는 각각의 새로 들어오는 유저들이 그 곳에 들어갈 것이라는 것을 의미하고, 우리가 작성하는 대화가 전부 모두에게 보여짐을 의미한다.이를 위해서, 우리는 앱과 상호작용 하기 전에 각각의 새로운 유저를 채팅방에 연결부터 해야한다. (일련의 로그인 과정)
이를 쉽게 만들기 위해, 우리는 복잡한 로그인 시스템을 만들지 않을 것이고, 대신 다음처럼 할 것이다. 우리는 단지 각 유저에게 닉네임을 물어볼 것이고 두 가지 중요한 가정을 할 것이다.
1. 각 유저들은 unique한 닉네임으로 로그인한다. 우리가 특별히 작업하지 않아도 데모를 위해 동일 닉네임을 사용하는 두 명이상의 유저가 없을 것이라고 가정할 것이다.
2. 기존의 닉네임은 기존 유저가 재로그인할 때만 사용된다?

이 튜토리얼의 다음 논의를 돕기 위할 뿐만 아니라 위의 두 번째 가정을 명확하게 만드는데 초점을 두기 위해서 서버 사이드 코드가 로그인된 모든 유저들에 대한 하나의 배열을 유지한다고 말할 수 있다. 사실, 그 배열은 각 유저 데이터의 세 조각을 저장한다.
- 포트에 연결할 때 서버에 의해 자동으로 할당되는 각 클라이언트의 unique ID 
- 유저의 닉네임
- 연결 상태 (유저가 연결되어 있는지 아닌지에 따라 true, false)

유저가 앱을 종료하고 서버의 소켓과의 연결이 없어진다고 해서 유저가 배열에서 사라지는 것이 아니다. 대신, 연결이 끊겼다고 마크될 뿐이다. 유저가 서버 리스트에서 완전히 삭제되게 하려면, 유저가 직접 앱에서 exit버튼을 눌러야 한다.

서두르지 말고, 모든 것을 적당한 순서 속에서 보면 혼란스럽지 않을 수 있다. 가장 먼저 우리가 해야할 액션은 물론 socket.io 라이브러리를 이용해서 유저의 닉네임을 서버에 보내는 것이다. SocketIOManager.swift 파일을 열어서 다음 method를 정의하자.

```swift
func connectToServerWithNickname(nickname: String, completionHandler: (userList: [[String: AnyObject]]!) -> Void) {
 
}
```

닉네임을 서버에 보내는 것은 다음의 한줄이면 충분하다.

```
func connectToServerWithNickname(nickname: String, completionHandler: (userList: [[String: AnyObject]]!) -> Void) {
    socket.emit("connectUser", nickname)
}
```

socket object의 emit 메소드는 우리가 Socket.IO 클라이언트 라이브러리를 사용해서 어떤 메시지든 서버에 보내기 위해 필요로 하는 메소드이다.

위 line은 단순히 서버에 connectUser라는 이름의 single 메시지를 보내는 것이고, nickname이라는 한 개의 파라미터를 같이 보낸다. 반면, 서버는 소켓으로 들어오는 메시지를 듣고 있다가, 메시지가 도착하면 당장 다음의 작업을 한다.
1. 유저가 새로운 유전지 아닌지 판단한다. 새로운 유저라면 내부의 배열(client ID, nickname, 연결상태)를 저장하고, 아니면 연결 상태를 true로 업데이트 한다.
2. 업데이트 된 유저 리스트를 return한다.

주의 : 이 부분의 서버 사이드 액션을 srv-SocketChat폴더의 index.js에서 볼 수 있다. 또 MARKDOWN_HASH69adb32223120a64c2bfaa1bcf690c7fMARKDOWN_HASH함수를 찾아봐라.

내가 말한 것에 기반해서, 우리의 다음 할 일은 유저 리스트에 대한 어떤 메시지든 소켓을 통해 듣고, 서버로부터 답이 오면 그를 붙잡는 것이다. 한 iOS 앱에 들어오는 메시지를 듣기 위해서, 우리는 아래처럼 socket object의 on 메소드를 사용해야 한다.


socket.on("SomeMessage") { ( dataArray, ack) -> Void in
 
}

명시해야 하는 첫 번째 인자는 우리가 관심을 가진 메시지의 literal이다. 당신이 서버 개발자가 아니라도, 당신이 대응해야 하는 모든 메시지 리터럴이 먼저 당신에게 알려질 것이다. 여기에서 유저 리스트로 들어왔으면 하는 그 리터럴은 “userList”이다.

두 번째 인자는 두 개의 파라미터를 가진 closure이다. 첫 번째는 서버에 의해 보내질 결과의 갯수에 따라 달라지는 원소들의 갯수가 포함된 NSArray object이다. 서버에서 single value를 리턴하더라도 클로져의 첫 번째 인자는 무조건 배열임을 기억하자(single value가 오면 배열의 첫 번째 인자에 존재한다). 두 번째 파라미터는 서버에게 메시지를 수신했다는 정보를 알리는데 쓰일 수 있는 acknowledge 메시지 이다. 물론 이 파라미터들의 네이밍은 당신에게 달려 있고, dataArray, ack는 여기서 내가 정한 것일 뿐이다.

예시로 돌아가서, 앱이 유저리스트 메시지를 listening하게 만들자.


유저 리스트가 서버로부터 수신되면, 유저리스트를 인자로 가지고 있는 completion handler를 콜한다. 객체에 대한 dictionary들의 배열인데, 이것이 위의 변환이 필요한 이유다.

중요한 부분 : 위의 socket.on 메소드는 서버가 유저리스트를 보낼때마다 Socket.IO에 의해 자동으로 콜 된다. 다른 방법으로는, 앱이 userList 메시지를 끊임없이 listening하고 있으면서, userList 하나가 도착하면 completion handler를 콜하게 만들 수도 있다. 물론, 이 모든 activity는 앱이 서버에서 연결이 끊어질 때 종료된다.

<br>
Displaying users
---
우리가 유저를 채팅방에 연결하는 메카니즘을 구현했고, 유저 리스트를 받아왔기 때문에 이제 이를 사용할 차례다. 먼저 UsersViewController.swift 파일을 열고, 새로운 custom method를 추가하자. 이 메소드는 앱이 켜졌을 때 유저가 nickname을 타입해넣을 수 있는 textfield와 함께 alert constroller를 보여주는 메소드이다.
이것이 해야 할 구현이다. 복붙하도록 하자.

OKAction body 부분이 우리가 뭔가 로직을 추가할 곳이다. 먼저 alert controller의 textfield가 제대로된 값을 포함하고 있는지 아닌지 체크할 것이다. 만약 아니라면 우리는 recursion을 통해 같은 메소드를 한 번 더 call할 것이고 똑같은 alert controller가 다시 뜰 것이다.

그러나 textfield가 값을 가지고 있다면, 우리는 먼저 nickname이라고 한 property에 먼저 저장할 것이다. 그리고 이전에 만들어둔 connectToServerWithNickname(_:completionHandler:) 메소드를 call할 것이다.

사용자 목록을 얻어서 비어 있지 않다는 걸 확인하면 UsersViewController class의 users property에 할당하고 tableview를 업데이트하고 볼 수 있게 만들자. 
위의 코드 조각으로 우리는 각 부분이 나머지 부분과 어떻게 조화롭게 상호작용 하는지 볼 것이다: 데이터를 받아들이고 보내는 이 클래스가 매개 클래스인 SocketIOManager를 사용한다. 그리고 결국 Socket.IO 라이브러리를 직접적으로 다룬다.
어디선가는 위 메소드를 콜하도록 만들어야 한다는걸 잊지말자. 내가 보기에는, 가장 적절한 위치는 viewDidAppear(:_) 메소드이다. UI가 적절하게 초기화 되었는지 확인할 수 있기 때문이다. 이렇게 만들자:

각 속성이 nil일때만 닉네임을 요청한다.

<br>
Leaving the chat room
---
nickname property가 nil일 때만 다시 묻는다는 점을 기억하자. 그 외에는 할 이유가 없다.
이제 유저의 정보를 tableview에 뿌릴 차례다. starter 프로젝트안에 관련된 메소드가 거으 l완성되어 있는 tableview를 찾을 거 t이다. 하지만 각 셀에 적절한 데이터를 보여줄 로직은 아직 빠져 있다. 아래를 보자:

앞서 말한 것처럼, 유저가 앱 사용을 멈추거나 앱이 백그라운드로 들어갈 때 서버와의 연결이 끊어진다. 그러나 유저 정보는 서버의 레코드에 남아 있다(서버 프로그램이 아직 돌아가고 있는 한). 물론 해당 유저의 "isConnected" 플래그가 false이긴 할 것이다. 우리의 iOS앱에서 이는 Offline 표시로 유저의 tableview에 보여질 것이다.

내가 묘사했던 것을 사실 지금 당장 테스트할 수 있다. 테스트해보기 위해, app을 디바이스든 시뮬레이터든 어디든 설치하고 실행해야 한다. 두 개의 디바이스(시뮬레이터와 진짜 디바이스)에 앱을 실행하여 각각 다른 닉네임을 입력하자. 연결되고 나면, 각 디바이스에 두 명의 유저가 보일 것이다. 둘 모두 Online이라고 닉네임 옆에 표시되어 있다. 한 디바이스에서 앱을 종료하고 다른 디바이스에서는 그대로 두면, 종료된 디바이스의 유저가 목록에는 있지만 status가 Offline으로 바뀐 것을 볼 수 있다.

서버가 유저들의 배열에서 완전히 삭제하게끔 하려면, 우리는 SocketIOManager.class 안에 새로운 custom 메소드를 만들어야 한다. 그안에서 특정한 메시지를 우리가 지우고자 하는 유저의 닉네임 정보를 가지고 서버에 emit할 것이다. 우리한테 필요한건 그뿐이다.

그러니, SOcketIOManager.swift 파일을 열고 아래의 새 메소드를 복붙하자.

서버는 "exitUser"라는 메시지를 받고나면 명시된 유저를 삭제해야 한다는 것을 즉시 이해한다. 그게 정확히 서버가 하는 일이다.

UsersViewController.swift파일로 돌아가서, 바로 exitUser(:_) IBAction  메소드를 보자. 여기가 위의 코드가 불릴 곳이다(caller). 그 액션이 로그아웃 프로세스이기 때문에 그 메시지가 서버로 보내지고 나면 유저의 리스트에서 유저의 닉네임을 지우고 그의 닉네임을 다시 입력하라고 요청해야 한다. 다음처럼 하자

앱을 다시 테스트함으로써 한 디바이스에서 Exit 버튼을 눌렀을 때 그 유저가 즉시 다른 디바이스의 리스트에서 사라진다는 것을 볼 수 있다. 테스트 하기 전엔 각 디바이스에 우리가 작업한 모든 부분을 다시 인스톨해서 진행해야 된다는 것을 기억하자. 이전에 추가된 사용자들이 전부 제거되도록 서버를 재시작 (Ctrl-C 해서 정지, node index.js로 시작) 하는 것도 좋은 방법이다.

<br>
Chatting
---

사용자 간의 실제 채팅을 구현할 때다. 예상 하다시피, 서버에 메시지를 보내고 새로운 것을 받아오기 위해 새로운 메소드를 SOcketIOManager 클래스에 만들 것이다. 당연히 두 동작을 위해서는 새로운 메시지 리터럴을 사용할 것이고, 이를 서버도 어떻게 답해야 할지 안다. 그리고, 우리는 메시지를 보내고 앱에 띄우는 데 부족한 코드를 모두 넣을 것이다.

급한게 먼저니까, SOcketIOManager 클래스를 열자. 새로운 메소드를 정의하고 그 body는 한줄만으로 채울 것이다. 그 한줄은 채팅 메시지와 유저의 닉네임을 서버로 보내는 emit 명령이다.

서버가 “chatMessage”라는 메시지를 받으면, 이는 결국 연결된 모든 유저에게 표시된다. 이 모든 통신 과정은 한순간에 이루어진다. 그래서 새로운 어떤 메시지든 모든 유저에게 실시간으로 보여질 것이다. 그리고 이것이 우리가 SocketIO 라이브러리를 이용해 아카이브하길 원하는 이유이다.

실제 메시지 외에, 서버도 유저에게 다른 쓸모 있는 정보들을 보낸다: 메시지 발신자의 닉네임과 메시지의 일시. 그것을 염두에 두고 새롭게 들어오는 채팅 메시지를 listening하는 새로운 메소드를 구현하자.

socket.on(…) 클로져의 dataArray 배열은 세 개의 원소를 가지고 있다: 보낸이의 닉네임, 실제 메시지와 메시지의 시간이다. 이 모두는 string 값으로 보내질 것이다. 그리고 딕셔너리에 추가될 것이다. 이 딕셔너리는 completion handler를 통해 그 메소드의 콜러에게 리턴된다. 지금부터 계속해서 새로운 채팅 메시지가 올 때마다 on 메소드를 자동으로 콜하게 됨을 다시 한 번 기억하자.

채팅 메시지를 보내고 받는 메카니즘이 이제 존재한다. 그러니 이걸 app flow에 전부 합치는 작업을 하자. 먼저 이 튜토리얼에서는 ChatViewController.swift 파일을 열어 sendMessage(:_) IBAction method를 위치시키자. 유저의 닉네임은 세그웨이를 따라 UsersViewController에서 ChatViewController로 전달됨을 유념하자. 그래서 우리가 여기서도 유저의 닉네임을 이용할 수 있다.

IBAction 메소드에서 보낼 텍스트가 있다는걸 확인할 것이다. 메시지를 보내고 나면 우리는 textview를 clear시키고 마지막으로는 keyboard를 아래로 숨길 것이다.

이제, 우리는 새로운 메시지를 받기 위한 로직을 구현해야 한다. viewDidAppear(_:) 메소드에 다음을 추가하자.

위와 같이, 자세한 새 채팅 메시지는 chatMessages 배열에 append될 것이다. 그리고 tableview는 새 메시지가 chat feed에 표시되게끔 리로드될 것이다.

tableview에 대해 말하자면, 셀 내용이 적절하게 표시되도록 만들자. 카톡처럼 내 메시지가 우리의 다른 유저에게 보내질 때 셀 내용은 오른쪽에 align 될 것이다. 반대로 다른 유저가 메시지를 보내면 셀 내용이 왼쪽으로 align될 것이다.

축하한다! 챗 앱이 이제 거의 다 준비가 됐다. 그러나 더 흥미로운 것들이 많이 남아 있으니까 우리는 여기서 멈추지 않을 것이다.

앱을 다시 테스트하자. 각각의 디바이스에 설치해서, Join Chat 버튼을 누르고, 유저들 사이에서 메시지를 보내보자.

<br>
Being Notified When Users Join and Leave the Chat
---
채팅 어플에서는 유저가 채팅방에 들어오고 나갈 때 알려주는 것이 아주 유용하다. 우리의 데모 앱에서는, 이거를 popup 라벨을 졸라 짧은 시간동안 그 유저에게 보여주는 것으로 구현할 것이다.

SocketIOManager.swift 파일으로 돌아가서, 두 개의 새로운 메시지(“userConnectUpdate”, “userExitUpdate”)를 듣기 위한 새로운 메소드를 구현할 것이다. 전자는 새로운 유저의 닉네임이 서버에 전달되고 나서 연결될 때 서버에 의해 보내지는 것이다. 반면 두 번째는 유저가 앱을 종료할 때나 유저가 Exit 버튼을 눌러서 유저리스트에서 완전히 삭제될 때 보내진다.

추가적으로 위 메시지가 들어올 때 우리는 알림(NSNotifications)을 보낼 것이다. ChatViewController 클래스에 있는 알림들을 보게될 것이고, 최종적으로 적당한 메시지와 함께 팝업 라벨을 보여줄 것이다.

여기 당신이 SocketIOManager.swift에 만들어야 하는 새로운 메소드가 있다. 우리가 두 메시지를 listen할 메소드이다:

첫 번째 케이스에서 서버는 모든 유저의 정보(ID, 닉네임, 연결상태)를 가지고 있는 딕셔너리를 리턴한다. 두 번째 케이스에서 서버는 채팅방을 나간 사람의 닉네임만 리턴한다. 그러나 각 경우에 우리는 알림의 object property를 이용하는 각각의 정보를 보내준다.

위 메소드는 어디선가는 불릴 것이다. 그렇지 않으면 이 앱이 두 메시지를 받아들일 수 없다. 위 메소드를 유저가 서버에 연결되자마자 콜할 최적의 장소는 좀전에 우리가 만들었던 connectToServerwithNickname(_:completionHandler:) 메소드이다. 따라서 이렇게 바꾸자.

이제 ChatViewController.swift를 열어서 viewDidLoad(_:) 메소드를 보자. 우리가 여기서 해야할 것은 위의 두 noti를 관찰하는 것이다.

위 코드 조각은 알림들을 선택하기 위해 우리가 명시한 두 새로운 메소드이다. 이는 다음 작업으로 각 메소드들을 정의해야 함을 의미한다.

첫 번째 것을 위해, 연결된 유저의 닉네임을 알림의 object 프로퍼티로부터 추출할 것이다. 그리고 우리는 라벨의 텍스트를 명시할 것이고 트리거를 만들 것이다.

showBannerLabelAnimated 메소드 뿐만 아니라 lblNewsBanner IBOutlet 프라퍼티도 이미 프로젝트에 존재한다.

비슷한 방법으로 우리는 연결이 끊긴 유저의 닉네임을 표시하는 아래의 메소드를 구현할 것이다.

만약 당신이 앱을 다시 킨다면, chat view controller로 가서 다른 유저를 연결하거나 연결을 끊어보라; 적절한 메시지의 라벨이 몇 초 만에 나타났다가 화면 위로 사라질 것이다.

<br>
Being Notified When Users Type Messages
---
마지막 흥미로운 주제는 다른 유저가 메시지를 치고 있을 때 알려주는 기능이다. 사실 우리의 목표는 현재 메시지를 치고 있는 유저의 닉네임을 라벨에 보여주는 것이고 아무도 아무것도 치고 있지 않으면 이를 숨기는 기능이다. 이를 가능하게 하기 위해서 우리는 한 유저가 타이핑을 시작하거나 멈출 때 서버에 알릴 것이다. 그리고 그 결과로 우리는 타이핑하고 있는 모든 유저의 딕셔너리를 받게 된다.

이전까지 많이 해온 같은 로직을 따라서 SocketIOManager.swift 파일을 열고 메시지가 타입되고 있으면 새로운 메시지를 서버에 emit하는 새로운 메소드를 추가하자.

우리는 또한 유저가 타이핑을 멈출때에도 비슷한 메시지를 보낼 것이다. 예를 들어 swipe down 제스쳐가 만들어지고 키보드가 dismissed된다면 그런 상황이 일어날 수 있다.

우리가 타이핑을 시작하거나 멈출 때마다 위의 것들 중 하나가 호출됨으로써 서버는 결과적으로 모든 타이핑을 하고 있는 유저들을 알려줄 수 있다. 그러므로 서버는 그 메시지를 보내줄 것이고 따라서 우리는 그것을 듣고 있어야 한다. 아래 코드가 listenForOtherMessages() 메소드에 추가될 것임을 주의하자.

우리는 이제 마지막으로 ChatViewController.swift 파일로 전환할 준비가 되었다. viewDidLoad(_:)메소드에서 위 알림을 계속 관찰할 것이다.

handleUserTypingNotification(_:) 메소드의 구현할 때 타이핑중인 유저가 나라면 라벨을 보여주지 않도록 하자. 게다가 나머지는 쉽다. 서버에서 반환된 딕셔너리에 포함된 모든 사용자의 닉네임을 포함하는 문자열을 작성하고 출력 메시지를 올바르게 준비한다.

잘했다. 하지만 아직 유저가 타이핑 하고 있는지 아닌지를 서버에 알리지 않았다. 첫 번째 경우에는 textview의 textViewShouldBeginEditing(_:)로 가서 delegate 메소드로 이동해서 다음처럼 업데이트 하자.

유저가 키보드를 없애서 타이핑을 멈췄다는 것을 나타내려면, dismissKeyboard()라는 메소드를 찾아내서 라인을 한 줄 추가하자.

바로 그거다! 위는 우리의 마지막 산고였다. 남은 일은 앱을 한 번 더 테스트하고 다른 유저가 타이핑할 때 알림을 받는 것이다.

<br>
Summary
---

이 튜토리얼의 끝에 다다라서, 이 데모 앱을 통해 iOS용 Socket.IO 클라이언트 라이브러리가 얼마나 쉬운지와 어떻게 작동하는지를 가능한한 최상의 방법으로 보여줄 수 있었다고 생각한다. 이 튜토리얼의 각 부분에서 실시간으로 서버와 메시지를 송수신하기 위해 채팅 어플을 작성했다. 물론 이 채팅 앱은 실제 앱이 되는 것과는 거리가 멀지만, 우리가 관심을 가진 부분은 핵심 기능이므로 우리가 성공적으로 해냈다고 확신한다.

Socket.IO는 모든 종류의 작업에 적합한 슈퍼무기로 구성되어 있는 것이 아니고 서버와 통신하는 전통적인 방법을 대체할 수는 없다. 그러나 특정 사례에 대해서는 훌륭한 도구라는 것을 알았고, 실시간 솔루션이 필요할때는 최상의 옵션이라고 생각했다. 어쨌든, 당신이 읽은 것이 유용했고 새로운 무언가를 배웠길 바란다. 더욱이, 내가 이 글을 쓰는데 쓴 시간만큼 이 글을 읽는 것을 즐겼기를 바란다.

참고문헌은 https://github.com/appcoda/SocketIOChat여기서 전부 확인할 수 있다.