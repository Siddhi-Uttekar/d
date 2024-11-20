## step 1 - data cleansing

```javascript
db.orders.insertMany([{
"order_id": "ORD002",
"customer_id": "CUST002",
"order_date": "2024-11-02",
"total_amount": 80.00,
"status": "Pending",
"payment_method": null,
"shipping_address": null,
"items": [
{ "product_id": "P003", "quantity": 1, "price": 80.00 }
]
}])
```

```javascript

# Set 'payment_method' to 'Unknown' if it's missing
db.orders.updateMany(
{ "payment_method": { $exists: false } }, // Check if the field does not exist
{ $set: { "payment_method": "Unknown" } } // Set default value
);

# Set 'payment_method' to 'Unknown' if it's null
db.orders.updateMany(
{ "payment_method": null }, // Check if the field is null
{ $set: { "payment_method": "Unknown" } } // Set default value
);

# Set 'shipping_address' to 'Not Provided' if it's null
db.orders.updateMany(
{ "shipping_address": null }, // Check if the field is null
{ $set: { "shipping_address": "Not Provided" } } // Set default value
);

db.orders.find().pretty();

```

## step 2 - data Integration using JSON

- import data.json file to show Integration

## step 3 - Data selection

- Selecting all total_amount values:

```javascript
db.orders.find({}, { "order_id": 1, "total_amount": 1, "\_id": 0 })
```

- This query selects all documents and extracts only the order_id and total_amount fields, excluding the MongoDB \_id.

## step 4 - data Transformation

- Transform the data for further analysis, such as aggregating the total spending by each
  customer.

```javascript 
db.orders.aggregate([
{
$group: {
_id: "$customer_id",
total_spending: { $sum: "$total_amount" }
}
}
])
```

## step 5 - Data Mining


```javascript
db.orders.aggregate([
{
$group: {
_id: "$customer_id",
 total_spent: { $sum: "$total_amount" }
}
},
{
$match: { total_spent: { $gt: 100 } }
}
])
```

## step 6 - Knowledge Representation

- Representing the results of the data mining query in a meaningful way, such as displaying high value customer and categorizing them.

- Define Spending Tiers
  First, you need to define the spending ranges for each category (1-5). For example:

Level 1 (Importance: 1): Customers who spent the least (e.g., $0 - $50)
Level 2 (Importance: 2): Customers who spent between $51 - $100
Level 3 (Importance: 3): Customers who spent between $101 - $200
Level 4 (Importance: 4): Customers who spent between $201 - $500
Level 5 (Importance: 5): Customers who spent above $500

```javascript
db.orders.aggregate([
{
$group: {
      _id: "$customer_id", // Group by customer_id
total_spent: { $sum: "$total_amount" }
}
},
{
$addFields: {
      importance: {
        $switch: {
          branches: [
            { case: { $lte: ["$total_spent", 50] }, then: 1 }, // Importance Level 1
{ case: { $and: [{ $gt: ["$total_spent", 50] }, { $lte: ["$total_spent", 100] }] }, then: 2 }, // Importance Level 2
{ case: { $and: [{ $gt: ["$total_spent", 100] }, { $lte: ["$total_spent", 200] }] }, then: 3 }, // Importance Level 3
{ case: { $and: [{ $gt: ["$total_spent", 200] }, { $lte: ["$total_spent", 500] }] }, then: 4 }, // Importance Level 4
{ case: { $gt: ["$total_spent", 500] }, then: 5 } // Importance Level 5
],
default: 0 // Default if no range matches, though in this case all should match
}
}
}
},
{
$project: { 
      _id: 0, 
      customer_id: "$\_id",
total_spent: 1,
importance: 1
}
}
])
```
