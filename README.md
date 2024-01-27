# SwiftUI Best Practices

This documents contains a collection of best practices for SwiftUI, Swift 5+ and iOS 17+.

⛔️ = Bad Practice (MR/PR won't be approved)

🆗 = Not Optimal (MR/PR may be approved)

✅ = Better (MR/PR will be approved)

❕ = Related (similar concepts)

- [SwiftUI](#swiftui)
    1. [Image Assets](#1-image-assets)
    2. [Geometry Reader](#2-geometry-reader)
    3. [Binding Properties Preview](#3-binding-properties-preview)
    4. [SwiftUI View in UIKit View Controller](#4-swiftui-view-in-uikit-view-controller)
    5. [Closing Parenthesis Styling](#5-closing-parenthesis-styling)
    6. [ViewBuilder vs AnyView](#6-viewbuilder-vs-anyview)
    7. [Preview Macros](#7-preview-macros)
    8. [Tint Images](#8-tint-images)
    9. [Async Image](#9-async-image)
    10. [Repeat Element](#10-repeat-element)
    11. [State Private](#11-state-private)

- [Swift](#swift)
    1. [Optional Downcasting](#1-optional-downcasting)
    2. [Opaque Generic Arguments](#2-opaque-generic-arguments)
    3. [Async Await](#3-async-await)
    4. [Preview Data](#4-preview-data)
    5. [Extend Type Array](#5-extend-type-array)
    6. [Name Formatter](#6-name-formatter)
    7. [Safe Subscript](#7-safe-subscript)
    8. [Guard Self](#8-guard-self)
    9. [If Nesting](#9-if-nesting)
    

# SwiftUI

Best practices to be used in iOS 17 or newer for SwiftUI & related functionality.

## 1. Image Assets

This applies to cases where a named image exists in the assets.

🆗 String may be misspelled. Importing it through extensions is unnecessary boilerplate.

```swift
Image("myImage")
```

✅ Use *ImageResource* which is less mistake-prone.

```swift
Image(.myImage)
```

*Tags: Image, Assets, ImageResource, SwiftUI*

## 2. Geometry Reader

⛔️ When using geometry reader, do not put the entire view inside it. 

```swift
GeometryReader { gr in
    MyView()
        .frame(width: gr.size.width, height: gr.size.height)
}
```

✅ Instead, apply a *.background* modifier which contains a geometry reader, which in turn contains a clear background. Use *.onAppear* and calculate the width and/or height of the clear background. Store these values in @State properties and apply them to the frame's width and/or height.

```swift
@State private var width: CGFloat = 0.0
@State private var height: CGFloat = 0.0

var body: some View {
    MyView()
        .frame(width: width, height: height) // Apply the width & height 
        .background {
            GeometryReader { gr in
                Color.clear
                    .onAppear {
                        width = gr.size.width // Set width
                        height = gr.size.height // Set height
                    }
            }
        }
}
```
*Tags: Geometry Reader, Background Modifier, OnAppear Modifier, Frame Modifier*

## 3. Binding Properties Preview

🆗 When previewing content containing @Binding variables, you may think of creating @State variables within the preview container.
```swift
struct ContentView_Previews: PreviewProvider {
    @State static var isValid = true
    ContentView(isValid: $isValid)
}
```
✅ Use the provided Binding.constant(*value*) structure. Can be used for *Bool*, *Int*, *Float*, *String*, and etc. types of values. 
```swift
struct ContentView_Previews: PreviewProvider {
    ContentView(isValid: Binding.constant(true))
}
```
❕ This is the alternative with #Preview macros, where a @State variable can also be used. Note: that the *static* keyword must be ommitted, since *static* can only be declared on types. Also the view must be returned with a *return* keyword.
```swift
#Preview {
    @State var isValid = true
    return ContentView(isValid: $isValid)
}
```
✅ Using Binding.constant(*value*) is still preferred.
```swift
#Preview {
    ContentView(isValid: Binding.constant(true))
}
```
*Tags: Binding, Constant, State, Preview, Macros*

## 4. SwiftUI View in UIKit View Controller

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}
```

Inside the UIKit View Controller:

```swift
import SwiftUI

override func viewDidLoad() {
    super.viewDidLoad()
    
    // 1 
    let vc = UIHostingController(rootView: ContentView())

    let contentView = vc.view!
    contentView.translatesAutoresizingMaskIntoConstraints = false
    
    // 2
    // Add the view controller to the destination view controller.
    addChild(vc)
    view.addSubview(contentView)
    
    // 3
    // Create and activate the constraints for the swiftui's view.
    NSLayoutConstraint.activate([
        contentView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
        contentView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
    ])
    
    // 4
    // Notify the child view controller that the move is complete.
    vc.didMove(toParent: self)
}
```
*Tags: UIKit, SwiftUI, Auto Layout, viewDidLoad, constraints*

## 5. Closing Parenthesis Styling

When there is a single parameter (in a function, view, init, etc.) write it on a single line:

```swift
func doSomething(myString: String) -> String {
    // ...
}
```

However when there are 2 or more parameters, write them each on a new line as such:

```swift
func doSomething(
    myString1: String,
    myString2: String,
) -> String {
    // ...
}
```

## 6. ViewBuilder vs AnyView

⛔️ Using a normal *var* that returns AnyView(...) is unnecessarily verbose. You also can't pass variables through parameters.

```swift
var someText: some View {
    if let text {
        return AnyView(Text(text))
    } else {
        return AnyView(Text("text is nil"))
    }
}
```

✅ In cases where you have an **if/else** or need to use parameters, use a **@ViewBuilder func**.

```swift
private var text: String?

public init(text: String?) {
    self.text = text
}

var body: some View {
    someText(text)
}

@ViewBuilder func someText(_ text: String?) -> some View {
    if let text {
        Text(text)
    } else {
        Text("text is nil")
    }
}
```

# 7. Preview Macros

You can quickly preview multiple views using Macros. You can rename the view & set them one-after-another.

```swift
#Preview("Light landscape", traits: .landscapeRight) {
    ContentView()
        .preferredColorScheme(.light)
}

#Preview("Dark") {
    ContentView()
        .preferredColorScheme(.dark)
}
```

❕ You can also preview UIKit View Controllers with Macros, to quickly preview data without the need to run the app:

Let's assume you have a UIViewController

```swift
final class ButtonContainer: UIViewController {
    private let buttonTitle: String
    
    init(buttonTitle: String) {
        self.buttonTitle = buttonTitle
        super.init(nibName: nil, bundle: nil)
    }
    
    override func viewDidLoad() {
        let button = UIButton(configuration: .borderedProminent(), primaryAction: nil)
        button.setTitle(buttonTitle, for: .normal)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

You can use Macros as follows:

```swift
#Preview("UIKit Portrait") {
    ButtonContainer(buttonTitle: "Next Screen")
}
```
*Tags: Preview, Preview Macros, UIKit, View Controller, UIViewController*

## 8. Tint Images

Suppose you want to tint a symbol Image. You need to set the *renderingMode* to *template* and *foregroundStyle* to the color of choice.
```swift
Image(systemName: "checkmark.circle.fill")
    .renderingMode(.template)
    .foregroundStyle(.red)
```
*Tags: Tint, Color, SF Symbols, Vector, Image*

## 9. Async Image

Download & display an image from a URL using async image:
```swift
AsyncImage(url: URL(string: "https://your_image_url_address"))
```
*Tags: Async, Image, URL, Download*

## 10. Repeat Element

Use **RepeatElement** to create the number of columns.

```swift
private var columns: [GridItem] = []

public init(
    numberOfColumns: Int = 3,
    columnSpacing: CGFloat = 12,
) {
    columns.append(
        contentsOf: repeatElement(
            GridItem(
                .flexible(),
                spacing: columnSpacing
                ),
            count: numberOfColumns
        )
    )
}
```
*Tags: Array, Repeat Element, Grid Item, Spacing, Columns, Rows*

## 11. State Private

Since @State variables should only be modified within the current scope, it's best to mark them as private:

```swift
@State private var score = 0
```

# Swift

Best practices to be used with Swift 5 or newer.

## 1. Optional Downcasting

```swift
let mystery: Any = Bool.random() ? 1 : "One"
```

🆗 Too verbose, hard to read:

```swift
if let _ = mystery as ? Int {
    print("It's an int")
}
```

✅ Clean & easy to read:

```swift
if mystery is Int {
    print("It's an int")
}
```

❕ Can also use guard if method should return:

```swift
guard mystery is Int else { return }
```

*Tags: Optional, Downcasting, Swift 5, If Let, Optional Pattern, Guard*

## 2. Opaque Generic Arguments

🆗 Hard to read:
```swift
func handle<T: Identifiable>(value: T) {
    // ...
}
```
✅ Clean & easy to read:
```swift
func handle(value: some Identifiable) {
    // ...
}
```
*Tags: Some Keyword, Opaque, Generics, Identifiable*

## 3. Async Await

In this example we'll be writing data to Firebase. 

✅ Wait for the action to be completed by using **try await** and make the method throwing, using **async throws**

```swift
func writeData(databaseURL: String, value: String) async throws {
    let ref = Database
        .database(url: databaseURL)
        .reference()
        .child("KeyHere")

    try await ref.setValue(value)
}
```
In case there's an error thrown, the view that calls the **writeData** will handle it. Use **Task** to handle asynchronous operations in the UI.

```swift
import FirebaseDatabase

struct ContentView: View {
    var body: some View {
        Button(
            "Write To Database",
            action: {
                Task { // Use "Task" to handle asynchronous operations
                    do {
                        try await writeData(databaseURL: "databaseURLHere", value: "MyValue")
                    } catch {
                        print(error) // Handle error through UI (alerts, etc.)
                    }
                }
            }
        )
    }
}
```

❕ Example with fetching data:

```swift
func fetchData(databaseURL: String) async throws {
    let ref = Database
        .database(url: databaseURL)
        .reference()
        .child("KeyHere")

    let data = try await ref.getData()
    // ... Read or use data ...
}
```
*Tags: Async, Await, Try, Asynchronous, Fetch, Read, Write, Database, Firebase*

## 4. Preview Data

Preview models quickly using a reusable preview data template. Take this sample **User** model:

```swift
struct User {
    private let id: Int
    private let name: String
    private let age: Int

    public init(
        id: Int,
        name: String,
        age: Int
    ) {
        self.id = id
        self.name = name
        self.age = age
    }
}

```
✅ We can create preview data (in debug mode) as such:

```swift
#if DEBUG
extension User {
    static var previewData: User {
        .init(
            id: 2029320392,
            name: "Bob",
            age: 30
        )
    }
}
#endif
```

We can use the previewData in #Previews as such:

```swift
#if DEBUG
#Preview {
    ContentView(user: .previewData)
}
#endif
```

*Tags: if DEBUG, Preview Data, Reusable, Template, extension, init, model, Preview*

## 5. Extend Type Array

In cases where you need to return an array of a specific/custom type (by adding modifications to the **self**). Lets take a sample **User** model:

```swift
struct User {
    private let id: Int
    private let name: String
    private let age: Int

    public init(
        id: Int,
        name: String,
        age: Int
    ) {
        self.id = id
        self.name = name
        self.age = age
    }
}

```

✅ Extend **Array** where *Element* is the custom model, in this case **User**. Create a static var array with 10 elements, with slight variation in the id number.

```swift
extension Array where Element == User {
    static var userArray: [User] {
        (0...9).map({
            .init(
                id: $0,
                name: "Bob", 
                age: 30
            )
        })
    }
}
```

❕ In cases where you don't need a unique/iterating element in the array, and exactly the same element will be repeating 10 times, use this:

```swift
extension Array where Element == User {
    static var userArray: [User] {
        [User](
            repeating: .init(
                id: 100,
                name: "Bob", 
                age: 30
            ),
            count: 10
        )
    }
}
```
*Tags: Array, Extension, Type*

## 6. Name Formatter

⛔️ Don't just put the first and last name in a string, as this may not be valid in other regions where names are formatted differently

```swift
let fullName = "\(givenName) \(familyName)"
```

✅ Use the **PersonNameComponentsFormatter** instead:

```swift
let personNameFormatter = PersonNameComponentsFormatter()

var fullName: String {
    var components = PersonNameComponents()
    components.givenName = givenName
    components.familyName = familyName
    return personNameFormatter.string(from: components)
}
```

❕ How to use with localization:

```swift
let japanLocale = Locale(identifier: "ja_JPN")
personNameFormatter.locale = japanLocale

print(fullName)
```
*Tags: Full Name, Last Name, First Name, Formatting, Localization, Locale, Person Name Components Formatter*

## 7. Safe Subscript 

⛔️ If you want to access an item in an array through indexing, you might crash at run-time if the index is out of range:

```swift
let myArray = ["a","b","c"]
let index = 10
let myItem = myArray[10] // this would crash at runtime
```

✅ Extend **Array** with a **safe** subscript, to check whether the index is within bounds, otherwise return nil

```swift
extension Array {
    subscript(safe index: Index) -> Element? {
        get {
            return indices.contains(index) ? self[index] : nil
        }
        set(newValue) {
            guard let value = newValue, indices.contains(index) else { return }
            self[index] = value
        }
    }
}
```
1. Use as a optional property: 
```swift
let myItem: String? = myArray[safe: 10]
```
2. Use with **if let** variable: 
```swift
if let myItem = myArray[safe: 10] {
    // ...
}
```
3. Use with **guard let** variable: 
```swift
guard let myItem = myArray[safe: 10] else {
    // ...
}
```
*Tags: Safe Subscript, Extend Array, Array Index, Optional*

## 8. Guard Self

🆗 We currently write the **self** with optional chaining:

```swift
publisher.sink { [weak self] newValue in
    self?.doSomething(with: newValue)
}
.store(in: &cancellables)
```

✅ You can guard the **self** and ommit it from the beginning. Can also be written as **self.doSomething(with: newValue)** but ommitting **self** is more concise.

```swift
publisher.sink { [weak self] newValue in
    guard let self else { return }
    doSomething(with: newValue) 
}
.store(in: &cancellables)
```
*Tags: Guard Let, Weak Self, Closures, Callbacks, Publisher, Combine*

## 9. If Nesting

⛔️ Nesting many **If** or **If Let** statements is difficult to read:

```swift
if let value = value {
    if value < 10 {
        //... do something
    }
}
```

✅ Instead, you can separate **If** or **If Let** statements with commas:

```swift
if let value = value,
    value < 10 {
    //... do something
}
```

*Tags: If, If Let, Nesting, Multiple Ifs, Conditional Binding*