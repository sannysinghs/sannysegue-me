---
title: 'Read this and you’ll never have to learn state in Compose again'
description: 'A practical guide to mutableStateOf, remember, rememberSaveable, and state hoisting in Jetpack Compose'
pubDatetime: 2026-05-01T09:40:59+0000
heroImage: /assets/img/2026/state-in-compose/state-in-compose-hero.webp
tags: ["android", "compose", "engineering", "state"]
---

## What is state in Compose?

State is just a value that changes over time and is read by your UI. Compose is declarative: instead of telling the UI how to change, you describe the UI for each state, and Compose handles the rest.

A composable function is a node in the composition tree. When state changes, Compose may re-run the relevant composables and update the UI automatically.

## The composable lifecycle

Every composable can go through three phases:

1. Enter composition
2. Recomposition
3. Exit compossition

The important part is recomposition: when state changes, Compose re-executes the composable functions that depend on that state.

## Stateless vs stateful composables

### Stateless composable

```kotlin
@Composable
fun AddToCart() {
    var count = 0

    Column {
        println("Column composed!")
        Text("Counter $count")

        Button(onClick = { count++ }) {
            Text("Click me")
        }
    }
}
```

This looks like the composable function has state, but it doesn’t. `count` is a plain local variable, and clicking the `Click me` button will not recompose this function.

That means when the button is clicked, the `count` value remains unchanged.

### Stateful composable

```kotlin
@Composable
fun AddToCart() {
    var count by mutableStateOf(0)

    Column {
        println("Column composed!")
        Text("Counter $count")

        Button(onClick = { count++ }) {
            Text("Click me")
        }
    }
}
```

Now let's make the `count` variable observable by using `mutableStateOf`. Compose now knows that when `count` changes, it should recompose the UI that reads it. In the log, you’ll see `Column composed!` on every button click.

But there’s still one catch.

## Remember or forget?

By definition, recomposition re-executes the composable function, so `count` still gets recreated on every recomposition.

What we want is for the `AddToCart` composable to remember the previous `count` value, mutate it on button click, and display the new value.

Without `remember`, `count` is recreated on every recomposition.

```kotlin
@Composable
fun AddToCart() {
    var count by remember { mutableStateOf(0) }

    Column {
        println("Column composed!")
        Text("Counter $count")

        Button(onClick = { count++ }) {
            Text("Click me")
        }
    }
}
```

`remember` tells Compose to keep this state object around across recompositions. That means `count` survives recomposition, and your counter actually increments instead of resetting.

### Want to survive rotation too?

Use `rememberSaveable`:

```kotlin
@Composable
fun AddToCart() {
    var count by rememberSaveable { mutableStateOf(0) }
    // ...
}
```

`rememberSaveable` keeps the value across recompositions and configuration changes, such as screen rotation.

## State hoisting

State hoisting is the pattern where the parent owns the state and passes it down to children as parameters.

That makes the child composable reusable, testable, and easier to reason about.

```kotlin
@Composable
fun AddToCart() {
    var count by remember { mutableStateOf(0) }

    AddToCartButton(count) { count++ }
}
```

```kotlin
@Composable
fun AddToCartButton(
    count: Int,
    onAdd: () -> Unit
) {
    Column {
        println("Column composed!")
        Text("Counter $count")

        Button(onClick = onAdd) {
            Text("Click me")
        }
    }
}
```

The parent `AddToCart` owns the state. The child `AddToCartButton` just displays it and reports events.

## Why this matters

- Plain local variables inside a composable are not state.
- `mutableStateOf` makes a value observable.
- `remember` keeps state alive across recomposition.
- `rememberSaveable` keeps it across rotation too.
- Hoist state whenever you can, so your composables stay reusable.

Once these pieces click, Compose state stops being mysterious and starts feeling simple.
