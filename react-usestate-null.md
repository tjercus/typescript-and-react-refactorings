# Refactoring `null` to `Maybe` in React State

## BEFORE

Using `null` directly in React state can introduce potential runtime errors and 
makes it harder to express the *absence* of a value in a functional and type-safe way.

```tsx
import React, { useState } from "react";

export const PersonSelector = () => {
  const [selectedRecordMaybe, setSelectedRecordMaybe] = useState<Person>(null);

  const selectPerson = (person: Person) => {
    setSelectedRecordMaybe(person);
  };

  return (
    <div>
      <button onClick={() => selectPerson({ id: 1, name: "Alice" })}>
        Select Alice
      </button>
      {selectedRecordMaybe && (
        <p>Selected: {selectedRecordMaybe.name}</p>
      )}
    </div>
  );
};
```

## AFTER

```tsx
import React, { useState } from "react";
import { Maybe } from "purify-ts";

export const PersonSelector = () => {
  const [selectedRecordMaybe, setSelectedRecordMaybe] = useState<Maybe<Person>>(
    Maybe.empty()
  );

  const selectPerson = (person: Person) => {
    setSelectedRecordMaybe(Maybe.of(person));
  };

  return (
    <div>
      <button onClick={() => selectPerson({ id: 1, name: "Alice" })}>
        Select Alice
      </button>
      {selectedRecordMaybe.map((p) => (
        <p key={p.id}>Selected: {p.name}</p>
      ))}
    </div>
  );
};
```

## EXPLANATION

The refactor brings several improvements:

    ✅ Type safety: useState<Person>(null) technically violates the type system 
    because null is not a Person. Using Maybe<Person> fixes that.

    🧠 Explicitness: Maybe makes it obvious that a value might be missing, 
    unlike null, which is often overlooked.

    🛡 Safer rendering: Using map on a Maybe ensures the JSX is only rendered when
     a Person is present, avoiding null dereferencing.

    🧪 Easier testing and logic composition: Maybe methods (map, chain, isNothing,
     etc.) make conditional logic more declarative and less error-prone.

Overall, using Maybe in state makes the component more robust and aligns better 
with functional programming principles.