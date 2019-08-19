# The Dependency Inversion Principle

> a. High-level modules should not depend on low-level modules. Both should depend on abstractions.

And

> b. Abstractions should not depend on details. Details should depend on abstractions.

The Dependency Inversion Principle basically says this:

> Depend on Abstractions. Do not depend on concretions.

## Migrating to support the Dependency Inversion Principle

In a traditional layered pattern software architecture design, higher level modules depend on lower level modules to do their job. For example, here's a very common layered architecture:

> Android UI -> Business Rules -> Data Layer

Let's assume that we have an expense tracking application that allows users to track their expenses. Given the traditional model above, when a user creates a new expense we would have three different operations happening.

* UI Layer: Allows user to enter data.
* Business Layer: Verifies that entered data matches a set of business rules.
* Data Layer: Allows for persistent storage of the expense data.

This might look like this:

```kotlin
findViewById<Button>(R.id.btn_save_expense).setOnClickListener { _ ->
    val expense: ExpenseModel = TODO() //... create the model from the view values
    val bl = BusinessLayer()
    if (bl.isValid(expense)){
        // Save it and Continue to the next screen
    } else {
        Toast.makeText(context, "Shucks, couldn't save expense. Error: ${bl.getValidationErrorFor(expense)}", Toast.LENGTH_SHORT).show()
    }
}
```

In the business layer we might have some code that resembles this pseudo code:

```kotlin
fun saveExpense(expense: Expense): Int {
    // some code to check for validity... then save
    // do some other logic, like check for duplicates/etc
    val dl = DataLayer()
    return dl.insert(expense)
}
```

The problem with the above code is that it breaks the Dependency Inversion Principle - namely item (a) from above: High-level modules should not depend on low-level modules. Both should depend on abstractions. The UI is depending upon a concrete instance of the business layer with this line:

```kotlin
val bl = BusinessLayer()
```

This forever ties the Android UI layer to the business layer, and the UI layer won't be able to do its job without the business layer.

The business layer also violates DIP, because it is depending upon a concrete implementation of the data layer with this line:

```kotlin
val dl = DataLayer()
```

If the higher-level modules should not depend on lower-level modules then how can an app do its job?

We definitely don't want a simple monolith class that does everything. Remember, we still want to adhere to the first SOLID principle too - the Single Responsibility Principle.

Changing your application from a traditional layered implementation to a dependency inverted architecture is done through a process known as **Ownership Inversion**.

## Implementing Ownership Inversion

We definitely don't want lower-level modules depending on higher-level modules either. We need to invert this relationship completely, from both ends.

How can this be done? With abstractions.

An interface is simply a contract that informs the consumer of the interface of all the possible operations an implementor may have.

This allows each layer to rely on an interface, which is an abstraction, rather than a concrete implementation.

Let's assume that you have that DataLayer class and it looks like this:

```kotlin
class DataLayer {
    fun insert(expense: Expense){
        // go to db and insert
    }

    fun find(id: Int): Expense {
        val expense: Expense = TODO() // find an expense in the db
        return expense
    }

    fun findAll(): List<Expense> {
        val expenses: List<Expense> = // find all expenses in the db/etc
        return expenses
    }
}
```

Since we want to depend an abstraction, we need to extract an interface off the class:

```kotlin
interface IDataLayer {
    fun insert(expense: Expense)
    fun find(id: Int): Expense
    fun findAll(): List<Expense>
}

class DataLayer : IDataLayer {
    override fun insert(expense: Expense){
        // go to db and insert
    }

    override fun find(id: Int): Expense {
        val expense = // find an expense in the db
        return expense
    }

    override fun findAll(): List<Expense> {
        val expenses: List<Expense> = // find all expenses in the db/etc
        return expenses
    }
}
```

Now you have an interface you can use to depend on! However, it still needs to be utilized because the business layer still depends on the concrete data layer. Going back to the business layer, you can change that code to have the dependency injected through the constructor like this:

```kotlin
class BusinessLayer(private val dataLayer: IDataLayer) {
    fun saveExpense(expense: Expense): Int {
        // some code to check for validity... then save
        // do some other logic, like check for duplicates/etc
        return dataLayer.insert(expense)
    }
}
```

The business layer now depends upon an abstraction - the IDataLayer interface. The data layer is now injected via the constructor via what is known as "Constructor Injection".

So where does this data layer come from? It comes from whoever creates the Business Layer object. In this case, it would be the Android UI. However, we know that our previous example illustrates that the Android UI is tightly coupled to the business layer because it is creating a new instance. We need the business layer to also be an abstraction.

We have abstract also the BusinessLayer and create an interface that the Android UI could rely on:

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var businessLayer: IBusinessLayer

    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

Finally, our higher-level modules are relying on abstractions (interfaces). Out abstractions are not depending on details, they're also depending upon abstractions.

Now, the UI layer is depending on the business layer interface, and the business layer interface is depending on the data layer interface.

## Wiring it together in Android

How do I rely on an abstraction in the Android UI layer if this is the top layer?

There are a couple ways you can solve it in Android using a creational pattern such as the factory of factory method pattern, or a dependency injection framework.

Example with a Dependency Injection framework:

```kotlin
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var businessLayer: IBusinessLayer

    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // businessLayer is null at this point, can't use it
        getInjector().inject(this) //this does the injection
        // businessLayer field is now injected valid and NOT NULL, we can use it

        // do something with the business layer...
        businessLayer.foo()
    }
}
```

If you don't use a creational pattern or dependency injection framework you'll be left writing code that looks like this:

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var businessLayer: IBusinessLayer

    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        businessLayer = BusinessLayer(DataLayer())
        businessLayer.foo()
    }
}
```

While this may not look too bad at this time, you'll eventually find that your object graph will grow to be quite large, and instantiating objects this way is very error-prone and breaks many of the SOLID principles.
