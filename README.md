todo-txt-js
===========

About
-----

This is a [todo.txt](https://github.com/ginatrapani/todo.txt-cli/wiki/The-Todo.txt-Format) parser written in pure JavaScript.
The source has no dependencies, though there are a few for the unit tests.

Parsing
-------

```
// Let's create a todo list.
var todoList = '+music Write a new song @guitar @home\n' + 
  '(B) 2014-04-10 ride 25 miles @bicycle +stayhealthy\n' + 
  '\n' +                                                    // NOTE: empty lines are ignored by the parser.
  'x 2014-03-02 buy milk @grocerystore\n' + 
  '(A) FILE TAXES! due:2014-04-15 for:me for:wife';

// Read the list into an array of todo items.
var items = TodoTxt.parseFile('Task1\n@work Task2\n(A) 2014-01-01 prioritized Task3 +someproject');

// Access properties on the new items.
console.log(items.length); // ==> 4

console.log(items[0].text);           // ==> 'Write a new song'
console.log(items[0].contexts);       // ==> ['@guitar', '@home']
console.log(items[0].projects);       // ==> ['+music']
console.log(items[0].priority);       // ==> null
console.log(items[0].createdDate);    // ==> null
console.log(items[0].complete);       // ==> false
console.log(items[0].completedDate);  // ==> null
console.log(items[0].addons);         // ==> {}

console.log(items[1].text);           // ==> 'ride 25 milds'
console.log(items[1].contexts);       // ==> ['@bicycle']
console.log(items[1].projects);       // ==> ['+stayhealthy']
console.log(items[1].priority);       // ==> B
console.log(items[1].createdDate);    // ==> Date object (April 10, 2014)
console.log(items[1].complete);       // ==> false
console.log(items[1].completedDate);  // ==> null
console.log(items[1].addons);         // ==> {}

console.log(items[2].text);           // ==> 'Buy milk'
console.log(items[2].contexts);       // ==> ['@grocerystore']
console.log(items[2].projects);       // ==> []
console.log(items[2].priority);       // ==> null
console.log(items[2].createdDate);    // ==> null
console.log(items[2].complete);       // ==> true
console.log(items[2].completedDate);  // ==> Date object (March 2, 2014)
console.log(items[2].addons);         // ==> {}

console.log(items[3].text);           // ==> 'FILE TAXES!'
console.log(items[3].contexts);       // ==> []
console.log(items[3].projects);       // ==> []
console.log(items[3].priority);       // ==> 'A'
console.log(items[3].createdDate);    // ==> null
console.log(items[3].complete);       // ==> false
console.log(items[3].completedDate);  // ==> null
console.log(items[3].addons);         // ==> {due: '2014-04-15', for: ['me','wife']} 
```

As you can see, add-ons are given a bit of special treatment. If an add-on key (the portion preceding the colon) appears 
only once in the item, the value is treated as a simple string. However, if the key appears more than once, 
then the parsed value will be an array containing each of the values in the string. Hence "for:me for:wife" becomes `{ for: ['me','wife'] }`.

Rendering
---------

Individual items can be rendered back into strings, as can the entire list. However, be aware that:

* The order of tokens in each item may be rearranged
* In a document, blank lines will be removed, but otherwise the order of tasks will be preserved.


```
var item = TodoTxt.parseLine();       // an empty parseLine acts like a constructor and gives you a blank object.
item.priority = 'X';
item.createdDate = new Date();
item.text = 'Dance!';
item.contexts.push('@work');
item.projects.push('+confuseMyCoworkers');
console.log(item.render());           // ==> (X) 2014-08-21 Dance! +confuseMyCoworkers @work

var list = TodoTxt.parseFile('(A) @work +dinosaur Open the door\n' +
  '(B) @work +dinosaur Get on the floor\n' +
  '(C) @work +dinosaur Everybody walk the dinosaur');


console.log(list.render());
// (A) Open the door +dinosaur @work
// (B) Get on the floor +dinosaur @work
// (C) Everybody walk the dinosaur +dinosaur @work

```


