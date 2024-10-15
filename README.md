# MEAN-Web-Stack

## MEAN Stack ia a combination of the following components:
- **MongoDB (Document database)** - Stores and allows to retrieve data.
- **Express (Back-end application framework)** - Makes request to Database for Readsand Writes.
- **Angular (Front-end application framework)** - Handles client and server requests.
- **Node.js (JavaScript runtime environment)** - Accepts requests and displays results to end user.

### Prerequisites
## Step 0 - Prerequisites
1. AWS account
2. Launch EC2 instance using ubuntu image
3. Configure security group with the following inbound rules:
Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
Allow traffic on port 22 (SSH) with source from any IP address.
4. Setup other configurtions
5. Connect to the instance using the CLI

## Step 1 - Install NodeJs
Update ubuntu
```sh
sudo apt update
```
Upgrade ubuntu
```sh
sudo apt upgrde
```
Locate the Node.js software from Ubuntu repositories
```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
Install NodeJs
```sh
sudo apt install -y nodejs
```
Verify that NodeJs and npm is instaalled
```sh
node -v
```
```sh
npm -v
```
## Step 2 - Install MongoDB
Import the public key used by the package management system from a terminal, install gnupg and curl if they are not already available:
```sh
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
```
```sh
sudo apt-get install gnupg curl
```
Create a List-File
```sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```
Once completed, update the system to activate the changes and add the MongoDB repository.
```sh
sudo apt-get update
```
Install the packages needed for the MongoDB version you want to run. In most cases, it is a good idea to select the current version of MongoDB.
```sh
sudo apt-get install -y mongodb-org
```
Start the database installed
```sh
 sudo systemctl start mongod
```
Verify the status of the mongodb databse
```sh
sudo systemctl status mongod
```
Install body-parser packege
```sh
sudo npm install body-parser
```
Createa foldr named ```Books```
```sh
mkdir Books && cd Books
```
In the Books directory, initialize npm project
```sh npm init
```
Add a file to it named ```server.js```
```sh
vi server.js
```
Copy and paste the web server code below into the server.js file
```sh
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3000);
app.listen(app.get('port'), function(){
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
## Step 3 - Install Express and set up routes to the server
Install Express & Mongoose
```sh
sudo npm install express mongoose
```
In ```Books``` folder, create a folder named ```apps```
```sh
mkdir apps && cd apps
```
Create a file name ```routes.js```
```sh
vi routes.js
```
Copy and paste the code 
```sh
    var Book = require('./models/book');
module.exports = function(app){
    app.get('/book', function(req,res){
        Book.find({}, function(err, result){
            if(err) throw err;
            res.json(result);
        });
    });
    app.post('/book', function(req, res){
        var book = new Book({
            name:req.body.name,
            isbn:req.body.isbn,
            author:req.body.author,
            pages:req.body.pages
        });
        book.save(function(err, result){
            if(err)throw err;
            res.json({
                message:"Successfully added book",
                book:result
            });
        });
    });
    app.delete("/book/:isbn", function(req, res){
        Book.findOneAndRemove(req.query, function(err,result){
            if(err) throw err;
            res.json({
                message: "Successfully deleted the book",
                book: result
            });
        });
    });
    var path = require('path');
    app.get('*', function(req,res){
        res.sendfile(path.json(__dirname + '/public', 'index.html'));
    });
};
```
In the ```apps```directory, create a folder named ```models```
```sh
mkdir models && cd models
```
Create a file named ```book.js```
```sh
nano book.js
```
Copy and paste
```sh
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema({
    name: String,
    isbn: {type: String, index: true},
    author: String,
    pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
## Step 4 - Access the routes with AngularJs
Change the directory back to ```Books```
```sh
cd ../..
```
Create a folder named ```public```
```sh
mkdir public && cd public
```
Add a file named ```script.js```
```sh
vim script.js
```
Copy and paste the configuration code into the ```script.js``` file
```sh
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
    // Fetch books from the server
    $http({
        method: 'GET',
        url: '/book'
    }).then(function successCallback(response) {
        $scope.books = response.data;
    }, function errorCallback(response) {
        console.log('Error:' + response);
    });

    // Delete a book
    $scope.del_book = function(book) {
        $http({
            method: 'DELETE',
            url: '/book/' + book.isbn // Use book.isbn directly in the URL
        }).then(function successCallback(response) {
            console.log(response);
            // Optionally, refresh the book list or remove the deleted book from $scope.books
        }, function errorCallback(response) {
            console.log('Error:' + response);
        });
    };

    // Add a new book
    $scope.add_book = function() {
        var body = {
            name: $scope.Name,
            isbn: $scope.Isbn,
            author: $scope.Author,
            pages: $scope.Pages
        };

        $http({
            method: 'POST', // Changed to POST for adding a book
            url: '/book',
            data: body,
            headers: {
                'Content-Type': 'application/json' // Set the content type to JSON
            }
        }).then(function successCallback(response) {
            console.log(response);
            // Optionally, refresh the book list or add the new book to $scope.books
        }, function errorCallback(response) {
            console.log('Error:' + response);
        });
    };
});
```
In ```public``` folder, create a file named ```index.html```
```sh
vi index.html
```
Copy and paste 
```sh
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
</head>    
    <body>
        <div>
            <table>
                <tr>
                    <td>Name:</td>
                    <td><input type="text" ng-model="Name"></td>
                </tr>
                <tr>
                    <td>Isbn:</td>
                    <td><input type="text" ng-model="Isbn"></td>
                </tr>
                <tr>
                    <td>Author:</td>
                    <td><input type="text" ng-model="Author"></td>
                </tr>
                <tr>
                    <td>Pages:</td>
                    <td><input type="text" ng-model="Pages"></td>
                </tr>
            </table>
            <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Isbn</th>
                    <th>Author</th>
                    <th>Pages</th>
                </tr>

                <tr ng-repeat="book in books">
                    <td>{{book.name}}</td>
                    <td>{{book.isbn}}</td>
                    <td>{{book.author}}</td>
                    <td>{{book.pages}}</td>

                    <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
                </tr>
            </table>
        </div>
    </body>
</html>
```
Change back to ```Books``` directory
```sh
cd ..
```
Start the server on the terminal by running this command
```sh
node server.js
```
![locaL](https://github.com/user-attachments/assets/2ed29fb3-7221-4d4f-ac53-697c789eee88)

Accessing the Web appilication on the browser 

The application is running on port 3000, which will be added to the security group. Add inbound rule, open TCP port ```3000``` fron anywhere ```0.0.0.0/0``` 

```sh
http:// your-public-ip:3000
```
![books](https://github.com/user-attachments/assets/5c4eb3ab-d783-45f8-aaab-0f5162bc1e9e)



## Conclusion


