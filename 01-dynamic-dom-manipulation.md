# 1. Dynamic DOM manipulation

Looking at our HTML, it looks like only the list should be dynamic, so we can keep the remaining markup as before, removing the example todos.
```html
<div id="root">
    <div class="todo-app">
      <h1 class="title">todos</h1>
      <div class="todos">
        <input class="add-todo" type="text" placeholder="Add new todo" />
        <ul class="todo-list">
          <!-- todos go here -->
        </ul>
      </div>
    </div>
</div>
```

Given that we're now constructing a dynamic list, we should first get a reference to the list container.
```js
const rootElement = document.querySelector('.todo-app');
const list = rootElement.querySelector('.todo-list');
```

Now let's create an `addTodo` function that adds a new entry to this list. We'll need to create a `li` element with two `span` elements inside it, as we had in the static markup.
```js
function addTodo(description) {
  // create todo `li` with `todo-item` class
  const todoElem = document.createElement('li');
  todoElem.setAttribute('class', 'todo-item');

  // create description `span` with description text
  const descriptionElem = document.createElement('span');
  descriptionElem.setAttribute('class', 'description');
  descriptionElem.innerText = description;

  // create remove `span`
  const removeElem = document.createElement('span');
  removeElem.setAttribute('class', 'remove');
  removeElem.innerText = '\u2716';

  // add DOM elements to list
  todoElem.appendChild(descriptionElem);
  todoElem.appendChild(removeElem);
  list.appendChild(todoElem);
}
```

We can call `addTodo`, just to make sure it's working.
```js
addTodo('Learn CSS');
addTodo('Learn HTML');
addTodo('Learn JavaScript');
```

But what we really want is to add new entries when we hit \<Enter\>, so we'll listen for `keypress` events on the `input` element and call `addTodo` from there.
```js
const input = rootElement.querySelector('.add-todo');

function handleKeyPress(event) {
  const value = event.target.value;
  const key = event.key;

  if (value !== '' && key === 'Enter') {
    addTodo(value);
    event.target.value = '';
  }
}

input.addEventListener('keypress', handleKeyPress);
```

Now, the only thing missing is the ability to toggle and remove todos. As you probably noticed when implementing `addTodo`, we have references to the `description` and `remove` elements, so we can add `click` event listeners to create callbacks for DOM manipulation.
```js
function toggleTodo(todoElem) {
  todoElem.classList.toggle('done');
}

function removeTodo(todoElem) {
  list.removeChild(todoElem);
}

function addTodo(description) {
  ...

  descriptionElem.addEventListener('click', event => toggleTodo(todoElem));

  removeElem.addEventListener('click', event => removeTodo(todoElem));
}
```

And it's done! A _Todo List_ application in less than 50 lines of JavaScript :tada:! The sad thing is, this is not a good way to build dynamic frontend applications. There are a couple of bad practices here that will make it very hard to evolve this app in the future:
- It's impossible understand the struture of the dynamic DOM at a glance. It's all obfuscated under those `document.createElement` and `element.setAttribute` calls.
- There is no separation between data, logic and UI:
  - It's impossible to know the current state of the application without iterating through DOM nodes.
  - Doing changes to the logic will always involve messing with the DOM.
  - What if we want to order items by sobre criteriadate?
  - What if we want to clear all the items?
  - What if we want to save/restore all the items to/from a backend server?

If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-01](https://stackblitz.com/edit/talkdesk-js-class-01?file=index.js). But don't look for too long, we'll do it better in the [next step](./02-unidirectional-data-flow.md).