# MongoDB Practice Questions — posts.json & restaurants.json

---

## 🗂️ posts.json — 10 Questions

---

### Q1. Add a new field "status" with value "active" to all published posts

```js
db.posts.updateMany(
  { "is_published": true },
  { $set: { "status": "active" } }
)
```

---

### Q2. Set the likes of the post with post_id 2 to 100

```js
db.posts.updateMany(
  { "post_id": 2 },
  { $set: { "likes": 100 } }
)
```
> ⚠️ `100` not `"100"` — likes is a number, not a string.

---

### Q3. Remove the "is_published" field from all posts written by "Rahul"

```js
db.posts.updateMany(
  { "author": "Rahul" },
  { $unset: { "is_published": "" } }
)
```
> ✅ The value `""` inside `$unset` doesn't matter — the field gets removed regardless.

---

### Q4. Remove the "tags" field entirely from posts that have 0 likes

```js
db.posts.updateMany(
  { "likes": 0 },
  { $unset: { "tags": "" } }
)
```
> ⚠️ Condition is `"likes": 0` — not `{ $size: 0 }` on tags. The question asks about posts with 0 likes.

---

### Q5. Increase the likes by 5 for all posts that have the tag "python"

```js
// Simple way — use $inc
db.posts.updateMany(
  { "tags": "python" },
  { $inc: { "likes": 5 } }
)

// Pipeline way — use $add
db.posts.updateMany(
  { "tags": "python" },
  [{ $set: { "likes": { $add: ["$likes", 5] } } }]
)
```
> ✅ Both are correct. Use `$inc` for simple increments — it's cleaner.  
> ✅ `$add` requires pipeline syntax `[{ }]` and `"$likes"` with `$`.

---

### Q6. Find posts where there is a comment by "Neha" with more than 3 likes

```js
db.posts.find({
  "comments": {
    $elemMatch: { "user": "Neha", "likes": { $gt: 3 } }
  }
})
```
> ✅ `$elemMatch` ensures both conditions match the **same** comment object.

---

### Q7. Find posts where a comment has exactly 0 likes and was made by "Amit"

```js
db.posts.find({
  "comments": {
    $elemMatch: { "user": "Amit", "likes": 0 }
  }
})
```

---

### Q8. Find the total number of likes for each author across all their posts

```js
db.posts.aggregate([
  { $group: { _id: "$author", "Total Likes": { $sum: "$likes" } } }
])
```
> ✅ `$sum: "$likes"` adds up the likes field. `$sum: 1` would just count documents.

---

### Q9. Find the author who has written the most number of posts

```js
db.posts.aggregate([
  { $group: { _id: "$author", "No Of Posts": { $sum: 1 } } },
  { $sort: { "No Of Posts": -1 } },
  { $limit: 1 }
])
```

---

### Q10. Find authors whose average likes are greater than 40, sorted by average likes (highest first)

```js
db.posts.aggregate([
  { $group: { _id: "$author", "Avg Likes": { $avg: "$likes" } } },
  { $match: { "Avg Likes": { $gt: 40 } } },
  { $sort: { "Avg Likes": -1 } }
])
```
> ✅ HAVING pattern — `$match` after `$group` filters on the computed result.  
> ⚠️ Don't forget the `$match` — sorting alone doesn't filter out the authors below 40.

---
---

## 🍽️ restaurants.json — 10 Questions

---

### Q11. Add a new field "status" with value "open" to all restaurants in "Manhattan"

```js
db.restu.updateMany(
  { "borough": "Manhattan" },
  { $set: { "status": "open" } }
)
```

---

### Q12. Set the cuisine of the restaurant with restaurant_id "30075445" to "Fusion"

```js
db.restu.updateOne(
  { "restaurant_id": "30075445" },
  { $set: { "cuisine": "Fusion" } }
)
```
> ✅ `updateOne` is correct since `restaurant_id` is unique.

---

### Q13. Remove the "address" field from all restaurants in "Staten Island"

```js
db.restu.updateMany(
  { "borough": "Staten Island" },
  { $unset: { "address": "" } }
)
```

---

### Q14. Remove the "grades" field from all restaurants whose cuisine is "Missing"

```js
db.restu.updateMany(
  { "cuisine": "Missing" },
  { $unset: { "grades": "" } }
)
```
> ⚠️ `"Missing"` is the **literal string value** of cuisine — not a missing/absent field.  
> If the field were actually absent you'd use `{ $exists: false }`, but the question says cuisine **is** "Missing".

---

### Q15. Add a new field "score_bonus" calculated as the first grade's score + 10

```js
db.restu.updateMany(
  {},
  [{
    $set: {
      "score_bonus": {
        $add: [{ $arrayElemAt: ["$grades.score", 0] }, 10]
      }
    }
  }]
)
```
> ✅ `$arrayElemAt: ["$grades.score", 0]` accesses the first element of the scores array.  
> ✅ Pipeline syntax `[{ }]` is required when using aggregation expressions inside updates.

---

### Q16. Find restaurants where a single grade entry has grade "A" and score less than 10

```js
db.restu.find({
  "grades": {
    $elemMatch: { "grade": "A", "score": { $lt: 10 } }
  }
})
```

---

### Q17. Find restaurants where any grade entry has grade "B" and score greater than 20

```js
db.restu.find({
  "grades": {
    $elemMatch: { "grade": "B", "score": { $gt: 20 } }
  }
})
```
> ⚠️ Must use `$elemMatch` on the `"grades"` array — `"grade"` and `"score"` are nested inside each grade object, not top-level fields.

---

### Q18. Find the total number of restaurants in each borough, sorted by count descending

```js
db.restu.aggregate([
  { $group: { _id: "$borough", "Total": { $sum: 1 } } },
  { $sort: { "Total": -1 } }
])
```

---

### Q19. Find cuisines where the total number of restaurants is greater than 100

```js
db.restu.aggregate([
  { $group: { _id: "$cuisine", "Total": { $sum: 1 } } },
  { $match: { "Total": { $gt: 100 } } }
])
```
> ✅ HAVING pattern — `$match` after `$group`.

---

### Q20. Find the borough that has the highest number of restaurants with "Italian" cuisine

```js
db.restu.aggregate([
  { $match: { "cuisine": /Italian/i } },
  { $group: { _id: "$borough", "Total": { $sum: 1 } } },
  { $sort: { "Total": -1 } },
  { $limit: 1 }
])
```
> ⚠️ `$match` filters by `"cuisine"` not `"borough"` — you want restaurants where cuisine is Italian, then group by borough.

---
---

## 🔁 Bonus — $match + $group Questions

---

### B1. Find the total number of published posts per author (only include published posts)

```js
db.posts.aggregate([
  { $match: { "is_published": true } },
  { $group: { _id: "$author", "Published Posts": { $sum: 1 } } }
])
```
> ✅ `$match` before `$group` = WHERE — filters documents before grouping.

---

### B2. Find the cuisine types available in "Brooklyn" and how many restaurants offer each

```js
db.restu.aggregate([
  { $match: { "borough": "Brooklyn" } },
  { $group: { _id: "$cuisine", "Total": { $sum: 1 } } },
  { $sort: { "Total": -1 } }
])
```

---

### B3. Among posts with more than 30 likes, find the total likes per tag

```js
db.posts.aggregate([
  { $match: { "likes": { $gt: 30 } } },
  { $unwind: "$tags" },
  { $group: { _id: "$tags", "Total Likes": { $sum: "$likes" } } },
  { $sort: { "Total Likes": -1 } }
])
```
> ✅ `$unwind` is needed to flatten the tags array before grouping by individual tag.

---

### B4. Find boroughs where the average grade score is greater than 10

```js
db.restu.aggregate([
  { $unwind: "$grades" },
  { $group: { _id: "$borough", "Avg Score": { $avg: "$grades.score" } } },
  { $match: { "Avg Score": { $gt: 10 } } },
  { $sort: { "Avg Score": -1 } }
])
```
> ✅ `$match` after `$group` = HAVING — filters on the computed average.

---

### B5. Find authors who have written at least 1 unpublished post, and show their total post count

```js
db.posts.aggregate([
  { $match: { "is_published": false } },
  { $group: { _id: "$author", "Unpublished Posts": { $sum: 1 } } },
  { $sort: { "Unpublished Posts": -1 } }
])
```

---

### B6. Find the top 3 boroughs by number of restaurants that have at least one "A" grade

```js
db.restu.aggregate([
  { $match: { "grades.grade": "A" } },
  { $group: { _id: "$borough", "Total": { $sum: 1 } } },
  { $sort: { "Total": -1 } },
  { $limit: 3 }
])
```
> ✅ `$match` first filters only restaurants that have an "A" somewhere in their grades array, then groups by borough.

---

### Quick Rule Reminder

| Pattern | When to use |
|---------|------------|
| `$match` → `$group` | Filter documents FIRST, then group (like WHERE) |
| `$group` → `$match` | Group first, then filter on computed result (like HAVING) |
| `$match` → `$group` → `$match` | Filter docs → group → filter groups (WHERE + HAVING) |
