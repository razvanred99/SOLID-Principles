# The Liskov Substitution Principle

The Liskov Substitution Principle was introduced by Barbara Liskov in 1987 in a keynot at a conference. The Liskov Substitution Principle states the following:

> Objects in a program should be repleaceble with instances of their subtypes without altering the correctness of that program.

## An Example of Replacing Object Instace with Subtypes

Java is a statically typed language. The compiler is very good at catching type errors and notifying us via the errors that it reports. We've all done this many times. You try to assing a String to a Long or vice versa, and the compiler tells you you've made a mistake. The compiler is also very good at allowing us to write code that adheres to the Liskov Subtituion Principle.

Let's assume that you0ve writing some Android code that allows you to work with the ```List``` type in Java.

```kotlin
val ids: ArrayList<Int> = getCustomerIds()
val customers: List<Customer> = customerRepository.getCustomersWithIds(ids)
```

At this point, you just care that the ```customerRepository``` returns a ```List<Customer>```. Perhaps you've even written the ```CustomerRepository```, but because your backend isn't done yet, you've decided to separate your interface from the implementation.

Assume the code looks like this:

```kotlin
interface CustomerRepository {
    fun getCustomersWithIds(ids: List<Int>): List<Customer>
}

class CustomerRepositoryImpl : CustomerRepository {
    override fun getCustomersWithIds(ids: List<Int>): List<Customer> {
        val customers: ArrayList<Customer> = api.getWholeLottaCustomers(ids)
        return customers
    }
}
```

In the code sample above, the customer repository needs a list of customer IDs so that it can obtain those customers. The customer repository only requires that that list of customer IDs be of type ```List<Int>```. When we call the repository we provide an ```ArrayList<Int>``` like so:

```kotlin
val ids: ArrayList<Int> = getCustomerIds()
val customers: List<Customer> = customerRepository.getCustomersWithIds(ids)
```

How can that code still work?

This is the Liskov Substitution Principle at work. Since ```ArrayList<Int>``` is a subtype of ```List<Int>```, the program will not falter: We're replacing the instance of the requested type with an instance of its subtype.

The customer repository is depending upon the contract provided by the ```List``` interface. The ```ArrayList``` is an implementation of the ```List``` interface, therefore, when the program runs, the customer repository will not see that the type is of ```ArrayList```, but as an instance of ```List```.

## What if a specific type is required

In the Java type system, ```List<E>``` actually implements ```Collection<E>```.

```kotlin
val ids: Collection<Int> = getCustomerIds()
val customers: List<Customer> = customerRepository.getCustomersWithIds(ids)
```

The above code will not compile because the ```getCustomersWithIds``` method only accepts ```List<Int>```. ```List``` does implement ```Collection```, but ```Collection``` does not implement ```List```. So while a ```List``` is a ```Collection```, a ```Collection``` is not necessarily a ```List```. In this example, the compiler can't prove that the ```Collection<Int>``` is for sure a ```List<Int>```. When presented in this manner, these types are not compatible.
