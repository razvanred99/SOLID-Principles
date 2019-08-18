# The Open/Closed Principle

> software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

You should strive to write code that doesn't have to be changed every time the requirements change. In Android, we're using Java, so this principle can be implemented with inherintance and polymorphism.

## An Example of the Open/Closed Principle

Let's assume that you have an application that has a requirement to calculate area for any given shape.

We abstract the area calculation into a class named `` `AreaManager` ` `. The ` ` `AreaManager` `` class has a single responsibility - to calculate the total area of the shapes.

Let's assume we're working with rectangle crops right now so we have a `` `Rectangle` `` class that represents this. Here's what these classes might look like:

``` kotlin
data class Rectangle(
    val length: Double,
    val height: Double
)

class AreaManager {
    fun calculateArea(vararg shapes: Rectangle): Double {
        return shapes.map {
            it.length * it.height
        }.sum()
    }
}
```

The `` `AreaManager` `` class does its job well until next week when we have a new type of crop show up - a circle:

``` kotlin
data class Circle(
    val radius: Double
)
```

Since there is a new shape we have to account for, we have to change the area manager:

```kotlin
class AreaManager {
    fun calculateArea(vararg shapes: Any) : Double {
       return shapes.map { shape ->
            when (shape) {
                is Rectangle -> shape.length * shape.height
                is Circle -> shape.radius.pow(2) * Math.PI
                else -> throw RuntimeException("Shape not supported!")
            }
        }.sum()
    }
}
```

The code is starting to smell already.

If we have a triangle show up, or any other polygon for that matter, we're going to be changing this class over and over.

This class violates the Open/Closed principle. It is not closed for modification and it is not open to extension. Every time a new shape comes along we have to modify the AreaManager.

How do we make the ```AreaManager``` class Open/Closed friendly?

## Implementing the Open/Closed Principle with Inheritance

Since the ```AreaManager``` is responsible for calculating the total area of all the shapes, and because the shape calculation is unique to each individual shape, it seems only logical to move the area calculation for each shape into its respective class.

But that makes the ```AreaManager``` still have to know about all the shapes? Because how dows it know that the object it's iterating over has an area method? Sure, this could be solved with reflection **or** we could have each of the shape classes inherit from an interface: the ```Shape``` interface:

```kotlin
interface Shape {
    fun getArea(): Double
}
```

Each class would implement this interface like this:

```kotlin
data class Rectangle(
    val length: Double,
    val height: Double
) : Shape {
    override fun getArea(): Double {
        return length * height
    }
}

data class Circle(
    val radius: Double
) : Shape {
    override fun getArea(): Double {
        return radius.pow(2) * Math.PI
    }
}
```

We can now make the ```AreaManager``` follow the Open/Closed principle by relying on this abstraction:

```kotlin
class AreaManager {
    fun calculateArea(vararg shapes: Shape): Double {
        return shapes.map(Shape::getArea).sum()
    }
}
```

We've made changes to the ```AreaManager``` that allow it to be closed for modification but open for extension. If we need to add a new shape, such as an octagon, the ```AreaManager``` will not need to be changed because it is open for extension through the ```Shape``` interface.
