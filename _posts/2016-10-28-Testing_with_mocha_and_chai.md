---
layout: post
title: Testing with Mocha and Chai
---

In the previous two articles we built an API using Node.js, with MySQL as our database and Sequelize for our ORM. Now we’re going to test our API by writing unit tests using Mocha and Chai.

Unit tests are something that was glossed over when I was in school. I think we might have talked about them for something like an hour, then we moved on. I feel that this was a pretty big oversight. You hear this often when reading about unit tests, but every method you write should have a test written for it. And ideally, you should write the tests before writing that method. Of course, this means that you should be programming with a plan rather than sitting down and just starting to code. Breaking these bad programming habits will make you a stronger developer, just as writing unit tests will make your code stronger.

## Setup dependencies

First things first, let’s install our dependencies using NPM. We’re going to install mocha, chai and chai-http. Mocha is the testing framework. Chai is the assertion library, and Chai-http extends Chai to do HTTP requests, which we’ll need to test our API.

```
npm install --save-dev mocha chai chai-http
```

We’re using another feature of NPM here, which is ```--save-dev```, or only install these dependencies as developer dependencies. If you were to push this project to a server there’s really no need to include your unit tests in production. On the server we would install our dependencies using ```npm install --only=production```.

## Write tests

If we’re going to use test our API we’re going to end up changing the data in our database. To avoid this we’ll setup a second database with dummy information that we can add, delete and generally mess with without worrying about our live data. We’ll use the same bookuser MySQL user from before.

```
mysql> CREATE DATABASE bookdbtest;
Query OK, 1 row affected (0.00 sec)

mysql> USE bookdbtest;
Database changed

mysql> CREATE TABLE authors (authorid INTEGER PRIMARY KEY AUTO_INCREMENT, authorname VARCHAR(250));
Query OK, 0 rows affected (0.05 sec)

mysql> CREATE TABLE books (bookid INTEGER PRIMARY KEY AUTO_INCREMENT, title VARCHAR(250), authorid INTEGER, FOREIGN KEY (authorid) REFERENCES authors(authorid));
Query OK, 0 rows affected (0.04 sec)

mysql> GRANT ALL PRIVILEGES ON bookdb.* TO 'bookuser'@'localhost';
Query OK, 0 rows affected (0.01 sec)
```

Now let’s quickly edit our ```models.js``` file to check for a change in environment variables. This means when we’re running our tests we’ll use our testing database, and otherwise we’ll use our production version.

```
// ...

if (process.env.NODE_ENV = 'test') {
  var sequelize = new Sequelize('bookdbtest', 'bookuser', 'bookpassword', {
    host: 'localhost',
    dialect: 'mysql',
    pool: {
      max: 5,
      min: 0,
      idle: 10000
    }
  });
} else {
  var sequelize = new Sequelize('bookdb', 'bookuser', 'bookpassword', {
    host: 'localhost',
    dialect: 'mysql',
    pool: {
      max: 5,
      min: 0,
      idle: 10000
    }
  });
}
// ...
```

Alright! Now we’ll make a subfolder called tests with the following structure:

```
tests/
--tests.js
--book.js
--author.js
```

```tests.js``` looks like this:

```
// tests/tests.js
process.env.NODE_ENV = 'test';

var server = require('../index');
var chai = require('chai');
var chaiHttp = require('chai-http');
var assert = chai.assert;

chai.use(chaiHttp);

require('./author')(chai, server, assert);
require('./book')(chai, server, assert);
```

I’ve split the tests for the two API routes up into two files. author.js looks like this:

```
// tests/author.js
module.exports = function(chai, server, assert) {

  describe('/POST to /api/author', function() {
    it('Should indicate successful addition', function(done) {
      chai.request(server)
      .post('/api/author')
      .send({
        authorname: 'Arthur Conan Doyle'
      })
      .end(function(err, res) {
        assert.equal(res.status, 200, 'Should return 200 OK status');
        assert.equal(res.body.success, true, 'Should indicate success.');
  
        done();
      });
    });
  });  

  describe('/POST to /api/author but without parameters', function() {
    it('Should fail and return 403 error', function(done) {
      chai.request(server)
      .post('/api/author') 
      .end(function(err, res) {
        assert.equal(res.status, 403, 'Should return 403 error.');
        assert.equal(res.body.success, false, 'Should indicate failure.');

        done();
      });
    });
  });

  describe('/GET to /api/author', function() {
    it('Should return JSON response of all authors.', function(done) {
      chai.request(server)
      .get('/api/author')
      .end(function(err, res) {
        assert.equal(res.status, 200, 'Should return 200 OK status');
        assert.equal(res.body.success, true, 'Should indicate success.');
        assert.typeOf(res.body.authors, 'array', 'Should return an array of all authors');

        done();
      });
    });
  });

  describe('/GET to /api/author/:id', function() {
    it('Should return JSON response of all authors.', function(done) {
      chai.request(server)
      .get('/api/author')
      .end(function(err, res) {
        author = res.body.authors[0]; //find an author

        chai.request(server)
        .get('/api/author/' + author.authorid)
        .end(function(err, res) {

          assert.equal(res.status, 200, 'Should return 200 OK status.');
          assert.equal(res.body.success, true, 'Should indicate success.');
          assert.typeOf(res.body.author, 'object', 'Should return the selected author as an object.');

          done();
        });
      });
    });
  });

};
```

And book.js looks like this:

```
// tests/books.js
module.exports = function(chai, server, assert) {

  describe('/POST to /api/book', function() {
    it('Should indicate successful addition', function(done) {
      chai.request(server)
      .get('/api/author')
      .end(function(err, res) {

        author = res.body.authors[0];

        chai.request(server)
       .post('/api/book')
       .send({
         title: 'A Study in Scarlet',
         authorid: author.authorid
       })
       .end(function(err, res) {

          assert.equal(res.status, 200, 'Should return 200 OK status');
          assert.equal(res.body.success, true, 'Should indicate success.');

          done();
        });
      });
    });
  });

  describe('/GET to /api/book', function() {
    it('Should return JSON response of all books', function(done) {
      chai.request(server)
      .get('/api/book')
      .end(function(err, res) {
        assert.equal(res.status, 200, 'Should return 200 OK status');
        assert.equal(res.body.success, true, 'Should indicate success.');
        assert.typeOf(res.body.books, 'array', 'Should return an array of all books');

        done();
      });
    });
  });

  describe('/GET to /api/book', function() {
    it('Should return JSON response of all books', function(done) {
      chai.request(server)
      .get('/api/book')
      .end(function(err, res) {
        book = res.body.books[0];

        chai.request(server)
        .get('/api/book/' + book.bookid)
        .end(function(err, res) {

          assert.equal(res.status, 200, 'Should return 200 OK status');
          assert.equal(res.body.success, true, 'Should indicate success.');
          assert.typeOf(res.body.author, 'object', 'Should return the requested book as an object');

          done();
        });
      });
    });
  });
};
```

Let’s break this down a bit. ```describe``` gives a title for the test, and is useful later on when we’re running the tests to find which one failed. Then comes ```it```, which gives your expectation for the test. it has an argument ```done```, which is our callback. Note if you forget to include your callback after running your test, the test will fail after a timeout. So don’t forget to include ```done()```!

We’re able to make a request to the node server, which is our server variable, using ```chai.request(server)```. Then we specify the HTTP method we want to test. Here I’ve written tests for GET and POST, but you can just as easily use PUT and DELETE. If you need to, we can send parameters by using the ```send``` method. If you need to set the headers in your request, you’ll use ```.set({'HeaderName': 'HeaderValue'})```.

Within your test you can use [asserts](https://www.npmjs.com/package/assert) to determine that your request is working as expected. There are a few examples above of what you can do. What should the HTTP status be? What is the type of the data that’s being returned? In additional to asserting that things are equal, you can also use ```assert.notEqual``` to determine that they aren’t, or even use ```assert.deepEqual``` or ```assert.notDeepEqual``` to test for deep equality. If you’re not a fan of the assert syntax, you can check out [should](https://www.npmjs.com/package/should) or [expect](https://www.npmjs.com/package/expect). They all serve the same purpose, but you might find one is a bit more intuitive for your tests.

Notice I’ve also nested a few of the HTTP calls. In some cases you need to know an ID number of an object in order to test the route properly, so you can make a request first to get that information. The catch here is the tests then take longer to run.

## Run the tests

The last thing we need to do before running our tests is to edit our ```package.json``` file to show it where our tests are. This is done by editing the scripts section.

```
{
"name": "bookapi",
"version": "1.0.0",
"description": "Book API for gregwood.tech post.",
"main": "index.js",
"scripts": {
"test": "mocha tests/tests.js"
},
// ...
```

Now give them a run using ```npm test```. You’ll see a ton of text fly by the screen:

```
$ npm test
bookapi@1.0.0 test /home/greg/Documents/BookDB
mocha tests/tests.js
Listening on port 3000


  /POST to /api/author
Executing (default): INSERT INTO `authors` (`authorid`,`authorname`) VALUES (NULL,'Arthur Conan Doyle');
    ✓ Should indicate successful addition (131ms)

  /POST to /api/author but without parameters
    ✓ Should fail and return 403 error

  /GET to /api/author
Executing (default): SELECT `authorid`, `authorname` FROM `authors` AS `authors`;
    ✓ Should return JSON response of all authors.

  /GET to /api/author/:id
Executing (default): SELECT `authorid`, `authorname` FROM `authors` AS `authors`;
Executing (default): SELECT `authors`.`authorid`, `authors`.`authorname`, `books`.`bookid` AS `books.bookid`, `books`.`title` AS `books.title`, `books`.`authorid` AS `books.authorid` FROM `authors` AS `authors` LEFT OUTER JOIN `books` AS `books` ON `authors`.`authorid` = `books`.`authorid` WHERE `authors`.`authorid` = '1';
    ✓ Should return JSON response of all authors.

  /POST to /api/book
Executing (default): SELECT `authorid`, `authorname` FROM `authors` AS `authors`;
Executing (default): INSERT INTO `books` (`bookid`,`title`,`authorid`) VALUES (NULL,'A Study in Scarlet',1);
    ✓ Should indicate successful addition

  /GET to /api/book
Executing (default): SELECT `bookid`, `title`, `authorid` FROM `books` AS `books`;
    ✓ Should return JSON response of all books

  /GET to /api/book
Executing (default): SELECT `bookid`, `title`, `authorid` FROM `books` AS `books`;
Executing (default): SELECT `bookid`, `title`, `authorid` FROM `books` AS `books` WHERE `books`.`bookid` = '1';
    ✓ Should return JSON response of all books


  7 passing (229ms)
```

If any of your tests fail, they’ll be listed at the bottom of the screen. Then investigate the test a bit further. Is the test asking for the correct information? If you’ve written the test correctly, then it’s time to go back to the code and edit it until the tests all pass. Unit tests become exceptionally useful as a project grows in size. If you’re going to edit an existing route, run ```npm test``` once you’re finished just to be sure that your update hasn’t broken your project. Check out the code on [my github](https://github.com/weirdvector/BookDB/tree/tests) if you’re interested in seeing the full project. Thanks for reading!