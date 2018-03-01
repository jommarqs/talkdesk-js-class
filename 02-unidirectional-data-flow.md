# 2. Unidirectional Data Flow

Instead of jumping straight to DOM manipulation, let's keep the data of our application in a separate object. For now we'll have some sample todos.

```js
const model = {
  todos: [
      { description: 'Learn HTML', done: true },
      { description: 'Learn CSS', done: true },
      { description: 'Learn JavaScript', done: false },
    ]
}
```

Using this data model, along with our sample markup from [step 0](./00-introduction.md) and some template string replacements, we can render the entire application by setting the `innerHTML` of our root node.
```html
<div id="root"></div>
```
```js
const rootElement = document.getElementById('root');

function render() {
  const todos = model.todos.map(todo => {
    const classes = todo.done ? 'todo-item done' : 'todo-item';
    return `
      <li class="${classes}">
        <span class="description">${todo.description}</span>
        <span class="remove">&#x2716</span>
      </li>
    `
  }).join('');
  
  rootElement.innerHTML = `
    <div id="root">
      <div class="todo-app">
        <h1 class="title">todos</h1>
        <div class="todos">
          <input class="add-todo" type="text" placeholder="Add new todo" />
          <ul class="todo-list">
            ${todos}
          </ul>
        </div>
      </div>
    </div>
  `;
}

render();
```

This means that we can now define operations over our data without having to deal with DOM nodes, and call `render` only when we need to rebuild the view. Note we now have a unique `id` for every todo, so they can be easily referenced in `toggleTodo` and `removeTodo` operations.

```js
const model = {
  todos: [],
  idCounter: 0
}

function addTodo(description) {
  model.idCounter++;
  model.todos = model.todos.concat({
    description: description,
    done: false,
    id: model.idCounter
  });
  return model.idCounter;
}

function toggleTodo(id) {
  model.todos = model.todos.map(todo => {
    if (todo.id !== id) {
      return todo;
    }
    return { id: todo.id, description: todo.description, done: !todo.done };
  });
}

function removeTodo(id) {
  model.todos = model.todos.filter(todo => todo.id !== id);
}
```
```js
const learnCSS = addTodo('Learn CSS');
const learnHTML = addTodo('Learn HTML');
const learnJS = addTodo('Learn JS');

toggleTodo(learnCSS);
toggleTodo(learnHTML);
render();
removeTodo(learnHTML);
render();
```

When calling `render` we get a brand new DOM tree, so we have to attach event listeners to handle the input submission and todo item clicks, as we did before.
A couple of things to note here:
- We're adding some extra information to `.todo-item` nodes using `data-*` attributes, so we can identify the corresponding todo ID when the elements are clicked.
- The `render` function is called again every time we do some data manipulation (_i.e._ when calling `addTodo`, `removeTodo` or `toggleTodo`).
```js
function handleInputChange(event) {
  const value = event.target.value;
  if (event.key === 'Enter' && value !== '') {
    addTodo(event.target.value);
    render();
  }
}

function handleDescriptionClick(event) {
  const todoElement = event.target.parentNode;
  // Convert `dataset.key` string to a number
  const id = Number.parseInt(todoElement.dataset.key);
  toggleTodo(id);
  render();
}

function handleRemoveClick(event) {
  const todoElement = event.target.parentNode;
  const id = Number.parseInt(todoElement.dataset.key);
  removeTodo(id);
  render();
}

function render() {
  // re-create DOM from model data
  rootElement.innerHTML = `
    ...
        <li class="${classes}" data-key="${todo.id}">
    ...
  `

  // add event listeners
  rootElement.querySelector('.add-todo')
    .addEventListener('keypress', handleInputChange);

  rootElement.querySelectorAll('.todo-item .description').forEach(element => {
    element.addEventListener('click', handleDescriptionClick);
  });

  rootElement.querySelectorAll('.todo-item .remove').forEach(element => {
    element.addEventListener('click', handleRemoveClick);
  });
}
```

And we're done. It's more code than the previous version, but we now have something that we can reason about:
- We can manipulate data, logic and view independently.
- It's easy to implement new features or change existing ones:
  - Sort todos?
    ```js
    function sortTodos() {
      model.todos.sort(...);
    }

    function handleSortButtonClick() {
      sortTodos();
      render();
    }

    function render() {
      rootElement.innerHTML = `
        ...
        <button class="sort">sort</button>
        ...
      `;

      rootElement.querySelector('.sort')
        .addEventListener('click', handleSortButtonClick);
    }
    ```
  - Load todos from server?
    ```js
    function loadTodos() {
      return api.fetchTodos().then(todos => {
        model.todos = todos;
      });
    }

    function handleDocumentReady() {
      loadTodos().then(render);
    }

    document.addEventListener('DOMContentLoaded', handleDocumentReady);
    ```

See the pattern?
- Interacting with the view triggers events.
- Event handlers manipulate data.
- Every time the data is changed, we also re-render the view.
- And the cycle continues...
```
            ┌────────────┐
      ╭─────┤   EVENTS   ◀─────╮
      │     └────────────┘     │
      │                        │
┌─────▼────╮              ┌────┴─────┐
|   DATA   |              |   VIEW   | 
└─────┬────┘              └────▲─────┘
      │                        │
      ╰────────────────────────╯
```

---

## Exercise
Implement _Clear all todos_:
- Add a "clear" button.
- When clicking the button, all the existing todos should be removed.

---
If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-02](https://stackblitz.com/edit/talkdesk-js-class-02?file=index.js). In the [next step](./03-creating-reusable-component.md) we'll create a reusable component out of this app.
