# If/else to ternary expression

## BEFORE

```ts
/**
 * Converts a number of bytes to a human-readable kilobyte string.
 *
 * @param bytes - The number of bytes
 * @returns A string formatted as "123.45kB" or "123B" if less than 1kB
 */
export const bytesToKilobytes = (bytes: number): string => {
  const kilobytes = bytes / 1024;
  if (kilobytes < 1) {
    return `${bytes}B`;
  } else {
    return `${kilobytes.toFixed(2)}kB`;
  }
};
```

## AFTER

```ts
/**
 * Converts a number of bytes to a human-readable kilobyte string.
 *
 * @param bytes - The number of bytes
 * @returns A string formatted as "123.45kB" or "123B" if less than 1kB
 */
export const bytesToKilobytes = (bytes: number): string => {
  const kilobytes = bytes / 1024;
  return kilobytes < 1 ? `${bytes}B` : `${kilobytes.toFixed(2)}kB`;
};
```

## EXPLANATION

The refactoring simplifies the conditional branching using a ternary expression, 
making the function more concise and declarative. This reduces boilerplate 
(if/else) and improves readability for a straightforward conditional logic.
While behavior remains identical, the code now aligns better with functional 
programming principles by focusing on expressions over statements.
It also removes multiple and early return points which is an anti-pattern.
When one works towards a composition of small functions then only one return
point is possible.