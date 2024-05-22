+++
title = "My Favorite Little Operator"
date = 2024-05-22
tags = ["programming", "php", "js", "dart"]
draft = false
+++

Sometimes, there are little things that solve so much that we take for
granted. There's also things that solve so much that aren't used as
often.

Recently, I've been relying on [??](https://en.wikipedia.org/wiki/Null_coalescing_operator), which is a very
interesting operator called "null-coalescing operator". A fairly
seasoned programmer probably uses it very often and wouldn't find this
as fascinating as I did, but for me, it's a game-changer.

Take this for instance:

```js
const somevar = null;
if (somevar === null) {
    console.log("non-null value");
} else {
    console.log(somevar);
}
```

Here, we print out `somevar`, and if it's null, we print a string. You
can also use a ternary operator to clean it up and rewrite it like
this.

```js
const somevar = null;
console.log(somevar === null ? "non-null value" : somevar);
```

But the `??` is much cleaner:

```js
const somevar = null;
console.log(somevar ?? "non-null value");
```

Pretty neat, eh? Notice how `null` is not the same as `undefined` in JS,
Dart, C#, etc. So, if I do this in JavaScript

```js
console.log(variablethatdoesntexist ?? "value that does exist");
```

I get an error.

PHP, as terrible as it may be, has the cool little `isset()` function, and `??` can be used, not only to check for nullity,
but to also see if a value is defined:

```php
print_r($variablethatdoesntexist ?? "value that does exist");
```

This is perfectly valid PHP code with readability. Whether it's good
practice or not is a different matter of discussion...
