# 4. Extracting Generic Bits

It's a little anoying that every time we call a data manipulation method (_i.e._ `addTodo`, `toggleTodo` and `removeTodo`) we have to call `render`. This gives us that nice unidirectional data flow, but we have to do it ourselves, and we may forget to call it, and that may lead to bugs.

So, let's introduce an `updateModel` method, that we can call every time we need to update our data, instead of manipulating the `model` object directly.
```js
updateModel(updated) {
  Object.assign(this.model, updated);
  this.render();
}
```

Now we can call this method and skip the explicit calls to `render` in the event handlers.
```js
class TodoApp extends App {
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

And we can digg a little further... Our next app may not be a _Todo List_, but we're probably going to follow the same pattern:
- Think about it in terms of data, events and views.
- Have a `model` object.
- Have a `render` method.
- Have an `updateModel` helper that enforces an _unidirectional data flow_.
- Have a `rootElement` in which `render` creates DOM.
- Have event handlers and bindings.

So, if we extract these common bits... we might have just discovered a framework 🤔.

```js
// framework code

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

```js
// application code

import { BaseApp } from 'framework';

class TodoApp extends BaseApp {
  constructor(rootElement) {
    super();
    this.model = { ... };
    ...
  }

  // no need to implement `updateModel` ourselves

  ...

  addTodo(description) {
    this.updateModel({ ... });
  }

  handleInputChange(event) {
    ...
    this.addTodo( ... );
    // no need to call `render` explicitly
  }

  ...

  render() { ... }
}
```

The framework code is pretty small at the moment, but it's a framework nonetheless. We can enhance it in several ways:
- Add lifecycle hooks that run before and after render.
- Implement optimizations to avoid rendering the DOM every time.
- Provide helpers to generate the view.
- Simplify all the boilerplate of event handlers and bindings.
- ...

And we have to stop here, because you may not realize it, but you just learned now [React](https://reactjs.org/) works.

---

## Exercises
- Implement _Clear all todos_:
  - Add a "clear" button or icon.
  - On click, all the existing todos should be removed.
- Make the title of each todo app customizable:
  - So we can do something like `new TodoApp(root, 'JavaScript Todos')`.
- Implement load and save using a [Firebase](https://firebase.google.com/) database.
  - Here's a public one https://talkdesk-js-class.firebaseio.com/
  - Pick a unique name for your database location (as long as it ends in `.json`):
    - I picked `todos.json`, choose anything else, otherwise you'll be rewriting my stuff.
  - Use it like this:
    ```js
    // load
    fetch('https://talkdesk-js-class.firebaseio.com/todos.json')
      .then(response => response.json())
      .then(todos => todos || []);

    // save
    fetch(
      'https://talkdesk-js-class.firebaseio.com/todos.json',
      { method: 'PUT', body: JSON.stringify(this.model) }
    );
    ```

---
If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-04](https://stackblitz.com/edit/talkdesk-js-class-04?file=index.js).
In the [next step](./05-porting-to-react.md) we'll port our application to [React](https://reactjs.org/), using the concepts that we learned so far.
