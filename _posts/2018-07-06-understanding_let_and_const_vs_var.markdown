---
layout: post
title:      "Understanding let & const vs. var"
date:       2018-07-06 18:23:13 +0000
permalink:  understanding_let_and_const_vs_var
---

In broad terms, const and let are reserved words for delcaring and assigning variables, just like var. However, there's a huge difference in how they behave. Let's start with var:

Consider what happens if we declare a variable twice:

```
var number = 2;
// => undefined

var number = 4;
// => undefined

number;
// => 4
```

Nothing, no error is thrown. This is not good! There is no reason to declare a variable twice. If it occurs, it is usually a mistake by the developer that forgot he already declared the variable. If so, it would be ideal for JS to throw an error. So already using var is prone to lead to bugs in your code. (In addition to other issues that might occur due to block-scoping).

However, if I declare a variable with let or const, and then try to declare it again:

```
let number = 2;
//=> undefined
 
let number = 4;
//=> Uncaught SyntaxError: Identifier 'number' has already been declared
```

Now, notice that the issue here lies in *declaring* the variable. But what if we want to *reassign* its value after it has been declared? 

This is where const and let differ. With let, the variable can still be *reassigned*:

```
let number = 2
//=> undefined
 
number = "some new value";
//=> "some new value"
```

Variables declared with const, on the other hand, cannot be reassigned. Additionally, at the moment of declaration, the value needs to be assigned immediately.  

```
let number;
//=> undefined

const number:
//=> Uncaught SyntaxError: Missing initializer in const declaration
```

Conclusion

Due to the bug-prone issues that come with var, it is best practice to always use const and never use var. However, if you are sure that the variable will be reassigned at a later point (for example if the variable will be used as a counter), then, of course, use let.
