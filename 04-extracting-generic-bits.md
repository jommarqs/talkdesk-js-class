# 4. Extracting Generic Bits

It's a little annoying that every time we call a data manipulation method (_i.e._ `addTodo`, `toggleTodo` and `removeTodo`) we have to call `render`. This gives us that nice unidirectional data flow, but we have to do it ourselves, and we may forget to call it, and that may lead to bugs.

So, let's introduce an `updateModel` method, that we can call every time we need to update our data, instead of manipulating the `model` object directly.
```js
updateModel(updated) {
  Object.assign(this.model, updated);
  this.render();
}
```

Now we can call this method and skip the explicit calls to `render` in the event handlers.
```js
class TodoApp {
  constructor(rootElement) {
    this.model = {
      todos: [],
      idCounter: 0
    };
    this.rootElement = rootElement;

    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleDescriptionClick = this.handleDescriptionClick.bind(this);
    this.handleRemoveClick = this.handleRemoveClick.bind(this);

    this.render();
  }

  updateModel(updated) {
    Object.assign(this.model, updated);
    this.render();
  }

  addTodo(description) {
    const id = this.model.idCounter + 1;
    this.updateModel({
      idCounter: id,
      todos: this.model.todos.concat({
        description: description,
        done: false,
        id: this.model.idCounter
      })
    });
  }

  toggleTodo(id) {
    this.updateModel({
      todos: this.model.todos.map(todo => {
        if (todo.id !== id) {
          return todo;
        }
        return { id: todo.id, description: todo.description, done: !todo.done };
      })
    });
  }

  removeTodo(id) {
    this.updateModel({
      todos: this.model.todos.filter(todo => todo.id !== id)
    });
  }

  handleInputChange(event) {
    const value = event.target.value;
    if (event.key === 'Enter' && value !== '') {
      this.addTodo(event.target.value);
    }
  }

  handleDescriptionClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.toggleTodo(id);
  }

  handleRemoveClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.removeTodo(id);
  }

  render() {
    // re-create DOM from model data
    const todos = this.model.todos.map(todo => {
      const classes = todo.done ? 'todo-item done' : 'todo-item';
      return `
      <li class="${classes}" data-key="${todo.id}">
        <span class="description">${todo.description}</span>
        <span class="remove">&#x2716</span>
      </li>
    `
    }).join('');

    this.rootElement.innerHTML = `
      <div class="todo-app">
        <h1 class="title">todos</h1>
        <div class="todos">
          <input class="add-todo" type="text" placeholder="Add new todo" />
          <ul class="todo-list">
            ${todos}
          </ul>
        </div>
      </div>
  `;

    // add event listeners
    this.rootElement.querySelector('.add-todo')
      .addEventListener('keypress', this.handleInputChange);

    this.rootElement.querySelectorAll('.todo-item .description').forEach(element => {
      element.addEventListener('click', this.handleDescriptionClick);
    });

    this.rootElement.querySelectorAll('.todo-item .remove').forEach(element => {
      element.addEventListener('click', this.handleRemoveClick);
    });
  }
}
```

But we can digg a little further... Our next app may not be a _Todo List_, but we're probably going to follow the same pattern:
- We'll think about it in terms of data, events and views.
- We'll have a `model` object, a `render` method, an `updateModel` helper and a `rootElement`.
- We'll define event handlers.

So, if we start extracting these recurring bits... we might have just discovered a framework 🤔.

Let's start by extracting `model`, `render()` and `updateModel()` to a separate "framework" file.
```js
class BaseApp {
  constructor() {
    this.model = {};
  }

  render() {
      throw new Error('The `render` method must be implemented');
  }

  updateModel(updated) {
    Object.assign(this.model, updated);
    this.render();
  }
}

export { BaseApp };
```

We can also offload some of that `render` logic to the framework. Setting the `innerHTML` of our `rootElement` and adding event listeners "by hand" can get very verbose and bug-prone. It would be a little more elegant if we just returned a "markup string" from our `render` method and the framework took care of the rest. We just have to find a way to declare event listeners directly into that string, and that's really easy with data-attributes (_e.g._ `data-on-click`, `data-on-keypress`, ...).
```js
class BaseApp {
  constructor(rootElement) {
    this.model = {};
    this.rootElement = rootElement;
  }

  render() {
    throw new Error('The `render` method must be implemented');
  }

  addEventListeners() {
    const events = [
      { query: '[data-on-click]', data: 'onClick', event: 'click' },
      { query: '[data-on-keypress]', data: 'onKeypress', event: 'keypress' }
    ];

    events.forEach(options => {
      this.rootElement.querySelectorAll(options.query).forEach(elem => {
        const cbName = elem.dataset[options.data];
        const callback = this[cbName];
        if (!callback) {
          throw new Error(`\`${cbName}\` is not defined`);
        }
        elem.addEventListener(options.event, callback);
      });
    });
  }

  renderToDOM() {
    this.rootElement.innerHTML = this.render();
    this.addEventListeners();
  }

  updateModel(updated) {
    Object.assign(this.model, updated);
    this.renderToDOM();
  }
}

export default BaseApp;
```

And we can keep adding features to our framework:
- Bind event listeners automatically:
  ```js
  addEventListeners() {
    ...
    elem.addEventListener(options.event, callback.bind(this));
    ...
  }
  ```
- Accept a root CSS selector:
  ```js
  constructor(rootSelector) {
    this.model = {};
    this.rootElement = document.querySelector(rootSelector);
  }
  ```

Using this tiny framework, we can now build applications following the pattern we've exploring up to this point. Just by extending `BaseApp`, all the complicated details about DOM manipulation and event handling are hidden away and the application code can be simple, compact and expressive.
```js
import 'style.css';
import BaseApp from './framework';

class TodoApp extends BaseApp {
  constructor(rootSelector) {
    super(rootSelector);
    this.model = {
      todos: [],
      idCounter: 0
    };
  }

  addTodo(description) {
    const id = this.model.idCounter + 1;
    this.updateModel({
      idCounter: id,
      todos: this.model.todos.concat({
        description: description,
        done: false,
        id: this.model.idCounter
      })
    });
  }

  toggleTodo(id) {
    this.updateModel({
      todos: this.model.todos.map(todo => {
        if (todo.id !== id) {
          return todo;
        }
        return { id: todo.id, description: todo.description, done: !todo.done };
      })
    });
  }

  removeTodo(id) {
    this.updateModel({
      todos: this.model.todos.filter(todo => todo.id !== id)
    });
  }

  handleInputChange(event) {
    const value = event.target.value;
    if (event.key === 'Enter' && value !== '') {
      this.addTodo(event.target.value);
    }
  }

  handleDescriptionClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.toggleTodo(id);
  }

  handleRemoveClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.removeTodo(id);
  }

  render() {
    // re-create DOM from model data
    const todos = this.model.todos.map(todo => {
      const classes = todo.done ? 'todo-item done' : 'todo-item';
      return `
      <li class="${classes}" data-key="${todo.id}">
        <span class="description" data-on-click="handleDescriptionClick">
          ${todo.description}
        </span>
        <span class="remove" data-on-click="handleRemoveClick">
          &#x2716
        </span>
      </li>
    `
    }).join('');

    return `
      <div class="todo-app">
        <h1 class="title">todos</h1>
        <div class="todos">
          <input class="add-todo" type="text" placeholder="Add new todo"
                 data-on-keypress="handleInputChange" />
          <ul class="todo-list">
            ${todos}
          </ul>
        </div>
      </div>
  `;
  }
}

new TodoApp('#root').renderToDOM();
new TodoApp('#root-2').renderToDOM();
new TodoApp('#root-3').renderToDOM();
```

And we must stop here... You may not realize it, but you just learned now [React](https://reactjs.org/) works and why using a framework is a good idea.

---

## Exercises
- Implement "load" and "save" using a [Firebase](https://firebase.google.com/) database.
  - Here's a public one [https://talkdesk-js-class.firebaseio.com/](https://talkdesk-js-class.firebaseio.com/):
  - Choose a unique name for your database location (as long as it ends in .json):
    - I picked `todos.json`, choose anything else, otherwise you'll be writing over my stuff.
  - Use it like this:
    ```js
    // load
    fetch('https://talkdesk-js-class.firebaseio.com/todos.json')
      .then(response => response.json())
      .then(data => data || {});
      .then(data => this.updateModel(data));

    // save
    fetch(
      'https://talkdesk-js-class.firebaseio.com/todos.json',
      { method: 'PUT', body: JSON.stringify(this.model) }
    );
    ```
- You may have noticed that all our todo lists interact with the same database. Like you did to customize the title, also add a `dbId`, which will be used for a specific list.
  - _e.g._ `new TodoApp(root, 'JavaScript Todos', 'javascript')` will use `https://talkdesk-js-class.firebaseio.com/javascript.json`.

---
If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-04](https://stackblitz.com/edit/talkdesk-js-class-04?file=index.js).
In the [next step](./05-porting-to-react.md) we'll port our application to [React](https://reactjs.org/), using the concepts that we learned so far.
