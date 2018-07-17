---
layout: post
title: Join statements in Sequelize
---

Let’s pick up where the last article left off and add in some more functionality that’s provided by Sequelize. The great thing about using a relational databases is the *relations*, or how your data is connected to other pieces of your data. Using traditional SQL we’re able to combine tables together using JOIN statements, which connect a table together with another table using the keys from tables. It might look something like this:

```
MYSQL> SELECT * FROM books INNER JOIN authors ON books.authorid = authors.authorid;
+--------+----------------------+----------+----------+------------------+
| bookid | title                | authorid | authorid | authorname       |
+--------+----------------------+----------+----------+------------------+
|      2 | Cryptonomicon        |        2 |        2 | Neal Stephenson  |
|      3 | Girlfriend in a Coma |        3 |        3 | Douglas Coupland |
|      4 | Neuromancer          |        4 |        4 | William Gibson   |
|      5 | Pattern Recognition  |        4 |        4 | William Gibson   |
+--------+----------------------+----------+----------+------------------+
```

If you remember from last time, our books table just has an ID field, the book title and a reference to the ID of the author. The author table has the author’s name and ID. When we join the tables together, we get one larger table that has repeated pieces of information: William Gibson shows up in our query table twice, because he’s got two books in our book table. We also have the ```authorid``` column repeated twice, because both tables contain it.

With Sequelize it works a little bit differently. If you look back at our ```models.js``` file we’ve already set up our associations:

```
Author.hasMany(Book, {
  foreignKey: 'authorid'
});
```

It’s pretty self-explanatory, one author can have many books, or a many-to-one relation. We could also set up a one-to-one associations if we needed, but here it wouldn’t make sense. Sequelize provides two options for setting up either of these associations: ```belongsTo``` and ```belongsToMany``` or ```hasOne``` and ```hasMany```. The difference between the two is that  ```hasOne``` puts the association in the target model. For the above example, Book is our target. ```belongsTo``` would put the association in the source model (Author for our example).

Now let’s edit our ```routes.js``` file and edit our GET author route, and this will start to make more sense.

```
app.get('/api/author/:id', function(req, res) {

  models.Author.find({
    where: {
      authorid: req.params.id
    },
    include: [{
      model: models.Book
    }]
  })
  .then(function(author) {
    res.json({
      success: true,
      author: author
    });
  })
  .catch(function(err) {
    res.status(500).json({
      success: false,
      message: err.message
    });
  });
});
```

All we’ve changed is added the include statement to the arguments of our find function. We don’t need to say anything about how the two tables are connected as that’s already been established in our ```models.js``` file. If you try and include a table without a defined reference you’ll get an error. Notice too that our include statement takes an array as argument, so if you had multiple associations to the first table you would add another object to the array. If you wanted to join a third table to the second table, you could include that in the reference object like so:

```
// ...
include: [{
  model: models.Book,
  include: [{
    model: models.Publisher
  }]
}]
// ...
```

This sort of nesting can go as deep as your associations allow, and can include their own where clauses and order-by statements.

The other difference you’ll notice is that instead of getting one table with all your results is now you get an object that contains an array (or object depending on your association) of all the referenced objects. Here’s an example of what I mean, by making a call to our ```/api/author/:id``` route:

```
{
  "success": true
  "author": {
    "authorid": 4
    "authorname": "William Gibson"
    "books": [
      {
        "bookid": 4
        "title": "Neuromancer"
        "authorid": 4
      },{
        "bookid": 5
        "title": "Pattern Recognition"
        "authorid": 4
      }
    ]
  }
}
```

This took a little bit for me to wrap my head around. I was expecting Sequelize to work in the same manner as a SQL Join, and return one big table of all the information. Instead, Sequelize is an ORM (Object Relational Mapper), so it returns only objects as defined in our ```models.js```. So we are returned an array titled the same as the table in our database, that contains an object of each match.

Try it out! Sequelize is a pretty powerful ORM for Javascript, but it can take a bit of getting used to if you are coming from plain MySQL. Check out the code on [my github](https://github.com/weirdvector/BookDB/tree/joins).

