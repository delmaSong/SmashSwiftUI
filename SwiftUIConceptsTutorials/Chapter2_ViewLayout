# View Layout
## Scailing views to complement text
### Section 1. Associate content with the text
문자열과 심볼을 동시에 사용한 `Label`에 대해 알아봅니다. 그리고 일관된 margin을 사용하기 위해 `background(alignment:content:)` 내에서 `Capsule`을 사용합니다.

```swift
import SwiftUI

struct KeywordBubbleDefaultPadding: View {
		// 재사용할 수 있도록 프로퍼티를 정의
    let keyword: String
    let symbol: String
    var body: some View {
				// 문자열과 이미지를 한 레이블에 나타낼 수 있도록 생성자가 마련되어있음
				// 심볼의 크기를 조정하고 컨텐츠를 알아서 정렬하기 때문에 수동으로 정렬 할 필요가 없음
        Label(keyword, systemImage: symbol)
						// 텍스트와 이미지에 모두 동일한 글꼴을 적용하게 됨
            .font(.title)
            .foregroundColor(.white)
						// 아무 파라미터도 넘기지 않으면 상하좌우에 디폴트 패딩이 생김
            .padding()
						// Capsule 사용을 통해 둥근 사각형 모양의 배경을 만들 수 있음
						// Capsule이 레이블의 텍스트와 심볼 뒤에 보여지도록 background(alignment:content:) 모디파이어 내부에 Capsule을 정의함
						// background 모디파이어가 padding() 뒤에 있기 때문에 Capsule의 크기에는 라벨 주변의 패딩영역이 포함됨
            .background(.purple.opacity(0.75), in: Capsule())
    }
}
```

### Section 2. Preview a custom view in Xcode

Xcode는 코드가 변경될 때 레이아웃을 미리 볼 수 있도록 캔버스를 제공합니다. SwiftUI에서 커스텀뷰의 미리보기를 확인하려면 `PreviewProvider` 프로토콜을 채택하면 됩니다. 다양한 조건에서 어떻게 보여지는지 확인할 수 있습니다.

```swift
// 타입변수인 previews가 정의된 PreviewProvider를 채택
struct KeywordBubbleDefaultPadding_Previews: PreviewProvider {
		// preview에서 보여질 정적 데이터 정의
    static let keywords = ["chives", "fern-leaf lavender"]
    static var previews: some View {
        VStack {
            ForEach(keywords, id: \.self) { word in
								// 확인하고자 하는 KeywordBubbleDefaultPadding에 키워드와 심볼을 제공하여 뷰를 생성함
                KeywordBubbleDefaultPadding(keyword: word, symbol: "leaf")
            }
        }
    }
}
```

Dynamic Text Size의 full range를 확인하려면 아래처럼 보기 설정을 변경하면 됨 (Variants > Dynamic Type Variants)

<img width="263" alt="스크린샷 2023-06-18 오후 9 26 36" src="https://github.com/TheSwiftists/effective-swift/assets/40784518/ba228a5d-26d6-49cc-adc2-7a1e23bccaaf">

### Section 3. Adjust dimensions with ScaledMetric
`pdding`, `width`, `height` 같은 값이 모든 환경에서 적절하게 보여지는 것은 아닙니다.
numeric value의 scale을 조정하는 dynamic 프로퍼티인 `ScaledMetric`을 사용해서 각 뷰의 다양한 조건에서 보여질 미세한 수치들을 조정할 수 있습니다.

```swift
import SwiftUI

struct KeywordBubble: View {
    let keyword: String
    let symbol: String
		// 이 프로퍼티래퍼는 dynamicTypeSize에 따라 적절하게 더 크거나 작게 값을 조정함
    @ScaledMetric(relativeTo: .title) var paddingWidth = 14.5
    var body: some View {
        Label(keyword, systemImage: symbol)
            .font(.title)
            .foregroundColor(.white)
						// padding()을 이용하면 기본 패딩값이 지정되는데, ScaledMetric인 값을 지정하면 label이 더 큰 텍스트크기를 사용할 때 적절하게 더 많은 패딩값을 추가함
            .padding(paddingWidth)
            .background {
                Capsule()
                    .fill(.purple.opacity(0.75))
            }
    }
}
```

## Layering content

<img width="30%" alt="스크린샷 2023-06-18 오후 9 26 04" src="https://github.com/TheSwiftists/effective-swift/assets/40784518/ec368a88-9b0a-4e91-82bb-5732dd57be26">

### Section 1. Define an overlay
z축으로 컨텐츠를 정렬할 때 ZStack을 사용할 수도 있고 `overlay(alignment:content:)`나 `background(_:in:fillStyle:)`같은 모디파이어를 사용할 수 있습니다.
ZStack은 스택 내의 다른 뷰들에 대한 고려 없이 이용가능한 공간만을 기반으로 사이즈를 잡습니다. 다른 컨텐츠의 크기가 다른 컨텐츠의 크기에 따라 달라지도록 지정하려면, secondary 컨텐츠 내에서 overlay나 background 모디파이어를 정의합니다.

```swift
import SwiftUI

struct CaptionedPhoto: View {
    let assetName: String
		//Captionq 뷰에 전달할 문자열
    let captionText: String
    var body: some View {
				// 앱에서 해당 이름의 사진이나 그래픽을 가져와 초기화하고 display 함
        Image(assetName)
						// Image는 기본적으로 이미지의 오리지널 사이즈대로 display 함
						// resizable()과 scaleToFit()은 가용한 범위 내에서 이미지를 fit함
            .resizable()
            .scaledToFit()
						// 해당 모디파이어의 클로저 안에 캡션을 정의하면 캡션이 이미지 앞에 속하게 됨
						// primary 뷰의 사이즈에 따라 overlay되는 사이즈도 조정됨
            .overlay(alignment: .bottom) {
                Caption(text: captionText)
            }
						// RoundedRectangle을 이용해 클리핑하면 이미지의 사이즈나 포지션을 변경하지 않고도 모서리를 둥글게 만들 수 있음
            .clipShape(RoundedRectangle(cornerRadius: 10.0, style: .continuous))
            .padding()
    }
}
```

### Section 2. Define a background
Caption 뷰는 `background(_:in:fillStyle:)` 모디파이어를 사용해 텍스트 뒤에 있는 컨텐츠를 부분적으로 가리는 모양으로 더 높은 대비를 제공합니다.

```swift
import SwiftUI

struct Caption: View {
    let text: String
    var body: some View {
        Text(text)
						// 텍스트와 그 아래 배경의 가장자리에 약간의 공간을 추가하는데, 코드의 구조와 뷰에서 그려지는 모습을 일치해서 생각하면 됨. 패딩은 텍스트와 배경 사이에 존재함.
            .padding()
						// background 수정자는 컨텐츠가 수정하는 뷰의 크기에 기반한다는 점에서 overlay 수정자와 유사함. 그러나 background는 컨텐츠를 뷰의 뒤에 넣고, overaly는 앞에 둔다는 점이 다름.
            .background(
								Color("TextContrast").opacity(0.75),
                in: RoundedRectangle(cornerRadius: 10.0, style: .continuous)
						)
            .padding()
    }
}
```
