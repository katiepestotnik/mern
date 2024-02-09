## Express React Build Pt 1

In this build we will

- Build an Express API
- Use Mongo/Mongoose with 2 related models
- Build a Full Crud Frontend with React

## Setup for Express Build

- Create a folder called express-react
- Inside this folder create another folder called backend
- Generate a React app called frontend `npx create-react-app frontend`

Your local folder structure should look like this...

```
/express-react
 -> /backend
 -> /frontend
```

## Setting up the Express app

- cd into `backend` folder
- create a new node project `npm init -y`
- install dependencies `npm install dotenv mongoose express cors`
- install dev dependencies `npm install --save-dev nodemon` if you need to.
- set up 'MVCR' folder structure `mkdir models controllers routes`
- setup npm scripts

```json
"scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
}
```

- make files `touch .env .gitignore server.js`

- put the following .gitignore

```
/node_modules
.env
```

- put the following in .env (make sure to use YOUR mongodb.com uri)

```
DATABASE_URL=mongodb+src://...
PORT=4000
```

## Starting Server.js

Let's build out the minimum to get server.js up and running

```js
///////////////////////////////
// DEPENDENCIES
////////////////////////////////
// get .env variables
require("dotenv").config();
// pull PORT from .env, give default value of 3000
const { PORT } = process.env;
// import express
const express = require("express");
// create application object
const app = express();

///////////////////////////////
// ROUTES
////////////////////////////////
// create a test route
app.get("/", (req, res) => {
  res.send("hello world");
});

///////////////////////////////
// LISTENER
////////////////////////////////
app.listen(PORT, () => console.log(`listening on PORT ${PORT}`));
```

- run the server `npm run dev` and make sure you see "Hello World" when you go to `localhost:4000`

## Add 'index.js' to folders

`index.js` is a very special file in Node. When you use `index.js`, thanks to Node resolving our CommonJS file imports for us, if there isn't a configuration set that specifies which file to look for first. Then it will default to using `index.js`, if there is a file of that name in the folder it's searching in. Most current build tools use Node, so they've inherited this behaviour.

What file will your express server always look for in a folder?

Let's go ahead and create those files for our folders

```js
 touch routes/index.js models/index.js controllers/index.js
```

Spoiler alert: When a request hits our express server, the request will be filtered as such when we're done connecting and setting everything up: 
`server.js -> /routes/index.js -> /controllers/index.js -> /models/index.js`

## Making and Connecting Routes with MVCR
In our `server.js`, lets start making our index and create routes for our future model of "People". For now, we will just make them test routes, then test they work and are connected properly via Postman! 

### Routes

The R in MVCR stands for 'Routes' and you can think of that as our 'index of Routes'. The request will come in, the server will look at our `/routes/index.js` for available routes available to the user. Let's also get rid of our test route since we know our server is connected and working. We need the '/' root route to hold all the route possibilities!

`server.js`
```js
//import all available routes in our /routes/index.js the user can use
const routes = require('./routes/index')

///////////////////////////////
// ROUTES
////////////////////////////////
// create a test route
// app.get("/", (req, res) => {
//     res.send("hello world");
// });

app.use('/', routes) //check the routes index.js for ALL routes so we save space on server.js

//catch all 404 route! 
app.use((req, res) => {res.status(404).json({message: "NOT A PROPER ROUTE"})})
```
Now that we have told our server to look at our router's index page, we will further direct it to the proper route resources. Index should only be used for exporting, but our ACTUAL routes will be in their own resource js page, like so: `/routes/peopleRoutes.js` or however you wish to name it. `[resource][Abbr].js` is fairly typical of a naming convention we will be using through MVCR. 

Create our People route: 
```js
touch routes/peopleRoutes.js
```
The purpose of our `/routes` folder is to be a directional index. To point a certain url to a certain controller. We do not perform any database queries in our `/routes`, instead we will point at the urls proper CONTROLLER in just a bit. 

In `/routes/peopleRoutes.js`: 
```js
const router = require('express').Router();
const { peopleCtrl } = require('../controllers') //all functions/methods imported from people's controller's index.js

// ROUTES - METHODS //
router.get('/', peopleCtrl.getPeople)
router.post('/', peopleCtrl.createPeople)

module.exports = router;
```
With exporting 'router', we are exporting these two routes to be used in `/routes/index.js` along with our controller methods (that aren't set up yet), so lets go and import them to router's index. 

`/router/index.js`
```js
const router = require("express").Router()
const peopleRoute = require("./peopleRoutes")//import the people routing js page

router.use('/people', peopleRoute) //any url beginning in /people will be directed to ./peopleRoutes and then use the request's HTTP method sent

module.exports = router
```
While we only have 1 resource 'People', this is where we can add as many routes as we need, making this express app scalable and ORGANIZED. Routes can easily and quickly disorganize your server.js if we keep everything in there. 

An example could look like: 
```js
const router = require("express").Router()
const peopleRoute = require("./peopleRoutes")
const userRoutes = require('./userRoutes')
const imageRoutes = require('./imageRoutes`)

router.use('/people', peopleRoute)
router.use('/user', userRoutes)
router.use('/images', imageRoutes)

module.exports = router
```

### Controllers

Now we're ready to create our controllers! This is where all our logic lives, every database query, json, etc for our routes! We already set up two methods in our `/routes/peopleRoutes.js` file. Remember what they are? `getPeople` and `createPeople`. Those will be the variables holding our controller logic! Stay tuned to see how we impliment it. 

First, lets start with our `/controllers/index.js` file and set it up. This index doesnt serve a higher purpose like our route's index file, but it's used to export all of our controller methods for a particular resource. In our case, we will have one export for all of our people logic.

`/controllers/index.js` 
```js
module.exports = {
    peopleCtrl: require('./peopleCtrls'), //exports all functions/methods inside the controller for us to use elsewhere, easier
}
```
That's it for that page! Now let's set up the meat and potatoes of our server, the database queries and route logic. 

We have to create our controller file first: 
Create our People route: 
```js
touch controllers/peopleCtrls.js
```

Remember those two methods from earlier? Let's make those now. 

`/controllers/peopleCtrls.js`
```js
// const db = require('../models') //this is where our db mongoose connection lives as well as our models

const getPeople = (req, res) => {
    // db.People.find({})  <-- db has all our models in it so we can use any of them here with one line! 
    res.send('getPeople')
}

const createPeople = (req, res) => {
    // db.People.create({name: 'testing'})
    // .then((res) => {console.log(res)})
    res.send('createPeople')
}

module.exports = {
    getPeople, 
    createPeople
}
```
Technically, all our routes are connected and we are now going to test them in postman before we write our logic.

## Adding a Database Connection

Adding your database connection code to your express server is a matter of preference, usually. We've been showing you to put it in `server.js` as it's the easiest to ensure a connection, but coding loves modularity! We can easily put our connection strings anywhere we want, and for our app, let's go ahead and keep our database connections in our `models` folder, where our database schemas are held. 

`/models/index.js`
```js
const mongoose = require("mongoose");
const {DATABASE_URL} = process.env
///////////////////////////////
// DATABASE CONNECTION
////////////////////////////////
// Establish Connection
mongoose.connect(DATABASE_URL, {
  useUnifiedTopology: true,
  useNewUrlParser: true,
});
// Connection Events
mongoose.connection
  .on("open", () => console.log("You are connected to mongoose"))
  .on("close", () => console.log("You are disconnected from mongoose"))
  .on("error", (error) => console.log(error));

```
- Make sure you see the Mongoose Connection message when the server restarts
- We will come back here once we set up our schemas to export them through our `/models/index.js`

## Adding the Person Model

First, create our model resource file: 

```js
touch models/Person.js
```

Let's add a Person model to `/models/Person.js`, then update our index and create route to add logic and queries. Make sure to add cors and express.json middleware!

`/models/Person,js`
```js
///////////////////////////////
// MODELS
////////////////////////////////
const { mongoose } = require("mongoose");

const formerAddressSchema = new mongoose.Schema({
    streetAddressL1: {type:'String'},
    streetAddressL2: {type:'String'},
    city: {type:'String'},
    state: {type:'String'},
    zip: {type:Number},
    country: {type:'String', default: 'USA'},
})

const PersonSchema = new mongoose.Schema({
    firstName: {type:'String', required: true},
    lastName: {type:'String', required: true},
    streetAddressL1: {type:'String', required: true},
    streetAddressL2: {type:'String'},
    city: {type:'String', required: true},
    state: {type:'String', required: true},
    zip: {type:Number, required:true},
    country: {type:'String', default: 'USA'},
    pastAddresses: [formerAddressSchema],
    phoneNumber: {
        type:'String', 
        required: true, 
        minLength: [14, 'Please enter your number using the following format, no spaces allowed: (111)-111-1111'], 
        maxLength: [14, 'Please enter your number using the following format, no spaces allowed: (111)-111-1111']}, //(111)-111-1111
    email: {type:'String', required: true},
  });
  
  const Person = mongoose.model("Person", PersonSchema);
  
  module.exports = Person
```
We have our Person schema, so all thats left is to export it in our `/models/index.js` so ANY folder or controller can use this. Yay modularization! 

`/models/index.js`
```js
//all code above this

module.exports = {
    Person: require('./Person')
}
```

`server.js`
```js
....

const app = express();
// add this - import middlware
const cors = require("cors");

...

///////////////////////////////
// MiddleWare
////////////////////////////////
app.use(cors()); // to prevent cors errors, open access to all origins
app.use(express.urlencoded({extended: true}))
app.use(express.json()); // parse json bodies

```
Now the controller logic:

`/controllers/peopleCtrls.js`
```js
const db = require('../models') //this is where our db mongoose connection lives as well as our models

// PEOPLE INDEX ROUTE
const getPeople = async (req, res) => {
    // db.People.find({})  <-- db has all our models in it so we can use any of them here with one line! 
    // res.send('getPeople')
    try{
        const foundPeople = await db.Person.find({})
        if(!foundPeople){
            res.status(404).json({message: 'Cannot find People'})
        } else {
            res.status(200).json({data: foundPeople})
        }
    } catch(err) {res.status(400).json({error: err.message})}
}

// PEOPLE CREATE ROUTE
const createPeople = async (req, res) => {
// db.People.create({name: 'testing'})
// res.send('createPeople')
    try{
        const createdPerson = await db.Person.create(req.body)
        createdPerson.save()
        if(!createdPerson){
            res.status(400).json({message: 'Cannot create Person'})
        } else {
            res.status(201).json({data: createdPerson, message: 'Person created'})
        }
    } catch(err) {res.status(400).json({error: err.message})}
}

module.exports = {
    getPeople,
    createPeople,
}
```

- create 3 people using postman to make post requests to /people

- test the index route with a get request to /people

### Running into Errors? 

- Check your console to see if any mongoose-related console log is present
- Make sure you are EXPORTING / IMPORTING. Modularization is all about that!
- Check your .env for the proper uri string
- Work backwards using console logs from controllers to help locate the issue

## Update and Delete

Let's add an Update and Delete API Route to `controllers/peopleCtrls.js` to finish up our 4 basic routes.

```js
// PEOPLE UPDATE ROUTE
const updatePerson = async (req, res) => {
    try{
        const updatedPerson = await db.Person.findByIdAndUpdate(req.params.id, req.body, {new: true})
        if(!updatedPerson){
            res.status(400).json({message: 'Could not update person'})
        } else {
            res.status(200).json({Data: updatedPerson, message: "Person updated"})
        }
    } catch (err) {
        res.status(400).json({error: err.message})
    }
}

// PEOPLE DESTROY ROUTE
const deletePerson = async (req, res) => {
    try {
        const deletedPerson = await db.Person.findByIdAndDelete(req.params.id)
        if(!deletedPerson){
            res.status(400).json({message: 'Could not delete person'})
        } else {
            res.status(200).json({Data: deletedPerson, message: "Person deleted"})
        }
    } catch (err) {
        res.status(400).json({error: err.message})
    }
}
```
But they're not connected to our routes, currently, so how do we do that? 

Add them to the export in that same file:
```js
module.exports = {
    getPeople, 
    createPeople,
    updatePerson, 
    deletePerson
}
```

But they're not assigned to a route yet! How do we fix that? 
<details>
    <summary>Check answer here!</summary>

    Assign the routes in `/routes/peopleRoutes.js`

    ```js
    // ROUTES - METHODS //
    router.get('/', peopleCtrl.getPeople)
    router.post('/', peopleCtrl.createPeople)
    router.put('/:id', peopleCtrl.updatePerson)
    router.delete('/:id', peopleCtrl.deletePerson)

    module.exports = router;
    ```

</details>

### Test your Routes in Postman! 
