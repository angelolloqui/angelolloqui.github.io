---
layout: post
title:  "Keyboard observer in Android with LiveData"
date:   2020-03-20 12:00:00
categories: 
    - livedata
    - android
permalink: /blog/:title
---


It is very common in mobile apps to need to update UI when keyboard is shown or dismissed. It might be to make some extra space in the screen, to scroll a list to a particular position or just to show some hints to the user while typing. Nevertheless, no matter what your reason is, there is no easy way to detect keyboard opening or closing in Android. Let's explore some ideas:

### Option 1: Focus listener ❌
A first approach would be to **detect a focus changes on a particular `EditText`**. Code looks like:

```
editText.setOnFocusChangeListener { _, hasFocus ->
    if (hasFocus) {
        keyboardOpen()
    } else {
        keyboardClosed()
    }
}
```

However, this solution does not solve the problem completely because when the keyboard is displayed (and field has focus), **if the user presses the phone's back button, then the keyboard is dismissed but the focus remains in the `EditText`**. You may think that you can solve this by overriding and detecting `onKeyDown` or `onBackPressed` on your activity, but in fact the events are not sent to your activity and there is no way to either detect the dismissal or to force the `EditText` to lose the focus with it.


### Option 2: ViewTree listener ⚠️
A second approach would be add a **global view tree layout listener**, so every time you have a layout change then you can compute the height difference between your activity root view and the visible display frame for it. Code looks like:

```
val globalLayoutListener = ViewTreeObserver.OnGlobalLayoutListener {
    val displayRect = Rect().apply { contentView.getWindowVisibleDisplayFrame(this) }
    val keypadHeight = contentView.rootView.height - displayRect.bottom
    if (keypadHeight > minKeyboardHeight) {
        keyboardOpen()
    } else {
        keyboardClosed()
    }
}
contentView.viewTreeObserver.addOnGlobalLayoutListener(globalLayoutListener)
```
Althought this solution works, it is quite cumbersome and leaves up to the developer big responsibilities like **removing the observer when fragment/activity is destroyed**. Besides, if you need to do this in many different views, you will end up with **a lot of repetition** of code that is not trivial. Lastly, the global layout listener will be fired multiples times, so your `keyboardOpen` and `keyboardClosed` methods will be fired lots of times with no need.


### Solution: LiveData + ViewTree listener ✅

If we iterate on *Option 2*, we can see that the layout listener can in fact be encapsulated in a component. If we extend that idea, and we make use of `LiveData`, we can come with a very elegant solution. 

Create a small behavior class encapsulating the logic in a `LiveData` subclass

```
open class KeyboardTriggerBehavior(activity: Activity, val minKeyboardHeight: Int = 0) : LiveData<KeyboardTriggerBehavior.Status>() {
    enum class Status {
        OPEN, CLOSED
    }

    val contentView = activity.findViewById<View>(android.R.id.content)

    val globalLayoutListener = ViewTreeObserver.OnGlobalLayoutListener {
        val displayRect = Rect().apply { contentView.getWindowVisibleDisplayFrame(this) }
        val keypadHeight = contentView.rootView.height - displayRect.bottom
        if (keypadHeight > minKeyboardHeight) {
            setDistinctValue(Status.OPEN)
        } else {
            setDistinctValue(Status.CLOSED)
        }
    }

    override fun observe(owner: LifecycleOwner, observer: Observer<in Status>) {
        super.observe(owner, observer)
        observersUpdated()
    }

    override fun observeForever(observer: Observer<in Status>) {
        super.observeForever(observer)
        observersUpdated()
    }

    override fun removeObservers(owner: LifecycleOwner) {
        super.removeObservers(owner)
        observersUpdated()
    }

    override fun removeObserver(observer: Observer<in Status>) {
        super.removeObserver(observer)
        observersUpdated()
    }

    private fun setDistinctValue(newValue: KeyboardTriggerBehavior.Status) {
        if (value != newValue) {
            value = newValue
        }
    }

    private fun observersUpdated() {
        if (hasObservers()) {
            contentView.viewTreeObserver.addOnGlobalLayoutListener(globalLayoutListener)
        } else {
            contentView.viewTreeObserver.removeOnGlobalLayoutListener(globalLayoutListener)
        }
    }
}
```

And then use it everywhere just like:

```
    // Some Fragment, but it works equally on an activity
    private var keyboardTriggerBehavior: KeyboardTriggerBehavior? = null
    ...
    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        val activity = this.activity ?: return
        keyboardTriggerBehavior = KeyboardTriggerBehavior(activity).apply {
            observe(viewLifecycleOwner, Observer {
                when (it) {
                    KeyboardTriggerBehavior.Status.OPEN -> keyboardOpen()
                    KeyboardTriggerBehavior.Status.CLOSED -> keyboardClosed()
                }
            })
        }
    }
```

There are a number of benefits of this solution:
- **It is highly reusable**, you can include this in any of your views very easily
- **It automatically detects the lifecycle events** thanks to the usage of `LiveData`, which means that it is secure to use since it will remove listeners automatically when your fragment/activity is destroyed
- **It behaves like any other observable** of your ViewModels, and it can even be passed down to the view's databinding
- **It will just fire updates when there is a keyboard state change** and not when any layout happens.

We have even added a `minKeyboardHeight` property in case you need to use this in a splitted view.

![Brilliant!](https://media.giphy.com/media/3otPoFIPdGqzjUWpeE/giphy.gif)