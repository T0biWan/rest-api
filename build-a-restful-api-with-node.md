# Build a REST API with Node.js

## DB

### Local

```
mkdir data\mongodb
```

### Remote

Go to [mLab](https://mlab.com/).

- Create an account
- Create MongoDB Deployment (a sandbox Database)
- Create a user for your new Database

## Setup

Use npm to set up your project

```
npm init
npm install --save express mongoose body-parser
npm install --save-dev nodemon
```

Open _package.json_ and add the following lines

```
"scripts": {
    "dev": "nodemon server.js",
    "start": "node server.js"
},
```

_package.json_ should look something like this

```
{
  "name": "rest-api",
  "version": "1.0.0",
  "description": "a basic implementation of a rest api with node",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon server.js",
    "start": "node server.js"
  },
  "repository": {
    "type": "git",
    "url": "<your url>"
  },
  "author": "<you>",
  "license": "ISC"
}
```

Now add directories to structure our application and create the neccessary files

```
mkdir api
mkdir api\controllers
mkdir api\models
mkdir api\routes

touch app.js
touch server.js
touch api\controllers\UserController.js
touch api\models\User.js
touch api\routes\UserRouter.js
touch data\db.js
```

Now write some code

```
// app.js
var express = require('express');
var app = express();
var db = require('./data/db');

var UserRouter = require('./api/routes/UserRouter');
app.use('/users', UserRouter);

module.exports = app;
```

```
// server.js
var app = require('./app');
var port = process.env.PORT || 3000; // PORT=4444 node server.js

var server = app.listen(port, function() {
  console.log('Express server listening on port: '+ port);
});
```

```
// api/controllers/UserController.js
var express = require('express');
var bodyParser = require('body-parser');
var User = require('../models/User');
var router = express.Router();
router.use(bodyParser.urlencoded({extended: true}));

// CREATES A NEW USER
exports.createUser = function(req, res) {
  User.create({
      name: req.body.name,
      email: req.body.email,
      password: req.body.password
    },
    function(err, user) {
      if (err) return res.status(500).send("There was a problem adding the information to the database.");
      res.status(200).send(user);
    });
};

// RETURNS ALL THE USERS IN THE DATABASE
exports.getUsers = function(req, res) {
  User.find({}, function(err, users) {
    if (err) return res.status(500).send("There was a problem finding the users.");
    res.status(200).send(users);
  });
};

// GETS A SINGLE USER FROM THE DATABASE
exports.getUser = function(req, res) {
  User.findById(req.params.id, function(err, user) {
    if (err) return res.status(500).send("There was a problem finding the user.");
    if (!user) return res.status(404).send("No user found.");
    res.status(200).send(user);
  });
};

// DELETES A USER FROM THE DATABASE
exports.deleteUser = function(req, res) {
  User.findByIdAndRemove(req.params.id, function(err, user) {
    if (err) return res.status(500).send("There was a problem deleting the user.");
    if (!user) return res.status(404).send("No user found.");
    res.status(200).send("User " + user.name + " was deleted.");
  });
};

// UPDATES A SINGLE USER IN THE DATABASE
exports.updateUser = function(req, res) {
  User.findByIdAndUpdate(req.params.id, req.body, {
    new: true
  }, function(err, user) {
    if (err) return res.status(500).send("There was a problem updating the user.");
    if (!user) return res.status(404).send("No user found.");
    res.status(200).send(user);
  });
};
```

```
// api/models/User.js
var mongoose = require('mongoose');
var UserSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String
});
mongoose.model('User', UserSchema);

module.exports = mongoose.model('User');
```

```
// api/routes/UserRouter.js
var express = require('express');
var bodyParser = require('body-parser');
var User = require('../models/User');
var UserController = require('../controllers/UserController');
var router = express.Router();
router.use(bodyParser.urlencoded({extended: true}));

router.route('/')
  .post(UserController.createUser)
  .get(UserController.getUsers);

router.route('/:id')
  .get(UserController.getUser)
  .delete(UserController.deleteUser)
  .put(UserController.updateUser);

module.exports = router;
```

```
// data/db.js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/<YourDB>'); // local, you can enter any name, if it does not exist, it will be created
mongoose.connect('<your url>'); // remote
```

## Start the database and the server

```
# local db
C:\<your oath>\MongoDB\Server\3.6\binmongod.exe --dbpath C:\<your path>\data\mongodb

# remote db
# ... it depends
```

```
npm run start
```
