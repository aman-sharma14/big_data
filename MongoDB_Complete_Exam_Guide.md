# MongoDB — Complete Exam Guide

---

## 1. STARTING MONGODB (Server + Shell)

```bash
# Start the MongoDB server (run in terminal 1)
mongod

# Start the MongoDB shell (run in terminal 2)
mongo

# OR in newer versions
mongosh
```

```js
// Check current database
db

// Show all databases
show dbs

// Switch to / create a database
use myDatabase

// Show all collections
show collections

// Drop current database
db.dropDatabase()

// Drop a specific collection
db.collectionName.drop()
```

---

## 2. IMPORTING JSON FILES

```bash
# Single JSON object per line (default)
mongoimport --db DATABASE_NAME --collection COLLECTION_NAME --type json --file path/to/file.json

# If JSON file is a single array [ {...}, {...}, {...} ] — use --jsonArray flag
mongoimport --db DATABASE_NAME --collection COLLECTION_NAME --type json --file path/to/file.json --jsonArray

# Examples from class:
mongoimport --db rest --collection restu --type json --file E:\restaurants.json
mongoimport --db IPL --collection match --file C:\matches.json --jsonArray
mongoimport --db blog --collection posts --type json --file post.json --jsonArray
```

> **⚠️ Edge Case**: If you forget `--jsonArray` on an array file, you'll get a parse error.  
> Always check if your JSON starts with `[` — if yes, add `--jsonArray`.

---

## 3. CREATE COLLECTION

```js
// Simple
db.createCollection("students")

// With options
db.createCollection("logs", {
  capped: true,          // fixed-size collection (Boolean)
  autoIndexID: true,     // auto index on _id (Boolean)
  size: 6142800,         // max size in bytes (number)
  max: 10000             // max number of documents (number)
})

// Check if created
show collections
```

---

## 4. INSERT

```js
// Insert one document
db.employees.insertOne({
  ename: "John",
  salary: 48000,
  skills: ["Python", "SQL"]
})

// Insert multiple documents
db.employees.insertMany([
  { ename: "Rina", age: 24, city: "Mumbai", skills: ["C", "Python"] },
  { empid: 203, ename: "Sahil", city: "Delhi", skills: ["Java"] }
])

// Nested document
db.emp.insertOne({
  empid: 104,
  ename: "Jenin",
  sal: 7500,
  age: 26,
  department: {
    did: 1,
    dname: "Testing",
    location: { city: "Mumbai", State: "Maharashtra" }
  }
})
```

> **⚠️ Edge Case**: MongoDB does NOT enforce a schema. Two documents in the same collection can have completely different fields. The `age` field can be a number in one doc and a string in another — MongoDB allows it.

---

## 5. FIND (READ)

```js
// Find all documents
db.posts.find()

// Find with condition
db.posts.find({ "author": "Rahul" })

// Pretty print
db.posts.find().pretty()

// Count results
db.posts.find({ "is_published": true }).count()
```

---

## 6. COMPARISON OPERATORS

| Operator | Meaning | Example |
|----------|---------|---------|
| `$gt` | greater than | `{ likes: { $gt: 50 } }` |
| `$gte` | greater than or equal | `{ likes: { $gte: 50 } }` |
| `$lt` | less than | `{ likes: { $lt: 30 } }` |
| `$lte` | less than or equal | `{ likes: { $lte: 30 } }` |
| `$ne` | not equal | `{ author: { $ne: "Rahul" } }` |
| `$eq` | equal (explicit) | `{ likes: { $eq: 45 } }` |

```js
// Age greater than 25
db.emp.find({ "age": { $gt: 25 } }).count()

// Salary between 30000 and 70000 (exclusive)
db.emp.find({ "salary": { $gt: 30000, $lt: 70000 } })

// Likes between 30 and 70 — same field, one query object
db.posts.find({ "likes": { $gt: 30, $lt: 70 } })
```

> **⚠️ Edge Case**: `$ne: "American"` will NOT match documents where the field doesn't exist at all — those are actually returned too. Use `$exists` + `$ne` together if needed.

---

## 7. LOGICAL OPERATORS

### $and
```js
// Explicit $and
db.posts.find({
  $and: [
    { "is_published": true },
    { "likes": { $gt: 50 } },
    { "tags": "ai" }
  ]
})

// Implicit AND (cleaner — conditions just comma separated)
db.posts.find({ "is_published": true, "likes": { $gt: 50 }, "tags": "ai" })
```

### $or
```js
db.emp.find({
  $or: [
    { "age": { $gt: 25 } },
    { "sal": { $gt: 6000 } }
  ]
})
```

### $and + $or together
```js
// Employee in CS dept AND (city is Mumbai OR salary > 50000)
db.emp.find({
  $and: [
    { "department": "CS" },
    {
      $or: [
        { "city": "Mumbai" },
        { "salary": { $gt: 50000 } }
      ]
    }
  ]
})
```

### $not
```js
// NOT with regex — excludes anything matching pattern
db.posts.find({ "cuisine": { $not: /American/i } })

// NOT with expression
db.emp.find({ "age": { $not: { $gt: 30 } } })
```

> **⚠️ Edge Case**: `$not` works with regex and expressions. For simple value exclusion, use `$ne` instead. `$not` cannot be used alone at the top level — it must wrap an operator.

### $nor
```js
// Documents that match NEITHER condition
db.emp.find({
  $nor: [
    { "dept": "HR" },
    { "salary": { $gt: 50000 } }
  ]
})
```

---

## 8. $IN and $NIN

```js
// $in — field matches ANY value in the array
db.emp.find({ "dept": { $in: ["HR", "IT", "CS"] } })

// $nin — field matches NONE of the values
db.emp.find({ "dept": { $nin: ["HR", "IT"] } })

// $in on array field (tags contains "mongodb" OR "python")
db.posts.find({ "tags": { $in: ["mongodb", "python"] } })
```

> **⚠️ Edge Case**: `$in` and `$nin` work with **both exact values and regex patterns**. :
```js
// Both Works
db.posts.find({ "cuisine": { $nin: [/American/i, /Chinese/i] } })

// 
db.posts.find({
  $and: [
    { "cuisine": { $not: /American/i } },
    { "cuisine": { $not: /Chinese/i } }
  ]
})

// OR using pipe in single regex
db.posts.find({ "cuisine": { $not: /American|Chinese/i } })
```

---

## 9. PROJECTION

```js
// Show only title, author, likes — hide _id
db.posts.find({}, { "title": 1, "author": 1, "likes": 1, "_id": 0 })

// Show all fields except comments
db.posts.find({}, { "comments": 0 })
```

> **⚠️ Edge Case**: You CANNOT mix 1s and 0s in the same projection object — except for `_id`. Either include what you want (all 1s) or exclude what you don't want (all 0s). `_id` is the only exception.
```js
// WRONG
db.posts.find({}, { "title": 1, "comments": 0 })  // ❌ Error

// CORRECT
db.posts.find({}, { "title": 1, "_id": 0 })        // ✅ Include mode
db.posts.find({}, { "comments": 0, "_id": 0 })     // ✅ Exclude mode
```

---

## 10. SORT, LIMIT, SKIP

```js
// Sort ascending (1) / descending (-1)
db.posts.find().sort({ "likes": -1 })

// Multiple field sort — ONE sort() call only
db.posts.find().sort({ "cuisine": 1, "borough": -1 })

// ❌ WRONG — second sort() overwrites the first
db.posts.find().sort({ "cuisine": 1 }).sort({ "borough": -1 })

// Limit
db.posts.find().limit(3)

// Skip
db.posts.find().skip(5)

// Chain: sort + skip + limit (pagination)
db.posts.find().sort({ "likes": -1 }).skip(5).limit(5)
```

---

## 11. $EXISTS

```js
// Documents WHERE field exists
db.students.find({ "address": { $exists: true } })

// Documents WHERE field does NOT exist
db.students.find({ "address": { $exists: false } })

// Field exists AND is not empty string
db.posts.find({ "address.street": { $exists: true, $ne: "" } })
```

> **⚠️ Edge Case**: A field with value `null` still passes `$exists: true` — it exists, just has a null value. If you want to exclude nulls too, add `$ne: null`.

---

## 12. $TYPE

```js
// Find documents where coord is a double (type 1)
db.restaurants.find({ "address.coord": { $elemMatch: { $type: "double" } } })

// Type can be string name or number
db.collection.find({ "field": { $type: "string" } })   // string
db.collection.find({ "field": { $type: "int" } })      // integer
db.collection.find({ "field": { $type: "double" } })   // double/float
db.collection.find({ "field": { $type: "bool" } })     // boolean
db.collection.find({ "field": { $type: "array" } })    // array
db.collection.find({ "field": { $type: "null" } })     // null
```

---

## 13. $MOD

```js
// Find documents where score is divisible by 7
db.restaurants.find({ "grades.score": { $mod: [7, 0] } })
//                                               ↑ divisor, ↑ remainder

// Scores with remainder 1 when divided by 3
db.collection.find({ "score": { $mod: [3, 1] } })
```

> **⚠️ Edge Case**: `$mod` works directly on array fields like `grades.score` — no `$elemMatch` needed. It checks if ANY element in the array satisfies the condition.

---

## 14. PATTERN MATCHING (REGEX)

```js
// Starts with "J"
db.students.find({ "name": /^J/ })

// Ends with "Patel"
db.students.find({ "name": /Patel$/ })

// Contains "Kumar" anywhere
db.students.find({ "name": /Kumar/ })

// Starts with "Sneha" and ends with "Reddy" (anything in between)
db.students.find({ "name": /^Sneha.*Reddy$/ })

// Case insensitive — i flag
db.students.find({ "name": /^rahul/i })

// Match "American" OR "Chinese"
db.posts.find({ "cuisine": /American|Chinese/i })

// NOT matching pattern
db.posts.find({ "cuisine": { $not: /American/i } })
```

> **⚠️ Edge Case**: Dataset values might have trailing spaces like `"American "`. If you query `$ne: "American"`, it will still match because `"American "` ≠ `"American"`. Use regex `/American/i` for safer matching.

---

## 15. ARRAY OPERATORS

### Checking array contains a value
```js
// Tags array contains "ai" (exact match check)
db.posts.find({ "tags": "ai" })

// Tags contains BOTH "ai" AND "machine learning" — use $all
db.posts.find({ "tags": { $all: ["machine learning", "ai"] } })
```

### $size
```js
// Comments array has exactly 0 elements
db.posts.find({ "comments": { $size: 0 } })

// Comments array has exactly 3 elements
db.posts.find({ "comments": { $size: 3 } })
```

> **⚠️ Edge Case**: `$size` only accepts exact numbers. You CANNOT do `{ $size: { $gt: 2 } }`. To check array length > N, use index trick instead.
> If you want to check if the array size is less than N, you can still use the "index trick."

For example, to check if the array length is less than 3 (i.e., 0, 1, or 2 elements), you would check if the element at index 2 (the 3rd element) does not exist (i.e., $exists: false).
{
  "yourArrayField.2": {
    "$exists": false
  }
}


### Checking array length > N (index trick)
```js
// Array has MORE than 1 element (index 1 exists means at least 2 elements)
db.posts.find({ "comments.1": { $exists: true } })

// Array has MORE than 3 elements
db.posts.find({ "comments.3": { $exists: true } })
```

### Dot notation on arrays
```js
// ANY element in grades.score is greater than 80
db.restaurants.find({ "grades.score": { $gt: 80 } })

// Access specific index — 2nd element (index 1) of grades array
db.restaurants.find({ "grades.1.score": 9 })

// Access specific index of coord (first coordinate)
db.restaurants.find({ "address.coord.0": { $gt: 40 } })
```

---

## 16. $ELEMMATCH

Used when **multiple conditions must match within the SAME array element**.

```js
// Without $elemMatch — matches across DIFFERENT elements (may be wrong)
db.books.find({ "reviews.rating": 5, "reviews.reviewer": "Jane" })
// ↑ This returns books where rating:5 exists in ANY review
//   AND reviewer:"Jane" exists in ANY review (could be different reviews!)

// With $elemMatch — BOTH conditions must match in the SAME review element
db.books.find({
  "reviews": {
    $elemMatch: {
      "rating": 5,
      "reviewer": "Jane"
    }
  }
})

// Score between 80 and 100 in the SAME grade entry
db.restaurants.find({
  "grades": {
    $elemMatch: {
      "score": { $gt: 80, $lt: 100 }
    }
  }
})
```

> **⚠️ Key Difference**:
> - `{ "grades.score": { $gt: 80, $lt: 100 } }` → `$gt: 80` could match one element, `$lt: 100` could match a different element
> - `{ "grades": { $elemMatch: { score: { $gt: 80, $lt: 100 } } } }` → BOTH conditions must be true in the SAME element

---

## 17. UPDATE OPERATORS

### $set — set / update a field value
```js
db.emp.updateOne({ "empid": 101 }, { $set: { "salary": 55000 } })
db.emp.updateMany({ "dept": "HR" }, { $set: { "status": "active" } })
```

### $inc — increment / decrement
```js
// Increment salary by 5000
db.emp.updateOne({ "empid": 101 }, { $inc: { "salary": 5000 } })

// Decrement (use negative value)
db.emp.updateOne({ "empid": 101 }, { $inc: { "salary": -1000 } })

// Increase likes by 10 for posts tagged "python"
db.posts.updateMany({ "tags": "python" }, { $inc: { "likes": 10 } })
```

### $unset — remove a field
```js
// Remove commission field from ALL documents
db.emp.updateMany({}, { $unset: { "commission": "" } })
// ↑ value "" doesn't matter, field gets deleted regardless
```

### $push — add to array (allows duplicates)
```js
db.posts.updateOne(
  { "post_id": 5 },
  { $push: { "comments": { "user": "John", "message": "awesome", "likes": 2 } } }
)
```

### $addToSet — add to array only if NOT already present
```js
// Add "featured" tag only if it doesn't already exist
db.posts.updateMany({ "likes": { $gt: 60 } }, { $addToSet: { "tags": "featured" } })
```

> **⚠️ $push vs $addToSet**: `$push` adds regardless (allows duplicates). `$addToSet` only adds if the value is not already in the array. Use `$addToSet` to avoid duplicate tags/values.

### $pull — remove matching elements from array
```js
// Remove "web" tag from ALL posts
db.posts.updateMany({}, { $pull: { "tags": "web" } })

// Remove all grades with score < 0
db.emp.updateMany({}, { $pull: { "grades": { "score": { $lt: 0 } } } })
```

### $rename — rename a field
```js
db.emp.updateMany({}, { $rename: { "ename": "name" } })
```

---

## 18. ARITHMETIC OPERATORS (in Aggregation Pipeline Updates)

```js
// $add — add 5000 bonus to all employees
db.employees.updateMany(
  { salary: { $exists: true } },
  [
    {
      $set: {
        salary: { $add: ["$salary", 5000] }
      }
    }
  ]
)

// $multiply — double the salary
db.employees.updateMany(
  {},
  [
    {
      $set: {
        salary: { $multiply: ["$salary", 2] }
      }
    }
  ]
)
```

> **⚠️ Note**: When using `$add` / `$multiply` inside an update, the second argument to `updateMany` must be an **array** `[ ]` (pipeline update syntax), not `{ }`. This is different from regular updates.

---

## 19. DELETE

```js
// Delete ONE document
db.emp.deleteOne({ "empid": 101 })

// Delete MANY documents matching condition
db.emp.deleteMany({ "dept": "Testing" })

// Delete ALL documents in collection
db.emp.deleteMany({})
```

---

## 20. INDEXING

```js
// Create index on single field (ascending)
db.mycol.ensureIndex({ "title": 1 })

// Create index (descending)
db.mycol.ensureIndex({ "title": -1 })

// Create index on multiple fields
db.mycol.ensureIndex({ "title": 1, "description": -1 })

// Modern method (same thing)
db.mycol.createIndex({ "title": 1 })

// View all indexes on a collection
db.mycol.getIndexes()

// Drop an index
db.mycol.dropIndex({ "title": 1 })
```

> **⚠️ ensureIndex vs createIndex**: Both work. `ensureIndex` is the older method, `createIndex` is the modern one. Both are valid in exams.

---

## 21. AGGREGATION PIPELINE

The aggregation pipeline processes documents through stages in order.

```js
db.collection.aggregate([
  { stage1 },
  { stage2 },
  { stage3 },
  ...
])
```

# MongoDB Aggregation — Quick Reference

---

## Pipeline Stages

| Stage | What it does | Example |
|-------|-------------|---------|
| `$match` | Filters documents (like `find`) | `{ $match: { "is_published": true } }` |
| `$group` | Groups documents, computes aggregates | `{ $group: { _id: "$author", "Total": { $sum: 1 } } }` |
| `$project` | Reshapes documents (show/hide/compute fields) | `{ $project: { _id: 0, title: 1, "Count": { $size: "$comments" } } }` |
| `$sort` | Sorts documents | `{ $sort: { "Total Likes": -1 } }` |
| `$limit` | Limits number of documents | `{ $limit: 5 }` |
| `$skip` | Skips documents | `{ $skip: 10 }` |
| `$unwind` | Flattens array field into individual documents | `{ $unwind: "$grades" }` |
| `$count` | Counts documents | `{ $count: "published_count" }` |

---

## Aggregate Expressions (used inside `$group` / `$project`)

| Expression | What it does | Example |
|-----------|-------------|---------|
| `$sum` | Sum values | `{ $group: { _id: "$dept", "Total Sal": { $sum: "$salary" } } }` |
| `$avg` | Average of values | `{ $group: { _id: null, "Avg Likes": { $avg: "$likes" } } }` |
| `$max` | Maximum value | `{ $group: { _id: null, "Max Score": { $max: "$grades.score" } } }` |
| `$min` | Minimum value | `{ $group: { _id: null, "Min Score": { $min: "$grades.score" } } }` |
| `$count` | Count documents | `{ $count: "total" }` |
| `$push` | Collect values into array (allows duplicates) | `{ $group: { _id: "$dept", "Names": { $push: "$name" } } }` |
| `$addToSet` | Collect unique values into array | `{ $group: { _id: "$dept", "Cuisines": { $addToSet: "$cuisine" } } }` |
| `$first` | First value in group | `{ $group: { _id: "$author", "First Post": { $first: "$title" } } }` |
| `$last` | Last value in group | `{ $group: { _id: "$author", "Last Post": { $last: "$title" } } }` |

---

## Full Pipeline Example (combining stages)

```js
db.posts.aggregate([
  { $match: { "is_published": true } },         // Stage 1: filter
  { $unwind: "$tags" },                          // Stage 2: flatten tags array
  { $group: {                                    // Stage 3: group & compute
      _id: "$author",
      "Total Likes": { $sum: "$likes" },
      "All Tags": { $addToSet: "$tags" }
  }},
  { $match: { "Total Likes": { $gt: 100 } } },  // Stage 4: HAVING filter
  { $sort: { "Total Likes": -1 } },              // Stage 5: sort
  { $skip: 0 },                                  // Stage 6: skip (pagination)
  { $limit: 5 },                                 // Stage 7: limit
  { $project: {                                  // Stage 8: reshape output
      _id: 0,
      author: "$_id",
      "Total Likes": 1,
      "All Tags": 1
  }}
])
```

> **Key Rule:** `$match` BEFORE `$group` = WHERE (filter documents first).  
> `$match` AFTER `$group` = HAVING (filter the grouped results).

---

### $match
```js
// Filter published posts only
db.posts.aggregate([
  { $match: { "is_published": true } }
])
```

### $group
```js
// Group by author, count posts
db.posts.aggregate([
  { $group: { _id: "$author", "Total Posts": { $sum: 1 } } }
])

// _id: null means group ALL documents into one result
db.posts.aggregate([
  { $group: { _id: null, "Max Likes": { $max: "$likes" } } }
])

// Total salary per department
db.emp1.aggregate([
  { $group: { _id: "$dname", "Total Sal": { $sum: "$sal" } } }
])
```

### $project
```js
// Show title + computed comment count (preferred for Q20 type questions)
db.posts.aggregate([
  {
    $project: {
      _id: 0,
      title: 1,
      "Comments count": { $size: "$comments" }
    }
  }
])
```

> **⚠️ $project vs $group for single-document reshaping**: Always use `$project` when reshaping individual documents. Use `$group` only when aggregating ACROSS multiple documents. With `$group`, the grouping field shows as `_id` which is messy.

### $sort in aggregation
```js
db.posts.aggregate([
  { $group: { _id: "$author", "Total Likes": { $sum: "$likes" } } },
  { $sort: { "Total Likes": -1 } }
])
```

### $unwind
```js
// Flattens the comments array — one document per comment
db.posts.aggregate([
  { $unwind: "$comments" }
])

// After unwind, you can group on comment fields
db.posts.aggregate([
  { $unwind: "$comments" },
  { $group: { _id: "$comments.user", "Total Comment Likes": { $sum: "$comments.likes" } } }
])
```

### $count
```js
db.posts.aggregate([
  { $match: { "is_published": true } },
  { $count: "published_count" }
])
```

---

### Common Pipeline Patterns

**Pattern 1: Filter → Group**
```js
// Average likes for published posts only
db.posts.aggregate([
  { $match: { "is_published": true } },
  { $group: { _id: null, "Avg Likes": { $avg: "$likes" } } }
])
```

**Pattern 2: Group → Filter (HAVING equivalent)**
```js
// Authors whose TOTAL likes > 100
db.posts.aggregate([
  { $group: { _id: "$author", "Total Likes": { $sum: "$likes" } } },
  { $match: { "Total Likes": { $gt: 100 } } }
])
```
> **⚠️ Rule**: `$match` BEFORE `$group` = filter documents first (like WHERE).  
> `$match` AFTER `$group` = filter groups (like HAVING in SQL).

**Pattern 3: Filter → Group → Sort**
```js
// Total salary for HR and IT departments, sorted highest first
db.emp1.aggregate([
  { $match: { dname: { $in: ["HR", "IT"] } } },
  { $group: { _id: "$dname", "Total Sal": { $sum: "$sal" } } },
  { $sort: { "Total Sal": -1 } }
])
```

---

## 22. $WHERE (JavaScript expressions)

```js
// Find posts where any comment has likes divisible by 3
db.posts.find({
  $where: function() {
    return this.comments.some(c => c.likes % 3 === 0);
  }
})

// Find grades where score divisible by 7
db.restaurants.find({
  $where: function() {
    return this.grades.some(g => g.score % 7 === 0);
  }
})
```

> **⚠️ Note**: `$where` is slower than native operators because it executes JavaScript. Use `$mod` when possible. Use `$where` only when the logic can't be expressed with standard operators.

---

## 23. NESTED DOCUMENT QUERIES

```js
// Query on nested field — always use quotes + dot notation
db.emp.find({ "department.location.city": "Mumbai" })

// Update nested field
db.emp.updateOne(
  { "empid": 104 },
  { $set: { "department.dname": "Development" } }
)
```

---

## 24. QUICK REFERENCE — EXAM CHEAT SHEET

### Braces Reminder
- `{}` → Curly braces = document / conditions / update operations
- `[]` → Square braces = arrays / pipeline stages / $and/$or conditions

### Dot Notation Rules
- Always wrap in quotes: `"address.coord.0"`
- `grades.1.score` → 2nd element's score
- `address.coord.0` → 1st coordinate value

### When to use $elemMatch
- Use when: multiple conditions must match in the **SAME** array element
- Skip when: single condition on an array field

### Operator Quick Reference

| Task | Operator |
|------|---------|
| Any one of these values | `$in` |
| None of these values | `$nin` |
| All of these values in array | `$all` |
| Multiple conditions on same array element | `$elemMatch` |
| Field exists | `$exists: true` |
| Divisible by | `$mod: [divisor, 0]` |
| Starts with | `/^word/` |
| Ends with | `/word$/` |
| Contains | `/word/` |
| Case insensitive | `/word/i` |
| NOT matching regex | `$not: /word/` |
| Increment value | `$inc` |
| Set value | `$set` |
| Remove field | `$unset` |
| Add to array | `$push` |
| Add to array (no dups) | `$addToSet` |
| Remove from array | `$pull` |

### Aggregation Stage Order (common)
```
$match → $unwind → $group → $match → $sort → $project → $limit
```

### $group _id meanings
- `_id: "$fieldName"` → group by that field
- `_id: null` → treat entire collection as one group
- `_id: { a: "$field1", b: "$field2" }` → group by multiple fields

---

## 25. TRICKY EDGE CASES SUMMARY

| Situation | Right approach |
|-----------|---------------|
| Exclude regex values | `$and` + `$not`, NOT `$nin` with regex |
| Array length > N | `"array.N": { $exists: true }` |
| Multiple conditions on SAME array element | `$elemMatch` |
| Sort on multiple fields | ONE `.sort({ f1: 1, f2: -1 })` call |
| Can't mix 1 and 0 in projection | Use only 1s OR only 0s (except `_id`) |
| $where is slow | Use native operators (`$mod`, etc.) when possible |
| `$add`/`$multiply` in updates | Use pipeline update syntax `[ { $set: ... } ]` |
| `$match` before `$group` | Filters documents (WHERE) |
| `$match` after `$group` | Filters groups (HAVING) |
| Trailing space in data | Use regex `/word/i` not exact `$ne: "word"` |
| Check empty array | `{ "field": { $size: 0 } }` OR `{ "field": { $eq: [] } }` |
| Check non-empty array | `{ "field.0": { $exists: true } }` |
