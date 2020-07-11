# MongoDB

## Mongoose
***1. reference to other documents***.   

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