# Model Data

## @State

읽고 쓸 수 있는 값으로 SwiftUI에서 관리되는 프로퍼티 래퍼 타입으로, 뷰 계층에 저장하는 given type으로 single source of truth로서 state를 사용함.(@State로 감싸서 선언된 프로퍼티가 single source of truth로서 역할한다는 뜻인듯)

`App`, `Scene`, `View`에서 `@State`를 적용한 프로퍼티를 선언해 초기값을 가지고있는 state를 만듦. SwiftUI는 프로퍼티의 저장공간을 관리하는데, 값이 변경되면 SwiftUI는 값에 종속된 뷰 부분을 업데이트 함. state의 기본 값에 접근하려면 `wrappedValue` 속성을 사용함. 그렇지 않고 래핑된 값 자체를 이용할 수도 있음.

state 프로퍼티를 private으로 선언해서 SwiftUI가 제공하는 스토리지와 충돌할 수 있는 멤버와이즈 이니셜라이저에서 해당 상태를 설정하지 않도록 해야 함.

값이 액세스해야 하는 뷰 계층의 가장 상위 뷰에서 state를 private으로 선언하고, 읽기 전용 액세스를 위해 바인딩으로 액세스가 필요한 모든 하위 뷰와 상태를 공유하면 모든 스레드에서 state 속성을 안전하게 변경할 수 있음.

(클래스의 인스턴스와 같은 참조 형식을 저장해야 하는 경우 `StateObject`를 사용함)

state 프로퍼티를 하위 뷰에 전달하면 SwiftUI는 값이 변경될 때마다 하위 뷰를 업데이트 하지만 하위 뷰는 값을 수정할 수 없음. **하위 뷰가 저장된 상태의 값을 수정할 수 있도록 하려면 State가 아니라 `Binding`을 전달해야 함.** 상태 값에 대한 바인딩은 state의 `projectedValue`에 접근해서 가져올 수 있고, 속성 이름 앞에 달러 사인($)을 붙여야 함.

```swift
// @State를 사용할 때
struct PlayButton: View {
    @State private var isPlaying: Bool = false // Create the state.

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") { // Read the state.
            isPlaying.toggle() // Write the state.
        }
    }
}

// @Binding을 사용할 때
struct PlayButton: View {
    @Binding var isPlaying: Bool // Play button now receives a binding.

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}

// @Binding을 사용하면서 ${변수명}을 통해 하위 뷰에 해당 속성 값을 전달할 때
struct PlayerView: View {
    @State private var isPlaying: Bool = false // Create the state here now.

    var body: some View {
        VStack {
            PlayButton(isPlaying: $isPlaying) // Pass a binding.
            // ...
        }
    }
}
```

StateObject와 마찬가지로, SwiftUI가 제공하는 스토리지 관리와 충돌할 수 있는memberwise 이니셜라이저에서 setting하는 것을 막기위해 State를 private으로 선언하자..

그리고 state 프로퍼티는 늘 기본값을 제공해서 초기화해야 함. 그리고 로컬 스토리지나 선언된 뷰와 하위 뷰에서만 state 프로퍼티를 사용함.

### @State 다섯 줄 요약

1. `private`으로 선언한다
2. 선언 시 기본값을 설정한다
3. 해당 state 프로퍼티가 전달된 부분에서는 읽기만 가능하고 수정은 할 수 없다(그러기 위해서는 Binding 변수를 사용하거나 state 변수의 `projectedValue`를 전달한다)
4. 클래스의 인스턴스에 `@State`를 이용하려면 `@StateObject`를 사용한다
5. 값 타입의 데이터를 국한된 UI에서 일시적인 상태를 나타낼 때 주로 사용한다(하위 뷰에 의해 값이 변경된다던지 하지 않을 때)

---

## @Binding

원본(source of truth)을 읽고 쓸 수 있도록 하는 프로퍼티 래퍼

데이터를 저장하는 프로퍼티와 뷰에 데이터를 보여주고 데이터를 변경하는 프로퍼티가 양방향으로 연결되도록 Binding을 하는 역할. 바인딩은 데이터를 직접 저장하는 대신, 다른 곳에 저장된 source of truth에 프로퍼티를 연결하는 역할을 함.

---

## @Environment

뷰의 environment에서 값을 읽어오는 프로퍼티 래퍼로, 프로퍼티 선언부에서 EnvironmentValues 키 패스를 이용해 읽을 값을 나타낼 수 있음.

아래처럼 colorScheme 프로퍼티에서 colorScheme 키패스를 사용해 현재 뷰에서 사용되는 ColorScheme을 읽어올 수 있음

```swift
@Environment(\.colorScheme) var colorScheme: ColorScheme
```

값이 변경되면 SwiftUI는 이 값에 의존하는 모든 뷰를 업데이트 함.

이 프로퍼티 래퍼를 사용해서 environment 값을 읽어올 수는 있지만 설정할 수는 없음. SwiftUI는 시스템 설정에 따라서 일부 environment 값을 자동으로 업데이트하거나 적절한 기본값을 제공하는 등의 역할을 함. 이 중 일부를 재정의 할 수 있는데, view의 `environment(_:_:)` 모디파이어를 통해서 할 수 있음.

SwiftUI가 제공하는 environment 값의 전체 목록은 [EnvironmentValues](https://developer.apple.com/documentation/swiftui/environmentvalues) 구조체의 프로퍼티를 참고하면 되고, 유저 커스텀 environment 값을 만드려면 [EnvironmentKey](https://developer.apple.com/documentation/swiftui/environmentkey) 프로토콜을 참고하면 됨

---

# @Observable

커스텀 타입 선언시 해당 매크로를 선언하여 관찰 가능한 상태로 만듦.
이 매크로는 컴파일 타임에 해당 타입이 관찰 가능하도록 코드를 generate 하고 해당 타입의 저장 속성에 초점을 맞춤. (→ 보통 해당 타입의 저장 속성을 관찰하나 봄)
참조타입과 값타입 모두에서 사용 가능함. 이 observable 매크로를 이용한 디자인 패턴을  Observation이라고 함. ([참고 링크](https://developer.apple.com/documentation/Observation))

```swift
@Observable
class Car {
    var name: String = ""
    var needsRepairs: Bool = false

    init(name: String, needsRepairs: Bool = false) {
        self.name = name
        self.needsRepairs = needsRepairs
    }
}
```

변경사항을 추적하려면 `[withObservationTracking(apply:onChange:)](https://developer.apple.com/documentation/observation/withobservationtracking(_:onchange:))` 함수를 사용하면 됨.

다음 코드에서는 `car`의 `name`이 변경될 때 `onChange` 클로저를 호출함. 하지만 `needRepairs`가 변경된 경우에는 호출되지 않는데, 왜냐면 `apply` 인자로 넘긴 클로저 부분에서 `name`만 읽고 있고 `needRepairs`는 읽고있지 않기 때문.

```swift
func render() {
    withObservationTracking {
        for car in cars {
            print(car.name)
        }
    } onChange: {
        print("Schedule renderer.")
    }
}
```
