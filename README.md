# SwiftUI-Best-Practices

This documents contains a collection of best practices for SwiftUI, Swift 5 and iOS 17.

⛔️ = Bad Practice (MR/PR won't be approved)

🆗 = Not Optimal (MR/PR may be approved)

✅ = Better (MR/PR will be approved)

❕ = Related (similar concepts)

# SwiftUI

Best practices to be used in SwiftUI views & related functionality.

## 1. Image Assets

This applies to cases where a named image exists in the assets.

🆗 String may be misspelled. Importing it through extensions is unnecessary boilerplate:
```swift
Image("myImage")
```
✅ Less mistake-prone:
```swift
Image(.myImage)
```
*Tags: Image, Assets, SwiftUI*

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
*Tags: *

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