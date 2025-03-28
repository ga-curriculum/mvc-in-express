<h1>
  <span class="headline">MVC in Express</span>
  <span class="subhead">MVC in Express</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to restructure a basic MEN stack app into one that conforms to MVC architecture.

## Implementing MVC in our application

Before we dive into the practical steps, let's review what controllers do in the [MVC (Model-View-Controller)](https://developer.mozilla.org/en-US/docs/Glossary/MVC) design pattern.

In MVC, Controllers are the intermediaries between Models and Views. They handle user requests, interact with Models to retrieve or update data, and then decide which View to render.

In simpler terms: whenever someone visits a page on your application, the controller figures out what the user wants, talks to the model to get the necessary data, and then sends it to the view to display.

## Creating controllers

To follow the MVC pattern, we need to create a dedicated place for our controllers. Let's walk through the steps.

### Step 1: Create a `controllers` directory

This directory will hold all controller files and help us keep our project organized by separating route logic from the main `server.js` file.

```bash
mkdir controllers
cd controllers
```

### Step 2: Create the `fruits.js` file

This file will contain all the logic for handling requests related to fruit.

```bash
touch fruits.js
```

## Step 3: Add your first controller function

Inside `controllers/fruits.js`, we’ll first import our Fruit model:

```js
// controllers/fruits.js

const Fruit = require('../models/fruit');
```

Now let’s move some logic from `server.js` into a named function here. For example, this route:

```js
// server.js

app.get('/fruits', async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render('fruits/index.ejs', { fruits: foundFruits });
});
```

...becomes a named function in the controller file:

```js
// controllers/fruits.js

const index = async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render('fruits/index.ejs', { fruits: foundFruits });
};
```

To make this function available in `server.js`, we need to export it using `module.exports`.

> In Node.js, every file is treated as a module. When we use `module.exports`, we’re deciding what that module makes available to the rest of the app. We export functions so other files (like `server.js`) can import and use them.

At the bottom of your controller file, add:

```js
module.exports = {
  index,
  // add future controller functions here
};
```

## Naming controller functions

RESTful naming conventions follow a standard that aligns with CRUD operations (Create, Read, Update, Delete). These conventions help keep your code predictable and easier to read.

- `index` = show all items
- `show` = show one item
- `create` = create an item
- `update` = update an item
- `delete` = remove an item
- `showNewForm` = render a form to create a new item
- `edit` = render a form to edit an item

> 💡 Note: We use `showNewForm` instead of `new` because `new` is a reserved word in JavaScript and can’t be used as a variable or function name.

| Action     | Description                                 | Route Example          | Controller Function Name |
| ---------- | ------------------------------------------- | ---------------------- | ------------------------ |
| Index      | Retrieves and displays a list of resources  | `GET /fruits`          | `index`                  |
| Show       | Displays a single resource                  | `GET /fruits/:id`      | `show`                   |
| New (Form) | Returns a form to create a new resource     | `GET /fruits/new`      | `showNewForm`            |
| Create     | Processes form data, creates a new resource | `POST /fruits`         | `create`                 |
| Edit       | Returns a form to edit an existing resource | `GET /fruits/:id/edit` | `edit`                   |
| Update     | Processes form data, updates a resource     | `PUT /fruits/:id`      | `update`                 |
| Delete     | Deletes a resource                          | `DELETE /fruits/:id`   | `delete`                 |

## Moving route logic to controller functions

Start by moving the logic for the index route:

```js
// controllers/fruits.js

const Fruit = require('../models/fruit');

const index = async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render('fruits/index.ejs', { fruits: foundFruits });
};

module.exports = {
  index,
};
```

## Import controller functions in `server.js`

Back in `server.js`, import the controller functions at the top:

```js
const fruitsCtrl = require('./controllers/fruits');
```

Then, replace the inline route handler with the named function:

```js
// Before:
app.get('/fruits', async (req, res) => {
  // logic...
});

// After:
app.get('/fruits', fruitsCtrl.index);
```

This keeps your route file clean and focused on routing only.

## 🎓 You Do: Move all routes to `fruitsCtrl`

Take time to move each route’s logic from `server.js` into its own named function in `controllers/fruits.js`. Then refactor each route to use the imported function. Test frequently as you go to make sure each route still works.

## Conclusion

By separating your application into Models, Views, and Controllers, you make your codebase easier to manage and scale. You’re also following a widely used architectural pattern used across many frameworks and programming languages.

After completing the refactor, your server file might look like this:

```js
// server.js

const fruitsCtrl = require('./controllers/fruits');

app.get('/', fruitsCtrl.home);
app.get('/fruits/new', fruitsCtrl.showNewForm);
app.post('/fruits', fruitsCtrl.create);
app.get('/fruits', fruitsCtrl.index);
app.get('/fruits/:fruitId', fruitsCtrl.show);
app.delete('/fruits/:fruitId', fruitsCtrl.delete);
app.get('/fruits/:fruitId/edit', fruitsCtrl.edit);
app.put('/fruits/:fruitId', fruitsCtrl.update);

app.listen(3000, () => {
  console.log('The express app is ready!');
});
```

And your controller file might look like this:

```js
// controllers/fruits.js

const Fruit = require('../models/fruit');

const home = (req, res) => {
  res.render('index.ejs');
};

const showNewForm = (req, res) => {
  res.render('fruits/new.ejs');
};

const create = async (req, res) => {
  req.body.isReadyToEat = req.body.isReadyToEat === 'on';
  await Fruit.create(req.body);
  res.redirect('/fruits');
};

const index = async (req, res) => {
  const foundFruits = await Fruit.find();
  res.render('fruits/index.ejs', { fruits: foundFruits });
};

// Add the rest of your controller functions here

module.exports = {
  home,
  showNewForm,
  create,
  index,
  // export others: show, edit, update, delete
};
```

Great work!
