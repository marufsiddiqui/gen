# Chapter VII: Delegation and recursion

{{ toc }}

## Revisiting our tree traversal

In a previous chapter, we outfitted our binary search tree with generator-based iterability. Venturing a bit further down that rabbit hole, we note that we used [breadth-first iteration](https://en.wikipedia.org/wiki/Breadth-first_search), meaning that we visited every value at the child level before moving to the grandchild level, etc.

```js
// a breadth-first tree traversal algorithm
var queue = this.root ? [this.root] : [];
while (queue.length > 0) {
  var node = queue.shift();
  yield node.value;
  if (node.left) { queue.push(node.left); }
  if (node.right) { queue.push(node.right); }
}
```

This works, but one of the cool things about BSTs is that they keep their values in sorted order. To take advantage of that, we need to switch to [in-order iteration](https://en.wikipedia.org/wiki/Tree_traversal#In-order). An easy way to do that is to use recursion, which means our generator will have to call itself.

## Tree structure

The way our tree works is that we have a `Tree` class and an inner `Node` class. Instances of `Tree` have a root which is null if the tree is empty. Otherwise, it's an instance of `Node`, which in turn can have left and right children, themselves instances of `Node`, and so on.

```
           tree
            |
          node:9
         /      \
    node:4      node:11
    /   \        /    \
null   node:6  null   null
```

## Adding recursion to a generator: attempt one

We'll make both `Tree` and `Node` iterable, and have `Tree` delegate to its root `Node`, if it exists. Then the root will recursively delegate to its left and right children, if they exist, etc.

```js
class Tree {
  // ...
  *[Symbol.iterator]() {
    if (this.root) {
      // delegate to root
      for (const val of this.root) {
        yield val;
      }
    }
  }
}

class Node {
  // ...
  *[Symbol.iterator]() {
    if (this.left) {
      // delegate to this.left
      for (const val of this.left) {
        yield val;
      }
    }
    yield this.value;
    if (this.right) {
      // delegate to this.right
      for (const val of this.right) {
        yield val;
      }
    }
  }
}
```

This works, but we've written too much code! It turns out that generators have a much easier way to do delegation.

## Generator delegation

Generators have a special syntax to delegate to any iterable. Whenever we `yield* obj`, where `obj` is iterable, we inline that sequence into the sequence we're currently generating. Read `yield* foo` as *yield all the things in foo*. Even with arbitrarily-deeply nested delegation, the consumer always sees a flat stream.

```js
function* a() {
  yield 1;
  yield* arr;
  yield 2;
}

var arr = [ 3, 4 ];

for (let n of a()) {
  console.log(n);
}
// => 1
// => 3
// => 4
// => 2
```

## Adding recursion to a generator: attempt two

Going back and applying this to our tree, we get this:

```js
class Tree {
  // ...
  *[Symbol.iterator]() {
    if (this.root) yield* this.root;
  }
}

class Node {
  // ...
  *[Symbol.iterator]() {
    if (this.left) yield* this.left;
    yield this.value;
    if (this.right) yield* this.right;
  }
}
```

And voila! We have an in-order iteration over every value in the tree. There's also a gist containing a fuller example of an [iterable binary search tree](https://gist.github.com/greim/17ccec50e8ac45a35edf08efec5fe059).

----------------

{{ next }}

----------------

{{ toc }}
