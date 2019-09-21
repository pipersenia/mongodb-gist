### Mongo DB Commands

Connection via mongo shell to sandbox cluster: <br>
`username: m001`
`password: m001-password`

```
mongo "mongodb+srv://m001-524sk.mongodb.net/test" --username m001
```

Connecting to mongodb m001 cluster: <br>
`mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/test?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl --username m001-student --password m001-mongodb-basics`

Show databases: <br>
```
MongoDB Enterprise m001-shard-0:PRIMARY> show dbs
admin  0.000GB
local  5.275GB
video  0.001GB
```

Show collections: <br>
```
MongoDB Enterprise m001-shard-0:PRIMARY> use video
switched to db video
MongoDB Enterprise m001-shard-0:PRIMARY> show collections;
movieDetails
```

Inserting document into a collection: `insertOne()`
```
MongoDB Enterprise m001-shard-0:PRIMARY> db.moviesScratch.insertOne({title: "Singh is King", year: 1982, imdb: "tt0019284"});
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5d8587772c3c229677ddcac9")
}
```

`_id` is an attribute that each document in mongodb must have. It can either be specified or let it be 
created by mongodb.  

Bulk insert into collection: `insertMany()`
```
MongoDB Enterprise m001-shard-0:PRIMARY> db.moviesScratch.insertMany([{title: "Singh is King", year: 1982, imdb: "tt0019284"}, {title: "Singh is King", year: 1983, imdb: "tt0019284"}, {title: "Singh is King", year: 1984, imdb: "tt0019284"}]);
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5d8588902c3c229677ddcacb"),
		ObjectId("5d8588902c3c229677ddcacc"),
		ObjectId("5d8588902c3c229677ddcacd")
	]
}
```

Check if the documents were inserted: `find()`

```
MongoDB Enterprise m001-shard-0:PRIMARY> db.moviesScratch.find()
{ "_id" : ObjectId("5d8587772c3c229677ddcac9"), "title" : "Singh is King", "year" : 1982, "imdb" : "tt0019284" }
{ "_id" : ObjectId("5d8588742c3c229677ddcaca"), "title" : "Singh is King", "year" : 1982, "imdb" : "tt0019284" }
{ "_id" : ObjectId("5d8588902c3c229677ddcacb"), "title" : "Singh is King", "year" : 1982, "imdb" : "tt0019284" }
{ "_id" : ObjectId("5d8588902c3c229677ddcacc"), "title" : "Singh is King", "year" : 1983, "imdb" : "tt0019284" }
{ "_id" : ObjectId("5d8588902c3c229677ddcacd"), "title" : "Singh is King", "year" : 1984, "imdb" : "tt0019284" }
```

- Ordered inserts vs Unordered insert
- Records will keep being inserted if "ordered" : False.
- Bulk insert will fail fast when it encounters error when "ordered": True

- dot notation for nested documents
- when using mongoshell, those filters using dot notation need to use `""`

Count operation: `count`
```
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({"awards.wins": 2}).count()
100
```

Filtering on array fields: <br>
`{ "cast.0": "Jeff Bridges" }`
Index in the array must be provided.

In case the index is omitted, then only documents with exact match are selected. <br>
e.g. `{writers: ["Ethan Coen", "Joel Coen"]}` will only select documents with writers field 
containing those values in that order.

```
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({writers: ["Ethan Coen", "Joel Coen"]}).count()
1

MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({writers: ["Joel Coen", "Ethan Coen"]}).count()
0
``` 
- Cursors: Iterating over the result set <br>
`find()` method returns a cursor which can then be used to iterate over the documents.
- Projections: Limiting fields in the result set <br>
Works like a select clause in a sql query. By default all columns are returned. Behavior can be 
changed by adding another parameter `{}` clause in the `find()`. Set the value of the field = `1`
for including in `projection` and `0` for being excluded from the projection. 
```
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({writers: "Ethan Coen"}, {title: 1}).pretty()
{
	"_id" : ObjectId("5d7dad8a8efc253c861f5b51"),
	"title" : "The Big Lebowski"
}
{
	"_id" : ObjectId("5d7dad8a8efc253c861f5fb6"),
	"title" : "No Country for Old Men"
}
{
	"_id" : ObjectId("5d7dad8a8efc253c861f62cf"),
	"title" : "Paris, je t'aime"
}
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({writers: "Ethan Coen"}, {title: 1, _id:0}).pretty()
{ "title" : "The Big Lebowski" }
{ "title" : "No Country for Old Men" }
{ "title" : "Paris, je t'aime" }
```
 
 Updating: Updating documents <br>
 `updateOne()` 
 - Uses field operators. e.g. `$set`
```
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.updateOne({writers: "Ethan Coen"}, {$set: {"AverageRating": "Excellent!"}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
MongoDB Enterprise m001-shard-0:PRIMARY> db.movieDetails.find({writers: "Ethan Coen"}, {title: 1, "AverageRating":1}).pretty()
{
	"_id" : ObjectId("5d7dad8a8efc253c861f5b51"),
	"title" : "The Big Lebowski",
	"AverageRating" : "Excellent!"
}
{
	"_id" : ObjectId("5d7dad8a8efc253c861f5fb6"),
	"title" : "No Country for Old Men"
}
{
	"_id" : ObjectId("5d7dad8a8efc253c861f62cf"),
	"title" : "Paris, je t'aime"
```
In the above example, only a single record was updated. 

[Update Operators](https://docs.mongodb.com/manual/reference/operator/update/)

`updateMany()`