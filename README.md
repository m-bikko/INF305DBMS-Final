# Connect
mongosh "mongodb+srv://cluster0.9zgcwn7.mongodb.net" --apiVersion 1 --username dias
pass: 2tYYiLFvsdcBUzJf

mongodb+srv://dias:<2tYYiLFvsdcBUzJf>@cluster0.9zgcwn7.mongodb.net/
mongodb+srv://dias:<password>@cluster0.9zgcwn7.mongodb.net/ 
f5p96VLbrP5gBqJf   vscode
mongodb+srv://<vscode>:<f5p96VLbrP5gBqJf>@cluster0.9zgcwn7.mongodb.net/



## Import example code

```monogsh
mongoimport --uri "mongodb://root:<PASSWORD>@atlas-host1:27017,atlas-host2:27017,atlas-host3:27017/<DATABASE>?ssl=true&replicaSet=myAtlasRS&authSource=admin" --collection myData --drop --file /somedir/myFileToImport.json
```

#Create db
```javadcript
// Users
db.createCollection("users")

// Addresses
db.createCollection("addresses")

// Orders
db.createCollection("orders")

// Order_Items
db.createCollection("order_items")

// Products
db.createCollection("products")

// Categories
db.createCollection("categories")

// Manufacturers
db.createCollection("manufacturers")

// Product_Details
db.createCollection("product_details")

// Reviews
db.createCollection("reviews")

// Payment_Methods
db.createCollection("payment_methods")
```

# Query

1. Get the total stock quantity of all products:
```javascript
db.products.aggregate([
  {
    $group: {
      _id: null,
      total_stock: { $sum: "$STOCK" }
    }
  }
])
```
2. Get the most frequently used payment method:
```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$PAYMENT_METHOD_ID",
      count: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "payment_methods",
      localField: "_id",
      foreignField: "PAYMENT_METHOD_ID",
      as: "payment_method"
    }
  },
  {
    $sort: { count: -1 }
  },
  {
    $limit: 1
  }
])
```
3. Get the average order value:
```javascript
db.orders.aggregate([
  {
    $group: {
      _id: null,
      average_order_value: { $avg: "$TOTAL_PRICE" }
    }
  }
])
```
4.Get the user with the highest total spending:
```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$USER_ID",
      total_spending: { $sum: "$TOTAL_PRICE" }
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "user_id",
      as: "user"
    }
  },
  {
    $sort: { total_spending: -1 }
  },
  {
    $limit: 1
  }
])
```
5. Get the number of reviews for a specific product:
```javascript
db.reviews.countDocuments({ PRODUCT_ID: 1 })
```
6. Get the total number of distinct manufacturers:
```javascript
db.manufacturers.distinct("MANUFACTURER_ID").length
```
7. Get the average rating of a product based on its reviews:
```javascript
db.reviews.aggregate([
  {
    $match: { PRODUCT_ID: 2 }//here
  },
  {
    $group: {
      _id: "$PRODUCT_ID",
      average_rating: { $avg: "$RATING" }
    }
  }
])
```
8. Get the users who have purchased a specific product:
```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "order_items",
      localField: "ORDER_ID",
      foreignField: "ORDER_ID",
      as: "order_items"
    }
  },
  {
    $match: {
      "order_items.PRODUCT_ID": 1//here
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "USER_ID",
      foreignField: "USER_ID",
      as: "user"
    }
  }
])
```
9. Find the most purchased product:
```javascript
db.order_items.aggregate([
  {
    $group: {
      _id: "$product_id",
      total_quantity: { $sum: "$quantity" }
    }
  },
  {
    $sort: { total_quantity: -1 }
  },
  {
    $limit: 1
  },
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "product_id",
      as: "product"
    }
  }
])

```
10. Find the most active user (based on the number of orders):
```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$user_id",
      total_orders: { $sum: 1 }
    }
  },
  {
    $sort: { total_orders: -1 }
  },
  {
    $limit: 1
  },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "user_id",
      as: "user"
    }
  }
])
```
11. Get the total revenue by category:
```javascript
db.order_items.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "product_id",
      as: "product"
    }
  },
  {
    $unwind: "$product"
  },
  {
    $group: {
      _id: "$product.category_id",
      total_revenue: { $sum: { $multiply: ["$quantity", "$price"] } }
    }
  },
  {
    $lookup: {
      from: "categories",
      localField: "_id",
      foreignField: "category_id",
      as: "category"
    }
  }
])
```
12. Find the product with the most reviews:

```javascript
db.reviews.aggregate([
  {
    $group: {
      _id: "$product_id",
      total_reviews: { $sum: 1 }
    }
  },
  {
    $sort: { total_reviews: -1 }
  },
  {
    $limit: 1
  },
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "product_id",
      as: "product"
    }
  }
])
```
13. Find all products that have never been ordered:
```javascript
db.products.find({
  product_id: {
    $nin: db.order_items.distinct("product_id")
  }
})
```
14. Find the manufacturer with the most products:
```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$manufacturer_id",
      total_products: { $sum: 1 }
    }
  },
  {
    $sort: { total_products: -1 }
  },
  {
    $limit: 1
  },
  {
    $lookup: {
      from: "manufacturers",
      localField: "_id",
      foreignField: "manufacturer_id",
      as: "manufacturer"
    }
  }
])
```
15. Find the most popular product category based on total quantity sold:
```javascript
db.order_items.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "product_id",
      as: "product"
    }
  },
  {
    $unwind: "$product"
  },
  {
    $group: {
      _id: "$product.category_id",
      total_quantity_sold: { $sum: "$quantity" }
    }
  },
  {
    $sort: { total_quantity_sold: -1 }
  },
  {
    $limit: 1
  },
  {
    $lookup: {
      from: "categories",
      localField: "_id",
      foreignField: "category_id",
      as: "category"
    }
  }
])
```


