# Chapter 1. App principles

SwiftUI는 선언형 프레임워크로, 앱의 구조를 구성하는 기본 구성 요소는 `App`, `Scene`, `View` 프로토콜이다.

### Section 1. App Structure

App Structure는 앱의 컨텐츠와 동작을 설명하며, SwiftUI 앱에는 하나의 기본 App Structrue만 존재 함.

```swift
import SwiftUI

// SwiftUI 앱의 진입점을 나타내기위해 App Structure에 @main을 적용함
// SwiftUI 앱에는 하나의 진입점만 포함되어 있음
// 둘 이상의 stucture에 @main을 적용하려 하면 컴파일 오류가 발생함
@main

// 앱의 기본이 되는 struct는 App 프로토콜을 따라야 함
struct MyApp: App {

		// App 프로토콜의 요구사항인 body를 구현해야 함
		// Scene으로 설명된 앱의 컨텐츠를 반환하는데, 이 안에는 UI를 정의하는 뷰 계층구조가 포함되어있음
		// WindowGroup, DocumentGroup, Settings같은 다양한 유형의 Scene을 제공할 수 있음
    var body: some Scene {
				// SwiftUI는 WindowGroup에 대한 플랫폼별 동작을 제공함
				// macOS, iPadOS에서 한 사람이 그룹에서 두 개 이상의 윈도우를 열 수 있음
				// macOS에서는 사용자가 WindowGroup의 여러 인스턴스를 탭 세트로 결합할 수 있음
        WindowGroup {
						// Scene에는 뷰 계층을 만드는 사용자 정의 뷰인 ContentView가 포함되어 있음
            ContentView()
        }
    }
}
```

### Section 2. Content View

SwiftUI에서 Scene에는 앱이 사용자 인터페이스로 표시하는 뷰 계층구조가 포함되어 있음.

뷰 계층은 다른 뷰와 관련된 뷰의 레이아웃을 정의함. 해당 샘플에서 WindowGroup의 Scene에는 ContentView가 다른 뷰를 사용해 구성된 뷰의 계층구조가 포함되어 있음
