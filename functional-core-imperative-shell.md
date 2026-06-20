# Functional-core/imperative-shell

The Problem: Logic mixed with side effects

## BEFORE, EXAMPLE 1

```tsx
// shell/PremiumButton.tsx (tightly coupled)
import React from "react";
import { useNavigate } from "react-router-dom";

interface PremiumButtonProps {
  isLoggedIn: boolean;
  hasPremiumAccess: boolean;
}

export const PremiumButton = ({
  isLoggedIn,
  hasPremiumAccess,
}: PremiumButtonProps) => {
  const navigate = useNavigate();

  const handleButtonClick = () => {
    // Logic and side effects are mixed!
    if (!isLoggedIn) {
      navigate("/login"); // Side effect
      return; // early return anti-pattern
    }

    if (!hasPremiumAccess) {
      window.alert("You need a premium subscription to view this page."); // Side effect
      return; // multiple return anti-pattern
    }

    navigate("/premium/dashboard"); // Side effect
  };

  return (
    <button onClick={handleButtonClick} className="btn-premium">
      Access Premium Hub
    </button>
  );
};
```

**Problems:**
- Logic is tangled with side effects (hard to test)
- Multiple returns make flow unclear
- If you want to reuse this logic elsewhere, you must duplicate it
- Testing requires mocking React hooks

## AFTER, EXAMPLE 1

### functional core

```ts
// core/navigationCore.ts

export type NavigationCommand =
  | { type: "Noop" }
  | { type: "Redirect"; url: string }
  | { type: "Alert"; message: string };

export type UserState = {
  isLoggedIn: boolean;
  hasPremiumAccess: boolean;
};

/**
 * Pure function: Decide where the user goes using pattern matching.
 * Single expression, no multiple returns, extremely easy to unit test!
 */
import { match } from "ts-pattern";

export const determinePremiumRoute = (user: UserState): NavigationCommand => 
  match(user)
    .with({ isLoggedIn: false }, () => ({
      type: "Redirect" as const,
      url: "/login",
    }))
    .with({ isLoggedIn: true, hasPremiumAccess: false }, () => ({
      type: "Alert" as const,
      message: "You need a premium subscription to view this page.",
    }))
    .with({ isLoggedIn: true, hasPremiumAccess: true }, () => ({
      type: "Redirect" as const,
      url: "/premium/dashboard",
    }))
    .exhaustive();
```

### imperative shell

```tsx
// shell/PremiumButton.tsx
import React from "react";
import { match } from "ts-pattern";
import { useNavigate } from "react-router-dom"; // Example router
import { determinePremiumRoute, UserState } from "./navigationCore";

interface PremiumButtonProps {
  user: UserState;
}

export const PremiumButton = ({ user }: PremiumButtonProps) => {
  const navigate = useNavigate(); // Imperative router hook

  // The Interpreter: This transforms data (commands) into actual side effects
  const executeCommand = (command: ReturnType<typeof determinePremiumRoute>) => 
    match(command)
      .with({ type: "Redirect" }, ({ url }) => {
        navigate(url); // Side effect!
      })
      .with({ type: "Alert" }, ({ message }) => {
        window.alert(message); // Side effect!
      })
      .with({ type: "Noop" }, () => {
        // No side effect
      })
      .exhaustive();

  const handleButtonClick = () => {
    // 1. Gather inputs (from props)
    // 2. Feed to Core to get a command
    const command = determinePremiumRoute(user);
    
    // 3. Execute the side effect
    executeCommand(command);
  };

  return (
    <button onClick={handleButtonClick} className="btn-premium">
      Access Premium Hub
    </button>
  );
};
```

## BEFORE, EXAMPLE 2

```tsx
// UserProfile.tsx
import React from "react";
import { logger } from "./services/logger"; // Direct side-effect dependency

interface ProfileProps {
  username: string;
  isFormValid: boolean;
}

export const UserProfile = ({ username, isFormValid }: ProfileProps) => {  
  const handleSave = () => {
    // ❌ Business logic and side effects are tightly coupled
    if (isFormValid) {
      // Logic + side effect 1
      logger.info(`User ${username} successfully updated their profile.`);
      // ... save to database
    } else {
      // Logic + side effect 2
      logger.error(`Validation failed for user: ${username}`);
      // ... show UI error toast
    }
  };

  return <button onClick={handleSave}>Save Profile</button>;
};
```

## AFTER, EXAMPLE 2

### functional core

```tsx
// core/profileCore.ts

export type LogCommand = 
  | { type: "LogInfo"; message: string }
  | { type: "LogError"; message: string };

/**
 * Pure Function: Decides what log command to generate.
 * Completely deterministic and highly testable.
 * Single expression, no multiple returns.
 */
import { match } from "ts-pattern";

export const determineSaveLog = (username: string, isFormValid: boolean): LogCommand =>
  match(isFormValid)
    .with(true, () => ({
      type: "LogInfo" as const,
      message: `User ${username} successfully updated their profile.`,
    }))
    .with(false, () => ({
      type: "LogError" as const,
      message: `Validation failed for user: ${username}`,
    }))
    .exhaustive();
```

### imperative shell

```tsx
// shell/UserProfile.tsx
import React from "react";
import { match } from "ts-pattern";
import { logger } from "./services/logger";
import { determineSaveLog, LogCommand } from "./core/profileCore";

interface ProfileProps {
  username: string;
  isFormValid: boolean;
}

export const UserProfile = ({ username, isFormValid }: ProfileProps) => {
  // The Interpreter: Converts LogCommands into actual logger side effects
  const executeLog = (command: LogCommand) => {
    return match(command)
      .with({ type: "LogInfo" }, ({ message }) => {
        logger.info(message);
      })
      .with({ type: "LogError" }, ({ message }) => {
        logger.error(message);
      })
      .exhaustive();
  };

  const handleSave = () => {
    // 1. Get the command from the pure core
    const logCommand = determineSaveLog(username, isFormValid);
    
    // 2. Execute the side effect in the shell
    executeLog(logCommand);
    
    // ... proceed with actual save/error UI workflows
  };

  return <button onClick={handleSave}>Save Profile</button>;
};
```

## EXPLANATION

The main pattern is to use a repeatable 2-layer split:

## Pure core
 - accepts explicit inputs
 - returns value or command description
 - contains of pure functions
 - the .ts files

## Imperative shell
 - reads globals (window, storage, i18next)
 - invokes side effects
 - delegates all branching logic to pure core
 - the .tsx files (React views and containers)

The core returns commands; the shell interprets them.