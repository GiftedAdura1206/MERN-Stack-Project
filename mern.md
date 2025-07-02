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

![alt text](<image 11.png>)

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