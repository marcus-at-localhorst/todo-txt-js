todo-txt-js
===========

About
-----

This is a [todo.txt](https://github.com/ginatrapani/todo.txt-cli/wiki/The-Todo.txt-Format) parser written in pure JavaScript.
The source has no dependencies, though there are a few for the unit tests.

`bower install todo-txt-js`

Usage
-----
`TodoTxt.parseFile(str)` treats a string as though it is a file containing many todo items separated by line breaks. It returns 
a specialized object with these properties:

* `.length` returns the number of items found in the list. Blank lines are ignored.
* `.items(query, sortFields)` returns an array of task objects, optionally filtered by a query and sorted (see below).
* `.render(query, sortFields)` converts the object back into a todo list, optionally using a query and sorted (see below). 
* `.removeItem(item)` removes an item from the list. `item` is expected to be an existing todo object, but any object that implements the `id()` function will do.
* `.addItem(item)` adds an item to the list. `item` can be either a valid todo item object or a string; if the latter, it will be parsed. If there is no created date on the item, the created date will be set to today.

`TodoTxt.parseLine(str)` treats a string as though it is a single task. It returns an object representation of the task with these properties:

* `id()`: a unique id for the task. 
* `isComplete()` : whether the task has been completed. Boolean, never null.
* `completedDate()` : The date of completion, if present. Date, may be null.
* `priority()` : The current priority. Single character between A-Z, may be null.
* `createdDate()` : The date the task was created, if present. Date, may be null.
* `contexts()` : A list of all the contexts (@home, @work, etc) in the task. Array of string, never null, may be empty.
* `projects()` : A list of all the projects (+health, +jobsearch, etc) in the task. Array of string, never null, may be empty.
* `addons()` : A list of all addons found within the project. Array of object, never null, may be empty. See "Add-Ons" below.
* `textTokens()` : A list of all words in the task that have not been categorized as anything else. Array of string, never null, may be empty (but what's the point of that?).
* `completeTask()` : marks the task completed and sets the completedDate to now.
* `uncompleteTask()` : marks the task incomplete and sets the completedDate to null.
* `setCreatedDate(dt)`: sets the created date to the passed in date. If a date is not passed in, then today's date
is used.
* `render()` : Converts the object back into a string. 


Parsing
-------
Both 'files' (tasks separated by line breaks) and 'lines' (individual tasks) can be parsed.

```
// Let's create a todo list.
var todoList = '+music Write a new song @guitar @home\n' + 
  '(B) 2014-04-10 ride 25 miles @bicycle +stayhealthy\n' + 
  '\n' +     // NOTE: empty lines are ignored by the parser.
  'x 2014-03-02 buy milk @grocerystore\n' + 
  '(A) FILE TAXES! due:2014-04-15 for:me for:wife';

// Read the list into an array of todo items.
var todos = TodoTxt.parseFile(todoList);

// Access properties on the new items.
console.log(todos.length);            // ==> 4

// Fetch items from the object
var items = todos.items();            

console.log(items[0].id());             // ==> (a UUID-like string)
console.log(items[0].contexts());       // ==> ['@guitar', '@home']
console.log(items[0].projects());       // ==> ['+music']
console.log(items[0].priority());       // ==> null
console.log(items[0].createdDate());    // ==> null
console.log(items[0].isComplete());     // ==> false
console.log(items[0].completedDate());  // ==> null
console.log(items[0].addons());         // ==> {}
console.log(items[0].textTokens());     // ==> ['Write', 'a', 'new', 'song']

console.log(items[1].contexts());       // ==> ['@bicycle']
console.log(items[1].projects());       // ==> ['+stayhealthy']
console.log(items[1].priority());       // ==> B
console.log(items[1].createdDate());    // ==> Date object (April 10, 2014)
console.log(items[1].isComplete());     // ==> false
console.log(items[1].completedDate());  // ==> null
console.log(items[1].addons());         // ==> {}

console.log(items[2].contexts());       // ==> ['@grocerystore']
console.log(items[2].projects());       // ==> []
console.log(items[2].priority());       // ==> null
console.log(items[2].createdDate());    // ==> null
console.log(items[2].isComplete());     // ==> true
console.log(items[2].completedDate());  // ==> Date object (March 2, 2014)
console.log(items[2].addons());         // ==> {}

console.log(items[3].contexts());       // ==> []
console.log(items[3].projects());       // ==> []
console.log(items[3].priority());       // ==> 'A'
console.log(items[3].createdDate());    // ==> null
console.log(items[3].isComplete());     // ==> false
console.log(items[3].completedDate());  // ==> null
console.log(items[3].addons());         // ==> {due: '2014-04-15', for: ['me','wife']} 
```

Add-Ons
-------
As you can see above, add-ons are given a bit of special treatment. If an add-on key (the portion preceding the colon) appears 
only once in the item, the value is treated as a simple string. However, if the key appears more than once, 
then the parsed value will be an array containing each of the values in the string. Hence "for:me for:wife" becomes `{ for: ['me','wife'] }`.

Querying
--------
The todo list `items()` and `render()` methods can be passed a query object to filter the output. The query object contains a subset of the properties of a todo item. The values of those properties may be:

* Simple values (string, date, boolean) for exact matching (e.g. `{ isCompleted: false }`)
* Arrays, for "contains" matching of array properties like contexts, projects, and textTokens. The values are treated as an AND search, so every value specified must be present in every item.
  * For example, `{ contexts: ['@home', '@work'] }` will only match items that have BOTH *@home* and *@work* in them.
* Functions, for custom comparison logic. The function will take one argument, which is the value of the property on each item. Return `true` if the item passes the custom test, `false` otherwise.
  * Example (with Underscore.js): `{contexts: function(contexts) { return _.contains(contexts, '@home') || _.contains(contexts, '@work'); }}` will match items that have EITHER *@home* OR *@work* contexts.

Sorting
-------
The second argument to `items()` and `render()` is an array of `sortFields`. A sortField is either an object that looks like `{field: FIELDNAME, direction: TodoTxt.SORT_DESC | TodoTxt.SORT_ASC}`, or just a string with the fieldName if you are doing an ascending sort.

Currently, you can only sort on the following fields: `isComplete`, `priority`, `completedDate`, `createdDate`.

This example (from the tests) shows off querying and sorting by multiple fields in multiple directions. First, all completed entries are removed by the query. Then the remaining items are sorted by 1) ascending priority, 2) descending creation date.

    it("sorts queried items with an array of sort objects", function() {
        var first = '(A) 2014-01-02 I am first +1';
        var second = '(A) 2014-01-01 I am second';
        var third = '(B) 2014-01-03 I am third';
        var fourth = 'I am fourth';
        var testFile = fourth + '\n' + second + '\nx 2014-03-03 (A) 2014-02-02 I am ignored.\n' + first + '\n' + third;
        var todos = TodoTxt.parseFile(testFile);
        expect(todos.length).toBe(5);
        var output = todos.render({ isComplete: false }, ['priority', { field: 'createdDate', direction: TodoTxt.SORT_DESC }]);
        expect(output).toBe([first, second, third, fourth].join('\n'));
    });

Rendering
---------

Individual items can be rendered back into strings, as can the entire list. By now you've notice that the properties on each item
are functions; this keeps the task largely immutable so rendering can return the original string without changing up the order
of tokens. When a list object is rendered, blank lines will be removed but the order of all tasks will be preserved.

```
var list = TodoTxt.parseFile('(A) @work +dinosaur Open the door\n' +
  '\n' +
  '(B) @work +dinosaur Get on the floor\n' +
  '(C) @work +dinosaur Everybody walk the dinosaur');


console.log(list.render());
// (A) @work +dinosaur Open the door
// (B) @work +dinosaur Get on the floor
// (C) @work +dinosaur Everybody walk the dinosaur

```



