# express-api
$ mkdir nodejs
$ cd nodejs
$ npm init / npm init -yes
$ npm install express -save (Express Framework)
$ npm install mysql -save (for connecting to DB)
$ npm install body-parser (This is a node.js middleware for handling JSON, Raw, Text and URL encoded form data.)

Server setup:
nodemon will help us to keep track of changes to our application by watching changed files and automatically restart the server.

$ npm install --save-dev nodemon

Create a file server.js

$ touch server.js

$ type code below on server.js

var express = require('express'),
app = express(),
port = process.env.PORT || 3000;
app.listen(port);

console.log('todo list RESTful API server started on: ' + port);

$ Create Database "nodejs db"

$ Create Table
CREATE TABLE IF NOT EXISTS `tasks` (
  `id` int(11) NOT NULL,
  `task` varchar(200) NOT NULL,
  `status` tinyint(1) NOT NULL DEFAULT '1',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
);
 
ALTER TABLE `tasks` ADD PRIMARY KEY (`id`);
ALTER TABLE `tasks` MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;

$ Populate the table with sample data:
INSERT INTO `tasks` (`id`, `task`, `status`, `created_at`) VALUES
(1, 'Find bugs', 1, '2016-04-10 23:50:40'),
(2, 'Review code', 1, '2016-04-10 23:50:40'),
(3, 'Fix bugs', 1, '2016-04-10 23:50:40'),
(4, 'Refactor Code', 1, '2016-04-10 23:50:40'),
(5, 'Push to prod', 1, '2016-04-10 23:50:50');

$ mkdir app (Create new folder named app)

$ cd app

inside app create additional folders named controller, model, routes

$ mkdir controller 
$ mkdir model
$ mkdir routes

$ touch appController.js (Inside controller create new file named appController.js)
$ touch appModel.js (Inside model create new file named appModel.js)
$ touch appRoutes.js (Inside routes create new file named approutes.js)

Structure would look like below diagram:
$ image structure diagram

Setting up the routes

Routing refers to determining how an application responds to a client request for a specific endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, PUT,PATCH,DELETE)

Each of our routes has different route handler functions, which are executed when the route is matched.

Below we have defined two basic routes

/tasks
/tasks/taskId
with different methods
‘/tasks’ has to methods(‘GET’ and ‘POST’), while ‘/tasks/taskId’ has GET, PUT and DELETE.
As you can see, we required the controller so each of the routes methods can call it’s respective handler function.
To do this, open the appRoutes.js file in the route folder and paste the code snippet below into

$ 
'use strict';
module.exports = function(app) {
  var todoList = require('../controllers/todoListController');

  // todoList Routes
  app.route('/tasks')
    .get(todoList.list_all_tasks)
    .post(todoList.create_a_task);
   
   app.route('/tasks/:taskId')
    .get(todoList.read_a_task)
    .put(todoList.update_a_task)
    .delete(todoList.delete_a_task);
    };

Let’s create a db connection wrapper, this will allow you to create connection on db which stored on a single file and can be reuse by other modules.

To do this create a new file name db.js under models folder
$ touch db.js
Paste the code snippet below:

'user strict';

var mysql = require('mysql');

//local mysql db connection
var connection = mysql.createConnection({
    host     : 'localhost',
    user     : 'root',
    password : '',
    database : 'mydb'
});

connection.connect(function(err) {
    if (err) throw err;
});

module.exports = connection;


Setting up model:

We can now reuse our db instance to other modules
Now create are Task object and provide function like creating new task, get all task, get task by id, update task by id and delete task by id.

Open appModel.js and paste code snippet below:

'user strict';
var sql = require('./db.js');

//Task object constructor
var Task = function(task){
    this.task = task.task;
    this.status = task.status;
    this.created_at = new Date();
};
Task.createTask = function createUser(newTask, result) {    
        sql.query("INSERT INTO tasks set ?", newTask, function (err, res) {
                
                if(err) {
                    console.log("error: ", err);
                    result(err, null);
                }
                else{
                    console.log(res.insertId);
                    result(null, res.insertId);
                }
            });           
};
Task.getTaskById = function createUser(taskId, result) {
        sql.query("Select task from tasks where id = ? ", taskId, function (err, res) {             
                if(err) {
                    console.log("error: ", err);
                    result(err, null);
                }
                else{
                    result(null, res);
              
                }
            });   
};
Task.getAllTask = function getAllTask(result) {
        sql.query("Select * from tasks", function (err, res) {

                if(err) {
                    console.log("error: ", err);
                    result(null, err);
                }
                else{
                  console.log('tasks : ', res);  

                 result(null, res);
                }
            });   
};
Task.updateById = function(id, task, result){
  sql.query("UPDATE tasks SET task = ? WHERE id = ?", [task.task, id], function (err, res) {
          if(err) {
              console.log("error: ", err);
                result(null, err);
             }
           else{   
             result(null, res);
                }
            }); 
};
Task.remove = function(id, result){
     sql.query("DELETE FROM tasks WHERE id = ?", [id], function (err, res) {

                if(err) {
                    console.log("error: ", err);
                    result(null, err);
                }
                else{
               
                 result(null, res);
                }
            }); 
};

module.exports= Task;

In here we require db.js to import our connection instance to mysql db and perform mysql queries.

Setting up the controller

Open appController.js file with your text edito and let’s deep dive into coding.
In this controller, we would be writing five(5) different functions namely: list_all_tasks, create_a_task, read_a_task, update_a_task, delete_a_task. We will export each of the functions for us to use in our routes.
Each of these functions uses different Task methods we created previously in appModel.js such as getTaskById, getAllTask, updateById, createTask and remove.

Open appController.js and paste code snippet below

'use strict';

var Task = require('../model/appModel.js');

exports.list_all_tasks = function(req, res) {
  Task.getAllTask(function(err, task) {

    console.log('controller')
    if (err)
      res.send(err);
      console.log('res', task);
    res.send(task);
  });
};

........

Earlier on, we had a minimal code for our server to be up and running in the server.js file.
In this section we will be connecting our handlers(controllers), database, the created models, body parser and the created routes together.
Open the server.js file created awhile ago and follow the following steps to put everything together.
Essentially, you will be replacing the code in your server.js with the code snippet from this section

use bodyParser Parse incoming request bodies in a middleware before your handlers, available under the req.body property.
It exposes various factories to create middlewares. All middlewares will populate the req.bodyproperty with the parsed body, or an empty object ({}) if there was no body to parse (or an error was returned).
Register our created routes in the server
On Server.js would have code below:

Start mySQL db sever
Start server.js
$ nodemon sever
–	This will start server and any changes made to the code will restart the server
Or

$ http://localhost:3000/tasks

$ {
"task":"create repo",
"status":"1"
}
