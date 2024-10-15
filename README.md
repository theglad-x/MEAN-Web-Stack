# MEAN-Web-Stack

## MEAN Stack ia a combination of the following components:
-**MongoDB (Document database)** - Stores and allows to retrieve data.
-**Express (Back-end application framework)** - Makes request to Database for Readsand Writes.
-**Angular (Front-end application framework)** - Handles client and server requests.
-**Node.js (JavaScript runtime environment)** - Accepts requests and displays results to end user.

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
