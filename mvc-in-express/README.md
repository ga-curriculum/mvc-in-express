# ![MVC in Express - tktk Microlesson Name](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to refactor their fruits routes to make use of a fruits controller completing the MVC structure for this application.

## Implementing MVC in our application

Before we dive into the practical steps, let's review what controllers do in the [MVC (Model-View-Controller)](https://developer.mozilla.org/en-US/docs/Glossary/MVC) design pattern.

In MVC:

- **Model**: Represents your data and business logic. It's directly connected to your database (in our case, the fruits data).
- **View**: These are the templates (like your EJS files) that present data to the user in a specific format.
- **Controller**: This is where we will focus now. Controllers are the intermediaries between Models and Views. They handle the user's requests, interact with Models to retrieve data, and decide which View to render.

In simpler terms, whenever someone visits a page on your application, it's the controller's job to figure out what the user is asking for, talk to the model to get that data, and then send it to the view to be displayed.

![tktk Hunter a visual of MVC here](https://developer.mozilla.org/en-US/docs/Glossary/MVC/model-view-controller-light-blue.png)

## Creating controllers

To adhere to the MVC pattern, we need to create a dedicated space for our controllers. Let's get to work!

### Step 1: Create a controllers directory

First, we'll set up a new directory to keep our controllers organized. This helps in separating our route handling logic from our main `server.js` file, making our code cleaner and easier to manage.

Open your terminal and navigate to the project's root directory. Then, create a new directory for called `controllers`.

```bash
mkdir controllers
cd controllers
```

### Step 2: Create the `fruitsController.js` file

Now, within the `controllers` directory, we'll create a file named `fruitsController.js`. This file will contain all the logic related to handling requests for anything to do with 'fruits' in your application.

Create the fruits controller:

```bash
touch fruitsController.js
```

### Step 3: Understanding `fruitsController.js`

First, at the top of `fruitsController.js`, we'll import the `Fruit` model just like we did in `server.js`:

```js
const Fruit = require("../models/fruit");
```

Once imported, you can use the `Fruit` model to perform CRUD operations in your controller functions.

In `fruitsController.js`, we'll be moving the logic that currently resides in your routes in the main server file (like showing all fruits, displaying a form to add a new fruit, etc.). Each of these actions will be defined as a function in `fruitsController.js`.

For example, if you have a route in your main server file that looks like this:

```js
app.get("/fruits", async (req, res) => {
  // logic to show all fruits
});
```

We will move this logic to `fruitsController.js` and encapsulate it within a new function like this:

```js
const index = async (req, res) => {
  // logic to show all fruits
};

module.exports = {
  index,
  // add each of your controller function names here
};
```

In Node.js, each file in a project is treated as a separate module. `module.exports` is an object that the current module returns when it is "required" in another module. When you assign a function or an object to `module.exports`, you're making it accessible to other modules.

In your `fruitsController.js` file, you will likely have multiple functions - each handling different routes (like index, show, create, etc.). To make all these functions accessible in other files (like your main server file), you need to add them to `module.exports`. Each property of this object is a function you wish to export. The property name will be how you access this function in other modules.

### Naming controller functions

RESTful naming conventions typically follow a pattern that aligns with CRUD operations (Create, Read, Update, Delete). When organizing a REST application into an MVC structure, we rely on the naming conventions common to this standard.

In REST, the HTTP method (GET, POST, PUT, DELETE) combined with the URL path gives a clear indication of the function's purpose.

- A GET request to `/fruits` suggests fetching and displaying all fruits. The `index` function is a standard name used in many frameworks and languages to represent the action of listing _all_ resources of a certain type.

- The term '`index`' in web development is commonly understood to represent a list or a collection of items. So, when a developer sees a function named index, they can quickly infer that this function is responsible for retrieving and displaying a list or collection of items— in our case, all the fruits.

You can confidently follow this pattern for all your routes in express:

| Action         | Description                                 | Route Example          | Controller Function Name |
| -------------- | ------------------------------------------- | ---------------------- | ------------------------ |
| Index          | Retrieves and displays a list of resources  | `GET /fruits`          | `index`                  |
| Show           | Displays a single resource                  | `GET /fruits/:id`      | `show`                   |
| New            | Returns a form for creating a new resource  | `GET /fruits/new`      | `new`                    |
| Create         | Processes form data, creates a new resource | `POST /fruits`         | `create`                 |
| Edit           | Returns a form for editing a resource       | `GET /fruits/:id/edit` | `edit`                   |
| Update         | Processes form data, updates a resource     | `PUT /fruits/:id`      | `update`                 |
| Destroy/Delete | Deletes a resource                          | `DELETE /fruits/:id`   | `destroy` or `delete`    |

## Moving route logic to controller functions

Let's start with the index route and move all route logic over to a new controller function:

```js
// controllers/fruitsController.js

const Fruit = require("../models/fruit");

const index = async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render("fruits/index.ejs", { fruits: foundFruits });
};

module.exports = {
  index,
};
```

## Import your controller functions in `server.js`

Now that we moved our route logic to our controller, we'll need to import this logic back into `server.js` and connect it with our route handlers.

In your `server.js` file, import the `fruitsController` just above your routes:

```js
const fruitsController = require("./controllers/fruitsController");
```

Now replace the inline route handler in your `server.js` file with the corresponding function from the `fruitsController`.

For example, your original route:

```js
app.get("/fruits", async (req, res) => {
  // logic to get all fruits
});
```

should be refactored to:

```js
app.get("/fruits", fruitsController.index);
```

We have replaced the anonymous callback function in the route handler with a named function being imported from the controller.

If you run your application, you should be able to visit `localhost:3000/fruits` and see your new controller function in action.

## 🎓 You Do: Move all routes to `fruitsController`

Take some time to move the logic from each route in `server.js` to a new function in `fruitController.js`. Then refactor each route handler in `server.js` to use a function imported from `fruitsController`. Test each function before moving on to the next to ensure that your application is still able to access each new route as you go.

## Conclusion

By the end of this process, you'll have cleanly separated the concerns of your application into models, views, and controllers. This will not only make your application more maintainable but also easier to extend and manage as it grows in complexity.

Your server file should now look something like this:

```js
//server.js

// all dependencies above

const fruitsController = require("./controllers/fruitsController");

app.get("/", fruitsController.home);
app.get("/fruits/new", fruitsController.newFruitForm);
app.post("/fruits", fruitsController.createFruit);
app.get("/fruits", fruitsController.index);
app.get("/fruits/:fruitId", fruitsController.showFruit);
app.delete("/fruits/:fruitId", fruitsController.deleteFruit);
app.get("/fruits/:fruitId/edit", fruitsController.editFruitForm);
app.put("/fruits/:fruitId", fruitsController.updateFruit);

app.listen(3000, () => {
  console.log("The express app is ready!");
});
```

Your controller file should look like this:

```js
// fruitsController.js
const Fruit = require("../models/fruit");

const home = async (req, res) => {
  res.render("index.ejs");
};

const new = (req, res) => {
  res.render("fruits/new.ejs");
};

const create = async (req, res) => {
  req.body.isReadyToEat = req.body.isReadyToEat === "on";
  await Fruit.create(req.body);
  res.redirect("/fruits");
};

// Display all fruits
const index = async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render('fruits/index.ejs', { fruits: foundFruits });
};

// ...continue for other routes

module.exports = {
  home,
  new,
  create,
  index,
  // export other functions
};
```
