## BEFORE

```ts
/**
 * Clears all localStorage keys that start with the app's PREFIX.
 * Useful for logout or resetting app state.
 */
export const clearAppLocalStorageState = (): void => {
  if (typeof window === "undefined" || !window.localStorage) {
    return;
  }
  const keysToRemove: string[] = [];
  for (let i = 0; i < window.localStorage.length; i++) {
    const key = window.localStorage.key(i);
    if (key?.startsWith(PREFIX)) {
      keysToRemove.push(key);
    }
  }
  keysToRemove.forEach((key) => window.localStorage.removeItem(key));
};

```

## AFTER

```ts
// :: a -> boolean
const isRealString = (val: unknown): val is string =>
  val !== null && val !== undefined;

/**
 * Clears all localStorage keys that start with the app's PREFIX.
 * :: Storage -> void
 */
export const clearAppLocalStorageState = (
  storage: Storage = window?.localStorage,
): void => {
  Maybe.fromNullable(storage).map((ls) =>
    pipe(
      range(0),
      map((i) => ls.key(i)),
      filter(isRealString),
      filter(startsWith(PREFIX)),
      forEach((key: string) => ls.removeItem(key)),
    )(ls.length),
  );
};
```

## EXPLANATION

1. Imperative Looping / Index-Based Iteration

    Pattern: for (let i = 0; i < ...; i++)

    Why it's an anti-pattern:

        Manages mutable state manually (let i, keysToRemove.push)

        Error-prone and harder to compose or reuse

        Breaks from declarative data processing style

2. Mutating Temporary State (keysToRemove.push)

    Pattern: Building up a mutable array as an intermediate step.

    Why it's an anti-pattern:

        Introduces side effects and makes code less declarative

        Makes reasoning and refactoring harder

3. Hardcoded dependency on browser API

    Pattern: window.localStorage is directly referenced from within the function

    Why it's questionable:

        Directly hardcoding dependencies makes unit testing harder and also ties
        the function to a runtime where window has to be present.
