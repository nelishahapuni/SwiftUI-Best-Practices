# SwiftUI Best Practices

This documents contains a collection of best practices for SwiftUI, Swift 5+ and iOS 17+.

⛔️ = Bad Practice (MR/PR won't be approved)

🆗 = Not Optimal (MR/PR may be approved)

✅ = Better (MR/PR will be approved)

❕ = Related (similar concepts)

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

## 3. Binding Properties (Preview)

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

## 2. Some Opaque Generic Arguments

🆗 Hard to read:
```swift
func handle<T: Identifiable>(value: T){
    /*...*/
}
```
✅ Clean & easy to read:
```swift
func handle(value: some Identifiable){
    /*...*/
}
```
*Tags: Some Keyword, Opaque, Generics, Identifiable*