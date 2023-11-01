# SwiftUI-Best-Practices

This documents contains a collection of best practices for SwiftUI, Swift 5 and iOS 17.

⛔️ = Bad Practice (MR/PR won't be approved)

🆗 = Not Optimal (MR/PR may be approved)

✅ = Better (MR/PR will be approved)

❕ = Related (similar concepts)

## Optional Downcasting

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
*Tags: Optional, Downcasting, Swift 5, If Let Optional Pattern, Guard*