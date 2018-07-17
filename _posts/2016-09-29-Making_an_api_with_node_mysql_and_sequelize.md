---
layout: post
title: Making an API with Node.js, MySQL and Sequelize
---

In this article we’ll use Node.js to set up a series of API routes to perform CRUD operations on a MySQL database. Most articles you’ll find on Node.js focus on using the MEAN Stack (MongoDB, Express, Angular and Node), but I’m still more used to using MySQL for my databases. Granted, I haven’t spent the time with MongoDB yet to be comfortable with a document database. But if you want to learn Node.js without throwing away your relational database skills this guide should help.

You’ll need ```node```, ```npm``` and ```mysql``` installed.

## Start your project

We’re going to make a database for books. Make a folder, then run npm init inside it. Fill out the prompts to create your package.json file. We’ll need to install a few NPM packages here, using the ```--save``` flag to add them to our ```package.json```.

```
npm install --save express mysql sequelize
```

Now let’s quickly write up our index.js file. This will be our entry point into our project.

```
// index.js

var express = require('express');
var port = 3000;
var app = express();

var bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended:true
}));

//setup routes
require('./routes')(app);

app.listen(port);
console.log('Listening on port ' + port)

module.exports = app;
```

It’s only a few lines of code, but that’s all we need to get our API running. app is what we’re calling our Express web app, and we’re listening on port 3000. The body parser is included so that we’re able to read the POST parameters of the requests that are sent into our API. We’re going to keep our routes in a separate file that we’ll write a bit later. For now let’s move onto the database.

## Make your database

We need to make a database and some table to store information about our books. I’m going to make a table for Authors and another for Books, and we’ll link them to show off some of the functionality of Sequelize. Personally I always create a user in MySQL specifically for the project. I do this because it’s not uncommon to have the credentials stored in plaintext in your configuration files (like we’ll do later) so it’s generally a good idea to limit the damage that user can do.

```
$ mysql -u root -p
Password:

mysql> CREATE DATABASE bookdb;
Query OK, 1 row affected (0.00 sec)

mysql> CREATE TABLE authors (authorid INTEGER PRIMARY KEY AUTO_INCREMENT, authorname VARCHAR(250));
Query OK, 0 rows affected (0.03 sec)

CREATE TABLE books (bookid INTEGER PRIMARY KEY AUTO_INCREMENT, title VARCHAR(250), authorid INTEGER, FOREIGN KEY (authorid) REFERENCES authors(authorid));

mysql> CREATE USER 'bookuser'@'localhost' IDENTIFIED BY 'bookpassword';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON bookdb.* TO 'bookuser'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

## Sequelize

Let’s connect up our database with our Node API using Sequelize. Sequelize is an object relational mapper (ORM) for Node.js that works with MySQL, PostgreSQL, MariaDB, SQLite, and some other database variants. ORMs are really useful because they translate rows from your database tables into objects that your program can use, saving you from writing all of the code yourself. It also translates these objects back into database rows whenever you need to insert or update your table. Let’s write up our models.js file to define what these objects should look like.

```
// models.js

var Sequelize = require('sequelize');

var sequelize = new Sequelize('bookdb', 'bookuser', 'bookpassword', {
  host: 'localhost',
  dialect: 'mysql',

  pool: {
    max: 5,
    min: 0,
    idle: 10000
  }
});

var Author = sequelize.define('authors', {
  authorid: {
    type: Sequelize.INTEGER,
    primaryKey: true
  },
  authorname: {
    type: Sequelize.STRING,
    defaultValue: 'Ernest Hemingway'
  }
}, {
  timestamps: false
});

var Book = sequelize.define('books', {
  bookid: {
    type: Sequelize.INTEGER,
    primaryKey: true
  },
  title: {
    type: Sequelize.STRING,
    defaultValue: 'A Farewell to Arms'
  },
  authorid: {
    model: Author,
    key: 'authorid',
    type: Sequelize.INTEGER,
    primaryKey: true
  }
});

Book.belongsTo(Author, {
  foreignKey: 'authorid'
});
```

First we establish our connection to the database by instantiating an object of Sequelize. This takes our database name, MySQL username and password. Then we let Sequelize know we’re using the MySQL dialect, because it can be used for other dialects as well. Then we define our two models, giving the table name as well as a field for each column. In our Book model’s authorid field, we let Sequelize know that authorid is actually the foreign key we’ll be referencing authors by. One of my favourite things about venturing into Node.js-land is how legible everything is. Reading through this definition is almost as simple as reading in English. Take the associations at the end: book belongs to author. We’ve set up a one to one association, just like that. Easy!

## Routes

Now comes the part of the exercise where we write a lot of code. Don’t panic though, like everything else in Node this is pretty straightforward. We just need to write routes for our CRUD operations using the HTTP get, post, put and delete methods for both our author table and book table. Let’s start setting up routes.js:

```
// routes.js

var models = require('./models');

module.exports = function(app) {

  app.get('/', function(req, res) {
    res.json({
      success: true,
      message: 'Welcome to our API!'
    });
  });

  //more routes go here
};
```

First things first: we require our models file that we just finished writing. All of our database operations can be accessed by using the models we set up. Then we’ve got our first route, which is really just a welcome screen. Let’s break that down a bit: app is our express web app that we created in ```index.js```. The ```get()``` method is the HTTP method that we’re using. For our API, get will read from the database, post will create entries for the database, put will update entries, and delete will delete. Each of these HTTP methods takes a function that has a request and response object. So we read information from the request, perform any logic, then write to our response. Since this is an API, all we’re going to return is JSON data. This is done using the ```res.json()``` method, and then just filling in our javascript object.

The author GET routes are similar:

```
app.get('/api/author', function(req, res) {

  models.Author.findAll()
  .then(function(authors) {
    res.json({
      success: true,
      message: 'Here\'s the list of our authors!',
      authors: authors
    })
  })
  .catch(function(err) {
    res.status(500).json({
      success: false,
      message: err.message
    });
  });
});
```

Here we’re using our Author model to find all authors. Once we’ve got the results, we call a function that has the results as an argument, then return that to our user in a JSON response. If we hit an error, we’ll jump to our catch block, which is similar except that we have an error instead of results. For now, we’ll send the error message in our JSON response back to the user. Ideally you’ll have any errors worked about before you post this code, so a user would never see any error message, but this is just an example. Also notice you can change the HTTP response code by calling the status() method.

Here’s the GET route for returning a single author by id:

```
app.get('/api/author/:id', function(req, res) {

  models.Author.find({
    where: {
      authorid: req.params.id
    }
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

Notice that I used find() instead of findAll() and that it takes an object as an argument that represents our SQL statement’s where clause. Also we use req.params to read the parameters that I defined up in the route.

Here’s the author’s POST route:

```
app.post('/api/author', function(req, res) {

  if (req.body.authorname) {

    models.Author.build({
      authorname: req.body.authorname
    })
    .save()
    .then(function(author) {
      res.json({
        success: true,
        message: 'Author ' + author.authorname + ' added to the database.'
      });
    })
    .catch(function(err) {
      res.status(500).json({
        success: false,
        message: err.message
      });
    });

  } else {
    res.status(403).json({
      success: false,
      message: 'Missing author\'s name.'
    });
  }
});
```

Here’s where we needed the body-parser module that we required back in index.js, so we can check to make sure that we’re not including a blank authorname. Next we call another method of our Author model, build(). In the arguments here we make an object that includes all of the information we know about the author. Then we can call .save() to save it to the database, then we either display the successful JSON to the user, or catch the error and show that instead.

Almost there! Here’s our PUT route:

```
app.put('/api/author/:id', function(req, res) {

  if (req.body.authorname) {

    models.Author.update({
      authorname: req.body.authorname
    }, {
      where: {
        authorid: req.params.id
      }
    })
    .then(function(author) {
      res.json({
        success: true,
        message: 'Author updated.'
      });
    })
    .catch(function(err) {
      res.status(500).json({
        success: false,
        message: err.message
      });
    });

  } else {
    res.status(403).json({
      success: false,
      message: 'Missing author\'s name.'
    });
  }

});
```

Now we’re using the update() method of our model to change the authorname, where our parameter ID matches up with an authorid.

And lastly our DELETE route:

```
app.delete('/api/author/:id', function(req, res) {

  models.Author.destroy({
    where: {
      authorid: req.params.id
    }
  })
  .then(function(author) {
    res.json({
      success: true,
      message: 'Author deleted! Sayonara!'
    });
  })
  .catch(function(err) {
    res.json({
      success: false,
      message: err.message
    })
  });
});
```

The only thing that’s a bit tricky here is Sequelize opted to name their delete method destroy() because delete is a reserved word in Javascript (although Express is fine having methods named delete…). It takes an object-style where clause just like we’ve seen above.

Now for our book routes:

```
app.get('/api/book', function(req, res) {
  models.Book.findAll()
  .then(function(books) {
    res.json({
      success: true,
      message: 'Here\'s the list of our books!',
      books: books
    })
  })
  .catch(function(err) {
    res.status(500).json({
      success: false,
      message: err.message
    });
  });
});

app.get('/api/book/:id', function(req, res) {
  models.Book.find({
    where: {
      bookid: req.params.id
    }
  })
  .then(function(book) {
    res.json({
      success: true,
      author: book
    });
  })
  .catch(function(err) {
    res.status(500).json({
      success: false,
      message: err.message
    });
  });
});

app.post('/api/book', function(req, res) {
  if (req.body.title && req.body.authorid) {

    models.Book.build({
      title: req.body.title,
      authorid: req.body.authorid
    })
    .save()
    .then(function(book) {
      res.json({
        success: true,
        message: 'Book ' + book.title + ' added to the database.'
      });
    })
    .catch(function(err) {
      res.status(500).json({
        success: false,
        message: err.message
      });
    });

  } else {
    res.status(403).json({
      success: false,
      message: 'Missing parameters for adding books.'
    });
  }
});

app.put('/api/book/:id', function(req, res) {

  if (req.body.authorid && req.body.title) {

    models.Book.update({
      authorid: req.body.authorid,
      title: req.body.title
    }, {
      where: {
        bookid: req.params.id
      }
    })
    .then(function(book) {
      res.json({
        success: true,
        message: 'Book updated.'
      });
    })
    .catch(function(err) {
      res.status(500).json({
        success: false,
        message: err.message
      });
    });

  } else {
    res.status(403).json({
      success: false,
      message: 'Missing author\'s name.'
    });
  }
});

app.delete('/api/book/:id', function(req, res) {

  models.Book.destroy({
    where: {
      bookid: req.params.id
    }
  })
  .then(function(book) {
    res.json({
      success: true,
      message: 'Book deleted! Arrivederci!'
    });
  })
  .catch(function(err) {
    res.json({
      success: false,
      message: err.message
    })
  });
});
```

And we’re done our API! Fire up your app by running ```node index.js``` and going to ```http://localhost:3000``` to see your handiwork. Keep in mind this is just our API, so all we’ll see here is our JSON responses. But this is a good start to building the backend of a web app. From here we can go ahead and make a nice looking Angular or Ember app, and fill it with data by using AJAX calls to the API routes we just wrote. You can test this API out with browser extensions like the Advanced REST Client or Postman in order to make all of the different types of requests that the API can handle. Testing Node.js applications is a topic I hope to cover in a later post.

I hope you enjoyed building this app and dipping your toes into the world of Node without needing to give up your relational database skills. As always, the code is on [my github](https://github.com/weirdvector/BookDB).