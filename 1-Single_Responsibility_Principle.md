# The Single Responsibility Principle

> __A class should have only one reason to change.__

Taking the case of a RecyclerView and its adapter.

A RecyclerView is a flexible view which is capable of displaying a data set to the screen. In order for this data to get to the screen, we need a RecyclerView adapter.

An adapter takes the data from the data set and adapts it to a view. The most exercised part of an adapter is arguably the ```onBindViewHolder``` method (and sometimes ```ViewHolder``` itself, but we'll just stick with this method). The RecyclerView's adapter has one responsibility: mapping an object to its corresponding view that will be displayed on the screen.

Assume these objects and RecyclerView. Adapter implementation:

``` kotlin
data class LineItem(
    val description: String,
    val quantity: Int,
    val price: Long
)

data class Order(
    val orderNumber: Int
) {
    val lineItems: List<LineItem> = ArrayList()
}

class OrderRecyclerAdapter (
    private val items: List<Order>,
    private val itemLayout: Int
) : RecyclerView.Adapter<OrderRecyclerAdapter.ViewHolder> {

    override fun onCreateViewHolder (
        parent: ViewGroup,
        viewType: Int
    ) = ViewHolder (
            LayoutInflater
                .from(parent.context)
                .inflate(itemLayout, parent, false)
        )

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        // TODO: bind the view here
    }

    override fun getItemCount() = items.size

    inner class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val orderNumber: TextView = itemView.findViewById(R.id.order_number)
        val orderTotal: ImageView = itemView.findViewById(R.id.order_total)
    }
}
```

Example of an implementation that could look like:

``` kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val order = items[position]
    holder.orderTotal.text = order.orderNumber.toString()
    val total = order.lineItems.map(LineItem::price).sum()
    val formatter = NumberFormat.getCurrencyInstance(Locale.US)
    val totalValue = formatter.format(total / 100.0)
    holder.orderTotal.text = totalValue
    holder.itemView.tag = order
}
```

This code above violates the Single Responsibility Principle.

Why?

The adapter's ```onBindViewHolder``` method is not only mapping from an Order object to the view, but is also performing price calculations as well as formatting. This violates the Single Responsibility Principle. The adapter should only be responsible for adapting an order object to its view representation. The ```onBindViewHolder``` is performing two extra duties that it should not be.

Why is this a problem?

Including multiple responsibilities in a class can cause various problems.

The calculation logic for the order is now coupled to the adapter: if you need to display the total of an order elsewhere (most likely you do) you'll have to replicate that logic. Once that happens, your application is exposed to traditional software logic duplication issues that we're all familiar with. You update the the code in one place and forget to update it in another location, etc.

The second issue is the same as the first - you've coupled the formatting logic to the adapter. What if that needs to be moved or updated? At the end of the day, we're making this class do more than it should, and now the application is more susceptible to bugs due to too much responsibility in one location.

This simple example can be easily fixed by extracting the order total calculation into the Order object and them moving the currency formatting into a currency formatter class of some sort. This formatter can then be used by the Order too.

An updated ```onBindViewHolder``` method could look like this:

``` kotlin
overrind fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val order = items[position]
    holder.orderNumber.text = order.orderNumber.toString()
    holder.orderTotal.text = order.orderTotal // A String, the calculation and formatting moved elsewhere
    holder.itemView.tag = order
}
```

## What is meant by "Responsibility" though

Uncle Bob says:

> In the context of the Single Responsibility Principle we define a responsibility as "a reason for change". If you can think of more than one motive for changing a class, then that class has more than one responisibility.

This is sometimes really hard to see - especially if you've been in the codebase for a long time. At thant point, this famous quote usually comes to mind:

> You can't see the forest for the trees.

In the context of software, this means you're too close to the details of your code to see the bigger picture. For example - the class you're working on may look great, but that's because you've been working with it for so long its hard to see that it may have multiple responsibilities.

The challenge is knowing when to apply SRP and when not to. Taking the adapter example into account, if we look at the code again, we see various things happening that could necessitate the need for change in different areas for different reasons:

``` kotlin
class OrderRecyclerAdapter (
    private val items: List<Order>,
    private val itemLayout: Int
) : RecyclerView. Adapter<OrderRecyclerAdapter. ViewHolder> {

    override fun onCreateViewHolder (
        parent: ViewGroup,
        viewType: Int
    ) = ViewHolder (
            LayoutInflater
                .from(parent.context)
                .inflate(itemLayout, parent, false)
        )

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val order = items[position]
        holder.orderNumber.text = order.orderNumber.toString()
        holder.orderTotal.text = order.orderTotal
        holder.itemView.tag = order
    }

    override fun getItemCount() = items.size

    inner class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val orderNumber: TextView = itemView.findViewById(R.id.order_number)
        val orderTotal: ImageView = itemView.findViewById(R.id.order_total)
    }
}
```

The adapter is inflating a view, it's binidng an order to a view, it's constructing the ViewHolder, etc. This class has multiple responsibilities.

Should these responsibilities be broken apart?

This ultimately depends on how the application is changing over time. If the application changes in ways that affect the way the view is assembled and its connecting functions (the logic of the view), then as Uncle Bob states, the design will smell of Rigidity because one change is requiring another to change. The change of the view construction is also requiring a change of the adapter itself, causing the design to become rigid. However, it can also be argued that if the application is not changing in ways that require different functions to change at different times, then there is no need to separate them. In this case, separating them would be adding unnecessary needless complexity.

So, what do we do?

## An Example to Illustrate Rigidity

Let's assume a new product requirement comes in that proclaims when an order's total amount is zero, the view should display a bright yellow "FREE" image on the screen instead of a textual total amount. Where would this logic go? In one code path, you need a TextView, and in another, you need an ImageView. There are two places code needs to be changed:

1. In the view
2. In the presentation logic

In most cases, this is applied at the adapter level. Unfortunately, this forces the Adapter to be changed when your view is changed. If the logic for this is in the adapter, then this forces the logic in the adapter to change, as well as the code for the view. This adds yet another responsibility to the adapter.

This is exactly the point where something like the Model-View-Presenter pattern offers necessary decoupling so that the classes do not become too rigid, yet provide the flexibility for the extension, composability, and testing. For example, the view would implement an interface that defines how it will be interacted with, and the presenter would perform the necessary logic. The presenter in a Model-View-Presenter pattern is responsible for only the view/display logic, nothing more.

Moving this logic from the adapter into the presenter would help make the adapter adhere more to the single responsible principle.

Any RecyclerView adapter does already a lot of things like:

* Inflating the view
* Creating the ViewHolder
* Recycling the ViewHolder
* Providing item count
* etc.

Since SRP is about single resposibility, you're probably wondering whether or not some of those behaviors should be extracted to adhere to SRP.

> An axis of change is only an axis of change if the changes actually occur. It is not wise to apply the SRP, or any other principle for that matter, if there is no symptom.

While the Adapter still performs various actions, that is, in fact, what it is designed to do. After all, a RecyclerView adapter is simply an implementation of the Adapter pattern. In this case, keeping the view inflation and view holder mechanisms in place does make sense; that's what this class's responsibility is. However, introducing additional behavior (like view logic) breaks SRP and can be avoided by using the Model-View-Presenter pattern or other refactorings.
