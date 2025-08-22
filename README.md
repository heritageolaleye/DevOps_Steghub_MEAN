## MEAN STACK IMPLEMENTATION: Building a Book Register app with MEAN Stack

One of the most popular JavaScript stack used for building web applications is the **MEAN Stack**. It stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js.

## What is this MEAN stack all about?

- **MongoDB**: it is a-flexible digital filing cabinet or container for your data.

- **Express.js**: It is the very helpful software that manages how your app talks to the internet.

- **AngularJS**: This is the artist/framework-of-arts of this group, it helps your web pages to look good and work smoothly.

- **Node.js**: This is the engine that powers everything behind the scenes.

These tools comes together to help build amazing web applications and one of the best thing about these group of tools is that they all speak one language = JavaScript, making life easier for developers!

## Let's Get Started!

## Step 0: Setting Up Your Workspace

First things first, we need a place to work. Let us set up an EC2 computer in the cloud (via AWS).

1. **We'll launch an EC2 instance using AWS.**

2. **We'll set up a special key to access our cloud computer securely.**

3. **We will also set up some security rules to specify what ports to allow and what to deny.**

![project1](/images/mean.jpg)

![project2](/images/mean2.jpg)

## Step 1: Installing Node.js

Node.js is like the center of our operation. follow the steps below to get it installed on your machine:

1. First, we'll update our cloud computer to make sure it has the latest information by running this command on our cli:

```bash
sudo apt update && sudo apt upgrade
```

![project3](/images/mean3.jpg)

2. Next thing, we will add some security certificates.

```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```
![project4](/images/mean4.jpg)

3. Now, let us install Node.js:

```bash
sudo apt-get install nodejs -y
```
![project5](/images/mean5.jpg)

## Step 2: Setting Up MongoDB

MongoDB is where we would store all our book information. Let's get set it up and make sure that it is running:

1. We would start by downloading a special key for MongoDB:

```bash
sudo apt-get install -y gnupg curl
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
```
![project6](/images/mean6.jpg)

2. Then, we will point out to our computer where to find MongoDB for installation by running this command:

```bash
echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
![project7](/images/mean7.jpg)

3. Now, let us update our computer's knowledge of where to get apps, then we will proceed and install MongoDB:

```bash
sudo apt update
sudo apt-get install -y mongodb-org
```

4. It is now time to start MongoDB and make sure it runs every time we turn on our computer:

```bash
sudo systemctl start mongo
sudo systemctl status mongo
```
![project8](/images/mean8.jpg)

5. We'll install (npm) - Node Package Manager and install a helper tool called body-parser:

```bash
sudo apt install -y npm
sudo npm install body-parser
```
![project9](/images/mean9.jpg)

![project10](/images/mean10.jpg)

6. Let's create a special folder for our project:

```bash
mkdir Books && cd Books
npm init
```
![project11](/images/mean11.jpg)

we would also create a file called server.js and put our special code in it:

```bash
vi server.js
```
```bash
const express = require('express');
const bodyparser = require('body-parser')
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
```

# Step 3: Setting Up Express and Our Routes

Express is the traffic controller for our app. It helps direct information to where it needs to go.

1. Let's install Express and a tool called Mongoose:

```bash
sudo npm install express mongoose
```
![project12](/images/mean12.jpg)

2. We will now create a new folder called **apps** and a file inside it called **routes.js**:

```bash
mkdir apps && cd apps
vi routes.js
```

```bash
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({
        message: 'Successfully added book',
        book: savedBook
      });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
          if (!result) {
        return res.status(404).json({ message: 'Book not found' });
      }
      res.json({ message: 'Book successfully deleted', book: result });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
```

![project13](/images/mean13.jpg)

3. We'll also create a 'models' folder and a file called 'book.js' inside it:

```bash
mkdir models && cd models
vi book.js
```

```bash
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true, index: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true, min: 1 }
}, {
  timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```

## Step 4: Creating Our Web Page with AngularJS

We will now implement AngularJS to help us create a nice-looking, interactive web page for our Book Register web application.

1. First, Let's create a 'public' folder and add a file called 'script.js':

```bash
cd ../..
mkdir public && cd public
vi script.js
```

```bash
angular.module('myApp', [])
  .controller('myCtrl', function($scope, $http) {
    function fetchBooks() {
      $http.get('/book')
        .then(response => {
          $scope.books = response.data;
        })
        .catch(error => {
          console.error('Error fetching books:', error);
        });
    }

    fetchBooks();

    $scope.del_book = function(book) {
      $http.delete(`/book/${book.isbn}`)
        .then(() => {
          fetchBooks();
        })
        .catch(error => {
          console.error('Error deleting book:', error);
        });
    };

    $scope.add_book = function() {
      const newBook = {
        name: $scope.Name,
        isbn: $scope.Isbn,
        author: $scope.Author,
        pages: $scope.Pages
      };

      $http.post('/book', newBook)
        .then(() => {
          fetchBooks();
          // Clear form fields
          $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
        })
        .catch(error => {
          console.error('Error adding book:', error);
        });
    };
  });
  ```
  ![project14](/images/mean14.jpg)

  2. Now, let us create our main web page file in the public folder, 'index.html':

```bash
vi index.html
```
```bash
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Book Management</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  <script src="script.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
    input[type="text"], input[type="number"] { width: 100%; padding: 5px; }
    button { margin-top: 10px; padding: 5px 10px; }
  </style>
</head>
<body>
  <h1>Book Management</h1>
  
  <h2>Add New Book</h2>
  <form ng-submit="add_book()">
    <table>
      <tr>
        <td>Name:</td>
        <td><input type="text" ng-model="Name" required></td>
      </tr>
      <tr>
        <td>ISBN:</td>
        <td><input type="text" ng-model="Isbn" required></td>
      </tr>
      <tr>
        <td>Author:</td>
        <td><input type="text" ng-model="Author" required></td>
      </tr>
      <tr>
        <td>Pages:</td>
        <td><input type="number" ng-model="Pages" required></td>
      </tr>
    </table>
    <button type="submit">Add Book</button>
  </form>

  <h2>Book List</h2>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>ISBN</th>
        <th>Author</th>
        <th>Pages</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="book in books">
        <td>{{book.name}}</td>
        <td>{{book.isbn}}</td>
        <td>{{book.author}}</td>
        <td>{{book.pages}}</td>
        <td><button ng-click="del_book(book)">Delete</button></td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```
3. We will now start our server and see what we have done.

```bash
cd ..
node server.js
```
![project15](/images/mean15.jpg)

Congratulations! Your Book Register app is now up and running. You can access it using your computer's public address on port 3300.

![project16](/images/mean16.jpg)

Go ahead and add some books to your register:

![project17](/images/mean17.jpg)

## Conclusion

The purpose of this documentation is to help/guide you through building a full-fledged web application using the MEAN stack! This powerful combination of MongoDB, Express.js, AngularJS, and Node.js would help you to you to create modern, efficient web apps all using JavaScript.