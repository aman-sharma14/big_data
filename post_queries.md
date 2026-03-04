# MongoDB Queries — post.json

---

### 1. Find published posts that have more than 50 likes and contain the tag "ai"

```js
// Original
db.posts.find({ $and: [{"is_published": true}, {"likes": {$gt: 50}}, {"tags": "ai"}] })

// ✅ Optimized (implicit AND — cleaner)
db.posts.find({ "is_published": true, "likes": {$gt: 50}, "tags": "ai" })
```

---

### 2. Display posts where the author is not "Rahul" and the post has between 30 and 70 likes

```js
// ✅ Correct
db.posts.find({ "author": {$ne: "Rahul"}, "likes": {$gt: 30, $lt: 70} })
```

---

### 3. Find posts that contain both tags "ai" and "machine learning"

```js
// ✅ Correct — $all checks both values exist in array
db.posts.find({ "tags": {$all: ["machine learning", "ai"]} })
```

---

### 4. Find posts where at least one comment has more than 2 likes

```js
// ✅ Correct — dot notation checks inside comments array
db.posts.find({ "comments.likes": {$gt: 2} })
```

---

### 5. Find posts that are not published but still have at least one comment

```js
// Option 1 — check if index 0 exists (cleanest)
db.posts.find({
  "comments.0": {$exists: true},
  "is_published": false
})

// Option 2 — $not with $size
db.posts.find({
  "comments": {$not: {$size: 0}},
  "is_published": false
})

// Option 3 — cleanest of all
db.posts.find({
  "comments": {$ne: []},
  "is_published": false
})
```

---

### 6. Display only the title, author, and likes of posts, excluding the _id field

```js
// ✅ Correct
db.posts.find({}, {"title": 1, "author": 1, "likes": 1, "_id": 0})
```

---

### 7. Display the top 3 most liked posts

```js
// ✅ Correct
db.posts.find().sort({"likes": -1}).limit(3)
```

---

### 8. Skip the first 5 posts and show the next 5 sorted by likes (descending)

```js
// ✅ Correct
db.posts.find().sort({"likes": -1}).skip(5).limit(5)
```

---

### 9. Find posts where the tags array includes "mongodb" but does not include "database"

```js
// Original
db.posts.find({ $and: [{"tags": "mongodb"}, {"tags": {$ne: "database"}}] })

// ✅ Optimized — $all + $nin in one condition
db.posts.find({ "tags": {$all: ["mongodb"], $nin: ["database"]} })
```

---

### 10. Increase likes by 10 for all posts having the tag "python"

```js
// ✅ Correct — $inc adds to existing value
db.posts.updateMany({"tags": "python"}, {$inc: {"likes": 10}})
```

---

### 11. Add a new tag "featured" to posts where likes are greater than 60, without duplicating the tag

```js
// ✅ Correct — $addToSet won't add if value already exists (unlike $push)
db.posts.updateMany({"likes": {$gt: 60}}, {$addToSet: {"tags": "featured"}})
```

---

### 12. Remove the tag "web" from all posts where it exists

```js
// ✅ Correct — $pull removes matching value from array
db.posts.updateMany({}, {$pull: {"tags": "web"}})
```

---

### 13. Add a new comment to the post with post_id = 5

```js
// ✅ Correct — $push appends to array
db.posts.updateOne(
  {"post_id": 5},
  {$push: {"comments": {"user": "John", "message": "awesome", "likes": 2}}}
)
```

---

### 14. Find posts where the comments array has exactly 0 elements

```js
// ✅ Correct — $size matches exact array length
db.posts.find({"comments": {$size: 0}})
```

---

### 15. Find posts where the comments field exists and has more than 1 element

```js
// ✅ Correct — if index 1 exists, array has at least 2 elements
db.posts.find({"comments.1": {$exists: true}})
```

---

### 16. Find the average number of likes for all published posts

```js
// ✅ Correct — $match first to filter, then $group with $avg
db.posts.aggregate([
  {$match: {"is_published": true}},
  {$group: {_id: null, "Avg Likes": {$avg: "$likes"}}}
])
```

---

### 17. Find the total number of posts written by each author

```js
// ✅ Correct — $sum: 1 counts documents per group
db.posts.aggregate([
  {$group: {_id: "$author", "Total Posts": {$sum: 1}}}
])
```

---

### 18. Find the maximum number of likes among all posts

```js
// ✅ Correct
db.posts.aggregate([
  {$group: {_id: null, "Max Likes": {$max: "$likes"}}}
])
```

---

### 19. List authors whose total likes across all their posts exceed 100

```js
// ✅ Correct — $group first, then $match filters on grouped result
db.posts.aggregate([
  {$group: {_id: "$author", "Total Likes": {$sum: "$likes"}}},
  {$match: {"Total Likes": {$gt: 100}}}
])
```

---

### 20. Display each post's title and number of comments

```js
// ✅ Option 1 — $project (PREFERRED for reshaping single document)
db.posts.aggregate([
  {$project: {
    _id: 0,
    title: 1,
    "Comments count": {$size: "$comments"}
  }}
])

// Option 2 — $group (title shows as _id — less clean)
db.posts.aggregate([
  {$group: {
    _id: "$title",
    "Comments count": {$sum: {$size: "$comments"}}
  }}
])
```
> Use **Option 1** — `$project` is for reshaping a single document. `$group` is for aggregating across multiple documents and shows title under `_id` which is messy.
