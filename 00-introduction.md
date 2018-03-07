# 0. Introduction

In this class we're building a _Todo List_ app. We're not going to bother much about markup and styling, but in the next few minutes I'll try to help you develop a sense of how we want our final app to behave.

So, we want a _Todo_ application.
```html
<div id="root">
	<div class="todo-app">
	</div>
</div>
```
```css
* {
  box-sizing: border-box;
}

body {
  font-family: 'Helvetica Neue', Helvetica, sans-serif;
  background-color: #eee;
}

.todo-app {
  margin: auto;
  width: 80%;
  min-width: 320px;
  max-width: 640px;
}
```

We can call it _"todos"_.
```html
<div id="root">
	<div class="todo-app">
		<h1 class="title">todos</h1>
	</div>
</div>
````
```css
.title {
  text-align: center;
  font-weight: 300;
  font-size: 4em;
  margin: 0;
  color: #00ADB5;
}
```

We need some way to add new todos, so we can use a text field.
```html
<div id="root">
	<div class="todo-app">
		<h1 class="title">todos</h1>
		<div class="todos">
			<input class="add-todo" type="text" placeholder="Add new todo" />
		</div>
	</div>
</div>
```
```css
.todos {
  box-shadow: 0px 10px 20px 0px rgba(0,0,0,0.5);
}

.add-todo {
  width: 100%;
  padding: 10px 20px;
  font-size: 1em;
  font-weight: 100;
  border: 1px solid #eee;
}
```

And we also have to show all those tasks in a list.
```html
<div id="root">
	<div class="todo-app">
		<h1 class="title">todos</h1>
		<div class="todos">
			<input class="add-todo" type="text" placeholder="Add new todo" />
			<ul class="todo-list">
				<li class="todo-item">
					<span class="description">Learn JS</span>
				</li>
				<li class="todo-item">
					<span class="description">Learn HTML</span>
				</li>
				<li class="todo-item">
					<span class="description">Learn CSS</span>
				</li>
			</ul>
		</div>
	</div>
</div>
```
```css
.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
  cursor: default;
}

.todo-item {
  background-color: #222831;
  color: white;
  border-top: 1px solid #393E46;
  font-size: 1em;
  display: flex;
}

.todo-item:hover {
  background-color: #393E46;
}

.todo-item .description {
  padding: 10px 20px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex-grow: 1;
}
```

Also, we'll need to be able to mark the tasks as complete.
```html
<div id="root">
	<div class="todo-app">
		<h1 class="title">todos</h1>
		<div class="todos">
			<input class="add-todo" type="text" placeholder="Add new todo" />
			<ul class="todo-list">
				<li class="todo-item">
					<span class="description">Learn JS</span>
				</li>
				<li class="todo-item done">
					<span class="description">Learn HTML</span>
				</li>
				<li class="todo-item done">
					<span class="description">Learn CSS</span>
				</li>
			</ul>
		</div>
	</div>
</div>
```
```css
.todo-item.done .description {
  color: #00ADB5;
  text-decoration: line-through;
}
```

Finally, we should also be able to remove any task from the list, so we'll add an _✖ button_.
```html
<div id="root">
	<div class="todo-app">
		<h1 class="title">todos</h1>
		<div class="todos">
			<input class="add-todo" type="text" placeholder="Add new todo" />
			<ul class="todo-list">
				<li class="todo-item">
					<span class="description">Learn JS</span>
					<span class="remove">&#x2716</span>
				</li>
				<li class="todo-item done">
					<span class="description">Learn HTML</span>
					<span class="remove">&#x2716</span>
				</li>
				<li class="todo-item done">
					<span class="description">Learn CSS</span>
					<span class="remove">&#x2716</span>
				</li>
			</ul>
		</div>
	</div>
</div>
```
```css
.todo-item .remove {
  visibility: hidden;
  padding: 10px 20px 10px 0;
}

.todo-item .remove:hover {
  color: #FFB6B9;
}

.todo-item:hover .remove {
  visibility: visible;
}
```
 
And this is how our application will look. There is already some interactivity with CSS alone, but we want this list to be dynamic:
 - A new task should be added when hitting \<Enter\>.
 - Clicking on a task should mark it as done.
 - Clicking on the remove symbol should remove the task.
 - We might even have time to save and load tasks from a server.

In the next steps we'll be constructing this dynamic application. We'll start in pure JavaScript, and successively build up concepts and tools; up to the point we know enough to do it in a framework like React.

It's not important if you don't pick up all the language syntax, features and nuances. Instead, I want you to focus on the rationale and motivation for each successive step or refactor.

---

If you want to explore a bit more, the final code for this step is available at [https://stackblitz.com/edit/talkdesk-js-class-00](https://stackblitz.com/edit/talkdesk-js-class-00?file=index.html). In the [next step](./01-dynamic-dom-manipulation.md) we're going to do some dynamic DOM manipulation.
