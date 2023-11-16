# Types

## Table of contents

<!-- toc -->

- [Literal types](#literal-types)
- [Union types](#union-types)
- [Default values with types](#default-values-with-types)
- [Type narrowing ()](#type-narrowing-)
- [Type predicates and type assertion](#type-predicates-and-type-assertion)
- [Optional values](#optional-values)
- [Type intersection](#type-intersection)
- [Enums?](#enums)
- [never?](#never)

<!-- tocstop -->

## Literal types 

```typescript
type Role = 'developer';
```

## Union types

```typescript
type Role = 'developer' | 'admin' | 'manager';
```

## Default values with types

```typescript
function getTheme(theme: 'dark' | 'light' = 'dark' ): void {
  if (theme === 'dark') {
    // do something
  }
  if (theme === 'light') {
    // do something
  }
}
```

## Type narrowing ()

Literal type narrowing with `===`:

```typescript
function getTheme(theme: 'dark' | 'light'): void {
  if (theme === 'dark') {
    // do something
  }
  if (theme === 'light') {
    // do something
  }
}
```

Primitive type narrowing with `typeof`. Remember possible string values for `typeof` are:

- `string`
- `number`
- `bigint`
- `boolean`
- `symbol`
- `undefined`
- `object`
- `function`

```typescript
function getNumberValue(arg: unknown): number | never {
    if (typeof arg === 'number') {
        return arg;
    }
    if (typeof arg === 'string') {
        return Number(arg)
    }
    throw new Error(`Unsupported format: ${JSON.stringify(arg)}`);
}
```

Class type narrowing with `instanceof`:

```typescript
class Product {
  getDescription() {
    return 'Product...';
  }
}

class Activity {
  getDescription() {
    return 'Activity...';
  }
}

type Entries = Product | Activity;

function viewEntries(entry: Entries): void {
  if (entry instanceof Product) {
    // do something
  }
  if (entry instanceof Activity) {
    // do something
  }
}

viewEntries(new Product());
viewEntries(new Activity());
```

Object type narrowing:

```typescript
type Founder = {
    name: 'John Founder';
    car: 'Audi';
  } | {
    name: 'Bill Bicycle';
    bike: 'Wheels';
  };

function meetTheFounder(founder: Founder) {
  console.log(`Meet your fonder ${founder.name}`);
  if (founder.name === 'John Founder') {
    // do something
    console.log('John');
  }
  if ('bike' in founder) {
    // do something
    console.log(`bike: ${founder.bike}`);
  }
}

meetTheFounder({
  name: 'John Founder',
  car: 'Audi'
});

meetTheFounder({
  name: 'Bill Bicycle',
  bike: 'Wheels'
});
```

## Type predicates and type assertion

To define a user-defined type guard, we need to define a function whose return type is a *type predicate* (`pet is Fish`). When type predicates are used like this, we must return a boolean that answers the question. In order to check the arg for properties we have to use a *type assertion* (`pet as Fish`). It tells TypeScript to treat the arg as if it is of type Fish:

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

In this next example, the parameter arg is of type `any`. This allows you to freely access properties of arg without TypeScript errors. However, using `any` can potentially lead to runtime errors if arg does not have the expected properties, as `any` bypasses TypeScript's type checking.

```typescript
type Salary = {
  amount: number;
};

// ❌ Not the best approach:
function isSalary(arg: any): arg is Salary {
  return 'amount' in arg && typeof arg.amount === 'number';
}

function paySalary(arg: unknown) {
  if (isSalary(arg)) {
    console.log(`Paying ${arg.amount}`);
  }
}
```

If we switch `any` to `unknown` then we have to switch to type assertions:

```typescript
// 😑 This is ok but could be even better:
function isSalary(arg: unknown): arg is Salary {
  return (
    (arg as Salary).amount !== undefined &&
    typeof (arg as Salary).amount === 'number'
  );
}
```

If we want handle this properly:

```typescript
// ✅ This covers all possible scenarios:
function isSalary(arg: unknown): arg is Salary {
  return (
    typeof arg === 'object' &&
    arg !== null &&
    'amount' in arg &&
    typeof (arg as Salary).amount === 'number'
  );
}
```

To explain:

First do an initial type check for Object:` typeof arg === 'object' && arg !== null` to ensure that `arg` is an `object` before proceeding with further checks. This is a common pattern when dealing with unknown types, as it guards against arg being a primitive, `null`, or `undefined`.

Then we can safely use the `in` operator to check for the existence of the amount property in arg. This is a concise way to check for property existence on an object.

Finally, type assertion `arg as Salary` is now only used once, in the final check for the type of amount.


## Optional values

```typescript
// Optional properties
type Employee = {
  id: number;
  currency: 'USD' | 'EUR';
  yearlyBonus?: number;
};

// Optional parameters
function paySalary(employee: Employee, special?: string) {
  if (special) {
    // do something
  }
}

// Optional values in classes:
class Engineer {
  tasks?: string[];
}
```

Also should remind about optional chaining, even though that is a native JS feature (as of ES202).

```typescript
type Manager = {
  team?: {
    scrumMaster?: {
      holdScrumMeeting: () => void;
    };
  };
};

// ? - JavaScript optional operator
function manage(manager: Manager) {
  manager.team?.scrumMaster?.holdScrumMeeting();
}
```


## Type intersection

```typescript

```

## Enums?

```typescript

```

## never?

```typescript

```