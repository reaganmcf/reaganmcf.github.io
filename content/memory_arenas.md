+++
title="Memory Arenas - a Rust programmer's best friend"
description="As an intermediate Rust programmer, one data structure I find myself using time and time again in my Rust projects is the Memory. In other languages, this data structure doesn't have as much usefulness, but I have found it to be a real superstar in Rust."
date=2022-06-09

[taxonomies]
tags = ["rust", "memory-arenas", "intermediate"]
categories = ["programming"]
+++

As an intermediate Rust programmer, one data structure I find myself using time and time again in my Rust projects is the Memory Arena. In other languages, this data structure doesn't have as much usefulness, but I have found it to be a real superstar in my Rust projects.

## What is it?

A Memory Arena is a data structure that, in its most abstract sense, is just a large contiguous piece of memory. In Rust, this is often just a `Vec` of some type `T` where instead of exposing `&T` and `&mut T`, you return 'keys' (usually indices) to items in the `Vec`. 

The benefit of this might not be directly apparent and seems like it might just make our API a little more verbose. But to understand the usefulness of memory arenas, I'll walk through a problem in my programming language [nakaka](https://github.com/nakala-lang/nakala), and discuss how memory arenas were the perfect solution.

### The Idea

Often when you are writing an interpreter for a programming language, you're dealing with "scopes" which can be nested.

Scopes just provide a way to define new variables and look up the values for existing variables. Let's define some `Scope` and `Value` structs for this subset of nakala to paint a better picture:

```rs
#[derive(Clone)]
pub enum Value {
  Int(i64),
  Bool(bool),
  ...
}

struct Scope {
  values: HashMap<String, Value>
}

impl Scope {
  pub fn new() -> Self {
    Self {
      values: HashMap::default(),
    }
  }
}
```

Let's also define the `RuntimeError` type that we will use in `get`, `define`, and `assign`

```rs
pub enum RuntimeError {
  UndefinedVariable,
  VariableAlreadyExists
}
```

Lastly, let's define the `get`, `assign`, and `define` functions for our Scope

```rs
impl Scope {
  ...,
  
  pub fn get(&self, name: &String) -> Result<Value, RuntimeError> {
    if let Some(entry) = self.values.get(name) {
      return Ok(entry.clone());
    }

    Err(RuntimeError::UndefinedVariable)
  }
  
  pub fn assign(&self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Occupied(mut e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      Err(RuntimeError::UndefinedVariable)
    }
  }

  pub fn define(&mut self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Vacant(e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      Err(RuntimeError::VariableAlreadyExists)
    }
  }
}
```

Now we have some `Scope` that we can get, assign, and define variables inside - pretty neat! However, there is a core problem

### The Problem
Let's look at the following example:

```js
let x = 10;
if someCondition {
  x = 5;
}
```

Let's visualize what these scopes look like by the time it's about to evaluate `x = 5`;
```
  ┌─────────────────────┐
  │Scope 1              │
  │  - x = Val::Int(10) │
  └─────────────────────┘

  ┌─────────────────────┐
  │Scope 2              │
  └─────────────────────┘
```

As you can see Scope 2 does not know that `x` is a variable, and it will end up returning `RuntimeError::UndefinedVariable`. 

What we want is something like this:
```
  ┌────────────────────┐
  │Scope 1             │
  │  - x = Val::Int(10)│
  └─────────▲──────────┘
            │
            │ (enclosed by)
  ┌─────────┴──────────┐
  │Scope 2             │
  └────────────────────┘
```

With this new design, we would be able to recursively `get`, `assign` and `define` no matter how deeply the scope is nested.

To do this, we need a way to "enclose" the new scope with the current scope before interpreting the body of the if statement. Let's redo our `Scope` struct

```rs
struct Scope {
  values: HashMap<String, Value>,
  enclosing: Option<Box<&mut Scope>>
}

impl Struct { 
  pub fn new(enclosing: Option<Box<&mut Scope>>) -> Self {
    values: HashMap::default(),
    enclosing
  }

  pub fn get(&self, name: &String) -> Result<Value, RuntimeError> {
    if let Some(entry) = self.values.get(name) {
      return Ok(entry.clone());
    }

    // Recursively lookup in enclosing scope
    if let Some(enclosing) = self.enclosing {
      enclosing.get(name)
    } else {
      Err(RuntimeError::UndefinedVariable)
    }
  }
  
  pub fn assign(&self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Occupied(mut e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      // Recursively lookup in enclosing scope
      if let Some(enclosing) = self.enclosing {
        enclosing.assign(name, val)
      } else {
        Err(RuntimeError::UndefinedVariable)
      }
    }
  }

  pub fn define(&mut self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Vacant(e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      // Recursively lookup in enclosing scope
      if let Some(enclosing) = self.enclosing {
        enclosing.define(name, val)
      } else {
        Err(RuntimeError::VariableAlreadyExists)
      }
    }
  }
}
```

Now we have borrow checker errors everywhere because we have to annotate the lifetime of the `Box<&mut Scope>` inside the `Struct` definition.

```
   Compiling playground v0.0.1 (/playground)
error[E0425]: cannot find value `enclosing` in this scope
  --> src/main.rs:25:7
   |
25 |       enclosing
   |       ^^^^^^^^^ not found in this scope

error[E0106]: missing lifetime specifier
  --> src/main.rs:18:25
   |
18 |   enclosing: Option<Box<&Scope>>
   |                         ^ expected named lifetime parameter
   |
help: consider introducing a named lifetime parameter
   |
16 ~ struct Scope<'a> {
17 |   values: HashMap<String, Value>,
18 ~   enclosing: Option<Box<&'a Scope>>
```

A full disclaimer here - I am not a Rust expert by any means, but I have probably spent 5 hours alone trying to get this design to comply with the borrow checker and failed. I also asked the Rust discord for help for over an hour and they could not help me. 

The main reason this is so hard is that you have to **prove that the enclosing scope _always_ lives longer than the scope itself**, which is hard to encode in lifetimes. At least for a mere mortal like me :).

But don't fret - we have memory arenas!

### The Solution

What we want is some data structure that allows us to reference a scope from wherever, and do "stuff" with that scope later on. We can use a memory arena!

Instead of having direct mutable references to the scopes, we will instead hand out `ScopeId`s which will index into our memory arena of scopes. This way the arena will "own" the memory for the duration of the program, and we can indirectly hand out more than one mutable reference to the same memory location.

In our interpreter, we will be passing around a new data structure, our `Environment` which will abstract away the memory arena implementation for the most part.

Let's start with our new `Scope` and `ScopeId` definitions:

```rs
type ScopeId = usize;

struct Scope {
  pub id: ScopeId,
  values: HashMap<String, Value>,
  pub enclosing: Option<ScopeId>
}
```

We now reference a Scope via an unsized integer, or `ScopeId`.

Our `get`, `assign` and `define` functions are back to what they were in our first iteration:

```rs
impl Scope {
  pub fn new(id: ScopeId, enclosing: Option<ScopeId>) -> Self {
    Self {
      id,
      values: HashMap::default(),
      enclosing
    }
  }
  
  pub fn get(&self, name: &String) -> Result<Value, RuntimeError> {
    if let Some(entry) = self.values.get(name) {
      return Ok(entry.clone());
    }

    Err(RuntimeError::UndefinedVariable)
  }
  
  pub fn assign(&mut self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Occupied(mut e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      Err(RuntimeError::UndefinedVariable)
    }
  }

  pub fn define(&mut self, name: String, val: Value) -> Result<(), RuntimeError> {
    if let Entry::Vacant(e) = self.values.entry(name) {
      e.insert(val);
      Ok(())
    } else {
      Err(RuntimeError::VariableAlreadyExists)
    }
  }
}
```

Now let's define our `Environment`

```rs
pub struct Environment {
  scopes: Vec<Scope>, // our scope memory arena!
  next_scope_id: ScopeId
}

impl Environment {
  pub fn new() -> Self {
    Self {
      scopes: vec![Scope::new(0, None)], // Global scope
      next_scope_id: 1
    }
  }
}
```

Our interpreter will want a way to be able to begin and end scopes as it walks our AST, so let's add those functions to our API.

```rs
impl Environment {
  ...,

  pub fn begin_scope(&mut self, enclosing: ScopeId) -> ScopeId {
    let id = self.next_scope_id;
    self.next_scope_id += 1;
    
    self.scopes.push(Scope::new(id, Some(enclosing)));

    id
  }
}
```

Notice how `begin_scope` returns a `ScopeId`. Instead of returning a `&mut Scope`, it returns this Id which it will then use to perform `get`, `assign` and `define`, like so:

```rs
impl Environment {
  ...,

  pub fn get(
    &self, 
    scope_id: ScopeId, 
    name: &String
  ) -> Result<Value, RuntimeError> {
    // SAFETY: Should never have an invalid scope_id
    let scope = self.scopes.get(scope_id).unwrap();

    match scope.get(name) {
      Ok(v) => Ok(v),
      Err(e) => {
        // recursively call enclosing scope
        if let Some(enclosing_id) = scope.enclosing {
          self.get(enclosing_id, name)
        } else {
          Err(e)
        }
      }
    }
  }

  pub fn assign(
    &mut self, 
    scope_id: ScopeId, 
    name: String, 
    val: Value
  ) -> Result<(), RuntimeError> {
    // SAFETY: Should never have an invalid scope_id
    let scope = self.scopes.get_mut(scope_id).unwrap();

    match scope.assign(name.clone(), val.clone()) {
      Err(e) => {
        if let Some(enclosing_id) = scope.enclosing {
          self.assign(enclosing_id, name, val)
        } else {
          Err(e)
        }
      }
      _ => Ok(())
    }
  }

  pub fn define(
    &mut self, 
    scope_id: ScopeId, 
    name: String, 
    val: Value
  ) -> Result<(), RuntimeError> {
    // SAFETY: Should never have an invalid scope_id
    let scope = self.scopes.get_mut(scope_id).unwrap();

    match scope.define(name.clone(), val.clone()) {
      Err(e) => {
        if let Some(enclosing_id) = scope.enclosing {
          self.assign(enclosing_id, name, val)
        } else {
          Err(e)
        }
      }
      _ => Ok(())
    }
  }
}
```

And that's it! This is the API that [nakala](https://github.com/nakala-lang/nakala) uses for its scoping. This API is powerful enough to handle first-class functions, closures, and `this` binding for functions. Even if can get the lifetime annotations to work, I doubt you will be able to get all those features with lifetime annotations.

## Conclusion

Memory Arenas are a simple yet incredibly powerful tool every intermediate to advanced Rust programmer should have in their back pocket. Lots of open source projects use this throughout the codebase for when you need multiple things to have mutable access to the same location in memory (not concurrently!).

The astute readers might have noticed there is no `delete_scope` function in the `Environment` impl block. Since our keys to the memory arena are indices, we can't resize or remove dead scopes without invalidating every other key. The solution involves using some sort of generational vector data structure, which I will address in part 2.
