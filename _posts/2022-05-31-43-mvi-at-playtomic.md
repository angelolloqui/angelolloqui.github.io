---
layout: post
title:  "MVI at Playtomic"
date:   2022-05-31 12:00:00
categories: 
    - ios
    - android
    - architecture
    - mvi
    - swiftui
    - compose
permalink: /blog/:title
---

Last summer we finally decided to move out of our classic MVP+UIKit/Android view architecture into a more modern one with **SwiftUI**/**Compose** as main actors for our view layer. Together with the UI framework changes, we found the need of switching to a more reactive architecture that better fits the nature of declarative UIs.

We spent some time analysing some of the most popular reactive architectures: MVVM, MVI and [TCA](https://github.com/pointfreeco/swift-composable-architecture). Without getting into much detail of our decision making (it would take a full post), we decided that **MVI was the best fitting for our project**. With it, we could get better **separation of concerns** and **state management** than in MVVM, **unidirectional data flow**, **single source of truth** and **easy of testing**, without the additional complexity added by TCA.

After around half a year working with MVI, the **team is highly satisfied**: all people in the team consider it a good/great choice and enjoys working with it, being the verbosity the only drawback.

Let us share a bit on how we do it:

## The state management layer

This is how our MVI base class looks like in both platforms

```
open class BaseMVIPresenter<S: ViewState, A: ViewAction> {
    public var currentViewState: S { _viewState.value! }
    public var viewState: Observable<S> { _viewState }
    fileprivate let _viewState: MutableObservable<S>

    public init(initialState: S) {
        _viewState = MutableObservable(value: initialState)
    }

    func dispatch(action: A) {
        fatalError("Must be implemented by the children")
    }
}

open class MVIPresenter<S: ViewState, A: ViewAction, R: ActionResult>: BaseMVIPresenter<S, A> {
    private var middlewares: [MVIMiddleware<S, A, R>] = []

    public func with(middleware: MVIMiddleware<S, A, R>) -> Self {
        middlewares.append(middleware)
        return self
    }

    open func handle(action: A, results:  @escaping (R) -> Void) {
        fatalError("Must be implemented by the children")
    }

    open func reduce(currentViewState: S, result: R) -> S {
        fatalError("Must be implemented by the children")
    }

    override public func dispatch(action: A) {
        middlewares.forEach { element in
            element.handle(action: action, presenter: self)
        }
        handle(action: action) { [weak self] result in
            Executor.execute(inBackground: false) {
                guard let self = self else { return }
                self.middlewares.forEach { middleware in
                    middleware.handle(result: result, presenter: self)
                }
                self._viewState.value = self.reduce(currentViewState: self.viewState.value!, result: result)
            }
        }
    }
}
```
```

interface BaseMVIPresenter<ViewState, ViewAction> {
    val currentViewState: ViewState get() = viewState.value!!
    val viewState: Observable<ViewState>
    fun dispatch(action: ViewAction)
}

abstract class MVIPresenter<S : ViewState, A : ViewAction, R : ActionResult>(initialState: S) : BaseMVIPresenter<S, A> {
    override val viewState: Observable<S>
        get() = _viewState
    private val _viewState = MutableObservable(value = initialState)
    internal var middlewares = mutableListOf<MVIMiddleware<S, A, R>>()

    abstract fun handle(action: A, results: (R) -> Unit)

    abstract fun reduce(currentViewState: S, result: R): S

    override fun dispatch(action: A) {
        middlewares.forEach { element ->
            element.handle(action = action, presenter = this)
        }
        handle(action) { result ->
            Executor.execute(inBackground = false) {
                this.middlewares.forEach { middleware ->
                    middleware.handle(result = result, presenter = this)
                }
                this._viewState.value = this.reduce(currentViewState = this.viewState.value!!, result = result)
            }
        }
    }
}

fun <S : ViewState, A : ViewAction, R : ActionResult, T : MVIPresenter<S, A, R>> T.with(middleware: MVIMiddleware<S, A, R>): T {
    middlewares.add(middleware)
    return this
}
```

> Note: In MVI there is no definition whether the data management part should be done in a presenter, view model or whatever. However, in our case we opted to call them "Presenters" to be more inlined with the rest of the app, but they are in practice maintaining state as classic ViewModel in Android.

Then, when building a feature, we need to provide the implementation of 2 methods:

- `handle`: This method takes the actions triggered by some other component (normally the view) and **handles the side effects**. It emits new events (called "action results") with the results of the effects, like for example a network call. It does not perform any state management or manipulation, it just emits new result events.

- `reduce`: Given a state and an action result, this method **computes the next state of the view**. Note that it behaves as a pure function, with no side effects.

![MVI diagram](/images/posts/43/mvi_diagram.png){:class="img-responsive"}

In contrast with other simpler implementations of MVI, we opted to split the code into these 2 methods to **isolate side effects from state manipulation**, which make our tests much simpler and our overall solution more robust and clean. An added benefit is that there are **no race conditions** possible like in other architectures, since all calls to `reduce` are executed in serial with no partial updates possible.

In addition, we added an extra piece around the presenters, called `Middleware`, that are capable of **reacting to events without doing state management**. For example, we can add all analytics tracking into a middleware or all navigation actions. This way, our presenter stays purist, just doing the state management part, and we have a set of small middlewares with a single other purpose, making it once again easier to test and maintain.

Lastly, you can see how both platform implementations are quite similar, and they both avoid the usage of platform specific APIs like Combine or Flows (although they are used internally) to maximize code reusal when transpiling and also reduce the learning curve.

An example Presenter would look like:

```
internal class LessonDetailPresenter(...): MVIPresenter<LessonDetailState, LessonDetailAction, LessonDetailResult>(LessonDetailState.none) {

    override fun handle(action: LessonDetailAction, results: (LessonDetailResult) -> Unit) {
        when (action) {
            is LessonDetailAction.onAppear -> loadLesson(results, lessonId)            
            is LessonDetailAction.tapConfirmCancelEnrollment -> unregisterFromLesson(results)
            is LessonDetailAction.resendConfirmation -> resendConfirmation(results)
            ...
        }
    }

    override fun reduce(currentViewState: LessonDetailState, result: LessonDetailResult): LessonDetailState {
        return when (result) {
            is LessonDetailResult.lessonLoading -> LessonDetailState.loading
            is LessonDetailResult.lessonLoaded -> LessonDetailState.detail(sections = result.lesson.mapToLessonDetail(me = userId))
            ...
        }
    }

    private fun loadLesson(results: (LessonDetailResult) -> Unit, lesson: LessonId) {
        results(LessonDetailResult.lessonLoading)
        activityService.fetchLesson(lessonId)
            .then { results(LessonDetailResult.lessonLoaded(it)) }
            .catchError { results(LessonDetailResult.loadLessonByIdFailed(error)) }
        }
    }
    ...
}
```
And some associated middleware for navigation:
```
internal class LessonDetailNavigator(...) : MVIMiddleware<LessonDetailState, LessonDetailAction, LessonDetailResult>() {

    override fun handle(action: LessonDetailAction, presenter: MVIPresenter<LessonDetailState, LessonDetailAction, LessonDetailResult>) {
        when (action) {
            LessonDetailAction.tapOpenMaps -> openMaps(presenter = presenter)
            LessonDetailAction.tapAddToCalendar -> addLessonToCalendar(presenter = presenter)
            ...
        }
    }

    override fun handle(result: LessonDetailResult, presenter: MVIPresenter<LessonDetailState, LessonDetailAction, LessonDetailResult>) {
        when (result) {
            is LessonDetailResult.loadLessonByIdFailed -> dismiss()
            else -> {}
        }
    }
    ...
}
```



## Our view layer
Then, our views basically just receive 2 parameters:

```
public struct LessonDetailView: View {
    @ObservedObject var state: ObservableViewState<LessonDetailState>
    let dispatcher: (LessonDetailAction) -> Void

    public var body: some View {
        ...
    }
}
```
```
@Composable
private fun LessonDetailView(
    viewState: LiveData<LessonDetailState>,
    dispatcher: (LessonDetailAction) -> Unit
) {
    ...
}
```

As you can see, we are making use of [state hoisting](https://developer.android.com/jetpack/compose/state#state-hoisting) for encapsulating the presenter injection (from the view layer, it does not know what class is behind the state management). This also makes our **code much more reusable**, and very **easy to setup the previews**, since we do not need to mock any data, service or presenter, just passing the view state down is enough. For this state hoisting we are making use of a parent `UIViewController`/`Fragment`, since our app is now a mixed app with only part of the views in SwiftUI/Compose. They look like this: 

```
final class LessonDetailViewViewController: SwiftUIViewController<LessonDetailState, LessonDetailAction, LessonDetailView> {
    override func contentView(
        viewState: ObservableViewState<LessonDetailState>,
        dispatcher: @escaping (LessonDetailAction) -> Void
    ) -> LessonDetailView {
        LessonDetailView(state: viewState, dispatch: dispatcher)
    }
}
```
```
class LessonDetailFragment : ComposeFragment<LessonDetailState, LessonDetailAction>() {
    @Composable
    override fun ContentView(
        viewState: LiveData<LessonDetailState>,
        dispatcher: (LessonDetailAction) -> Unit
    ) = LessonDetailView(viewState = viewState, dispatcher = dispatcher)    
}
```

Where the `SwiftUIViewController` and `ComposeFragment` are base classes that inject the presenter and create the `UIHostingController`/`ComposeView` for using SwiftUI/Compose inside with the content returned by the concrete `contentView` method on each case.


## Conclusions
There are tons of options and architectures to use with SwiftUI/Combine. In Playtomic, we opted for a MVI version where we have a clear separation of  the different responsibilities, single source of truth and a simple and unidirectional data flow. It also allows for very simple views and easy transpilation between platforms, with 
only one drawback so far: the extra boilerplate needed.

## References
- [GoDaddy Studio’s Journey with State Management](https://www.godaddy.com/engineering/2021/11/05/android-state-management-mvi/): Great article explaining some of the issues they found with MVP, MVVM and MVI in its simpler form. We find ourselves very aligned with their journey.




