# Documenting the process of creating a Social Media Site Fullstack Notes

# Initialising the project
1. Create the project folder and give it a name
2. add a .gitignore file
3. add to it node_moodules/ folder
4. initialise the repository by typing
> git init
> npm init 
5. Andwer all the questions and the license should be MIT

## Express & MongoDB Setup

### MongoDb Setup on MongoDB Atlas

1. Signup/In> Create Project > Create a cluster > AWS provider > Any Region > Tier M0 Free forever > Cluster name - choose name : Create
2. Security tab > Add User > enter username and password > Privileges - Read and Write : add
3. IP whitelist > Allow access from anywhere - for testing purposes only
4. Cluster > Connect > Copy connection string
5. To access data > Collections to Tables

### Install Dependencies and Basic Express Setup

- ../folder-name-of-your-choice > ../folder-name-of-your-choice/.gitignore >

#### /.gitignore

```git
node_modules/
```

> npm init -y

> npm install express express-validator bcryptjs config gravatar jsonwebtoken mongoose request

```md
notes:
bcrypt - encryption,
config - for saving default values,  
gravatar - capturing avatars associated with emails
jsonwebtoken - tokens to be used for authentication,
mongoose - communication with MongoDB,
request - helps us with api communications\*\*
```

- Install dev dependencies
  `npm i -D nodemon concurrently`
- Create main file
  /server.js

##### /server.js

```js
const express = require(‘express’);
const app = express();
// get this API route and send a response to the browsers
app.get('/', (req, res) => res.send('API Running...'));

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => console.log(`Server Running on http://localhost:${PORT}`);

```

- create start scripts in package.json

#### /package.json

```json
// script that heroku will run
"start":"node server",
"server": "nodemon server"
// later we will have a client script that will run react & a dev script that will run both at the same time using concurrently package - server and client

```

: save

> npm run server

- postman > GET http://localhost:5000 > : send
  > API Running...
- Make a git commit as an initial commit - at the end we will send this to heroku

### Connecting to MongoDB with Mongoose

- Log into MongoDB Atlas> on the cluster - click Connect > Connect your application> copy connection string
  /config > /config/default.json - to create global values for the application

#### /config/default.json

```json
{
	"mongoURL": "mongodb+srv://<username>:<password>@devsocial.4ynvm.mongodb.net/?retryWrites=true&w=majority"
}
```

- add a username and password for your setup > inside config folder add another file db.js >

#### /config/db.js

```js
const mongoose = require('mongoose');
const config = require('config');
const db = config.get('mongoURI');

// mongoose.connect(db) will connect to Atlas but it gives us a promise.
// instead of using .then() we will use async await instead because its the new standard and it makes the code much cleaner

// since we need something to call in server.js, we'll create a function

const connectDB = async () => {
	// whenever we use async await, we use a try catch block
	try {
		await mongoose.connect(db);
		console.log('MongoDB Connected...');
	} catch (err) {
		console.error(err.message);
		// since we want to exit process to fail,
		process.exit(1);
	}
};
// export the connectDB function
module.exports = connectDB;
```

#### /server.js

```js
// add this after the express requirement at the top
const connectDB = require('config/db');

// add this after the express app initialisation to connect to the databaase
connectDB();
```

> npm run server

- if it says
  > pass option { useNewUrlParser: true }
- in the db.js

#### /config/db.js

```js
// pass the option into mongoose.connect by adding it as an object
await mongoose.connect(db, {
	useNewUrlParser: true,
});
```

> MongoDB Connected...

### Route Files with Express Router

- We want to break this down by resource i.e. Users, Profile, Auth & Posts
  /routes - all routes will return json >
  /routes/api - no server rendered templates, all will happen on the frontend - Reactjs side >
  /routes/api/users.js >
  /routes/api/auth.js >
  /routes/api/posts.js >
  /routes/api/profile.js

#### /routes/api/users.js

```js
const express = require('express');
const router = express.Router();

// @route   GET api/users
// desc     Test Route
// access   Public - we dont need a tken for authentication. Private - we do need one
router.get('/', (req, res) => res.send('User Route...'));

module.exports = router;
```

#### /routes/api/profile.js

```js
const express = require('express');
const router = express.Router();

// @route   GET api/profile
// desc     Test Route
// access   Public - we dont need a tken for authentication. Private - we do need one
router.get('/', (req, res) => res.send('Profile Route...'));

module.exports = router;
```

#### /routes/api/posts.js

```js
const express = require('express');
const router = express.Router();

// @route   GET api/posts
// desc     Test Route
// access   Public - we dont need a tken for authentication. Private - we do need one
router.get('/', (req, res) => res.send('Posts Route...'));

module.exports = router;
```

#### /routes/api/auth.js

```js
const express = require('express');
const router = express.Router();

// @route   GET api/auth
// desc     Test Route
// access   Public - we dont need a tken for authentication. Private - we do need one
router.get('/', (req, res) => res.send('Auth Route...'));

module.exports = router;
```

- We need to add them to server.js to access them

#### /server.js

```js
// add these lines just below the app.get() route
// Define routes
app.use('/api/users', require('./routes/api/users'));
app.use('/api/auth', require('./routes/api/auth'));
app.use('/api/posts', require('./routes/api/posts'));
app.use('/api/profile', require('./routes/api/profile'));
```

#### Test route with Postman

- postman > GET http://localhost:5000/api/users > : send
  > User Route...
- postman > GET http://localhost:5000/api/auth > : send
  > Auth Route...
- postman > GET http://localhost:5000/api/posts > : send
  > Posts Route...
- postman > GET http://localhost:5000/api/profile > : send

  > Profile Route...

- Lets organise our different requests in postman
  Create these collections> Posts - for post routes > Profiles - for all Profile Routed> User & Auth - routes for Users and Authentication

## User API Routes & JWT Authentication

```js
// We are setting up the user model so that we are able to send an email, password, name etc. to create a user in our database. To do this we need to create a model first.
// We will create a user first> then authentication> then move on to profile> then post
// We will need to create a model for each of our resources(Users, Authentication, Profiles and Posts)
// We will start with the user.
```

/modals > /modals/User.js

```js
// User.js is where we will add the schema - which holds the particular fields we would want this particular field to have
```

#### /modals/User.js

```js
const mongoose = require('mongoose');
const UserSchema = new mongoose.Schema({
	name: {
		type: String,
		required: true,
	},
	email: {
		type: String,
		required: true,
		unique: true
	},
	password: {
		type: String,
		required: true,
	}
	avatar: {
		type: String,
	},
	date: {
		type: Date,
		// automatic current date and time
		default: Date.now
	}
});

// we export a variable called User and set it to mongoose.modal
// which takes in 2 arguments.
// mongoose.modal('the modal name', 'and the schema we have just created')

module.exports = User = mongoose.modal('user',UserSchema)
```

- We save this and shouldnt have to touch this file again unless we are adding new fields to the User resource.

### Registering new Users, validation responses(for cleaner data)

#### /routes/api/users.js

```js
// lets start work on the registration route which will
// be a POST request in users

// @route   POST api/users
// desc     Register user
// access   Public - we dont need a tken for authentication. Private - we do need one

// We need to send data to this route
// we need to send a name, email and password to register this user
// so we wrap the response in curly brackets to add other lines of code.

router.post('/', (req, res) => {
	console.log(req.body);
	// but to use req.body, you need to initialise the middleware of the
	// body parser. We used to install  body parser, bring it in
	// and then initialise it but now its actuallly included with express
	res.send('User Route...');
});
```

#### /server.js

```js
// under connectDB();

// Init Middleware
// instead of doing app.use(bodyParser.json()) like we used to
// and then we pass in an object of extended:false
app.use(express.json({ extended: false }));
// this one line should allow us to get the data from req.body
```

#### Test route with Postman

> POST http://localhost:5000/api/users

- headers tab > key: content-type > value: application/json
- body tab > choose raw

```json
{
	"name": "Andrew Bwogi"
}
```

- when you send that, in postman you still see 'User route...'
- But on the console you see { name : 'Andrew Bwogi'}
- So we can send whatever we want an access it with **req.body**

- Before we start on the data, we need to start on validation.

#### /routes/api/users.js

```js
// under the router import at the top,

const { check, validationResult } = require('express-validator/check');

// its all in the validator documentation
// google express-validator got to the .md readme file
// and click on the documentation page

// in the Route, pass this valicator, check as a middleware
// for errors you pass in validationResult(req) for any errors
// if there is, send a response.status() to show what needs to be corrected.
// this will throw back an object with the details of what should be corrected.

// lets modify our post method by adding a second parameter in a set of square brackets []
router.post('/',[
	// we will run the check function to check for the name
	// then we can pass in a second parameter for a custom message
	// otherwise it will give us a generic message
check('name', 'Name is required').not().isEmpty(),
// if we want the message to be there and not empty
check('email' 'Please enter a valid Email').isEmail(),
check('password', 'Please enter a password with 6 or more characters').isLength({min:6}),
], (req, res) => {
	// console.log(req.body);
	const errors = validationResult(req)
	if(!errors.isEmpty()){
		// if errors in not empty, then its a bad request (400)
		// return a json object with an array of
		// whatever is in the errors variable
		return res.status(400).json({errors: errors.array()})
	}

	res.send('User route...')
}
```

#### Test route with Postman

> POST http://localhost:5000/api/users

- headers tab > key: content-type > value: application/json
- body tab > choose raw

```json
{
	"name": "Andrew Bwogi"
}
```

- when we send this, see the request status as **_400 bad request_**
- and then we also have an array of errors

```json
{
	"errors": [
		{
			"location": "body",
			// param is the actual field in the body of the url, email in this case
			"param": "email",
			// the custom message we added in the user route during validation
			"msg": "Please include a valid email!"
		},
		{
			"location": "body",
			"param": "password",
			"msg": "Please enter a password with 6 or more characters!"
		}
	]
}
```

- if we submit without any name, email or password, then

```json
{
	"errors": [
		{
			"location": "body",
			"param": "name",
			"msg": "Name is required!"
		},
		{
			"location": "body",
			"param": "email",
			"msg": "Please include a valid email!"
		},
		{
			"location": "body",
			"param": "password",
			"msg": "Please enter a password with 6 or more characters!"
		}
	]
}
```

- you dont have to worry about empty strings say an empty name field, its already factored in which is harder to do with custom validation

#### Test route with Postman

> POST http://localhost:5000/api/users

- headers tab > key: content-type > value: application/json
- body tab > choose raw

```json
{
	"name": "Andrew Bwogi",
	// i will use an email that has an avatar
	// so that this works when gravatar is used later...
	"email": "andrew.bwogi@gmail.com",
	"password": "123456"
}
```

- when you send, you see status **_200 OK_** meaning data was successfully sent.

##### Save in postman

- got to the dropdown arrow on the Save button next to the Send button and select **_Save As_** > Choose the Users & Auth collection> change name to Register User : Save
- Therefore we can always go to User & Auth whenever we want to register a user.

### Lets add logic to User Registration

#### /routes/api/user.js

```js

const User = require('../../modals/User'); // 1.
const gravatar = require('gravatar'); // 2.
const bcrypt = require('bcryptjs'); //3.




router.post('/',[
	check('name', 'Name is required').not().isEmpty(),
	check('email' 'Please enter a valid Email').isEmail(),
	check('password', 'Please enter a password with 6 or more characters').isLength({min:6}),
], async (req, res) => {
	const errors = validationResult(req)
	if(!errors.isEmpty()){
		return res.status(400).json({errors: errors.array()})
	}
		// we destructure req.body
	const { name, email, password } = req.body;

	try{
		// we want to see if the user exists in the database
		// if the user exists, we will send back an error message because we want one unique email

		// then we bring in our user modal - right below 'express-validator/check' import, add line (1.) above
		// then we use User.findOne() which returns a promise. So instead of using .then, we use async await.
		// the function is modified to use async await
		let user = await User.findOne({ email }) // search by email. {email:email} is the same as {email}
		if(user){
			res.status(400).json({errors: [{ msg: 'User already exists' }]})

			// deprecation warning to add createIndexes instead
			// solved by going to db.js after newUrlParser: true, add useCreateIndex:true
			// the error goes away
		}

		// Get User's Avatar
		// import gravatar at 2. first
		const avatar = gravatar.url(email, {
			s: '200', // size
			r: 'pg', // rating
			d: 'mm'
		})

		// then we create a new instance of a user.
		// We then create an object with the name, email and password

		user = new User({
			name,
			email,
			password,
			avatar
		})
		// this doesnt save the user though and the password is not encrypted


		// Encrypt User's Password
		// we need to bring in bcrypt at 3.
		// we create a salt to do the hashing with
		// we get a promise from bcrypt.genSalt. So we use await
		// we pass in the rounds - 10 in this case recommended in the documentation
		// the higher the stronger the hash/ password
		const salt = await bcrypt.genSalt(10);
		user.password = await bcrypt.hash(password, salt);

		// now we can save the user
		await user.save(); // anything that returns a promise, you have to add await


		// Return Jsonwebtoken. In the frontend when a user registers, a jsonwebtoken is returned so that they can log in right away
		// refer to https://jwt.io which breaks down a jsonwebtoken and its composition. basically 3 parts(header: algorithm or token type, the payload: data, & verify signature)
		// in the payload, we add the id so that we can identify which user it is with the token.

		// the way this works with the jsonwebtoken package we have installed is we need to first sign it,

		// later we will protect our routes by creating a piece of middleware that will verify the token to either allow the user access or respond with an error message saying 'invalid token'



	}catch(err){
		console.error(err.message);
		res.status(500).send('Server Error!')

	}

	res.send('User route...')
}
```

#### Test route with Postman

> POST http://localhost:5000/api/users

- headers tab > key: content-type > value: application/json
- body tab > choose raw

```json
{
	"name": "Andrew Bwogi",
	"email": "andrew.bwogi@gmail.com",
	"password": "123456"
}
```

- when you send, you see **User Registered...**

- Go to mongodb Atlas database, go to the cluster> collections > You'll see a collection called users > when you click on it you will see an object inserted automatically with everything

## Profile API Routes

## Post API Routes

## Getting started with the Reacts front-end

Lets set up react within the social media folder called client. But we also need to set up concurrently to run both the frontend and the backend server together.

> npx create-react-app client

check if the reactjs server is running by running

> npm start

within the client folder

To run the server and the client concurrently, we need to set it up within the package.json file.

#### /package.json

```json
{
	// lets add other scripts in here.

	"client": "npm start --prefix client",
	"dev": "concurrently \"npm run server\" \"npm run client\""
}
```

We are going to add some dependencies for the client side. For example:

1. axios - to make http requests. Could use the fetch API but we are doing specific things only axios can do.
2. react-router-dom
3. redux
4. react-redux
5. redux-thunk - which is middleware to allow us to make asynchronous requests
6. redux-devtools-extension - We will also use react dev tools extensively in the course
7. moment - date and time management
8. react-moment - so that we can use moment with the components

> npm i axios react-router-dom redux react-redux redux-thunk redux-devtools-extension moment react-moment

## Redux Setup & Alerts

## React User Authentication

## Dashboard and Profile Management

## Profile Display

## Posts and Comments

## Prepare & Deploy

## Issues Added and Features
