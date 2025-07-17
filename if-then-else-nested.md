# Nested if/else statements to a chained and flat composition

The examples are adapted from https://dev.to/rametta/basic-monads-in-javascript-3el3

## BEFORE

```ts
const data = {
  first: "Jason",
  level: 85,
  cool: true,
  shirt: {
    size: "m",
    color: "blue",
    length: 90,
    logo: {
      color1: "#abc123",
      color2: "#somehexcolor"
    }
  }
}

if (data) {
  if (data.shirt) {
    if (data.shirt.logo) {
      if (data.shirt.logo.color1 !== "black") {

        // Color1 is valid, now lets continue
        console.log(data.shirt.logo.color1)

      } else {
        console.error ("Color1 is black")
      }
    } else {
      console.error ("No logo")
    }
  } else {
    console.error ("No shirt")
  }
} else {
  console.error ("No data")
}

```

## AFTER

```ts
import { chain, caseOf, Right, Left, Either } from "purify-ts/Either";
import { chain, pipe } from "ramda";

const hasData = (data: Person): Either<string, Shirt> =>
  data ? Right(data.shirt) : Left("No data");

const hasShirt = (shirt: Person): Either<string, Logo> =>
  shirt ? Right(shirt.logo) : Left("No shirt");

const hasLogo = (logo: Person): Either<string, string> =>
  logo ? Right(logo.color1) : Left("No logo");

const isNotBlack = (color: string): Either<string, string> =>
  color !== "black" ? Right(color) : Left("Color is black");

pipe(
  hasData,
  chain(hasShirt),
  chain(hasLogo),
  chain(isNotBlack),
  caseOf({
    Right: console.log,
    Left: console.error
  })
)(data); // ramda's pipe gets the data as a separate, last argument
```

## EXPLANATION

The imperative version suffers from the dreaded 'Pyramid of doom' problem,
the indentation points out that there is a serious structural code smell.
There is IO (console.log) threaded trough the program.

The after version is better because:
- readability is improved dramatically by composing the algorithm. The indentation
is reduced and the logical steps are all individually isolated.
The main algorithm (steps) is now clearly readable and separated from their details.
- It pushes the IO (the console logging) to the edges of the program. 
That means that there are more pure functions that can be unit tested and don't have IO 
mixed inside of them. Having IO inside of pure functions don't make them pure anymore, 
which means they would be harder to unit test and could be a source of bugs.
Console logging is not a big deal in javascript, but if the IO was making a 
network request, then this type of programming makes a big difference, 
because all logic/validation would be independent of IO and easier to test and maintain.
 