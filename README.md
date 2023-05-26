# 01.SwiftUI Essentials
- 작성된 코드가 언제나 뷰의 the source of truth로서 역할을 하기 때문에 Inspector를 통해 속성을 변경하면 코드에 바로 반영됨

### App Protocol
앱의 동작과 구조를 표현하는 타입
- App 프로토콜을 따르는 구조체를 선언해서 앱을 만들면, 계산 속성인 `body` 내에 앱의 컨텐츠들을 정의해야 함
- 앱의 시작점임을 나타내기 위해 `@main` 어트리뷰트와 함께 사용되고, App 프로토콜이 `main()`을 제공함
- App 프로토콜의의 `body` 부분은 Scene 프로토콜을 따르는 인스턴스로 구성되어있고 각각의 scene은 뷰 계층구조의 root view를 포함함. 그리고 시스템에 의해 관리되는 life cycle을 가지고 있음
- 모든 scene을 통해 공유되는 state를 선언할 수 있음
    - `@StateObject`를 이용해 데이터 모델을 초기화하고 `@ObservedObject`나 `@EnvironmentObject`를 통해 뷰에 해당 모델을 제공할 수 있음

### @EnvironmentObject
- `@EnvironmentObject`
    - 어트리뷰트를 통해 상위 뷰에서 데이터를 수신하기 위해 뷰 계층 구조에서 하위에 있는 뷰에서 이 속성을 사용
- `environmentObject(_:)`
    - 뷰 하위 계층에 ObservableObject를 전달하기 위해 상위에 있는 뷰에서 사용

### 프리뷰에서 디바이스별 확인하는 방법
```swift
struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        ForEach(["iPhone SE (2nd generation)", "iPhone XS Max"], id: \.self) { deviceName in
                    LandmarkList()
                        .previewDevice(PreviewDevice(rawValue: deviceName))
                        .previewDisplayName(deviceName)
                }
    }
}
```

## List 사용하기
- `List`는 identifiable한 데이터로 동작함
- 목록에서 두개 이상의 동적 뷰를 보여주거나 정적 뷰와 동적 뷰를 같이 보여주어야 할 때는 `List`가 아니라 `ForEach`를 사용하고 값도 `ForEach`에 전달

### 데이터를 identifiable하게 만드는 방법 두가지
1. 데이터가 `Identifiable` 프로토콜을 준수하게 만듦

    ```swift
    // Landmark가 Identifiable 을 따를 때
    List(landmarks) { landmark in ... }
    ```

2. 데이터를 고유하게 식별하는 속성을 key path로 전달
    ```swift
    // Landmark가 Identifiable 을 따르지 않지만 구별 가능한 유일한 속성이 있을 때
    List(landmarks, id: \.id) { landmark in ... }
    ```
