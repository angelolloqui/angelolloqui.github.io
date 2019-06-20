---
layout: post
title:  "Playtomic's Shared Architecture using Swift and Kotlin"
date:   2018-08-11 11:30:34
categories: 
    - swift
    - kotlin
    - architecture
    - swiftkotlin
    - playtomic
permalink: /blog/:title
---

Choosing the technology stack is one of the first and most important decisions when starting any project from scratch. At [Playtomic](https://playtomic.io), we knew that we wanted to pick a stack for the mobile apps that would allow us to deliver the best possible experience given our limited resources available.

Mobile stacks range over a plethora of alternatives:

#### Web technologies

Solutions like responsive web apps or progressive web apps allow you to leverage your web frontend experience while having a single project for all. Some native capabilities can not be used but for most apps it would be enough. Distribution is through web browsers and not inside an App Store which may be an advantage or disadvantage depending on your case

#### Hybrid

Next step on the road to native you can choose to use an hybrid framework like Phonegap/Cordoba, which also uses web technologies but get wrapped into an app bundle, offering extra capabilities and an improved UX over pure web.

#### Multiplatform Solutions

There are several multiplatform solutions like Xamarin, ReactNative or the newer Flutter. They all have their own selling points and disadvantages, but in general they offer a unified solution to build native apps by using a single language and, to some degree, a single UI layer with custom components, sharing most of the code across the platforms while delivering a good UX, very close (if not the same) than the one delivered by native apps.

#### Native

The industry standard, and the one that brings the best UX is also the one that requires the most resources. Building native means building an app per platform, each one with its own language, set of tools, libraries and frameworks.



## Our dilemma

Without getting into too much detail about each one and our analysis, we knew that we wanted Playtomic to be a leader in the sports area, and being a mobile first project we wanted it to bring the best possible experience to our end users.

We also knew that once picked, that technology stack would be our one and only one stack for long since we do not have the resource power to maintain a “Frankenstein app” built in parts with different stacks or Big Bang refactors to completely migrate from one to another. We wanted “the stack” to be production ready and with enough community and maturity to have the “certainty” that it will be supported for our project’s life.

That basically left us with 3 candidates: **Xamarin**, **ReactNative** and **Native**; and from those first two we were much more appealed by React than Xamarin because of its programming model, the possibilities to share code with a future web frontend and the amazing community and tooling.

On the other hand, when selecting a solution, you also have to consider the team you have. In our case, we were expert native developers, with experience in both platforms (Android and iOS) and with little to no experience in React or other multiplatform stacks. Besides, at that moment, there was no web frontend developer or anyone within the company with enough React experience to coach the mobile team during the learning curve.

Having all that in mind, **our best fit was native**. It delivers almost everything we wanted (best UX, industry standard, maturity, long term vision, great tools, great community,...) except for one important aspect: **cost**

As explained before, going native would mean to build 2 apps (iOS/Android), which has an extra cost compared to the single app of multiplatform solutions. But how much? Let’s try to put very rough numbers with an example (*note that these are not real numbers, they are just an estimate based on our own previous experience, don’t take them too seriously but just as an illustration*):

* **Native**: Let’s say you are building a feature that takes you 100h on iOS. Then, porting it to Android would take around 80h extra (not 100 because there is always knowledge that can be “ported” from the first platform). A total of 180h

* **Multiplatform**: The same feature would take you around 120h, depending on how much you can reuse and the technology used. It is not write once run everywhere but close enough to add only a small percentage of extra work over a single platform.

So, roughly 180h vs 120h, or in other words **around 50% extra time to go native**. That is quite a lot! Especially for a small team like ours.

So, our next question was: **can we find a way of building a native app maximizing reusability across platforms and keeping the cost down**, close to the one delivered by multiplatform solutions? And if so, will it take us less than 1-2 months work of setup? (That was the time we had until the product and design teams would start delivering well defined features to build)

I had participated in the past of some very small projects (Proof Of Concepts and a minor library) using this approach with excellent results. But building a full app is a completely different challenge, especially when the application grows.


## Shared foundations

So, we started building with one objective in mind: **reusability across native platforms**

What we did was to split the app in 3 main parts for both platforms:

![Application architecture](/images/posts/39/diagram.png){:class="img-responsive"}

*  **Anemone SDK**: framework used to connect to our backend and provide persistence. It provides Model, Service and some utilities.
*  **Playtomic UI**: framework with visual components like custom textfields, input validators, generic datasources/adapters, animations, ...
*  **Application code**: where our modules and features are built. It includes and uses the former two.

We made sure that both frameworks offered the same public API (different implementations) and we also built a few facades over concepts that would be used across the app and that were provided differently on each platform, keeping the same API again. To name a few:

*   `Promise` ([iOS](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/promise/Promise.swift) [Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/promise/Promise.kt))
*   `HttpClient` ([iOS](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/http/HttpClient.swift) [Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/http/OkHttpClient.kt))
*   `JSONObject` ([iOS](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/json/JSONObject.swift) [Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/json/JSONObject.kt))
*   `NavigationManager` ([iOS](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/manager/navigation/NavigationManager.swift) [Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/manager/navigation/NavigationManager.kt))
*   `LocationManager` ([iOS](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/manager/location/LocationManager.swift) [Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/manager/location/LocationManager.kt))
*   ...

> (You can check all code in these repos: [Swift](https://github.com/angelolloqui/blog-shared-architecure-swift) [Kotlin](https://github.com/angelolloqui/blog-shared-architecure-kotlin))

We also picked the combination of Swift/Kotlin because of their enormous similarities and we used [SwiftKotlin](https://github.com/angelolloqui/SwiftKotlin) to minimize the time needed to transpile code from iOS to Android.

Finally, we added a few extensions over foundation objects to provide some of the missing methods on one or the other language (ex: `compactMap`, `flatMap`, `let`,...)

## Internals

### AnemoneSDK

Our SDK basically offers Models and Services. It makes heavy use of networking and  promises (with a facade on top of [JDeferred](https://github.com/jdeferred/jdeferred) and [PromiseKit](https://github.com/mxcl/PromiseKit)) to deal with asynchronous calls. We chose Promises over RX alternatives because of simplicity, since we do not need very complex operations on this end. A few examples:

#### IHttpClient

A common interface to deal with networking. On iOS, [it is implemented with `NSURLSession`](https://github.com/angelolloqui/blog-shared-architecure-swift/blob/master/http/HttpClient.swift) while the equivalent [implementation in Android](https://github.com/angelolloqui/blog-shared-architecure-kotlin/blob/master/http/OkHttpClient.kt) uses `OKHttp` library

```
public protocol IHttpClient {
    func request(_ httpRequest: HttpRequest) -> Promise<HttpResponse>
}

extension IHttpClient {

    public func get(endpoint: String, params: [String: Any]?) -> Promise<Data> {
        return request(HttpRequest(method: HttpMethod.get, url: endpoint, queryParams: params, bodyParams: nil, headers: nil))
            .then { $0.body }
    }

    public func post(endpoint: String, params: [String: Any]?) -> Promise<Data> {
        return request(HttpRequest(method: HttpMethod.post, url: endpoint, queryParams: nil, bodyParams: params, headers: nil))
            .then { $0.body }
    }

    public func put(endpoint: String, params: [String: Any]?) -> Promise<Data> {
        return request(HttpRequest(method: HttpMethod.put, url: endpoint, queryParams: nil, bodyParams: params, headers: nil))
            .then { $0.body }
    }

    public func patch(endpoint: String, params: [String: Any]?) -> Promise<Data> {
        return request(HttpRequest(method: HttpMethod.patch, url: endpoint, queryParams: nil, bodyParams: params, headers: nil))
            .then { $0.body }
    }

    public func delete(endpoint: String, params: [String: Any]?) -> Promise<Data> {
        return request(HttpRequest(method: HttpMethod.delete, url: endpoint, queryParams: params, bodyParams: nil, headers: nil))
            .then { $0.body }
    }

}
```
```
interface IHttpClient {
    fun request(httpRequest: HttpRequest): Promise<HttpResponse>

    fun get(endpoint: String, params: Map<String, Any>?): Promise<ByteArray> =
            request(HttpRequest(method = HttpMethod.get, url = endpoint, queryParams = params, bodyParams = null, headers = null))
                    .then(map = { it.body })

    fun post(endpoint: String, params: Map<String, Any>?): Promise<ByteArray> =
            request(HttpRequest(method = HttpMethod.post, url = endpoint, queryParams = null, bodyParams = params, headers = null))
                    .then(map = { it.body })

    fun put(endpoint: String, params: Map<String, Any>?): Promise<ByteArray> =
            request(HttpRequest(method = HttpMethod.put, url = endpoint, queryParams = null, bodyParams = params, headers = null))
                    .then(map = { it.body })

    fun patch(endpoint: String, params: Map<String, Any>?): Promise<ByteArray> =
            request(HttpRequest(method = HttpMethod.patch, url = endpoint, queryParams = null, bodyParams = params, headers = null))
                    .then(map = { it.body })

    fun delete(endpoint: String, params: Map<String, Any>?): Promise<ByteArray> =
            request(HttpRequest(method = HttpMethod.delete, url = endpoint, queryParams = params, bodyParams = null, headers = null))
                    .then(map = { it.body })

}

```
###### String similarity: 82.28%

#### User

Model object that contains information of a user. Note how thanks to the common JSON interface code looks almost identical and behaves equally for example when we get a Number where a String was expected.

```
public struct User: JSONMappable {
    public let id: UserId
    public let email: String
    public let fullName: String
    public let picture: String?
    public let isValidated: Bool
    public let linkedTenants: [Tenant]?
    public let phone: String?
    public let acceptsPrivacy: Bool?
    public let acceptsCommercial: Bool?

    public init(json: JSONObject) throws {
        self.id = try UserId(json.getAny("user_id"))
        self.email = json.optString("email") ?? ""
        self.fullName = try json.getString("full_name")
        self.picture = json.optString("picture")
        self.isValidated = try json.getBoolean("is_validated")
        self.linkedTenants = json.optJSONArray("linked_tenants")?.flatMap { (json: JSONObject) in
            try? Tenant(json: json)
        }
        self.phone = json.optString("phone")
        self.acceptsPrivacy = json.optBoolean("accepts_privacy")
        self.acceptsCommercial = json.optBoolean("accepts_commercial")
    }

    public func linkedTenant(_ id: TenantId) -> Tenant? {
        return linkedTenants?.first { $0.id == id }
    }

}
```
```
public class User: JSONMappable {
    public val id: UserId
    public val email: String
    public val fullName: String
    public val picture: String?
    public val isValidated: Boolean
    public val linkedTenants: List<Tenant>?
    public val phone: String?
    public val acceptsPrivacy: Boolean?
    public val acceptsCommercial: Boolean?

    @Throws(JSONException::class)
    public constructor(json: JSONObject) : super(json) {
        id = UserId(json.getAny("user_id"))
        email = json.optString("email") ?: ""
        fullName = json.getString("full_name")
        picture = json.optString("picture")
        isValidated = json.getBoolean("is_validated")
        linkedTenants = json.optJSONArray("linked_tenants")?.flatMap { json ->
            try { Tenant(json = json) } catch (t: Throwable) { null }
        }
        phone = json.optString("phone")
        acceptsPrivacy = json.optBoolean("accepts_privacy")
        acceptsCommercial = json.optBoolean("accepts_commercial")
    }

    public fun linkedTenant(id: TenantId): Tenant? =
            linkedTenants?.firstOrNull { it.id == id }

}
```
###### String similarity: 87.77%


#### TenantService

An example of service to query the Tenants microservice (for club's information). See how there is a minor difference due to the language difference when using generics, but for the rest are almost identical.

```
class TenantService: ITenantService {
    let httpClient: IHttpClient

    init(httpClient: IHttpClient) {
        self.httpClient = httpClient
    }

    func search(name: String, pagination: PaginationOptions?) -> Promise<[Tenant]> {
        let endpoint = "/v1/tenants"
        var params: [String: Any] = [
            "tenant_name": name,
            "playtomic_status": "ACTIVE"
        ]
        if let pagination = pagination {
            params += pagination.params
        }
        return httpClient.get(endpoint: endpoint, params: params)
            .then { JSONTransformer().map($0) }
    }

    func search(coordinate: Coordinate, radius: Int, sportId: SportId?, pagination: PaginationOptions?) -> Promise<[Tenant]> {
        let endpoint = "/v1/tenants"
        var params: [String: Any] = [
            "coordinate": coordinate,
            "radius": radius,
            "playtomic_status": "ACTIVE"
        ]
        if let sportId = sportId {
            params["sport_id"] = sportId
        }
        if let pagination = pagination {
            params += pagination.params
        }
        return httpClient.get(endpoint: endpoint, params: params)
            .then { JSONTransformer().map($0) }
    }

    func fetchDetail(id: TenantId) -> Promise<Tenant> {
        let endpoint = "/v1/tenants/\(id)"
        return httpClient.get(endpoint: endpoint, params: nil)
            .then { JSONTransformer().map($0) }
    }

}
```
```
class TenantService(private val httpClient: IHttpClient): ITenantService {

    override fun search(name: String, pagination: PaginationOptions?): Promise<List<Tenant>> {
        val endpoint = "/v1/tenants"
        val params = mutableMapOf<String, Any>(
                "tenant_name" to name,
                "playtomic_status" to "ACTIVE"
        )
        if (pagination != null) {
            params += pagination.params
        }
        return httpClient.get(endpoint = endpoint, params = params)
                .then(JSONTransformer(Tenant::class.java)::mapArray)
    }

    override fun search(coordinate: Coordinate, radius: Int, sportId: SportId?, pagination: PaginationOptions?): Promise<List<Tenant>> {
        val endpoint = "/v1/tenants"
        val params = mutableMapOf<String, Any>(
                "coordinate" to coordinate,
                "radius" to radius,
                "playtomic_status" to "ACTIVE"
        )
        if (sportId != null) {
            params["sport_id"] = sportId
        }
        if (pagination != null) {
            params += pagination.params
        }
        return httpClient.get(endpoint = endpoint, params = params)
                .then(JSONTransformer(Tenant::class.java)::mapArray)
    }


    override fun fetchDetail(id: TenantId): Promise<Tenant> {
        val endpoint = "/v1/tenants/${id}"
        return httpClient.get(endpoint = endpoint, params = null)
                .then(JSONTransformer(Tenant::class.java)::mapObject)
    }

}
```
###### String similarity: 83.93%

### PlaytomicUI

This module contains UI components and utilities such as custom textfields, tag views, sliders, generic table cells, generic datasources/adapters, input validators, animations, view stylers,... they are packed into a framework/library and used by the application. A very simple example:

```
open class TextFieldEmailValidatorBehavior: TextFieldValidatorBehavior {
    open override func configure() {
        super.configure()
        regularExpression = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}"
    }
}
```
```
class TextFieldEmailValidatorBehavior(textView: TextView) : TextFieldRegexValidatorBehavior(
        textView = textView,
        regularExpression = "^[(A-Za-z0-9\\._\\+\\-)]+@[(A-Za-z0-9\\.\\-)]+\\.[(A-Za-z)]{2,4}$"
)
```
###### String similarity: 70.75%

### Application

Our application code is divided into **modules** and **managers**.

Each module can have its own internal architecture and communicates with the others through the use of **Coordinators**. For most of our modules we use an **MVP pattern**, as the presenter manipulation required is pretty small (remember our Anemone SDK deals with our network and persistence, so that would be the Entity and Repository in a VIPER architecture, and our Coordinators correspond to the Routers). In some cases, where there is business logic or complex data manipulation involved in clients, we also use **Interactors**. Nevertheless, modules could be implemented in MVVM or other patterns as long as they publish a Coordinator for the rest to consume.

The benefit of splitting code this way is that our Presenters and Interactors have no platform dependencies so they can be transpiled with almost no work (especially using [SwiftKotlin](https://github.com/angelolloqui/SwiftKotlin)). Coordinators are also very similar and quick to transpile, leaving the Managers and the Views as the only parts that require specific work per platform.

![Transpiling presenters](/images/posts/39/transpiling.gif){:class="img-responsive"}

> See in the above screenshot how with just a minor fix on the Kotlin code we get a fully working Android version of a Presenter by transpiling the iOS one in about 15 seconds.


Let’s see a few examples of each of this:

#### AuthCoordinator

```
class AuthCoordinator: IAuthCoordinator {

    let managerProvider: IManagerProvider
    let authenticationService: IAuthenticationService

    init(managerProvider: IManagerProvider, authenticationService: IAuthenticationService) {
        self.managerProvider = managerProvider
        self.authenticationService = authenticationService
    }

    func loginIntent() -> INavigationIntent {
        let presenter = LoginPresenter(
            coordinator: self,
            appEventManager: managerProvider.appEventManager,
            messageBarManager: managerProvider.messageBarManager,
            navigationManager: managerProvider.navigationManager,
            authenticationService: authenticationService)
        let loginController = R.storyboard.authStoryboard.loginViewController()

        return PresenterNavigationIntent(presenter: presenter, viewController: loginController)
    }

    //.... others ....
}
```
```
class AuthCoordinator(
        private val managerProvider: IManagerProvider,
        private val authenticationService: IAuthenticationService)
    : IAuthCoordinator {

    override fun loginIntent(): INavigationIntent {
        val presenter = LoginPresenter(
                coordinator = this,
                appEventManager = managerProvider.appEventManager,
                messageBarManager = managerProvider.messageBarManager,
                navigationManager = managerProvider.navigationManager,
                authenticationService = authenticationService)
        val loginFragment = LoginFragment()

        return PresenterNavigationIntent(presenter, loginFragment)
    }

    //.... others ....
}
```
###### String similarity: 76.11%

#### LoginPresenter

```
class LoginPresenter: Presenter<ILoginView> {
    let coordinator: IAuthCoordinator
    let appEventManager: IAppEventManager
    let messageBarManager: IMessageBarManager
    let navigationManager: INavigationManager
    let authenticationService: IAuthenticationService    

    init(coordinator: IAuthCoordinator,
         appEventManager: IAppEventManager,
         messageBarManager: IMessageBarManager,
         navigationManager: INavigationManager,
         authenticationService: IAuthenticationService) {
        self.coordinator = coordinator
        self.appEventManager = appEventManager
        self.messageBarManager = messageBarManager
        self.navigationManager = navigationManager
        self.authenticationService = authenticationService
    }

    override func viewPresented() {
        super.viewPresented()
        view?.setIsLoading(false)
        if authenticationService.isLoggedIn() {
            skipLogin()
        }
    }

    func skipLogin() {
        self.view.let { self.navigationManager.dismiss(view: $0, animated: true) }
    }

    func login(email: String, password: String) {
        view?.setIsLoading(true)
        authenticationService.login(user: email, password: password).then { [weak self] _ in
            guard let `self` = self else { return }
            self.view.let { self.navigationManager.dismiss(view: $0, animated: true) }
            self.appEventManager.sendEvent(AppEvent.loginWithCredentials(success: true))
        }.always { [weak self] in
            self?.view?.setIsLoading(false)
        }.catchError { [weak self] (error) in
            self?.messageBarManager.showError(error: error)
            self?.appEventManager.sendEvent(AppEvent.loginWithCredentials(success: false))
        }
    }

    func rememberPassword() {
        navigationManager.show(coordinator.requestPasswordIntent(), animation: NavigationAnimation.push)
    }

}
```
```
class LoginPresenter(
        private val coordinator: IAuthCoordinator,
        private val appEventManager: IAppEventManager,
        private val messageBarManager: IMessageBarManager,
        private val navigationManager: INavigationManager,
        private val authenticationService: IAuthenticationService)
    : Presenter<ILoginView>() {

    override fun viewPresented() {
        super.viewPresented()
        view?.setIsLoading(false)
        if (authenticationService.isLoggedIn()) {
            skipLogin()
        }
    }

    fun skipLogin() {
        this.view?.let { this.navigationManager.dismiss(view = it, animated = true) }
    }

    fun login(email: String, password: String) {
        view?.setIsLoading(true)
        authenticationService.login(email, password)
                .then {
                    this.view?.let { this.navigationManager.dismiss(view = it, animated = true) }
                    appEventManager.sendEvent(AppEvent.loginWithCredentials(success = true))
                }
                .always { 
                    view?.setIsLoading(false) 
                }
                .catchError { error ->
                    messageBarManager.showError(error = error)
                    appEventManager.sendEvent(AppEvent.loginWithCredentials(success = false))
                }
    }

    fun rememberPassword() {
        navigationManager.show(coordinator.requestPasswordIntent(), animation = NavigationAnimation.push)
    }

}
```
###### String similarity: 73.59%

#### LoginView

```
class LoginViewController: PresenterViewController<LoginPresenter> {
    @IBOutlet weak var usernameTextField: PlaytomicTextField!
    @IBOutlet weak var passwordTextField: PlaytomicTextField!
    @IBOutlet weak var loginButton: UIButton!
    @IBOutlet weak var dismissButton: UIButton!
    @IBOutlet weak var loadingIndicator: UIActivityIndicatorView!

    override func viewDidLoad() {
        super.viewDidLoad()

        usernameTextField.configure(
            inputType: .email,
            labelText: R.string.localizable.auth_login_user_field(),
            errorMessage: R.string.localizable.auth_login_user_error(),
            validators: [
                TextFieldEmailValidatorBehavior(textField: usernameTextField.textField)
            ],
            editTextDidChangeCallback: { [weak self] in self?.reloadLoginButtonState() }
        )

        passwordTextField.configure(
            inputType: .password,
            labelText: R.string.localizable.auth_login_password_field(),
            errorMessage: R.string.localizable.auth_login_password_error(),
            validators: [
                TextFieldLengthValidatorBehavior(textField: passwordTextField.textField, minLength: 5, maxLength: nil)
            ],
            editTextDidChangeCallback: { [weak self] in self?.reloadLoginButtonState() }
        )
        reloadLoginButtonState()
    }

    @IBAction func login() {
        view.endEditing(true)
        presenter.login(email: usernameTextField.text, password: passwordTextField.text)
    }

    @IBAction func skipLogin() {
        presenter.skipLogin()
    }

    @IBAction func rememberPassword() {
        presenter.rememberPassword()
    }

    func reloadLoginButtonState() {
        let fieldsValid = usernameTextField.isValid && passwordTextField.isValid
        let loading = loadingIndicator.isAnimating
        loginButton.isEnabled = fieldsValid && !loading
    }

    // ****  View Interface  ****

    func setIsLoading(loading: Bool) {
        if newValue {
            loadingIndicator.startAnimating()
        } else {
            loadingIndicator.stopAnimating()
        }
        reloadLoginButtonState()
    }

}
```
```
class LoginFragment : PresenterFragment<LoginPresenter>(R.layout.login_fragment), ILoginView {

    @BindView(R.id.username_edit_text_custom)
    lateinit var usernameCustomEditText: PlaytomicTextField

    @BindView(R.id.password_edit_text_custom)
    lateinit var passwordCustomEditText: PlaytomicTextField

    @BindView(R.id.login_button)
    lateinit var loginButton: Button

    @BindView(R.id.toolbar_back_button)
    lateinit var dismissButton: ImageButton

    @BindView(R.id.loading_indicator)
    lateinit var loadingIndicator: ProgressBar

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        usernameCustomEditText.configure(
                inputType = PlaytomicTextField.InputType.email,
                labelText = R.string.auth_login_user_field,
                errorMessage = R.string.auth_login_user_error,
                validators = listOf(
                        TextFieldEmailValidatorBehavior(usernameCustomEditText.editText)
                ),
                editTextDidChangeCallback = ::reloadLoginButtonState
        )
        usernameCustomEditText.nextTextField = passwordCustomEditText

        passwordCustomEditText.configure(
                inputType = PlaytomicTextField.InputType.password,
                labelText = R.string.auth_login_password_field,
                errorMessage = R.string.auth_login_password_error,
                validators = listOf(
                        TextFieldLengthValidatorBehavior(passwordCustomEditText.editText, 5, null)
                ),
                editTextDidChangeCallback = ::reloadLoginButtonState
        )
        reloadLoginButtonState()
    }

    @OnClick(R.id.login_button)
    internal fun login() {
        hideKeyboard()
        presenter.login(email = usernameCustomEditText.text, password = passwordCustomEditText.text)
    }

    @OnClick(R.id.auth_login_forget_password_button)
    internal fun rememberPassword() {
        presenter.rememberPassword()
    }

    private fun reloadLoginButtonState() {
        val fieldsValid = usernameCustomEditText.isValid && passwordCustomEditText.isValid
        val loading = loadingIndicator.visibility == View.VISIBLE
        loginButton.isEnabled = fieldsValid && !loading
    }

    // ****  View Interface  ****

    override fun setIsLoading(loading: Boolean) {
        loadingIndicator.visibility = if (loading) View.VISIBLE else View.GONE
        reloadLoginButtonState()
    }

}
```
###### String similarity: 70.8%

----

As you can see, code is basically the same except for some language differences (constructors, keywords,...) that can be quickly transpiled. Moreover, by using PlaytomicUI components and some of our extensions, code is similar even on the View layer. The main work on this part corresponds to laying out elements in Interface Builder (iOS) or in layout XMLs (Android).

An interesting note to make here is that we could  have decided to write the Views in code or with tools like [Layout](https://github.com/schibsted/layout). That would make possible to reuse much more here as well, but we intentionally chose not to because we wanted to keep this part (the one the user actually sees and experiences) as standard as possible. This also allows us to use and follow platform components and conventions when desired and the de facto developer tools available, hence keeping a moderate learning curve and taking advantage to a full extent of our native development expertise.


## The good, the bad and the ugly

After 1.5 years working with the explained Shared Architecture, conventions and tools, we have a pretty solid view of what’s working for us and what is not working that well. Let me try to make an introspection:

### The good

* **Team unification**: there is no Android/iOS team distinction because the same developer always transpiles his work to the other platform. This results in extreme union, platform parity and less disputes/blockages
* **Team performance**: developing app code is much faster than writing 2 platforms independently. It typically takes just a few minutes to transpile Presenters, Interactors, Coordinators, Models and Services. XML and Xib files takes the rest of the time, and every now and then some code in managers. In average, we take about 30% extra time to convert from one to the other platform, depending on the amount and complexity of the views involved, pretty close to multiplatform solutions.
* **Fully native UX**: Visual components and app performance is the same than any other native app. Besides, there is no extra penalty on app size nor app launch time like in multiplatform solutions.
* **Long term vision**: we use the facto tools, frameworks and languages on each platform, and we have no important dependencies. We can have the certainty that code will be valid for many years, even if at some point team grows and we stop sharing code they will still be valid standard native projects independently.
* **Good abstractions and code quality**: The fact that we want to reuse as much code as possible forces developers to think very carefully the abstractions they want to build. It encourages for proper separation of concerns, single responsibility classes, more testing (to verify the transpilation), etc. In fact I would even say that code reviews are also more effective as you can compare the PR side by side with the counterpart and detect issues on a higher level. Quality is not just desirable but it is also perceived as an actual productivity boost from day 1.
* **Reduced project’s cognitive load**: Having 1 code base makes understanding the project and remembering the internal details much easier.

### The bad

* **Extra architecture work**: it is no secret that building these shared abstractions and extensions take time. In our case we dedicated about 1 month to architectural foundations, and since then we have had to make some changes and additions every now and then. The total overhead is difficult to calculate, but it is noticeable especially at the beginning.
* **Hidden bugs from language differences**: transpilation works great, most of the time 💥. However, during these 18 months working on it, we have encountered 3 or 4 times bugs derived from language differences that were unexpected. Especially important is the Value type (`struct`) in Swift that has no counterpart in Kotlin or the sort in place of arrays in Kotlin. These differences impose restrictions and are a source of bugs if not considered properly.
* **Maximum common factor of language features**: in parallel to the previous bullet, having to share code imposes restrictions on the way you use a language (or more transpilation work is required). As a result, we tend to write code using the maximum common factor of language features. A few examples of limitations on Swift are value types and protocol extensions, while in Kotlin the usage of decorators or default parameters in interfaces.
* **View layer needs work per platform**: writing views require specific work per platform. That has an impact on development time, testing and bug fixing that would not ocurr that much with multiplatform solutions with shared UI.
* **Learning curve**: all the architectural components and code conventions that we use are specific from this project and therefore require some learning. Nevertheless, to be fair all projects have their own internal conventions and architecture design, so at least having the same across both platforms means that there is only 1 curve to pass and not 2.

### The ugly

* **Hybrid conventions**: Kotlin and Swift are very similar but they use different naming conventions. For example, in Kotlin/Java constants are written in upper case while in Swift they aren’t. Or the classical `I` prefix so common in Java does not exist in Swift (the closest would be to suffix `Protocol` to the name). As a result, when sharing code you have to either come with a mix of conventions or penalize the transpilation process with more manual edition to adapt from one to the other. We started with conventions per platform and we are gradually moving into a single convention that feels to us like the best of both worlds and which is becoming our de facto mobile team convention (but it would look “ugly” to external developers)
* **Replicate changes manually**: transpilation works great when building new features because you can copy&paste the full transpiled code. However, when maintaining code, it is up to the developer to remember to apply the same change made on platform A into platform B. As a result, we have sometimes forgotten to replicate, resulting in some inconsistencies on app behavior. We are controlling that through PR, forcing both platforms to have the same kind of changes and reviewing them in parallel, but there is still the case for human error.
* **Team scaling**: having such a small team helps when using this approach since it requires lots of communication between members. We are not sure how this would scale with more developers, but we suspect it won’t the day we have 6+ people or so. Besides, we are “forced” to hire (or teach) multiplatform experts as long as we want to keep sharing code efficiently.

Overall, when looking back, we feel the decision has been **the correct one for our team**. That does not mean that using React would have been a mistake (probably not), but we are very satisfied with the results we are getting. Sure, we run into some issues every now and then, and we have had to invest a couple of months on making the abstractions (this time would have gone to learning React anyway), but **we have now the best UX possible with a very decent development speed**. Moreover, we are **not dependent on some third party framework or tool** (even SwiftKotlin could be removed and just transpile code manually, which is not that bad anyway) what gives us long term confidence, and we are **free to chose the application architecture** we prefer per module (MVP, MVVM, VIPER, REDUX,...). We can also leverage all of the **native goodies** the instant they are announced and we can use the **team knowledge to full extent**.


> String similarity calculated with [Tools4Noobs](https://www.tools4noobs.com/online_tools/string_similarity/)