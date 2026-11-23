# MongoDB (mongosh) — DS2 test cheat sheet

Examples use `db.students`. Run commands in **mongosh** (JavaScript syntax).

## Database / collection

| Command | Meaning |
|---|---|
| `show dbs` | List databases. |
| `use ds2` | Switch/create database on first write. |
| `db.getName()` | Current database name. |
| `show collections` | List collections. |
| `db.students.drop()` | Delete collection and its documents. |
| `db.runCommand({ping: 1})` | Test connection. |

## CRUD

| Command | Meaning |
|---|---|
| `db.students.insertOne({name:"Alice", age:22})` | Insert one document. |
| `db.students.insertMany([{name:"Alice"},{name:"Bob"}])` | Insert multiple documents. |
| `db.students.find()` | All documents. |
| `db.students.findOne({name:"Alice"})` | First matching document. |
| `db.students.countDocuments({age:{$gte:20}})` | Count matches. |
| `db.students.updateOne({name:"Alice"},{$set:{age:23}})` | Update first match. |
| `db.students.updateMany({city:"Prague"},{$inc:{age:1}})` | Update all matches; increment. |
| `db.students.replaceOne({name:"Alice"},{name:"Alice",age:24})` | Replace entire document (except `_id`). |
| `db.students.deleteOne({name:"Alice"})` | Delete first match. |
| `db.students.deleteMany({age:{$lt:18}})` | Delete all matches. |

## Query operators

| Command / filter | Meaning |
|---|---|
| `{age:{$eq:22}}` or `{age:22}` | Equal. |
| `{age:{$ne:22}}` | Not equal (also matches missing field). |
| `{age:{$gt:20}}` / `$gte` | Greater than / or equal. |
| `{age:{$lt:25}}` / `$lte` | Less than / or equal. |
| `{age:{$in:[20,22,24]}}` | Any of these values. |
| `{age:{$nin:[20,22]}}` | None of these values. |
| `{age:{$gte:20,$lt:25}}` | Between 20 inclusive and 25 exclusive. |
| `{age:{$gte:20},city:"Prague"}` | Implicit AND. |
| `{$or:[{age:22},{city:"Brno"}]}` | Either condition. |
| `{$and:[{age:{$gt:20}},{age:{$lt:25}}]}` | Explicit AND. |
| `{$nor:[{city:"Prague"},{age:22}]}` | Neither condition. |
| `{email:{$exists:true}}` | Field exists (even if null). |
| `{email:{$exists:false}}` | Field missing. |
| `{email:{$type:"string"}}` | Match BSON type. |
| `{name:{$regex:"^a",$options:"i"}}` | Name starts with a, case-insensitive. |

Use: `db.students.find(FILTER)`.

## Arrays and nested fields

| Command / filter | Meaning |
|---|---|
| `{courses:"Redis"}` | Array contains `Redis` (or scalar equals it). |
| `{courses:{$in:["Redis","Mongo"]}}` | Contains at least one of these. |
| `{courses:{$all:["Redis","Mongo"]}}` | Contains **both**. |
| `{grades:{$elemMatch:{$gte:2,$lte:4}}}` | **Same array element** satisfies both. |
| `{"address.city":"Prague"}` | Nested field via dot notation. |
| `{"items.price":{$gt:100}}` | Any nested array item has price >100. |
| `{items:{$elemMatch:{type:"book",price:{$gt:100}}}}` | Same array object meets both. |
| `{courses:{$size:2}}` | Array has exactly 2 elements. |
| `{$expr:{$gt:[{$size:"$courses"},2]}}` | Computed array length >2. |

`$size` in ordinary queries only matches an exact integer; use `$expr` for comparisons. `$size` errors if the value is missing/non-array, so guard it when necessary.

## Projection, sorting, paging

| Command | Meaning |
|---|---|
| `db.students.find({age:{$gt:20}},{name:1,age:1,_id:0})` | Include name/age, hide `_id`. |
| `db.students.find({},{password:0,_id:0})` | Exclude fields. |
| `db.students.find().sort({age:1})` | Ascending (`-1` descending). |
| `db.students.find().limit(5)` | First five. |
| `db.students.find().skip(10).limit(5)` | Page starting at offset 10. |
| `db.students.find().pretty()` | Pretty-print output. |
| `db.students.distinct("city")` | Unique city values. |

Projection: normally don't mix `1` and `0` except `_id:0`.

## `$expr` — compare fields / calculate

```javascript
db.students.find({$expr:{$gt:["$score","$minimum"]}})
db.students.find({$expr:{$eq:[{$size:"$courses"},2]}})
db.students.find({$expr:{$gt:[{$add:["$points",5]},20]}})
db.students.find({$expr:{$gte:[{$multiply:["$price","$quantity"]},100]}})
db.students.find({$expr:{$lt:[{$subtract:["$end","$start"]},10]}})
db.students.find({$expr:{$and:[{$gt:["$age",20]},{$lt:["$age",30]}]}})
```

Within `$expr`, field references have `$` prefix; `$gt`, `$eq` etc. take **arrays** of expressions.

## Update operators

| Command | Meaning |
|---|---|
| `db.students.updateOne({name:"Alice"},{$set:{city:"Brno"}})` | Set/create field. |
| `db.students.updateOne({name:"Alice"},{$unset:{city:""}})` | Remove field. |
| `db.students.updateOne({name:"Alice"},{$inc:{score:2}})` | Add 2 to numeric field. |
| `db.students.updateOne({name:"Alice"},{$push:{courses:"Redis"}})` | Append array item (duplicates allowed). |
| `db.students.updateOne({name:"Alice"},{$addToSet:{courses:"Redis"}})` | Add if not present. |
| `db.students.updateOne({name:"Alice"},{$pull:{courses:"Redis"}})` | Remove matching array elements. |
| `db.students.updateOne({name:"Eve"},{$set:{age:21}},{upsert:true})` | Insert if no match. |

## Aggregation pipeline

Each stage processes results of the previous stage.

```javascript
db.students.aggregate([
  {$match:{age:{$gte:20}}},                // filter
  {$group:{_id:"$city",count:{$sum:1},avgAge:{$avg:"$age"}}},
  {$sort:{count:-1}},                      // largest first
  {$limit:5}
])
```

| Stage/operator | Meaning |
|---|---|
| `{$match:{city:"Prague"}}` | Filter documents. |
| `{$project:{name:1,_id:0}}` | Select/compute fields. |
| `{$group:{_id:"$city",n:{$sum:1}}}` | Group by city and count. |
| `{$sort:{age:-1}}` | Sort descending. |
| `{$skip:10}` / `{$limit:5}` | Pagination. |
| `{$unwind:"$courses"}` | One output document per array element. |
| `{$lookup:{from:"courses",localField:"courseId",foreignField:"_id",as:"course"}}` | Join collections. |
| `{$count:"total"}` | Count pipeline output. |
| `{$addFields:{double:{$multiply:["$score",2]}}}` | Compute additional field. |
| `{$group:{_id:null,total:{$sum:"$price"},min:{$min:"$price"},max:{$max:"$price"}}}` | Aggregate all documents. |

## Indexes

| Command | Meaning |
|---|---|
| `db.students.createIndex({email:1})` | Ascending index. |
| `db.students.createIndex({city:1,age:-1})` | Compound index. |
| `db.students.getIndexes()` | List indexes. |
| `db.students.find({email:"a@b.com"}).explain("executionStats")` | Inspect query plan. |
| `db.students.dropIndex({email:1})` | Remove index. |

## Test pitfalls

- `find(filter, projection)` — second argument is **projection**, not another filter.
- `$in` = **any**, `$all` = **every listed value**.
- `$elemMatch` ensures conditions apply to **one element**; without it they may match different elements.
- `{x:null}` matches null **or missing**; `{x:{$exists:true}}` includes null.
- `{x:{$ne:5}}` also matches missing `x`.
- Don't repeat the same JavaScript object key: `{age:{$gt:20},age:{$lt:30}}` overwrites one condition. Combine operators.
- Regex `^` start, `$` end, `i` case-insensitive; escape special characters as needed.
- Use `// comment` or `/* comment */` in mongosh.

Docs: https://www.mongodb.com/docs/mongodb-shell/ and https://www.mongodb.com/docs/manual/aggregation/
