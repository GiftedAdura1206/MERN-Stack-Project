# MERN Web stack
The MERN stack is a popular JavaScript-based web development stack that includes:

MongoDB: A NoSQL, document-oriented database for storing data.

ExpressJS: A Node.js framework for building server-side applications.

ReactJS: A JavaScript library by Facebook for building dynamic user interfaces.

Node.js: A JavaScript runtime for executing code outside the browser.

In a MERN application, users interact with React on the frontend, which communicates with an Express server running on Node.js. This server handles requests, interacts with MongoDB as needed, and returns data to the frontend for display.

In order to complete this project you will need an AWS account and create a new EC2 Instance of t2.micro family with **Ubuntu Server 20.04 LTS (HVM) image** 

![alt text](<images/image 1.png>)

![alt text](<images/image 2.png>)

## 1. Backend configuration
Update ubuntu

`sudo apt update`

Upgrade ubuntu

`sudo apt upgrade -y`

  

 Lets get the location of Node.js software from [Ubuntu repositories](https://github.com/nodesource/distributions#deb)

`curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash`

Install Node.js

`sudo apt-get install -y nodejs`

The command above installs both `nodejs` and `npm.` NPM is a package manager for Node like `apt` for `Ubuntu`, it is used to install Node modules & packages and to manage dependency conflicts.

Verify the node installation

`node -v`

Verify the node installation

`npm -v`

![alt text](<images/image 3.png>)

## 2. Application Code Setup

Create a new directory for your To-Do project:

`mkdir Mern-Todo`

Run the command below to verify that the `Mern-Todo` directory is created with `ls` command

`ls`

change your current directory to the newly created one:

`cd Mern-Todo`

Use the command 

`npm init`

![alt text](<images/image 4.png>)

This command initializes your project and creates a `package.json` file, which holds app details and dependencies. Follow the prompts—press `Enter` to accept defaults, then type `yes` to generate the file.

Run `ls` to confirm that you have package.json file created.

Next is to Install ExpressJs and create the Routes directory.

## 3. Install ExpressJS

`npm install express`

![alt text](<images/image 5.png>)

Create a file `index.js` with 

`touch index.js`

Run `ls` to confirm that your `index.js` file is successfully created

Install the `dotenv` module with

`npm install dotenv`

![alt text](<images/image 6.png>)

Open the `index.js` file with the command below

`vi index.js`

Type the code below into it and save.

    const express = require('express');
    require('dotenv').config();

    const app = express();

    const port = process.env.PORT || 5000;

    app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "\*");
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
    next();
    });

    app.use((req, res, next) => {
    res.send('Welcome to Express');
    });

    app.listen(port, () => {
    console.log(Server running on port ${port})

    });

Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.

Now, start the server to check if it is working. In the terminal, navigate to the directory containing your `index.js` file and run

`node index.js`

Now thw **Server running on port 5000** in your terminal.

![alt text](<images/image 7.png>)

Now we need to open this port 5000 in EC2 Security Groups.

![alt text](<images/image 8.png>) 
![alt text](<images/image 9.png>) 
![alt text](<images/image 10.png>)

`http://<PublicIP-or-PublicDNS>:5000`

![alt text](<images/image 11.png>)

## 4. Routes

Our To-Do application must support three core functions:

-   Creating new tasks

-   Displaying all tasks

-   Deleting completed tasks

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

To handle each task, we will define specific routes that serve as endpoints for the To-Do application. Let us start by creating a routes folder to organize these route definitions.

`mkdir routes`

Change directory to `routes` folder.

`cd routes`

Now, create a file `api.js` with the command below

`touch api.js`

Open the file with the command below

`vim api.js`

Copy below code in the file.

    const express = require ('express');
    const router = express.Router();

    router.get('/todos', (req, res, next) => {
    
    });
    
    router.post('/todos', (req, res, next) => {
    
    });
    
    router.delete('/todos/:id', (req, res, next) => {`
    
    })
    
    module.exports = router;


## Models

Since the application will use MongoDB, a NoSQL database, we need to define a schema and create a model.

In JavaScript-based applications, models play a central role in managing and interacting with data. A model is constructed using a schema, which serves as a blueprint for the structure of documents in the database. The schema defines the fields, their data types, validation rules, and default values.

Additionally, schemas can include static and instance methods to simplify data manipulation, as well as virtual properties, computed fields that behave like regular fields but are not stored in the database.

To create a Schema and a model, install mongoose which is a `Node.js` package that makes working with mongodb easier.

Change directory back `Mern-Todo` folder with 

`cd ..`

Install Mongoose (ensure that you are in the `Mern-Todo` project directory)

`npm install mongoose`

Open the file created with 

`vim todo.js` 

Then paste the code below in the file:

    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;

    // Create schema for todo
    const TodoSchema = new Schema({
        action: {
            type: String,
            required: [true, 'The todo text field is required'],
        },
    });

    // Create model for todo
    const Todo = mongoose.model('todo', TodoSchema);
    
    module.exports = Todo;

Now we need to update our routes from the file `api.js` in 'routes' directory to make use of the new model.

In the routes directory, open `api.js` with:

`vim api.js`

Delete its contents using `:%d` and paste the code below

    const express = require('express');
    const router = express.Router();
    const Todo = require('../models/todo');

    router.get('/todos', (req, res, next) => {
        // This will return all the data, exposing only the id and action field to the client
        Todo.find({}, 'action')
          .then((data) => res.json(data))
          .catch(next);
    });

    router.post('/todos', (req, res, next) => {
        if (req.body.action) {
            Todo.create(req.body)
                .then((data) => res.json(data))
                .catch(next);
        } else {
          res.json({
            error: 'The input field is empty',
        });
      }
    });

    router.delete('/todos/:id', (req, res, next) => {
      Todo.findOneAndDelete({ _id: req.params.id })
        .then((data) => res.json(data))
        .catch(next);
    });

    module.exports = router;
The next piece of our application will be the MongoDB Database.

## Connecting to a Database
We will use mLab, a MongoDB Database-as-a-Service (DBaaS), to host our database. To get started, sign up for a free shared cluster [here](https://www.mongodb.com/cloud/atlas/register). During setup, select AWS as the cloud provider and choose the region closest to your location for optimal performance.

![alt text](<images/image 12.png>)

![alt text](<images/image 13.png>)

![alt text](<images/image 14.png>)

![alt text](<images/image 15.png>)

Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing)

![alt text](<images/image 16.png>)

![alt text](<images/image 17.png>)

#### IMPORTANT NOTE

Make sure you change the time of deleting the entry from 6 Hours to 1 Week

In the `index.js` file, we specified `process.env` to access environment variables, but we have not yet created this file. So we need to do that now.

Create a file in your `Mern-Todo` directory and name it `.env`.

    touch .env
    vi .env

Add the connection string to access the database in it, just as below:

`DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'`

Ensure to update <username>, <password>, <network-address> and <database> according to your setup

Here is how to get your connection string

![alt text](<images/image 18.png>) 

![alt text](<images/image 19.png>) 

![alt text](<images/image 20.png>)

Now we need to update the `index.js` to reflect the use of `.env` so that `Node.js` can connect to the database.

Simply delete existing content in the file, and update it with the entire code below.

`vim index.js`

    const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const routes = require('./routes/api');
    const path = require('path');
    require('dotenv').config();

    const app = express();

    const port = process.env.PORT || 5000;

    //connect to the database
    mongoose.connect(process.env.DB, { useNewUrlParser: true})
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

Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the `index.js` application file.

Start your server using the command:

`node index.js`

![alt text](<images/image 21.png>)

This means the backend is configured. Now we are going to test it.

## Testing Backend Code without Frontend using RESTful API

We have completed the backend and database setup for our To-Do application, but the frontend UI (to be built with ReactJS) is still pending. In the meantime, we will use a RESTful API client to test and interact with the backend.

we will use Postman to test our API. Click [Install Postman](https://www.postman.com/downloads/) to download and install postman on your machine.

Click [HERE](https://www.youtube.com/watch?v=FjgYtQK_zLE) to learn how perform basic operartions on Postman

Please ensure that all API endpoints are thoroughly tested to confirm they are functioning as expected. For endpoints requiring a request body, include JSON payloads with the required fields, as specified in our codebase.

Open your Postman, **create a POST request** to the API `http://<PublicIP-or-PublicDNS>:5000/api/todos`. This request sends a new task to our To-Do list so the application could store it in the database.

Sample POST value:

    {
         "action": "build a mern stack application"
    }
Make sure your set header key Content-Type as `application/json`

![alt text](<images/image 22.png>)

**Create a GET** request to your API on http://<PublicIP-or-PublicDNS>:5000/api/todos. This request retrieves all existing records from our To-do application (backend requests these records from the database and sends it us back as a response to GET request).

![alt text](<images/image 23.png>)

# Frontend Creation

With the backend and API for our To-do app complete, we will now develop a web client UI using the create-react-app command to scaffold the frontend.

In the same root directory as your backend code, which is the Mern-Todo directory, run:

`npx create-react-app client`

This will create a new folder in your `Mern-Todo` directory called `client`, where you will add all the react code.

### Running a React App

1.  Install [Concurrently](https://www.npmjs.com/package/concurrently). It is used to run more than one command simultaneously from the same terminal window.

    `npm install concurrently --save-dev`

2.  Install [Nodemon](https://www.npmjs.com/package/nodemon). It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

    `npm install nodemon --save-dev`

3.  In `Mern-Todo` folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.

        "scripts": {
        "start": "node index.js",
        "start-watch": "nodemon index.js",
        "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
        },

    ![alt text](<images/image 24.png>)

#### Configure Proxy in `package.json`

1.  Change directory to 'client

    `cd client`

2.  Open the `package.json` file

    `vi package.json`

3.  Add the key value pair in the package.json file 

    `"proxy": "http://localhost:5000"`

    **N.B.** The proxy configuration added in step 3 enables direct access to the application from the browser using the server URL, such as http://localhost:5000, instead of requiring the full path, like http://localhost:5000/api/todos.
        ![alt text](<images/image 25.png>)

`cd` into the `Mern-Todo` directory, and simply run:

`npm run dev`

![alt text](<images/image 26.png>) 

This means the app is up and running on `localhost:3000` or `http://public-IP:3000`

In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule.

![alt text](<images/image 27.png>)


### Creating the React Components

One key advantage of React is its use of components, which promote code reusability and modularity. In our To-Do app, we will implement two stateful components and one stateless component.

To begin, navigate to your `Mern-Todo` directory and run:

`cd client`

move to the src directory

`cd src`

Inside your `src` folder create another folder called `components`

`mkdir components`

Move into the components directory with

`cd components`

Inside `components` directory create three files `Input.js`, `ListTodo.js` and `Todo.js`.

`touch Input.js ListTodo.js Todo.js`

Open `Input.js`file

`vi Input.js`

Copy and paste the following

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


To incorporate Axios which is a Promise-based HTTP client for browser and Node.js environments, navigate to your client directory via the terminal and execute yarn add axios or npm install axios.


Move to the src folder

`cd ..`

Move to clients folder

`cd ..`

Install Axios

`npm install axios`

Go to `components` directory

`cd src/components`

open your `ListTodo.js`

`vi ListTodo.js`

in the ListTodo.js copy and paste the following code

    import React from 'react';

    const ListTodo = ({ todos, deleteTodo }) => {
    return (
        <ul>
        {todos && todos.length > 0 ? (
            todos.map((todo) => {
            return (
                <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                {todo.action}
                </li>
            );
            })
        ) : (
            <li>No todo(s) left</li>
        )}
        </ul>
    );
    };

    export default ListTodo;

Then in your `Todo.js` file you write the following code

    import React, { Component } from 'react';
    import axios from 'axios';
    import Input from './Input';
    import ListTodo from './ListTodo';

    class Todo extends Component {
    state = {
        todos: [],
    };

    componentDidMount() {
        this.getTodos();
    }

    getTodos = () => {
        axios
        .get('/api/todos')
        .then((res) => {
            if (res.data) {
            this.setState({
                todos: res.data,
            });
            }
        })
        .catch((err) => console.log(err));
    };

    deleteTodo = (id) => {
        axios
        .delete(`/api/todos/${id}`)
        .then((res) => {
            if (res.data) {
            this.getTodos();
            }
        })
        .catch((err) => console.log(err));
    };

    render() {
        let { todos } = this.state;

        return (
        <div>
            <h1>My Todo(s)</h1>
            <Input getTodos={this.getTodos} />
            <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
        </div>
        );
    }
    }

    export default Todo;


We need to make little adjustment to our react code. Delete the logo and adjust our `App.js`

Move to the src folder

`cd ..`

Make sure that you are in the src folder and run

`vi App.js`

Copy and paste the code below into it

    import React from 'react';
    import Todo from './components/Todo';
    import './App.css';

    const App = () => {
    return (
        <div className="App">
        <Todo />
        </div>
    );
    };

    export default App;

After pasting, exit the editor.

In the `src` directory open the `App.css`

`vi App.css`

Then paste the following code into `App.css`

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

In the `src` directory open the `index.css`

`vim index.css`

Copy and paste the code below:

    body {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    box-sizing: border-box;
    background-color: #282c34;
    color: #787a80;
    }

    code {
    font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
    }

Go to the Mern-Todo directory

`cd ../..`

run:

`npm run dev`

![alt text](<images/image 28.png>)

Congratulations!