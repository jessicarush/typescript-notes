# Function overloads

Some JavaScript functions can be called with a variety of arguments and types. For example, you might write a function to produce a Date that takes either a timestamp (one argument) or a month/day/year specification (three arguments).

In TypeScript, we can specify a function that can be called in different ways by writing *overload signatures*. To do this, write some number of function signatures (usually two or more), followed by the body of the function.

The reason you would want to do this is to have more accurate types on your resulting calls:

```typescript
function oneYearAgo(date: Date | string) {
  const oneYearAgo = new Date(date);
  oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

  if (typeof date === 'string') {
    return oneYearAgo.toLocaleDateString();
  } else {
    return oneYearAgo;
  }
}
const lastYearDate = oneYearAgo(new Date()); // const lastYearDate: string | Date
const lastYearString = oneYearAgo('6/9/2026'); // const lastYearString: string | Date
```

If we define function signatures for each possible scenario, we get the correct types `Date` or `string` instead of `string | Date`.

```typescript
function oneYearAgo(date: Date): Date;
function oneYearAgo(date: string): string;
function oneYearAgo(date: Date | string) {
  const oneYearAgo = new Date(date);
  oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

  if (typeof date === 'string') {
    return oneYearAgo.toLocaleDateString();
  } else {
    return oneYearAgo;
  }
}
const lastYearDate = oneYearAgo(new Date()); // const lastYearDate: Date
const lastYearString = oneYearAgo('6/9/2026'); // const lastYearString: string
```