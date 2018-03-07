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

We need to build the list items dynamically, so we first need get a reference to the list container.
```js
const rootElement = document.getElementById('root');
const list = rootElement.querySelector('.todo-list');
```

Now let's create an `addTodo` function that adds a new entry to this list. Here we'll be creating `li` elements, each containing two `span` elements, just like we had in the static markup.
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

Just to make sure this is working, we can call `addTodo`.
```js
addTodo('Learn CSS');
addTodo('Learn HTML');
addTodo('Learn JavaScript');
```

But what we really want is `addTodo` to be called in respose to the \<Enter\> key. So we'll need to grab a reference to the `input` element, listen for `keypress` events and call `addTodo` from there.
```js
const input = rootElement.querySelector('.add-todo');

function handleKeyPress(event) {
  const value = event.target.value;
  const key = event.key;

  if (key === 'Enter' && value !== '') {
    addTodo(value);
    event.target.value = '';
  }
}

input.addEventListener('keypress', handleKeyPress);
```

Now, the only thing missing is the ability to toggle and remove todos. As you probably noticed when implementing `addTodo`, we have references to the `description` and `remove` elements, so we can also add `click` event listeners to those elements and perform some DOM manipulation from there.
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

And it's done! A _Todo List_ application in less than 50 lines of JavaScript :tada:! The sad part is, this is not a good way to build dynamic frontend applications. It's a reasonable approach for a fast prototype, but I made a couple of bad decisions that will make it very hard to maintain and evolve this app in the future:
- It's impossible understand the struture of the dynamic DOM at a glance. It's all obfuscated behind `document.createElement` and `element.setAttribute` calls.
- There is no separation between data, logic and UI:
  - It's impossible to know the current state of the application without iterating through DOM nodes.
  - Doing changes to the logic will always involve messing with the UI, and _vice-versa_.
  - What if we want to order items alphabetically?
  - What if we want to clear all the items?
  - What if we want to save/restore all the items to/from a backend server?

---

If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-01](https://stackblitz.com/edit/talkdesk-js-class-01?file=index.js). But don't look for too long, we'll do it better in the [next step](./02-unidirectional-data-flow.md).
