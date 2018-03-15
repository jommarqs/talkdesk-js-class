# 5. Porting to React

✨TODO ✨

---

## Exercises

### 1. Implement "load" and "save" using a [Firebase](https://firebase.google.com/) database.
  - Here's a public one [https://talkdesk-js-class.firebaseio.com/](https://talkdesk-js-class.firebaseio.com/):
  - Choose a unique name for your database location (as long as it ends in .json):
    - I picked `todos.json`, choose anything else, otherwise you'll be writing over my stuff.
  - Use it like this:
    ```js
    // load
    fetch('https://talkdesk-js-class.firebaseio.com/todos.json')
      .then(response => response.json())
      .then(data => data || {})
      .then(data => {
      /* do domething with your data */
        console.log(data);
      });

    // save
    fetch(
      'https://talkdesk-js-class.firebaseio.com/todos.json',
      { method: 'PUT', body: JSON.stringify(this.model) }
    );
  - Here a bit of markup which already has some CSS for it:
    ```html
    <div className="options">
      <span className="save-todos">🚀</span>
      <span className="load-todos">⬇</span>
    </div>
    ```

### 2. Implement filters for your todos.
    - Add "all", "todo" and "done" filters so you can show only a partial list of all todos:
        - "all" shows all the todo items.
        - "todo" shows items not completed.
        - "done" shows completed items.
  - Here a bit of markup which already has some CSS for it:
    ```html
    <div className="filters">
      <span>all</span> | <span>todo</span> | <span>done</span>
    </div>
    ```

---
