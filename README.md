# ios_sdk
iOS SDK Guide

어드민에 매체를 등록한 후 APP_ID 를 발급 받는다.
APP_ID 를 info.plist 파일에 등록한다.



# 초기화
## S2Offerwall
- @objc public static func initSdk(listener:InitializeListener?)
- InitializeListener
```swift
@objc
public protocol InitializeListener : NSObjectProtocol {
    
    @objc optional func onSuccess()
    
    @objc optional func onFailure()
}
```
- 호출 예시
```swift
class ViewController: UIViewController, S2Offerwall.InitializeListener {
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        S2Offerwall.initSdk(listener: self)
    }

    func onSuccess() {
        NSLog("on Success")
    }
    
    func onFailure() {
        NSLog("on Failure")
    }
}
```

또는

- @objc public static func initSdk(onCompletion:@escaping (Bool) -> ())
- 호출 예시
```swift
class ViewController: UIViewController {
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        S2Offerwall.initSdk() { result in
            if (result) {
                NSLog("init succcess")
            }
            else {
                NSLog("init failure")
            }
        }
    }
}






``` 
- EventListener
```java
public interface EventListener {
    // login://param
    // 오퍼월에서 광고 참여 시점에는 사용자 정보가 필요하다. 이때 매체의 경우 로그인 아이디를 설정해야한다.
    // 
    void onLoginRequested(String param);
    
    // close://param
    // 오퍼월이 닫혀야 할 경우 아래 이벤트가 호출된다. 매체에서는 적절한 액션을 수행하도록 구현한다.
    void onCloseRequested(String param);
    }
```
- 호출예시
```java
S2Offerwall.setEventListener(new S2Offerwall.EventListener() {
    @Override
    public void onLoginRequested(String param) {
        S2Log.d("## login requested " + param);
        S2Offerwall.setUserName(MainActivity.this, "testuser");
    }

    @Override
    public void onCloseRequested(String param) {
        S2Log.d("## close requested " + param);
    }
});
```
# 사용자 정보 설정
광고 참여를 위해서는 사용자 정보가 필요하다. 일반적으로 매체의 로그인 ID 를 사용한다.

- S2Offerwall.setUserName(Context context, String userName)

설정한 사용자 정보를 삭제하기 위해서는 아래 API 를 호출한다.
- S2Offerwall.resetUserName(Context context)
  
# S2OfferwallActivity
오퍼월을 Activity 로 띄워준다. 어드민에서 설정한 플레이트먼트 명칭을 파라메터로 전달해야한다.
- S2Offerwall.startActivity(Context context, String placementName)
- 호출예시
```java
S2Offerwall.startActivity(MainActivity.this, S2Offerwall.MAIN);
```

# S2OfferwallView
오퍼월을 View 로 제공한다. S2OfferwallView 를 생성하여 화면에 추가한 후에 EventListener를 설정하고 이후 표시할 플레이스명을 호출한다.
- public void load(final String placementName)

- 사용예시

```java
private S2OfferwallView createOfferwallView() {
    S2OfferwallView offerwallView = new S2OfferwallView(this, null);
    ViewGroup.LayoutParams layoutParams = new ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.MATCH_PARENT);

    offerwallView.setLayoutParams(layoutParams);

    //setContentView(offerwallView);
    ViewGroup offerwallLayout = findViewById(R.id.offerwall_layout);
    offerwallLayout.addView(offerwallView);

    offerwallView.setEventListener(new S2Offerwall.EventListener() {
        @Override
        public void onLoginRequested(String param) {
            // ...
        }

        @Override
        public void onCloseRequested(String param) {
            // ...
        }
    });

    return offerwallView;
}


S2OfferwallView offerwallView = createOfferwallView();
offerwallView.load("placementName");

```

# 개인 정보 수집 동의
오퍼월 최초 사용시 개인 정보 수집 동의 다이알로그가 뜨고 여기에서 동의를 해야만 오퍼월로 진입하게 된다. 
S2Offerwall.startActivity() 를 호출하는 경우 내부에서 이에 대한 처리를 진행하지만 만약 S2OfferwallView 를 사용하여 직접 오퍼월 View 를 띄우는 경우에는
개인 정보 수집 동의 다이알로그를 띄우도록 구현을 해야한다. 아래의 구현 예시를 참고하여 구현한다.

```java

S2Offerwall.showConsentDialog(MyActivity.this, () -> {
    // 오퍼월 view 를 띄운다.
});

```

만약 자체적으로 오퍼월 관련 개인 정보 수집동의를 받는 경우에는 SDK가 제공하는 동의 창을 비활성화해야한다. 아래의 API 를 사용하여 동의 창 기능을 on/off 할 수 있다.

- S2Offerwall.setConsentDialogRequired(Context context, boolean required)


