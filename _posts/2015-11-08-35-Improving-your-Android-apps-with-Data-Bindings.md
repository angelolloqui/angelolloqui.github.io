---
layout: post
title:  "Improving your Android apps with Data Bindings"
date:   2015-11-08 11:30:34
categories:  
    - android
    - data binding
    - clean code
    - libraries
    - tools
permalink: /blog/:title
---

[DataBindings](https://en.wikipedia.org/wiki/Data_binding) is a concept that has been there for quite some time with great success in other technologies. However, the Android stack lacked them (at least in an formal way) until June 2015, when Google introduced them in beta during the Google/IO (still in beta version at the moment of writing this post). But, how does DataBinding work? why is DataBinding so important? can it actually help and improve the way we develop Android apps? will this be the new standard when developing Android apps? I really believe so, so let’s get started! there is a lot to cover!

#### Benefits

*   **Declarative vs Imperative**: Data Binding expressions are [declarative expressions](https://en.wikipedia.org/wiki/Declarative_programming). Declarative code is much easier to read and understand than their [imperative](https://en.wikipedia.org/wiki/Imperative_programming) counterpart because they express what you want to do instead of how you want to do it, and it removes the “time of execution” from the problem as well. In the next example you can see how the imperative version (left) mixes all kind of responsibilities in the code - dealing with temporal state- and results in a much longer/complex version than the [functional](https://en.wikipedia.org/wiki/Functional_programming) counterpart on the right. ![](/ckeditor_assets/pictures/22/content_screen_shot_2015-11-09_at_19_54_16.png?1447095303)

*_Note_: The right side is [functional](https://en.wikipedia.org/wiki/Functional_programming), a different paradigm, but a pure functional language can be considered declarative and in any case the example shows very clearly the problem with the imperative version.

*   **Much less code**: Having expressions in layouts remove a lot of the required code in the activities/fragments to manage the content displayed in the layout elements. Less code means faster development and less bugs.

*   **Reduced dependency between layouts and code**: Layouts can now describe themselves how to draw the information received, so there is no longer need for all of those highly dependent setText and alike in your fragments/activities. The result is a much more reusable layout and activity/fragment

*   **Contextual information**: When dealing with layouts it is very common to check how a particular field will be filled in. Without bindings you would normally need to look for the activity/fragment that uses the layout, then look where the findViewById is performed, and look for the usages to find out how that view gets filled in with your data. With bindings, all your context is within the same file (the XML) so it is immediately visible. More time and effort saved, with no language switch (XML/Java).

*   **Less error prone**: In addition to the reduced code, data binding expressions are protected against null pointers exceptions if the data passed is null. So, code like this: 

    if (order == null) {
       User user = order.getUser(); 
       if (user != null && user.getName() != null) {
               textField.setText(user.getName());       
       }
    }

becomes simply:

    android:text="@{order.user.name}"

Which wil work even if order or user or name are null. The result is a much less fragile code avoiding errors derived from developers forgetting to do some null checks.

*   **Compile time resolved expressions**: In contrast with other technologies, Android data binding are compile time resolved, so you can detect issues during compilation and your app performance does not get affected.

*   **Easy to adopt**: Adopting bindings is very easy and does not require big refactors. Moreover, you can even decide to include it in your existing projects gradually instead of an all-or-none approach. 

*   **New paradigms and architectures**: They open the doors for better integration of other programming paradigms and architectures like [Reactive Programming](https://en.wikipedia.org/wiki/Reactive_programming) or [MVVM](http://blog.stablekernel.com/mvvm-on-android-using-the-data-binding-library).

*   **Easier to test activities/fragments**: Less code and dependencies on your activities managing the view results in much easier code to test. This is especially the case if you follow a MVVM approach. 

#### Basic Concepts

When working with Data Bindings in Android there are many features to explore. However, I do not want this post to become a huge so let’s explore only some of the most important concepts. Please, refer to the [android documentation](https://developer.android.com/tools/data-binding/guide.html) to get all the information and the installation instructions:

##### Using data variables

The most basic thing in bindings are the expressions. An expression is written in the layout XML and can be composed by several operands, but in general they get the form of:

    <TextView android:text="@{product.name}"…>

But how does it work? where does the product come from? 

In order to use the variable product you first need to declare it in the layout XML itself, on the top, as part of the <data> tag. So, for example:

    <?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
    	   xmlns:tools="http://schemas.android.com/tools">
    	   <data>
    	       <variable name="product" type="com.angelolloqui.Product" />
    	   </data>
    	   <LinearLayout…>

The moment you declare the data tag, the system will create a DataBinding class with the name of the xml plus the “Binding” suffix. So, if your layout XML file is called product\_detail\_layout.xml, then a new class with the name ProductDetailLayoutBinding will be automatically generated by the SDK, including setters for the variables declared in the XML. In addition to the class, a new R like class called BR will also be generated, containing the name of the variables you have declared so far for the data binding.

Then, all the remaining work left is to pass the variable data from your activity/fragment to the layout. You can do this replacing your old view inflation and findViewById related code:

    setContentView(R.layout.product_view);
    TextView nameTextView = (TextView) findViewById(R.id.nameTextView);
    TextView priceTextView = (TextView) findViewById(R.id.priceTextView);
    ...
    nameTextView.setText(product.name);
    priceTextView.setText(product.price);
    ...

By something like:

    ProductViewBinding binding = DataBindingUtil.setContentView(this, R.layout.product_view);
    binding.setProduct(product);

And that is all! you now have the product passed to the view, and the view will be the one deciding in which way to print the product name, price,...  with expression like the one above. But there is more than that, because you can use data binding in virtually any property, so for example you could bind the visibility as well with:

    <View android:visibility="@{product.isAvailable? View.VISIBLE : View.GONE}"…>

Or enable a button:

    <Button android:enabled="@{product.isAvailable}" …>

We can even go a step further and call any instance method or static class method that we have in our project, so we could even do things like:

    <TextView android:text="@{DateUtils.formatDate(product.creationDate)}"…>

Or any other combination of attributes that you can think of.

In brief, all your old code doing the findViewById, setText, setVisibility, etc can be removed and replaced by a much shorter and safer version (as explained in benefits)

##### Sending Events

So now we know how to pass information down, from the activity/fragment to the layout. But that is only half of the job, the other half is notifying events up (like a button press). With databindings you can also do that. All you need to do is define a method that receives a View as first parameter (normally an interface):

    public interface MyHandler {
       public void doMyAction(View view);
    }

And pass the implementor to the view as any other variable:

    public class ProductActivity extends Activity implements MyHandler {
    …
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       …   
       binding.setHandler(this);
    }
    
    @Override
    public void doMyAction(View view) {
    	//This is my action
    }

Then, in your view, you can use it as other bindings:

    <data>   
       <variable name="handler" type="com.angelolloqui.MyHandler" />
    </data>
    …
    <Button
       android:onClick="@{handler.doMyAction}"/>

When the button gets clicked then the handler method doMyAction will be invoked. Note that at the moment of writing this post, the data binding listeners were not accepting extra parameters, so in some cases you will need to hack a bit around that. Nevertheless, this is a [known issue](https://code.google.com/p/android/issues/detail?id=185097) that I believe will be added soon.

One important difference with pre-databinding expressions is that the handler can be anything. For this example I opted to implement the handler in the activity to keep it simple, but it could be any other object. 

#### Advanced concepts

So far we have seen the very basics of data binding. However, there are many other features. Let’s explore a few:

##### Layout composition

If your view is made of small pieces, you can include them as usual and pass the data down to the included views as follows:

    <LinearLayout>
           <include layout="@layout/name"
               bind:user="@{user}"/>
           <include layout="@layout/contact"
               bind:user="@{user}"/>
    </LinearLayout>

##### Binding Adapters

Being able to use data binding expressions in android properties is great, but what if we want something that does not come out of the box in Android? there is where the @BindingAdapter comes into play. Let’s explore a concrete example: adding capabilities on ImageView to load a remote image with an data binding expression in the XML.

First, we need to create the class that will handle the load, and a static method annotated with @BindingAdapter with the first parameter receiving the destination object (ImageView), and any other parameter with the data (the url):

    public class ImageViewBindingAdapter {
       private static ImageLoader imageLoader;
    
       @BindingAdapter("imageUrl")
       public static void loadImage(ImageView view, String url) {
          if (url != null) {
              imageLoader.get(url, ImageLoader.getImageListener(view, 0, 0));
          }
       }
    }

Just by including the previous class, we can use the following expressions anywhere in our app:

    <ImageView
       app:imageUrl="@{product.imageUrl}" …/>

Which will automatically trigger the loadImage method with the product data.

##### Dynamic binding

We have seen how to pass the data to the bindings by using the setters. However, sometimes you might want to be more dynamic. For example, you could have a generic RecycleViewAdapter that passes the data to the populated items, but for that you need to be flexible about the variable names. You could do that by writing something like:

    public void onBindViewHolder(BindingHolder holder, int position) {
       final T item = mItems.get(position);
       holder.getBinding().setVariable(myVariableId, item);
       holder.getBinding().executePendingBindings();
    }

Where myVariableId could have been passed to the adapter with a setter or alike:

    adapter.setMyVariableId(BR.item);

or even more dynamic with the variable name as a String with something like:

    Field field = BR.getField("item");
    adapter.setMyVariableId(field.getInt(field));

For example, in my current project I can populate a recycle view by using this technique in combination with the binding adapters in the following way:

    <android.support.v7.widget.RecyclerView
       app:itemLayout="@{R.layout.order_list_row_layout}"
       app:itemName="@{`order`}"
       app:items="@{orders}"   
       />

    public class RecycleViewBindingAdapter {
    	
    	@BindingAdapter({"itemLayout", "itemName", "items"})
    	public static  void configureAdapter(RecyclerView recyclerView, @LayoutRes int layoutId, String variableName, List items) {
    		GenericRecyclerAdapter adapter = new GenericRecyclerAdapter();
    		adapter.setLayoutId(layoutId);
    adapter.setItemName(variableName);
    		adapter.setItems(items);
       		recyclerView.setAdapter(adapter);
       }
       ...
    }

The previous expressions will automatically populate the recycle view with all orders, and inflate a order\_list\_row_layout with each order as a variable. No more adapter setup or information management in my activities!

#### Problems

So, the benefits are humongous, but it is fair to point out that not everything is smooth yet. I guess most (if not all) of the issues will be fixed with upcoming releases, but for now these are the main difficulties:

*   **Code completion:** When writing the XMLs there is no autocomplete, and Android Studio is also not capable of showing up (in red or some other way) when your expressions are incorrect.
*   **Error messages:** Without code completion you would expect nice error messages during compilation time to quickly detect the expression mistake. However, at the current beta, the compiler can only tell you the XML and line of code, but not the exact expression part that is wrong.
*   **Refactoring and usages not available:** Android Studio is currently not supporting refactors of expressions, so if you refactor your code you will have compile time errors on any expression using the method. Likewise, the "Find usages" does not show expressions.
*   **Numbers in text**: When you assign a text to a TexView (or other similar component) you can easily fall in the “number trap”. If you do something like:

    android:text="@{orderItem.quantity}"

You will get a runtime error similar to:

    android.content.res.Resources$NotFoundException: String resource ID #0x1

This is basically because the Int is treated as a resource Id, and not automatically converted to a String. It is very easy to solve, just convert the Int to String with something like a toString() but be aware of it! 

*   **Events can not contain parameters:** As explained above, a missing feature is the ability to pass extra information when sending events. This will however be resolved in a future version 2.

#### Conclusion

This has been a pretty long post. We have covered the **basics of Data Binding** and we explained the most **important features** of this new technology. We have also covered a few of the current **open issues**, none of them very critical but somehow annoying.

All in all, DataBinding is a **very exciting technology**, with great benefits on code quality and huge potential. I have no doubts that this will very quickly become the standard way of building Android apps, so why not start today?