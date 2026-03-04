# MongoDB Queries — restaurants.json

> **Collection:** `db.restu` &nbsp;|&nbsp; **Import:** `mongoimport --db rest --collection restu --type json --file restaurants.json`

---

### 1. Find all restaurants in the borough "Brooklyn"

```js
db.restu.find({ "borough": "Brooklyn" })
```

---

### 2. Find restaurants that serve "American" cuisine (case-insensitive, handles trailing spaces)

```js
// ✅ Safe — regex handles trailing spaces like "American "
db.restu.find({ "cuisine": /American/i })
```

---

### 3. Find restaurants in "Manhattan" that have a grade "A"

```js
// ✅ Correct — dot notation on array field
db.restu.find({ "borough": "Manhattan", "grades.grade": "A" })
```

---

### 4. Display only the name, cuisine, and borough of all restaurants (hide _id)

```js
// ✅ Correct — projection include mode
db.restu.find({}, { "name": 1, "cuisine": 1, "borough": 1, "_id": 0 })
```

---

### 5. Find restaurants NOT serving "American" or "Chinese" cuisine

```js
// ✅ Correct — $and + $not with regex (safer than $nin with regex)
db.restu.find({
  $and: [
    { "cuisine": { $not: /American/i } },
    { "cuisine": { $not: /Chinese/i } }
  ]
})

// OR using pipe in single regex
db.restu.find({ "cuisine": { $not: /American|Chinese/i } })
```

---

### 6. Find restaurants where any grade score is greater than 80

```js
// ✅ Correct — dot notation checks any element in grades array
db.restu.find({ "grades.score": { $gt: 80 } })
```

---

### 7. Find restaurants where a grade score is between 80 and 100 in the SAME grade entry

```js
// ✅ Correct — $elemMatch ensures BOTH conditions match in the SAME element
db.restu.find({
  "grades": {
    $elemMatch: {
      "score": { $gt: 80, $lt: 100 }
    }
  }
})

// ❌ Wrong — $gt could match one element, $lt could match a different element
db.restu.find({ "grades.score": { $gt: 80, $lt: 100 } })
```

---

### 8. Find restaurants where the score of the 2nd grade entry is exactly 9

```js
// ✅ Correct — dot notation with index (grades.1 = second element)
db.restu.find({ "grades.1.score": 9 })
```

---

### 9. Find restaurants where the first coordinate (longitude) is greater than 40

```js
// ✅ Correct — access first element of coord array
db.restu.find({ "address.coord.0": { $gt: 40 } })
```

---

### 10. Find restaurants where the coordinates are of type "double"

```js
// ✅ Correct — $elemMatch + $type on nested array
db.restu.find({ "address.coord": { $elemMatch: { $type: "double" } } })
```

---

### 11. Find restaurants whose name starts with "Mc"

```js
// ✅ Correct — ^ anchors to start
db.restu.find({ "name": /^Mc/ })
```

---

### 12. Find restaurants whose name ends with "Restaurant"

```js
db.restu.find({ "name": /Restaurant$/i })
```

---

### 13. Find restaurants where any grade score is divisible by 7

```js
// ✅ Correct — $mod works directly on array fields
db.restu.find({ "grades.score": { $mod: [7, 0] } })
```

---

### 14. Find restaurants in "Brooklyn" or "Queens"

```js
// ✅ Option 1 — $in (cleanest)
db.restu.find({ "borough": { $in: ["Brooklyn", "Queens"] } })

// Option 2 — $or
db.restu.find({
  $or: [
    { "borough": "Brooklyn" },
    { "borough": "Queens" }
  ]
})
```

---

### 15. Find restaurants that have more than 3 grade entries

```js
// ✅ Correct — if index 3 exists, array has at least 4 elements (> 3)
db.restu.find({ "grades.3": { $exists: true } })
```

---

### 16. Sort restaurants by name alphabetically (A → Z) and show only the first 10

```js
db.restu.find({}, { "name": 1, "_id": 0 }).sort({ "name": 1 }).limit(10)
```

---

### 17. Find the total number of restaurants in each borough

```js
// ✅ Correct — $group by borough, count with $sum: 1
db.restu.aggregate([
  { $group: { _id: "$borough", "Total Restaurants": { $sum: 1 } } },
  { $sort: { "Total Restaurants": -1 } }
])
```

---

### 18. Find the average score for each cuisine type

```js
// ✅ Correct — $unwind grades first, then $group by cuisine
db.restu.aggregate([
  { $unwind: "$grades" },
  { $group: { _id: "$cuisine", "Avg Score": { $avg: "$grades.score" } } },
  { $sort: { "Avg Score": -1 } }
])
```

---

### 19. Find cuisines where the maximum score across all their restaurants exceeds 100

```js
// ✅ Correct — $unwind → $group → $match (HAVING pattern)
db.restu.aggregate([
  { $unwind: "$grades" },
  { $group: { _id: "$cuisine", "Max Score": { $max: "$grades.score" } } },
  { $match: { "Max Score": { $gt: 100 } } }
])
```

---

### 20. List each restaurant's name along with the number of grades it has received

```js
// ✅ Option 1 — $project (PREFERRED — reshaping individual documents)
db.restu.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      "Grade Count": { $size: "$grades" }
    }
  }
])

// Option 2 — $group (title shows as _id — less clean)
db.restu.aggregate([
  {
    $group: {
      _id: "$name",
      "Grade Count": { $sum: { $size: "$grades" } }
    }
  }
])
```
> Use **Option 1** — `$project` is for reshaping a single document. `$group` is for aggregating across multiple documents and shows name under `_id` which is messy.
