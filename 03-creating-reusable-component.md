# 3. Creating a Reusable Component

What if we want to have multiple todo lists? Let's create a reusable component! Basically, all we have to do is to encapsulate everything in a class:
- Place `model` and `rootElement` in the constructor.
- Transform all function to instance methods, and change all references to `this.<ref>`.
- Call `render` at the end of the constructor.

```js
class TodoApp {
  constructor(rootElement) {
    this.model = {
      todos: [],
      idCounter: 0
    };
    this.rootElement = rootElement;
    this.render();
  }

  addTodo(description) {
    this.model.idCounter++;
    this.model.todos = this.model.todos.concat({
      description: description,
      done: false,
      id: this.model.idCounter
    });
  }

  toggleTodo(id) {
    this.model.todos = this.model.todos.map(todo => {
      if (todo.id !== id) {
        return todo;
      }
      return { id: todo.id, description: todo.description, done: !todo.done };
    });
  }

  removeTodo(id) {
    this.model.todos = this.model.todos.filter(todo => todo.id !== id);
  }

  handleInputChange(event) {
    const value = event.target.value;
    if (event.key === 'Enter' && value !== '') {
      this.addTodo(event.target.value);
      this.render();
    }
  }

  handleDescriptionClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.toggleTodo(id);
    this.render();
  }

  handleRemoveClick(event) {
    const todoElement = event.target.parentNode;
    const id = Number.parseInt(todoElement.dataset.key);
    this.removeTodo(id);
    this.render();
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

There is only a small edge case that we need to take care of. The methods we're passing to event handlers will be run outside the instance context. Meaning that during execution, the `this` keyword will not be app instance. To fix this, we need to explicitly `bind` these handlers to the app instance. We can do it in the constructor, before calling `render`.
```js
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
```

And we're done 🎊! Now we can use multiple instances of our _Todo App_, each with it's own data.
```html
<div id="root"></div>
<div id="root-2"></div>
<div id="root-3"></div>
```
```js
new TodoApp(document.getElementById('root'));
new TodoApp(document.getElementById('root-2'));
new TodoApp(document.getElementById('root-3'));
```

---

## Exercises
- Implement _Clear all todos_:
  - Add a "clear" button or icon.
  - On click, all the existing todos should be removed.
- Make the title of each todo app customizable:
  - So we can do something like `new TodoApp(root, 'JavaScript Todos')`.

---
If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-03](https://stackblitz.com/edit/talkdesk-js-class-03?file=index.js). In the [next step](./04-extracting-generic-bits.md) we'll do a some improvements and extract the generic bits.
