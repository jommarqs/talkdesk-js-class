# 2. Unidirectional data flow

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

This means that we can how define operations over our data without having to deal with DOM nodes, and call `render` only when we need to rebuild the view. Note we now have a unique `id` for every todo, so they can be easily referenced in `toggleTodo` and `removeTodo` operations.

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
