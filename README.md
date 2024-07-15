# SwiftUI Best Practices

This documents contains a collection of best practices for SwiftUI, Swift 5+ and iOS 17+.

⛔️ = Bad Practice (MR/PR won't be approved)

🆗 = Not Optimal (MR/PR may be approved)

✅ = Better (MR/PR will be approved)

❕ = Related (similar concepts)

- [SwiftUI](#swiftui)
    1. [Image Assets](#1-image-assets)
    2. [Custom Modifiers](#2-custom-modifiers)
    3. [Binding Properties Preview](#3-binding-properties-preview)
    4. [SwiftUI View in UIKit View Controller](#4-swiftui-view-in-uikit-view-controller)
    5. [Closing Parenthesis Styling](#5-closing-parenthesis-styling)
    6. [ViewBuilder vs AnyView](#6-viewbuilder-vs-anyview)
    7. [Preview Macros](#7-preview-macros)
    8. [Tint Images](#8-tint-images)
    9. [Async Image](#9-async-image)
    10. [Repeat Element](#10-repeat-element)
    11. [State Private](#11-state-private)
    12. [Platform Customizations](#12-platform-customizations)
    13. [WebView](#13-webview)
    14. [EnvironmentObject & Singleton](#14-environmentobject--singleton)
    15. [Animations](#15-animations)
    16. [Mock HTTP Client](#16-mock-http-client)
    17. [Set Environment Object](#17-set-environment-object)
    18. [Navigation Path Router](#18-navigation-path-router)
    19. [Animating Numbers](#19-animating-numbers)
    20. [Format Currency](#20-format-currency)
    21. [Custom Property Wrappers](#21-custom-property-wrappers)
    22. [Firebase Query](#22-firebase-query)
    23. [StoreKit Paywall](#23-storekit-paywall)
    24. [Configurable Button](#24-configurable-button)

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
    10. [Comparing Strings](#10-comparing-strings)
    11. [JavaScript Functions](#11-javascript-functions)
    12. [Make Binding Hashable And Equitable](#12-make-binding-hashable-and-equitable)
    13. [If Value](#13-if-value)
    14. [Comparing Arrays](#14-comparing-arrays)
    15. [Defer Completion Handler](15-defer-completion-handler)
    16. [Reserve Opacity Array](#16-reserve-opacity-array)
    17. [Task Groups](#17-task-groups)
    18. [Subscript Default](#18-subscript-default)
    19. [Static Thread Safe](#19-static-thread-safe)
    20. [Conditional Modifiers](#20-conditional-modifiers)

- [Tips](#tips)
    1. [Remove Cached SwiftUI Previews](#1-remove-cached-swiftui-previews)
    2. [Prevent Dismiss Bottom Sheet](#2-prevent-dismiss-bottom-sheet)
    3. [Display Device Info SwiftUI](#30-display-device-info)
    4. [Redacted](#4-redacted)
    5. [Text Selectable](#5-text-selectable)
    6. [Multiline String](#6-multiline-string)
    7. [Button Repeat Behavior](#7-button-repeat-behavior)
    8. [Image in Text](#8-image-in-text)
    9. [Links within Text](#9-links-within-text)
    10. [Test Localization](#10-test-localization)
    11. [Hide Status Bar & Home Indicator](#11-hide-status-bar--home-indicator)
    12. [Multi Date Picker](#12-multi-date-picker)
    13. [Is Multiple Of](#13-is-multiple-of)
    14. [List Section Spacing](#14-list-section-spacing)
    15. [Reserve Space for Text](#15-reserve-space-for-text)
    16. [Allows Hit Testing](#16-allows-hit-testing)
    17. [Detect Low Power Mode](#17-detect-low-power-mode)
    18. [Compositing Group](#18-compositing-group)

- [Resources](#resources)
    

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

## 2. Custom Modifiers

You can create custom reusable modifiers:
```swift
struct PrimaryLabel: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.black)
            .foregroundStyle(.white)
            .font(.largeTitle)
            .cornerRadius(10)
    }
}
```

✅ And you can use it as such:
```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
            .modifier(PrimaryLabel())
    }
}
```

*Tags: Custom, Modifiers, Reusable*

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

## 7. Preview Macros

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

Use a custom made CachedAsyncImage to cache images ([CachdAsyncImage Package](https://github.com/lorenzofiamingo/swiftui-cached-async-image))

```swift
CachedAsyncImage(urlRequest: logoURLRequest, urlCache: .imageCache)
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
*Tags: Private, Encapsulation, State Vars*

## 12. Platform Customizations

If you want to add platform-specific modifications, you can use platform-specific extensions:

```swift
extension View {
    func iOS<Content: View>(_ modifier: (Self) -> Content) -> some View {
        #if os(iOS)
        return modifier(self)
        #else
        return self
        #endif
    }
}

extension View {
    func macOS<Content: View>(_ modifier: (Self) -> Content) -> some View {
        #if os(macOS)
        return modifier(self)
        #else
        return self
        #endif
    }
}

extension View {
    func tvOS<Content: View>(_ modifier: (Self) -> Content) -> some View {
        #if os(tvOS)
        return modifier(self)
        #else
        return self
        #endif
    }
}

extension View {
    func watchOS<Content: View>(_ modifier: (Self) -> Content) -> some View {
        #if os(watchOS)
        return modifier(self)
        #else
        return self
        #endif
    }
}
```
*Tags: Extensions, Customizaton, iOS, macOS, tvOS, watchOS*

## 13. WebView

### Open a URL in Safari
```swift
Link("My Website", destination: URL(string: "https://mywebsite.com")!)
```

### WKWebView in SwiftUI using UIViewRepresentable
First create the UIViewRepresentable:

```swift
import SwiftUI
import WebKit

struct WebView: UIViewRepresentable {
    // 1
    let url: URL
    // 2
    func makeUIView(context: Context) -> WKWebView {
        return WKWebView()
    }
    // 3
    func updateUIView(_ webView: WKWebView, context: Context) {
        let request = URLRequest(url: url)
        webView.load(request)
    }
}
```
Then you can use it as a normal view in SwiftUI:
```swift
struct ContentView: View {
    // 1
    @State private var isPresentWebView = false

    var body: some View {
        Button("Open WebView") {
            // 2 
            isPresentWebView = true
        }
        .sheet(isPresented: $isPresentWebView) {
            NavigationStack {
                // 3 Here is the web view
                WebView(url: URL(string: "https://mywebsite.com")!)
                    .ignoresSafeArea()
                    .navigationTitle("My Website")
                    .navigationBarTitleDisplayMode(.inline)
            }
        }
    }
}
```
*Tags: WebView, SwiftUI, UIViewRepresentable, Website, Link, URL*

## 14. EnvironmentObject & Singleton

If you need to use a Manager for the Location, User Defaults, Notifications, etc. use a signleton:

```swift
class UserSettings: ObservableObject {
    @Published var username: String = "Guest"
}
```
Then if you need to interact with the View, use an Environment Object:

```swift
struct ContentView: View {
@EnvironmentObject var settings: UserSettings

var body: some View {
        Text("Hello, \(settings.username)")
    }
}

struct RootView: View {
    var body: some View {
        ContentView()
            .environmentObject(UserSettings())
    }
}
```
*Tags: Environment Object, Observable, Singleton*

## 15. Animations

Scaling animations with a state value of the scale, effect, and animation.
```swift
struct ContentView: View {
    @State private var scale = 1.0

    var body: some View {
        Button("Press here") {
            scale += 1
        }
        .scaleEffect(scale)
        .animation(.easeIn, value: scale)
    }
}
```
*Tags: Animation, Transition, Scale, Slide, Effect*

## 16. Mock HTTP Client

```swift

struct ContentView: View {
    @Environment(\.httpClient) private var httpClient
    @State private var products: [Product] = []

    var body: some View {
        VStack {
            List(products) { product in
                Text(product)
            }.task {
                do {
                    products = try await httpClient.loadProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
        }
    }
}

#Preview {
    ContentView()
        .environment(\.httpClient, MockedHTTPClient())
}

```

*Tags: Mock API, HTTP, HTTPS, Testing*


*Tags: Observed Object, Firebase, Realtime Database, View Model*

## 17. Set Environment Object

⛔️ You cannot set the value of an environment variable directly, you'll get a compiler error since it's a 'get-only' property.

```swift
struct ContentView: View {
    @Environment(\.environmentFlag) private var environmentFlag
    
    var body: some View {
        Button {
            environmentFlag = true
        } label: {
            Text("Make flag true!")
        }
    }
}
```

✅ Instead, use a @State **localFlag** variable and pass its value in the environment var.

```swift
struct ContentView: View {
    @State private var localFlag = false
    @Environment(\.environmentFlag) private var environmentFlag
    
    var body: some View {
        Button {
            myFlag = true
        } label: {
            Text("Make flag true!")
        }
        .environment(\.environmentFlag, localFlag)
    }
}
```

❕ Even better to use a @Published variable inside a ObservableObject viewModel, and access it through a @ObservedObject.

```swift
class MyViewModel: ObservableObject {
    @Published var localFlag = false
}

struct ContentView: View {
    @ObservedObject var viewModel: MyViewModel
    @Environment(\.environmentFlag) var environmentFlag
    
    var body: some View {
        Button {
            myFlag = true
        } label: {
            Text("Make flag true!")
        }
        .environment(\.environmentFlag, viewModel.localFlag)
    }
}
```

*Tags: Environment, Flag, Boolean, Get-Only, Setting, Published, ObservedObject, Observable, ViewModel*

## 18. Navigation Path Router

Inside the **Destination** enum, we define all the possible navigation destinations.

```swift
final class Router: ObservableObject {
    
    public enum Destination: Codable, Hashable { 
        case livingroom
        case bedroom(owner: String)
    }
    
    @Published var navPath = NavigationPath()
    
    func navigate(to destination: Destination) {
        navPath.append(destination)
    }
    
    func navigateBack() {
        navPath.removeLast()
    }
    
    func navigateToRoot() {
        navPath.removeLast(navPath.count)
    }
}
```

Then, we need to create a Route instance from our application's entry point and inject it as an environment object to be used in every view.

```swift
@main
struct RoutingApp: App {
    @ObservedObject var router = Router()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $router.navPath) {
                HomeView()
                .navigationDestination(for: Router.Destination.self) { destination in
                    switch destination {
                    case .livingroom:
                        LivingroomView()
                    case .bedroom(let owner):
                        BedroomView(ownerName: owner)
                    }
                }
            }
            .environmentObject(router)
        }
    }
}
```

Finally, we need to get the router instance from the views and use it whenever we need it.

Example in Home View:

```swift
struct HomeView: View {

    @EnvironmentObject var router: Router
    var ownerName: String

    var body: some View {
        VStack {
            Button {
                router.navigate(to: .livingroom)
            } label: {
                Text("Go to Living room")
            }
            Button {
                router.navigate(to: .bedroom(owner: "John"))
            } label: {
                Text("Go to John's Bedroom")
            }
            Button {
                router.navigateBack()
            } label: {
                Text("Go back")
            }
        }
    }
}
```

Example in Living Room View:

```swift
struct LivingRoomView: View {

    @EnvironmentObject var router: Router
    var ownerName: String

    var body: some View {
        VStack {
            Button {
                router.navigate(to: .bedroom(owner: "Jane"))
            } label: {
                Text("Go to Jane's room")
            }
            Button {
                router.navigateBack()
            } label: {
                Text("Go back")
            }
        }
    }
}
```

Example in Bedroom View:

```swift
struct BedroomView: View {

    @EnvironmentObject var router: Router

    var body: some View {
        VStack {
            Button {
                router.removeLast(navPath.count)
            } label: {
                Text("Go back to Home")
            }
            Button {
                router.navigateBack()
            } label: {
                Text("Go back")
            }
        }
    }
}
```

*Tags: Environment, Object, Router, Coordinator, Navigation, Stack*

## 19. Animating Numbers

Add a fancy animation when changing numbers:

```swift
@State private var number: Int = 100

var body: some View {
    VStack {
        Text(String(number))
            .contentTransition(.numericText())
            .foregrounStyle(.red)
        Button("Random") {
            withAnimation {
                number = .random(in: 0 ..< 100)
            }
        }
    }
}
```

*Tags: Animation, Effect, Number, Integers*

## 20. Format Currency

```swift
Text(1000000, format: .currency(code: "USD")) // this will be formatted to 1,000,000.00
```

## 21. Custom Property Wrappers

Create a custom property wrapper struct when you need to use a projected value. For example, when you need to automatically valuate an email:

```swift
@propertyWrapper
struct Email {
  var wrappedValue: String
  var projectedValue: Bool {
    let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
    let emailPred = NSPredicate(format: "SELF MATCHES %@", emailRegEx)
    return emailPred.evaluate(with: wrappedValue)
  }
}
```

You can use the custom wrapper @Email and to access the projected value, you need to put a $(dollar sign) before the property name (e.g. $email).

```swift
struct Person {
    let name: String
    @Email var email: String
}
```

## 22. Firebase Query

How to fetch all elements with a certain collection path name.

❕ DocumentID is an easy way to tell Firebase how to map this struct to a Firestore document. 

```swift
struct Books: Codable, Identifiable {
    @DocumentID var id: String?
    var title: String
    var author: String
    var pages: Int
}

struct ContentView: View {
    @FirestoreQuery(collectionPath: "books") var books: [Books]
    
    var body: some View {
        List {
            ForEach(books) { book in
                VStack {
                    Text(book.title)
                    Text(book.author)
                    Text(book.pages)
                }
            }
        }
    }
}
```

You can also apply functions to the query:

```swift
books.order(by: "pages", descending: true).limit(to: 3)
```

## 23. StoreKit Paywall

Create a custom paywall with StoreKit

```swift
import SwiftUI
import StoreKit

struct SubscriptionView: View {
    var body: some View {
        SubscriptionStoreView(productIDs: ["myapp.monthly", "myapp.yearly"]) {
            HeaderView()
        }
        .subscriptionStorePolicyDestination(for: .termsOfService) {
            Text("Terms of Service")
        }
        .subscriptionStorePolicyDestination(for: .privacyPolicy) {
            Text("Privacy Policy")
        }
        .subscriptionStorePolicyForegroundStyle(.white)
        .subscriptionStorePickerItemBackground(.thinMaterial)
        .subscriptionStoreControlStyle(.prominentPicker)
        .subscriptionStoreControlBackground(
            LinearGradient(colors: [.indigo, .black], startPoint: .top, endPoint: .bottom)
        )
        .storeButton(.visible, for: .redeemCode)
        .storeButton(.visible, for: .restorePurchases)
        .storeButton(.visible, for: .cancellation)
        .tint(.indigo)
    }
}
```

## 24. Configurable Button 

```swift
struct ClaimButton: View {
    let configuration: Configuration
    let action: () -> Void

    var body: some View {
        Button {
            action()
        } label: {
            HStack(spacing: 8) {
                if let icon = configuration.icon {
                    Image(systemName: icon)
                        .resizable()
                        .frame(width: 25, height: 25)
                }
                if configuration.isLoading {
                    ProgressView()
                        .progressViewStyle(.circular)
                        .tint(configuration.textColor)
                }
                Text(configuration.text)
            }
            .padding(12)
            .font(.title2)
            .foregroundColor(configuration.textColor)
            .frame(maxWidth: .infinity)
            .background {
                RoundedRectangle(cornerRadius: 12)
                    .stroke(configuration.borderColor, lineWidth: 2.0)
                    .background {
                        RoundedRectangle(cornerRadius: 12)
                            .fill(configuration.backgroundColor)
                    }
            }
        }
        .disabled(configuration.disabled)
    }
}
```
Extension of the claim button to add configurations: 

```swift
extension ClaimButton {
    struct Configuration {
        let icon: String?
        let text: String
        let textColor: Color
        let backgroundColor: Color
        let borderColor: Color
        let isLoading: Bool
        let disabled: Bool

        // Initializer with default values
        init(
            icon: String? = nil,
            text: String,
            textColor: Color = .purple,
            backgroundColor: Color = .purple.opacity(0.2),
            borderColor: Color = .purple,
            isLoading: Bool = false,
            disabled: Bool = false
        ) {
            self.icon = icon
            self.text = text
            self.textColor = textColor
            self.backgroundColor = backgroundColor
            self.borderColor = borderColor
            self.isLoading = isLoading
            self.disabled = disabled
        }
        ...
    }
}
```

Create different button styles

```swift
// Default (normal) state
static var normal: Configuration {
    Configuration(
        text: "Claim Coupon"
    )
}

// Loading state
static var loading: Configuration {
    Configuration(
        text: "Claiming...",
        isLoading: true,
        disabled: true
    )
}

// Disabled state
static var disabled: Configuration {
    Configuration(
        text: "Claim Coupon",
        textColor: .secondary,
        backgroundColor: .secondary.opacity(0.2),
        borderColor: .secondary.opacity(0.7),
        disabled: true
    )
}

// Confirmed state
static var confirmed: Configuration {
    Configuration(
        icon: "checkmark.circle.fill",
        text: "Claimed!",
        textColor: .green,
        backgroundColor: .green.opacity(0.2),
        borderColor: .green,
        disabled: true
    )
}
```
Seeing it in action:

```swift
@Observable class ViewModel {
    var claimButtonConfiguration: ClaimButton.Configuration = .normal

    // Testing function
    func claimCoupon() {
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            self.claimButtonConfiguration = .loading
            DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
                self.claimButtonConfiguration = .confirmed
                DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                    self.claimButtonConfiguration = .disabled
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        self.claimButtonConfiguration = .normal
                    }
                }
            }
        }
    }
}
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

## 10. Comparing Strings

⛔️ This is not the most efficient way to compare case-sensitive strings:

```swift
let searchQuery = "cafe"
let databaseValue = "Cafe"

if searchQuery.lowercased() == databaseValue.lowercased() {
    // ... do something
}
```

✅ Instead, use this is a much better approach:

```swift
let searchQuery = "cafe"
let databaseValue = "Café"

let comparisonResult = searchQuery.compare(
    databaseValue,
    options: [.caseInsensitive, .diacriticInsensitive]
)
if comparisonResult == .orderedSame {
    // ... do something 
}
```

*Tags: Strings, Upper Case, Lower Case, Case Sensitive, Comparison, Diacritic Insensitive*

## 11. JavaScript Functions
Receive Parameters from WebView

```swift
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    let navigationRequestURL = navigationAction.request.url

    if let scheme = navigationRequestURL?.scheme,
        scheme == "myappscheme" {
        guard let params = navigationRequestURL?.query else {
            // no params
            decisionHandler(.cancel)
            return
        }
        // Parse parameters 
        let paramsArray = params.components(separatedBy: "=")

        if paramsArray[safe: 0] == "paramsArrayIdentifier" { // if there are multiple params
            // Parse parameters - number of incoming parameters (array length) should be known
            if let someParams = paramsArray[safe: 1]?.components(separatedBy: "&") {
                doSomethingWithParams(param1: someParams[safe: 0],
                                        param2: someParams[safe: 1],
                                        param3: someParams[safe: 2],
                                        param4: someParams[safe: 3])
            } else if paramsArray[safe: 0] == "singleParamName" {
                doSomething()
                decisionHandler(.allow)
                return
            }
        }
         decisionHandler(.cancel)
            return
    }
    decisionHandler(.allow)
}
```

### Call JS function from webview with .evaluateJavaScript(...)
```swift
myWebView.evaluateJavaScript("myJSFunc(\"\(data)\",\"\(success)\",\"\(status)\")", completionHandler: nil)
```
*Tags: WebView, JS, JavaScript, Website*

## 12. Make Binding Hashable And Equitable

```swift
extension Binding: Equatable where Value: Equatable {
    public static func == (lhs: Binding<Value>, rhs: Binding<Value>) -> Bool {
        lhs.wrappedValue == rhs.wrappedValue
    }
}

extension Binding: Hashable where Value: Hashable {
    public func hash(into hasher: inout Hasher) {
        hasher.combine(self.wrappedValue.hashValue)
    }
}
```

## 13. If Value

🆗 Create a value and assign to it, based on the outcome of the if-else statement:

```swift
let comment: String

if Int.random(in: 0...3).isMultiple(of: 2) {
    comment = "It's an even integer"
} else {
    comment = "It's an odd integer"
}
```

✅ Make the if statement part of the value:

```swift
let comment = if Int.random(in: 0...3).isMultiple(of: 2) {
    "It's an even integer"
} else {
    "It's an odd integer"
}
```

❕ This is best when there are multiple "if else" statments. For short if/else, it's best to use ternary operators. 

## 14. Comparing Arrays

Compare array elements with custom logic inside closure

✅ Best for single-use scenarios, no need to conform to Equitable 

```swift
let employees1 =[
    Employee(name: "Alice", age: 39, role: "Engineer"),
    Employee(name: "Bob", age: 25, role: "Designer")
]

let employees2 = [
    Employee(name: "Charlie", age: 28, role: "Engineer"),
    Employee(name: "Dave", age: 32, role: "Designer")
]

let areRolesEqual = employees1.elementsEqual(employees2) {
    $0.role = $1.role
}

print(areRolesEqual)
```

## 15. Defer Completion Handler

⛔️ In this case, the function will return, even if there is no completion handler

```swift
func fetchData(url: URL, _ completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        guard error == nil else {
            // completion(.failure(error!))
            return
        }

        guard let data else {
            completion(.failure(NetworkError.noData))
            return
        }

        completion(.success(data))
    }
    .resume()
}
```

✅ To ensure that the completion handler is always called, create a property **let result: Result<Data, Error>** and defer the completion handler. This way, the compiler would report an error when **result** is unassigned.

❕ Note: **defer statement** - a block of code that will be executed just before the current code block return/exits (normally or with an error)

```swift
func fetchData(url: URL, _ completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        let result: Result<Data, Error>

        defer {
            completion(result)
        }

        guard error == nil else {
            result = .failure(error!)
            return
        }

        guard let data else {
            result = .failure(NetworkError.noData)
            return
        }

        result = .success(data)
    }
    .resume()
}
```

## 16. Reserve Capacity Array

If you know the number of elements in an array, you can use the **.reserveCapacity()** method to keep the array in the same place in memory, which will optimize the app.

```swift
var arrayTwo: [Int] = []
array.reserveCapacity(1)
array.append(1)
```

## 17. Task Groups

You want to fetch multiple **Movie** objects, using the movies' IDs. You can do that with task groups, where you first fecth all the ids, then inside .withThrowingTaskGroup(of: ) add a task with method .getMovie for each ID. 

❕ Use .reserveCapacity() since you know the # of items

```swift
func getMovie(withId id: UUID) async throws -> Movie {
    return try await network.fetchMovie(withId: id)
}
```

```swift
func fetchFavorites(user: User) async throws -> [Movie] {
    // fetch ids for favorites from a remote source
    let ids = await getFavoriteIds(for: user)

    // load all favorites concurrently
    return try await withThrowingTaskGroup(of: Movie.self) { group in
        var movies = [Movie]()
        movies.reserveCapacity(ids.count)

        // adding tasks to the group and fetching movies
        for id in ids {
            group.addTask {
                return try await self.getMovie(withId: id)
            }
        }

        // grab movies as their tasks complete, and append them to the `movies` array
        for try await movie in group {
            movies.append(movie)
        }

        return movies
    }
}
```

## 18. Subscript Default

```swift
class BirdSightingsStore {
    private var birdSightings: [BirdSpecies: [Date]] = [:]

    // ⛔️ Old way with nil-coalescing operator
    func addSighting(of birdSpecies: BirdSpecies) {
        let currentValue = birdSpecies[birdSpecies] ?? []
        birdSightings[birdSpecies] = currentValue + [Date()]
    }

    // ✅  Improved with subscript(_:default:)
    func addSighting(of birdSpecies: BirdSpecies) {
        birdSightings[birdSpecies, default: []].append(Date())
    }
}
```

## 19. Static Thread Safe

⛔️ The following code is not thread-safe:
- Static props are shared among all instances of the class & can be accessed/modified from mult threads concurrently
- No synchronization mechanism present (data races/inconsistent results)
- If mult threads attempt to modify the prop - incorrect/incomplete modifications, data corruption, crashes

```swift
class Solution {
    static var nums: [Int] = []
}
```

✅ Thread safe - concurrent access to the names array should be synced w/locks, serial queues, etc.

```swift
class ThreadSafeSolution {
    private static var nums: [Int] = []
    private static let queue = DispatchQueue(label: "com.example.some-queue")

    static var countNums: Int {
        queue.sync {
            nums.count
        }
    }

    static func addNum(_ num: Int) {
        queue.async(flags: .barrier) {
            nums.append(num)
        }
    }
}
```

✅ Singleton - SettingsManager - Applying the above solution to singletons
- mult threads can read, but only one can modify at a time

```swift
class SettingsManager {
    static let shared = SettingsManager()
    private let serialQueue = DispatchQueue(label: "com.example.SettingsManager.serialQueue")
    private var settings: [String: Any] = [:]

    private init() {}

    func setValue(_ value: Any, forKey key: String) {
        serialQueue.async(flags: .barrier) {
            self.settings[key] = value
        }
    }

    func value(forKey key: String) -> Any? {
        var result: Any?
        serialQueue.sync {
            result = self.settings[key]
        }
        return result
    }
}

```

## 20. Conditional Modifiers

Almost all modifiers accept a **nil** value for no changes, so *always* do:

```swift
@State var overlay: Bool

var body: some View { 
    Image(systemName: "photo")
        .resizable()
        .overlay(overlay ? Circle().foregroundColor(Color.red) : nil) // use nil
}
```

# Tips

## 1. Remove Cached SwiftUI Previews

Go to terminal and type:

**xcrun simctl --set previews delete all**

## 2. Prevent Dismiss Bottom Sheet

You can prevent users from dismissing a bottom sheet by swiping down with the modifier:

```swift
.interactiveDismissDisabled()
```

## 3. Display Device Info

```swift
Text(UIDevice.current.systemName) // iOS
Text(UIDevice.current.systemVersion) // 17.4
Text(UIDevice.current.model) // iPhone
Text(UIDevice.current.name) // iPhone 15 Pro Max

Text(.now, style: .time) // 8:30 PM
Text(.now, style: .date) // 27 January 2024
```

## 4. Redacted

Apply placehodler effect while waiting for text to be fetched

```swift
HomeView()
    .redacted(reason: .placeholder)
```

Privacy Sensitive - Hide sensitive data like API keys with this setting

```swift
Text("hpOxZkqwhMlKJasB")
    .privacySensitive(true)
    .redacted(reason: .privacy)
```

## 5. Text Selectable

```swift
Text("Hello")
    .textSelection(.enabled)
```

## 6. Multiline String

⛔️ If you have multiple new lines in a string, this becomes difficult to read

```swift
let string = "1st line\n2nd line\n3rd line\n4th line"
```

✅ Instead, use the multiline string (""") and seperate each line as desired:

```swift
let string = """
1st line
2nd line
3rd line
4th line
"""
```

## 7. Button Repeat Behavior

You can configure a button to trigger an action repeatedly, while you hold the button:

```swift
Button("Tap Here") {
    print("tapped...")
}
.buttonRepeatBehavior(.enabled)
```

## 8. Image in Text

Insert images within text by using the ‘+’ operator.

```swift
Text("Made with ")
+
Text(Image(systemName: "heart.fill"))
    .foregroundStyle(.red)
+
Text(" in Paris")
```

## 9. Links within Text

```swift
Text("Visit the [Apple](https://www.apple.com) website")
```

## 10. Test Localization

```swift
#Preview {
    ContentView()
        .environment(\.locale, Locale(identifier: "it")) // Italy
}
```

## 11. Hide Status Bar & Home Indicator

```swift
.statusBarHidden()
.persistentSystemOverlays(.hidden)
```

## 12. Multi Date Picker

```swift
struct ContentView: View {
    @State private var selectedDates: Set<DateComponents> = []

    var body: some View {
        MultiDatePicker("Select Dates", selection: $selectedDates)
    }
}
```

## 13. Is Multiple Of

Instead of using modulo (%) use the method .isMultiple(of: n) for better readibility:

```swift
let myInteger = Int.random(in: 0...10)

if myInteger.isMultiple(of: 2) {
    print("it's even")
} else {
    print("it's odd")
}
```

## 14. List Section Spacing

Since iOS 17.0, you can change the spacing of lists:

```swift
.listSectionSpacing(.default) // default
.listSectionSpacing(.compact) // very thin
.listSectionSpacing(70) // custom
```

## 15. Reserve Space for Text 

```swift
Text("Hello, this is a text")
    .lineLimit(2, reserveSpace: true)
    .background(Color.mint)
```

## 16. Allows Hit Testing

If there's underlying content & you cannot access it, then you can use .allowsHitTesting(false)

```swift
Text("Hello")
    .allowsHitTesting(false) 
```

## 17. Detect Low Power Mode

```swift
if ProcessInfo.processInfo.isLowPowerModeEnabled {
    // do something or show a View
} else {
    // do something
}
```

## 18. Compositing Group

If you want to apply the same modifiers to a group, you can use the **.compositingGroup()** modifier.

```swift
VStack {
    // ... some views
}
.compositingGroup()
.shadow(
    color: .black,
    radius: 30,
    x: 0,
    y: 5
)
```

# Resources

- Swift Style Guide (Google) - https://google.github.io/swift/#file-comments
- SwiftUI tips and tricks - https://www.hackingwithswift.com/quick-start/swiftui/swiftui-tips-and-tricks 
- Vincent Pradeilles (Youtube Tutorials, Tips & Tricks) - https://www.youtube.com/@v_pradeilles
- Firebase Query - https://medium.com/firebase-developers/firestorequery-swiftui-the-easiest-way-to-listen-for-real-time-updates-32f436cfa26b 
- Task Groups - https://www.donnywals.com/swift-concurrencys-taskgroup-explained/
- StoreKit Paywall - https://media.licdn.com/dms/image/D4D22AQFLxqtJvKsO-w/feedshare-shrink_2048_1536/0/1713180461576?e=1720656000&v=beta&t=dcI_1lqTo93v5_upHCN-SJkCAngR0Jn2fEJlTWaaj2U
- Masking & Inverted Masking - https://media.licdn.com/dms/image/D5622AQFOlGAD_7ZkUA/feedshare-shrink_2048_1536/0/1710857641606?e=1720656000&v=beta&t=HJgr6NTtEkLENzhsc55vt29rz8bNfLKJGEdyR2A-odE
- AppStorage & User Defaults - https://holyswift.app/using-userdefaults-to-persist-in-swiftui/
- Swift Package Manager Fetching - https://ahmdyasser.medium.com/why-fetching-packages-using-swift-package-manger-takes-too-much-time-138982a0fba5
