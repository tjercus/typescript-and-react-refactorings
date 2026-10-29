# Nested Option calls to a flat pipe

To flatten functional, nested `Option` calls, use `pipe`.

## BEFORE

```ts
import { Option } from "effect"

const result = Option.getOrElse(
  Option.map(
    Option.map(
      Option.filter(notificationsOption, predicateFn),
      mapFn1
    ),
    mapFn2
  ),
  () => defaultValue
)
```

## AFTER

Using the `.pipe` method that every `Option` has:

```ts
import { Option } from "effect"

const result = notificationsOption.pipe(
  Option.filter(predicateFn),
  Option.map(mapFn1),
  Option.map(mapFn2),
  Option.getOrElse(() => defaultValue)
)
```

OR using the standalone `pipe` function:

```ts
import { pipe, Option } from "effect"

const result = pipe(
  notificationsOption,
  Option.filter(predicateFn),
  Option.map(mapFn1),
  Option.map(mapFn2),
  Option.getOrElse(() => defaultValue)
)
```

## EXPLANATION

The nested version has to be read inside-out: the first step (`Option.filter`)
is buried in the middle, and the last step (`Option.getOrElse`) is on the outside.
Every extra step adds another level of indentation, and the arguments of each
call (`mapFn1`, `mapFn2`, `() => defaultValue`) end up far away from the
function they belong to.

The piped version is better because:
- it reads top to bottom, in the same order the data flows: filter, map, map,
  fall back to a default.
- it is flat. Adding, removing or reordering a step is a one-line change instead
  of rebalancing parentheses and indentation.
- each step sits next to its own argument, so it is clear which function does what.
- `Option` functions in Effect are dual: called with only the function argument
  (`Option.map(mapFn1)`) they return a function that waits for the `Option`.
  That makes every step a small, reusable building block that composes naturally.
- the type of the value is inferred step by step, so a type error points at the
  exact step that is wrong instead of somewhere in a deeply nested expression.

Choose `.pipe` when you start from an `Option` value you already have.
Choose the standalone `pipe` when the starting value is not an Effect data type,
or when you prefer the same style everywhere.
