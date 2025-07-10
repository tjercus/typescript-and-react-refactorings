# Big function refactored to Zoom Levels

## BEFORE

The following function handles user registration by parsing the input, validating it, 
transforming it, and saving it. However, all of these concerns are tangled together 
in one large function, making it hard to reason about, test, and maintain.
Other problems:
- it has multiple return points

```ts
const handleUserRegistration = async (input: unknown): Promise<Either<string, string>> => {
  try {
    const parsed = JSON.parse(String(input));

    if (!parsed.email || !parsed.name) {
      return Left("Missing required fields");
    }

    const email = parsed.email.trim().toLowerCase();
    const name = parsed.name.trim();

    if (!email.includes("@")) {
      return Left("Invalid email");
    }

    const user = {
      id: uuid(),
      email,
      name,
      registeredAt: new Date().toISOString(),
    };

    const saved = await saveToDb(user);
    return saved ? Right("User registered") : Left("Failed to save user");
  } catch {
    return Left("Unexpected error");
  }
};
```

## AFTER

### Zoom Level 1 – Top-Level Pipeline

```ts
export const handleUserRegistration = (input: unknown): EitherAsync<string, string> =>
  EitherAsync.fromEither(() => parseInput(input))
    .chain(validateUserInput)
    .map(toDomainUser)
    .chain(saveUser);

```

### Zoom Level 2 – Intermediate Steps

```ts
import { z } from "zod";
import { Either, Left, Right } from "purify-ts/Either";
import { EitherAsync } from "purify-ts/EitherAsync";

const RawUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});

type RawUser = z.infer<typeof RawUserSchema>;

const parseInput = (input: unknown): Either<string, unknown> =>
  Either.encase(() => JSON.parse(String(input)))
    .mapLeft(() => "Invalid JSON");

const validateUserInput = (raw: unknown): EitherAsync<string, RawUser> =>
  EitherAsync.fromEither(() =>
    RawUserSchema.safeParse(raw).success
      ? Right(RawUserSchema.parse(raw))
      : Left("Validation failed")
  );

const toDomainUser = ({ email, name }: RawUser): User => ({
  id: uuid(),
  email: sanitizeEmail(email),
  name: sanitizeName(name),
  registeredAt: new Date().toISOString(),
});

const saveUser = (user: User): EitherAsync<string, string> =>
  EitherAsync(async () => {
    const success = await saveToDb(user);
    return success ? Right("User registered") : Left("Failed to save user");
  });

```

### Zoom Level 3 – Pure Utility Functions

```ts
import { pipe, trim, toLower} from "ramda";

const sanitizeEmail = pipe(trim, toLower);
const sanitizeName = trim;
```

### Supporting types

```ts
type User = {
  id: string;
  email: string;
  name: string;
  registeredAt: string;
};

```

## EXPLANATION

The original function mixed concerns and side effects in a single blob, making it difficult to test, extend, or understand. By applying the **zoom level programming** pattern, we restructure the logic across three abstraction layers:

- **Zoom Level 1** shows the full flow as a composition of steps — easy to scan and understand.
- **Zoom Level 2** extracts domain logic into named functions. We use **Zod** for schema validation and type inference, and **EitherAsync** from `purify-ts` to sequence async steps with failure handling.
- **Zoom Level 3** consists of tiny, pure utilities composed with Ramda, making transformations reusable and side-effect free.

This approach enhances **clarity**, **modularity**, and **robustness**, and makes each level easier to test and reason about independently. Zoom level programming is not just aesthetic — it’s a scalable pattern for real-world functional codebases.