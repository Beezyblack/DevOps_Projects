# Deploying A MERN Stack Application On AWS Cloud

A MERN stack consists of a collection of technologies used for the development of web applications. It comprises of MongoDB, Express, React and Node which are all JavaScript technologies used for creating full-stack applications and dynamic websites.

MongoDB: is a document-oriented NoSQL database technology used in storing data in the form of documents.

React: a javascript library used for building interactive user interface based on components.

Express: its a web application framework of Node js which is used for building server-side part of an appication and also creating Restful APIs.

Node: Node.js is an open-source, cross-platform runtime environment for building fast and scalable server-side and networking applications.

I will be building a simple todo list application and deploying on AWS cloud EC2 machine.

## Step 1

### Creation of an EC2 Ubuntu Instance

Provisioned an EC2 Ubuntu instance on AWS


After provisioning my EC2 Instance, Log into the instance via ssh

### Configuring the Backend

Run `sudo apt update` and `sudo apt upgrade`  to update all default ubuntu dependencies to ensure compatibility during package installation.

Next up will be to install nodejs, first we get the location of nodejs form the ubuntu repository using the following command. `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

Then we install node.js on the server by running the command below
`sudo apt-get install -y nodejs`

Now verify the node installation with the command below

`node -v`

### Setting up the application

In setting up the application we first create a directory that will house our codes and packages and all subdirectories to represent components of our application by running the command below:

`mkdir todo`

Run the command `ls` to verify that the Todo directory is created.


Inside this directory we will initialise our project running the command `npm init`. This enables javascript to install packages useful for spinning up our application.


Run the command `ls` to confirm that the package.json file has been created.

## Step 2

### ExpressJS Installation

Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. Therefore it simplifies development, and abstracts a lot of low level details.

We will be installing express which is nodejs framework and will be helpful when creating routes for our application via HTTP requests. To use express, we will install it using npm:

`npm install express`

Create an index.js file with the command `touch index.js` which will contain code useful for spinning up our express server


Install the ``dotenv`` module with the command ``npm install dotnev``


Open the ``index.js`` file with the command ``vim index.js`` and type the code as seen in the image below.


img


Now we start our server with the command ``node index.js``. The code in the image above is useful for starting up our application through the port specified in the code.


Now we need to edit our inbound rules and open port 5000 in EC2 scurity groups.


img


Accessing my servers public IP followed by port 5000 to see if the server is properly configured.


img


### Defining The Routes For Our Application

There are three actions that our To-Do application needs to do:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

For each task we will create ``routes`` folder which will contain code pointing to the three main endpoints used in a todo application. These endpoints will be helpful in interacting with our client_side and database via restful apis.

```bash
mkdir routes
cd routes
touch api.js
```

Open the file with the command ``vim api.js`` and copy the code below in.

```bash
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```


## Step 3

### Creating Models

A model is at the heart of JavaScript based applications, and it is what makes it interactive. Since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.

Models are also used to define database schema, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. This is important so that we will be able to define the fields stored in each Mongodb document.


Inside the Todo folder, run `` npm install install mongoose``

Create a models folder by running the command ``mkdir models`` and then create a file in it by running the command ``touch todo.js`` write the code below into the ``todo.js`` file

To open the file created run ``vim todo.js``

```bash
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

img


Now we update our routes from the file ``api.js`` in the routes directory to make use of the new model

In Routes directory, ``open api.js`` with ``vim api.js``, delete the code inside with :%d command and paste there code below into it then save and exit.

```bash
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

img

## Step 4

### Create A MongoDB Database

We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS).

Login into mLab and create a cluster


img


Create a database and a collection


img


To connect mongoose(application_db) to our database service we connect it using the connection credential provided by mLab.



img



In the ``index.js`` file, we specified ``process.env`` to access environment variables, but we have not yet created this file. So we need to do that now. Copy the connection code and save it inside a ``.env`` file which should be created in the parent ``Todo`` folder

``DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'``


Update the ``<username>``, ``<password>``, ``network-address>`` and ``<database>`` according to your setup to the one specified by me when I created my database and collection.




Now update the ``index.js`` with the code below to reflect the use of ``.env`` so that Node.js can connect to the database. This is to point mongoose to the database service we created using mLab.



```bash
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```


img



Start my server by running the command ``node index.js`` a message ‘Database connected successfully’ to show our backend is configured


img



### Test Backend Code Using Postman

So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. To see if it works without a frontend, we will be using postman to test the endpoints.


On postman make a POST request to our database while specifying an action in the body of the request.

img

Then make a GET request to see if we can ger back what has been posted into the database.

img

We can also make a delete request which deletes and entry using the id of each entry.

img


## Step 5

### Frontend Creation

Frontend is the user interface for a web client (browser) to interact with the application via API.

In the Todo folder which is the same folder containing the backend code run ``npx create-react-app client`` this creates a folder containing the necessary packages required for react to work.

img

Install ``concurrrently`` and ``nodemon`` which are dependencies required for react to work.

```bash
npm install concurrently --save-dev
npm install nodemon --save-dev
```

img


Configure the ``package.json`` file by changing the code in the image below to this

```bash
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

img

Configure proxy in ``package.json`` to ensure we can access our site via the url using ``http://localhost:5000``


img


Now we run ``npm run dev`` inside the todo folder and the server is opened on port localhost:3000. Then we set inbound security group rule for port 3000.


img


img


### Creation of React Components

The advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.
Inside the ``components`` directory create three files  ``Input.js ListTodo.js Todo.js``.



Open the ``Input.js`` file by running the command ``vi Input.js`` and insert the code below in

```bash
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```

img


Ensure ``axios`` is installed in the client directory by running the command ``npm install axios``


In the ``ListTodo.js`` directory insert the code below

```bash
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```

img


In the ``Todo.js`` directory insert the code below


```bash
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```

In the src directory open the ``App.css`` by running the command ``vi App.css`` and insert the code as shown below

```bash
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```

In the ``src`` directory open the ``index.css`` file by running the command and insert the code below

```bash
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```

For the final step go into the root directory ``Todo`` and run ``npm run dev``. To-do app is ready and functional.

img



