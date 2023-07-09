# ****Driving changes in your UI with state and bindings****

뷰에서 state를 사용하여 데이터 dependency들을 표시하고 바인딩을 통해 다른 뷰들과 해당 dependency들을 공유할 수 있습니다.

SwiftUI 앱의 UI는 뷰의 계층구조로 뷰를 구성합니다. 각각의 뷰는 몇 데이터에 의존성을 가집니다. 외부의 이벤트나 사용자 터치로 인해 해당 데이터가 변경되면 SwiftUI는 해당 변경사항을 반영하도록 뷰를 자동으로 업데이트합니다.

아래 예제는 State 변수를 사용해 데이터 dependency를 나타내고 Binding 프로퍼티 래퍼를 통해 다른 뷰와 데이터를 공유하는 방식을 보여줍니다.

## Section 1. Seperate properties and imperative code from view

뷰가 여러가지의 상태 데이터를 관리해야 하는 경우, 각각의 뷰 내에서 해당 데이터를 관리하는 것이 도움될 수 있습니다. 이러한 접근은 선언형 인터페이스 코드를 더 읽기 쉽게 만드는 것과 상태 변경을 쉽게 구현하는 유닛테스트를 만드는 데 도움이 됩니다.

```swift
// RecipeEditorConfig.swift

import Foundation

struct RecipeEditorConfig {
		// RecipeEditor 뷰에서 보여질 데이터를 저장합니다
    var recipe = Recipe.emptyRecipe()
    var shouldSaveChanges = false
    var isPresented = false

		// 아래 메서드는 사용자가 Add Recipe 버튼을 탭 한 경우 호출됩니다
		// 뷰의 상태를 변경해 새 레시피를 편집합니다
    mutating func presentAddRecipe(sidebarItem: SidebarItem) {
				// 편집할 레시피로서 empty recipe를 만듭니다
				// 정적메서드인 emptyRecipe()는 Recipe의 새로운 인스턴스를 반환합니다
        recipe = Recipe.emptyRecipe()

        switch sidebarItem {
        case .favorites:
            // Associate the recipe to the favorites collection.
            recipe.isFavorite = true
        case .collection(let name):
            // Associate the recipe to a custom collection.
            recipe.collections = [name]
        default:
            // Nothing else to do.
            break
        }

        shouldSaveChanges = false
        isPresented = true
    }

		// presentAddRecipe(sidebarItems:)와 비슷한 동작을 하지만, 현 메서드는 이미 있는 레시피를 편집하는 동작을 함
    mutating func presentEditRecipe(_ recipeToEdit: Recipe) {
        recipe = recipeToEdit
        shouldSaveChanges = false
        isPresented = true
    }

		// 레시피에 대한 변경사항을 저장하고 RecipeEditor 뷰를 닫도록 설정
    mutating func done() {
        shouldSaveChanges = true
        isPresented = false
    }

		// 레시피에 대한 변경사항을 저장하지 않고 RecipeEditor 뷰를 닫도록 설정    
    mutating func cancel() {
        shouldSaveChanges = false
        isPresented = false
    }
}
```

## Section 2. Bind the view to its state data

recipe editor가 필요로하는 데이터를 포함한 구조체와 editor의 상태를 변경하는 메서드가 있는 상태에서 `RecipeEditor` 뷰가 어떻게 `RecipeEditorConfig`를 이용하는지 알아봅시다

```swift
// RecipeEditor.swift
import SwiftUI

// RecipeEditor는 View 프로토콜을 채택합니다
struct RecipeEditor: View {

		// 뷰에서 보여지기 위해 사용하는 데이터들을 포함하고있는 RecipeEditorConfig 타입을 config 라는 바인딩 변수로 선언합니다
		// Binding 프로퍼티 래퍼는 뷰에 필요한 데이터의 양방향의 읽기-쓰기 바인딩을 제공합니다
		// 그렇지만 RecipeEditor는 데이터를 소유하지 않고 다른 뷰가 RecipeEditor가 바인딩하고 사용하는 RecipeEditorConfig의 인스턴스를 생성하고 소유합니다
    @Binding var config: RecipeEditorConfig

    var body: some View {
        NavigationStack {
							// 레시피를 편집하는데 필요한 인풋필드를 가지고있는 RecipeEditorForm이 있습니다
							// RecipeEditor는 바인딩 변수인 config를 RecipeEditorForm에 전달합니다
							// 바인딩 변수임을 나타내도록 앞에 $를 붙입니다
							// RecipeEditorForm은 바인딩 변수로 config를 받아, 받은 config 데이터에 대해 읽고 쓰는 작업을 합니다
            RecipeEditorForm(config: $config)
                .toolbar {
                    ToolbarItem(placement: .principal) {
                        Text(editorTitle)
                    }

											// 사용자가 취소버튼을 탭하면 RecipeEditorConfig에 정의된 cancel() 메서드를 호출해 shouldSaveChanges 변수와 isPresented 변수의 상태를 변경합니다
                    ToolbarItem(placement: cancelButtonPlacement) {
                        Button {
                            config.cancel()
                        } label: {
                            Text("Cancel")
                        }
                    }

                    ToolbarItem(placement: saveButtonPlacement) {
                        Button {
                            config.done()
                        } label: {
                            Text("Save")
                        }
                    }
                }
            #if os(macOS)
                .padding()
            #endif
        }
    }

    private var editorTitle: String {
        config.recipe.isNew ? "Add Recipe" : "Edit Recipe"
    }

    private var cancelButtonPlacement: ToolbarItemPlacement {
        #if os(macOS)
        .cancellationAction
        #else
        .navigationBarLeading
        #endif
    }

    private var saveButtonPlacement: ToolbarItemPlacement {
        #if os(macOS)
        .confirmationAction
        #else
        .navigationBarTrailing
        #endif
    }
}
```

## Section 3. Create a state variable in another view

`RecipeEditor` 뷰는 RecipeEditorConfig를 바인딩 변수로 가지고 있습니다. 에디터 뷰는 가지고있는 바인딩 데이터를 읽고 쓸 수 있으나 에디터 자체의 레시피 데이터를 소유하지는 않습니다.대신, `ContentListView`가 자기 자신의 데이터를 생성하고 소유합니다. 그리고 SwiftUI는 content list view의 생명주기에서 해당 데이터를 관리합니다.

```swift
// ContentListView.swift

struct ContentListView: View {
    @Binding var selection: Recipe.ID?
    let selectedSidebarItem: SidebarItem
    @EnvironmentObject private var recipeBox: RecipeBox
		// State 프로퍼티래퍼는 RecipeEditorConfig 인스턴스를 생성하고 관리하는 역할을 한다
		// 뷰의 상태가 변경될때마다(recipeEditorConfig에 포함된 데이터가 변경되면) SwiftUI는 뷰를 다시 초기화하고 body 내의 뷰를 다시 빌드해 데이터의 현재 상태를 뷰에 반영함
    @State private var recipeEditorConfig = RecipeEditorConfig()

    var body: some View {
        RecipeListView(selection: $selection, selectedSidebarItem: selectedSidebarItem)
            .navigationTitle(selectedSidebarItem.title)
            .toolbar {
                ToolbarItem {
                    Button {
                        recipeEditorConfig.presentAddRecipe(sidebarItem: selectedSidebarItem)
                    } label: {
                        Image(systemName: "plus")
                    }
                    .sheet(isPresented: $recipeEditorConfig.isPresented,
                           onDismiss: didDismissEditor) {
										// RecipeEditor는 $recipeEditorConfig를 전달받아 해당 데이터를 읽거나 쓸 수 있음
                        RecipeEditor(config: $recipeEditorConfig)
                    }
                }
            }
    }

    private func didDismissEditor() {
        if recipeEditorConfig.shouldSaveChanges {
            if recipeEditorConfig.recipe.isNew {
                selection = recipeBox.add(recipeEditorConfig.recipe)
            } else {
                recipeBox.update(recipeEditorConfig.recipe)
            }
        }
    }
}
```

---

# **Creating a custom input control that binds to a value**

값에 바인딩되는 커스텀 컨트롤을 통해 유니크한 인터랙션을 제공합니다.

SwiftUI는 값을 바인딩해 인터랙션시 값을 변경할 수 있는 Slider나 TextField같은 input 컨트롤을 제공하는데요, 모든 앱마다 각각 다른 인터랙션을 주고받을 수 있도록 만들 수 있습니다.

## Section 1. Design a custom control

커스텀 컨트롤을 구현하기 전에, 컨트롤에 필요한 데이터가 무엇인지, 해당 데이터로 무엇을 하는지, 앱 내에서 해당 데이터를 어떻게 보여줄건지를 정해야합니다.

```swift
// StarRating.swift

import SwiftUI

struct StarRating: View {
		// 레시피의 등급을 표시할 rating이라는 이름의 Binding 변수를 선언합니다
		// Binding 프로퍼티래퍼를 통해 다른 뷰가 값을 생성하더라도 여기에서 읽고 쓸 수 있습니다
    @Binding var rating: Int
    private let maxRating = 5

    var body: some View {
        HStack {
							// HStack 내에서 maxRating만큼 별의 갯수를 생성합니다
            ForEach(1..<maxRating + 1, id: \.self) { value in
                Image(systemName: "star")
                    .symbolVariant(value <= rating ? .fill : .none)
                    .foregroundColor(.accentColor)
                    .onTapGesture {
                        if value != rating {
                            rating = value
                        } else {
                            rating = 0
                        }
                    }
            }
        }
    }
}
```

## Section 2. Make the control interactive

StarRating은 레시피의 등급을 나타내기 위해 사용되는데, 사용자의 인터랙션을 받기 위해 onTapGesture(count:perform:) 메서드를 사용합니다.

```swift
// StarRating.swift

import SwiftUI

struct StarRating: View {
    @Binding var rating: Int
    private let maxRating = 5

    var body: some View {
        HStack {
            ForEach(1..<maxRating + 1, id: \.self) { value in
                Image(systemName: "star")
                    .symbolVariant(value <= rating ? .fill : .none)
                    .foregroundColor(.accentColor)
											// Image 인스턴스에 사용자와 인터랙션할 수 있는 onTapGesture 메서드를 연결함
// 탭 제스처를 입력받았을 때의 value를 rating으로 주입하여 입력받은 value대로 rating을 업데이트 함
                    .onTapGesture {
                        if value != rating {
                            rating = value
                        } else {
                            rating = 0
                        }
                    }
            }
        }
    }
}
```

## Section 3. Display the custom control in other views

커스텀 input control을 사용하는 방법에 대해 알아봅니다.

```swift
// RegularTitleView.swift

import SwiftUI

// 레시피의 제목과 부제목을 등급과 함께 표시하는 뷰
struct RegularTitleView: View {
		// 다른 뷰에서 받은 레시피를 저장하는 바인딩 변수를 정의
		// 이 바인딩을 통해서 뷰가 Recipe의 인스턴스의 데이터를 읽고 쓸 수 있으나 뷰가 이 recipe 변수의 소유자는 아님
    @Binding var recipe: Recipe

    var body: some View {
        VStack(alignment: .leading) {
            Text(recipe.title)
                .font(.largeTitle)
						// recipe.rating을 StartRating에 전달하여 StartRating이 해당 속성을 읽고 쓸 수 있도록 함. $ 사인은 바인딩 변수를 전달하고 있음을 나타냄
						// 유저 인풋이 있는 경우, SwiftUI는 뷰를 다시 그려서 변경된 rating을 반영하도록 함
            StarRating(rating: $recipe.rating)
        }
        Spacer()
        Text(recipe.subtitle)
            .font(.subheadline)
    }
}
```

# ****Defining the source of truth using a custom binding****

### Section 1. Specifying the source of truth

```swift
import SwiftUI

struct DetailView: View {
    @Binding var recipeId: Recipe.ID?
    @EnvironmentObject private var recipeBox: RecipeBox
    @State private var showDeleteConfirmation = false

		/*
recipe 데이터를 가져오기 위해 State 변수를 선언하는 대신 계산 속성의 recipe를 선언함.
		recipe 변수는 Recipe를 반환하지 않고 Binding 타입의 Recipe를 반환하는데,
이렇게 하면 source of truth의 역할로서 상태 공유가 가능해짐
*/
    private var recipe: Binding<Recipe> {
// Binding은 값에 대한 읽기, 쓰기 권한을 제공하는데 init(get:set:) 메서드를 통해서 recipe 변수를 정의함
        Binding {
							// id를 이용해 레시피 검색
            if let id = recipeId {
                return recipeBox.recipe(with: id) ?? Recipe.emptyRecipe()
            } else {
                return Recipe.emptyRecipe()
            }
        } set: { updatedRecipe in
						// 사용자가 레시피 등급등을 변경했을 때 호출됨
            recipeBox.update(updatedRecipe)
        }
    }

    var body: some View {
        ZStack {
            if recipeBox.contains(recipeId) {
								// recipe를 전달함. Binding 자체를 전달하기 때문에 $사인이 붙지 않음
                RecipeDetailView(recipe: recipe)
                    .navigationTitle(recipe.wrappedValue.title)
                    #if os(iOS)
                    .navigationBarTitleDisplayMode(.inline)
                    #endif
                    .toolbar {
                        RecipeDetailToolbar(
                            recipe: recipe,
                            showDeleteConfirmation: $showDeleteConfirmation,
                            deleteRecipe: deleteRecipe)
                    }
            } else {
                RecipeNotSelectedView()
						}
        }
    }

    private func deleteRecipe() {
        recipeBox.delete(recipe.id)
    }
}
```
