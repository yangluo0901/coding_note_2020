# MongoDB

## Mongoose
***<u>1. reference to other documents</u>***.   

use `ref` to point to referenced collection

```
const bookSchema = new mongoose.Schema({
	bookName: String,
	authors: [{mongoose.Schema.Types.ObjectID, ref: "persion"}]        //use array
});

const personSchema = bew mongoose.Schema({
	name: String,
});

const Book = new mongoose.model("Book", bookSchema);
const Person = new mongoose.model("Person", personSchema);
```

add a new document as a ref    

```
const newPerson = new Person({
	_id: new mongoose.Types.ObjectID,
	name: "Lee Sin"
});

/************************* if create a new book and add new author ***************************/

const newBook = new Book({
	name: "Harry Potter",
	authors: [newPerson._id]
});
newBook.save()


/************************** if the book already exists *********************************/

// Method 1:
Book.findOne({_id: <bookID>}, function(err, book){
	book.authors.push(newPerson._id);
	book.save();
});

//Method 2:

Book.update({<filter>},{$push:{authors: newPerson}});
```

link two collections by reference:

```
Book.findOne({ <filter> }, function(err, book){
	// find out target author
	Persion.findOne({ <filter> }, function(err, author){
		book.authors.push(author._id);
		book.save();
	});
});
```

#### <u>2. $or / $and</u>    

```
Tag.find({$or: [{name: "Tree"}, {name: "Binary Search"}]}, function (err, tree) {
    console.log(tree);
    });
```
Use $and in the same way     



#### <u>3. find document from an __Array__ by  array element's property value</u>    

* __if the array is just an array of string or ObjectID__:         
  
```
Post.find({tags: <tag._id> }, function (err, post) {
    console.log(tree);
    });
```

* __if the element in the array is objects with multiple properties__:    
  
```
const Post = mongoose.Schema({
	title: String,
	tags:[{
		name: String,
		description: String,
		category: String
	}]
});

// only one condition
Post.find({"tags.name" : <tagName>}, function(err){
	...
})

//multiple conditions:
Post.find({"tags.name" : {$all:[ <tagName1>, <tagName2>, ... ]}}, function(err){
	...
});

```

__Note__: $all is AND, $in is OR

​     

#### <u>4. Delete from Array: by `$pull`</u>

```javascript
List.findOneAndUpdate({_id: <list_id>}, {$pull: {items: {_id: <item_id>}}}, function(){
    ...
})
```



## Troubleshooting
***1.Erro: Address already in use***		    
` sudo killall mongod`