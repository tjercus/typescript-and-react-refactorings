# if/then/else to pattern matching

## BEFORE

```ts
type Status = "idle" | "loading" | "success" | "error";

const getMessage = (status: Status): string => {
  if (status === "idle") {
    return "Waiting to start...";
  } else if (status === "loading") {
    return "Loading...";
  } else if (status === "success") {
    return "Success!";
  } else if (status === "error") {
    return "Something went wrong.";
  } else {
    return "Unknown status";
  }
}

```

## AFTER

```ts
import { match, P } from "ts-pattern";

type Status = "idle" | "loading" | "success" | "error";

const getMessage = (status: Status): string => {
  return match(status)
    .with("idle", () => "Waiting to start...")
    .with("loading", () => "Loading...")
    .with("success", () => "Success!")
    .with("error", () => "Something went wrong.")
    .exhaustive(); // ensures all cases are handled
}
```

## EXPLANATION

Problems:

    Easy to forget a case (no compiler help).

    No exhaustiveness checking.

    Verbose and harder to visually scan.

    Multiple return points

Benefits of refactoring:

    Exhaustiveness checking at compile time.

    Easy to extend with new cases (e.g. "retrying").

    Declarative structure improves readability.

    Composable for more complex matching (nested, guards, etc.).

    Single return point reinforces clarity, composability, and consistency, especially as complexity grows.