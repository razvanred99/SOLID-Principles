# The Interface Segregation Principle

> Make fine grained interfaces that are client-specific.

Another way to put this is:

> Many client-specific interfaces are better than one general purpose interface.

## Many client-specific interfaces

Let's start by taking as example the Android View.

The ```View``` class is the root superclass for all Android views. It is the root of ```TextView```, ```Button```, ```LinearLayout```, ```Checkbox```, etc.

Let's now assume that you are one of the developers who is writing the Android operating system. You realize that most likely every view might need to be clicked on. You create an interface called OnClickListener that is nested in the View class that looks like this:

```kotlin
interface OnClickListener {
    fun onClick(v: View)
}
```

As time progresses you find out that you need a listener for long press events. Seeing that it's something simple you decide to just throw it into the OnClickListener.

```kotlin
interface OnClickListener {
    fun onClick(v: View)
    fun onLongClick(v: View)
}
```

Some more time passes and you realize that you need to add some touch listeners to the view as well. The interface is still rather small so you decide to add another method to it.

Now your interface looks like this:

```kotlin
interface OnClickListener {
    fun onClick(v: View)
    fun onLongClick(v: View)
    fun onTouch(v: View, event: MotionEvent)
}
```

At this point, you decide to change the name of the interface from ```OnClickListener``` to ```ViewInteractions```, or something similar. Why? Mainly because a touch event is different than a click event.

This interface is becoming a problem - it's becoming generic and polluted.

## Why are generic polluted interfaces a problem

Using the same ```OnClickListener``` from above, let's image we're developing an app using the Android SDK that we've built. I want to attach a click listener to a button, so I wire it up like this:

```kotlin
val btnCreate: Button = findViewById(R.id.create)
btnCreate.setOnClickListener(
    object : View.OnClickListener {
        override fun onClick(v: View){
            myDatabase.createTask()
        }

        override fun onLongClick(v: View){
            // do nothing, we're not long clicking
        }

        override fun onTouch(v: View, event: MotionEvent) {
            // do nothing, we're not worried about touch
        }
    }
)
```

Those two last methods, ```onLongClick``` and ```onTouch``` are not doing anything. Sure, we could put some code there, but what if I don't need it? Most likely I'm only worried about a click, not the touch, not the long press.

This interface is too generic because it's requiring the client to implement all the methods, even if it doesn't not need them. It's trying to do too much.

By applying the definition, you eliminate the pollution in the client from unnecessary callbacks and code and that is not needed for the operation we're implementing. The client should only have to implement interface that it needs and no more.

Thankfully, the folks over at Google are very familiar with the Interface Segregation Principle, and have done separated interfaces in order to implement the operations: ```OnClickListener```, ```OnLongClickListener``` and ```OnTouchListener```.

There are times when you might need your interface to have multiple methods, and that's ok. For example, the Android TextView has the method ```addTextChangedListener```. The ```TextWatcher``` interface supplies three methods:

```kotlin
interface TextWatcher : NoCopySpan {
    fun beforeTextChanged(s: CharSequence, start: Int, count: Int, after: Int)

    fun onTextChanged(s: CharSequence, start: Int, before: Int, count: Int)

    fun afterTextChanged(s: Editable)
}
```

This is not a generic interface. These methods are all very specific to the interface and the client will most likely want to interact with them, therefore packaging them together in the same interface is the right thing to do.
