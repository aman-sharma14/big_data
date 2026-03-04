# MongoDB Practice Questions — restaurants.json (Advanced)

> **Collection:** `db.restu` | **Import:** `mongoimport --db rest --collection restu --type json --file restaurants.json`

---

### Q1. Find restaurants where a single grade entry has score between 50 and 80 AND grade is "B"

```js
db.restu.find({
  "grades": {
    $elemMatch: { "grade": "B", "score": { $gte: 50, $lte: 80 } }
  }
})
```
> ✅ `$elemMatch` ensures both conditions match the **same** grade element.

---

### Q2. Find restaurants where the 3rd grade element has a score greater than 10

```js
db.restu.find({ "grades.2.score": { $gt: 10 } })
```
> ✅ `grades.2` = index 2 = 3rd element (0-indexed).

---

### Q3. Find restaurants where any grade entry has grade "C", score greater than 50, on or after 2014-01-01

```js
db.restu.find({
  "grades": {
    $elemMatch: {
      "grade": "C",
      "score": { $gt: 50 },
      "date": { $gte: ISODate("2014-01-01T00:00:00.000Z") }
    }
  }
})
```
> ✅ All three conditions must match within the **same** grade entry — `$elemMatch` handles this.

---

### Q4. Find restaurants whose name starts with 'C' and ends with 's'

```js
db.restu.find({ "name": /^C.*s$/ })
```
> ✅ `^C` = starts with C, `.*` = anything in between, `s$` = ends with s.

---

### Q5. Find restaurants whose name contains exactly 4 characters

```js
db.restu.find({ "name": /^.{4}$/ })
```
> ✅ `.` matches any character, `{4}` means exactly 4 times, anchored by `^` and `$`.

---

### Q6. Find restaurants whose borough starts with a vowel (A, E, I, O, U)

```js
// Try on borough first
db.restu.find({ "borough": /^A|^E|^I|^O|^U/ })

// No borough starts with a vowel in this dataset — try on name instead
db.restu.find({ "name": /^A|^E|^I|^O|^U/ })
```
> ✅ `|` acts as OR between patterns. Each `^` anchors to the start.  
> 💡 Cleaner alternative using a character class:
```js
db.restu.find({ "name": /^[AEIOU]/i })
```

---

### Q7. Find restaurants whose street contains a number in it

```js
db.restu.find({ "address.street": /\d/ })
```
> ✅ `\d` matches any digit (0–9).

---

### Q8. Find restaurants where score is divisible by 5 but NOT by 10

```js
db.restu.find({
  $and: [
    { "grades.score": { $mod: [5, 0] } },
    { "grades.score": { $not: { $mod: [10, 0] } } }
  ]
})
```
> ✅ Must use explicit `$and` — you cannot write `"grades.score"` twice as separate keys (second silently overwrites the first).

---

### Q9. Find restaurants where score divided by 3 gives remainder 2

```js
db.restu.find({ "grades.score": { $mod: [3, 2] } })
```
> ✅ Syntax: `$mod: [divisor, remainder]` → `[3, 2]` = divide by 3, expect remainder 2.

---

### Q10. Find restaurants where zipcode field is missing entirely

```js
db.restu.find({ "address.zipcode": { $exists: false } })
```

---

### Q11. Find restaurants where building number is an empty string (handles spaces too)

```js
db.restu.find({ "address.building": /^$/ })
```
> ✅ Regex `^$` matches truly empty strings.  
> ✅ Field path is `"address.building"` — building is nested inside address.

---

### Q12. Find restaurants where coord field has exactly 2 elements

```js
db.restu.find({ "address.coord": { $size: 2 } })
```

---

### Q13. Find restaurant_id, name and score for restaurants in Queens — sorted by score descending then name ascending

```js
db.restu.find(
  { "borough": "Queens" },
  { "restaurant_id": 1, "name": 1, "grades.score": 1, "_id": 0 }
).sort({ "grades.score": -1, "name": 1 })
```
> ⚠️ `grades.score` is an array — sorting on it uses the highest score in the array per document. Acceptable for exam purposes.

---

### Q14. Display unique boroughs only

```js
// Using aggregation
db.restu.aggregate([
  { $group: { _id: "$borough" } }
])

// Simpler — using distinct
db.restu.distinct("borough")
```

---

### Q15. Find top 3 restaurants with the highest single score ever recorded

```js
// ❌ Wrong — find().sort() on an array field doesn't isolate individual scores
db.restu.find().sort({ "grades.score": -1 }).limit(3)

// ✅ Correct — unwind first to separate each grade into its own document
db.restu.aggregate([
  { $unwind: "$grades" },
  { $sort: { "grades.score": -1 } },
  { $limit: 3 },
  { $project: { _id: 0, name: 1, "grades.score": 1 } }
])
```
> ✅ `$unwind` flattens the grades array — each grade becomes its own document, then sort works correctly on individual scores.

---

### Q16. Find restaurants that are either in Manhattan with Italian cuisine OR in Brooklyn with Chinese cuisine

```js
db.restu.find({
  $or: [
    { "borough": "Manhattan", "cuisine": /Italian/i },
    { "borough": "Brooklyn", "cuisine": /Chinese/i }
  ]
})
```
> ✅ Each `$or` condition uses implicit AND (comma-separated) — Manhattan + Italian, Brooklyn + Chinese.

---

### Q17. Find restaurants where neither the score is above 80 nor the grade is "C"

```js
db.restu.find({
  $nor: [
    { "grades.score": { $gt: 80 } },
    { "grades.grade": "C" }
  ]
})
```
> ✅ `$nor` returns documents that match **neither** condition.

---

### Q18. Find restaurants in Bronx whose name starts with 'M' or ends with 'a'

```js
db.restu.find({
  "borough": "Bronx",
  "name": /^M|a$/
})
```

---

### Q19. Find restaurants where ALL scores are above 10

```js
db.restu.find({ "grades.score": { $not: { $lte: 10 } } })
```
> ✅ Trick — if NO score is ≤ 10, then ALL scores must be > 10.  
> Negating the opposite condition is the only way to express "all elements" in MongoDB without aggregation.

---

### Q20. Find restaurants where the latest grade (1st element) is "A" and the oldest grade (last element) is "C"

```js
db.restu.aggregate([
  { $match: { "grades.0.grade": "A" } },
  {
    $project: {
      name: 1,
      firstGrade: { $first: "$grades" },
      lastGrade: { $last: "$grades" }
    }
  },
  { $match: { "lastGrade.grade": "C" } }
])
```
> ✅ `grades.0.grade` directly accesses the first element.  
> ✅ `$first` and `$last` in `$project` get the first and last array elements.  
> ✅ MongoDB doesn't support negative indexing like `grades.-1` — this aggregation approach is the correct way.

---

### Q21. Find restaurants where a score appears more than once in grades (very challenging — skip for exam)

```js
// Beyond normal exam scope — no standard operator handles this directly
// Would require $group + $filter + JavaScript — not expected in class exams
```

---

### Q22. Find restaurant names and their total number of grades using aggregation

```js
db.restu.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      "Total Grades": { $size: "$grades" }
    }
  }
])
```
> ✅ Use `$project` — reshaping per document, not aggregating across documents.  
> ✅ `"$grades"` with `$` references the field value inside aggregation.

---

### Q23. Find the average score per borough using aggregation

```js
db.restu.aggregate([
  { $unwind: "$grades" },
  { $group: { _id: "$borough", "Avg Score": { $avg: "$grades.score" } } }
])
```
> ✅ `$unwind` first — without it `$avg` on an array field gives unreliable results.

---

### Q24. Find the top 5 cuisines by number of restaurants

```js
db.restu.aggregate([
  { $group: { _id: "$cuisine", "Total": { $sum: 1 } } },
  { $sort: { "Total": -1 } },
  { $limit: 5 }
])
```
