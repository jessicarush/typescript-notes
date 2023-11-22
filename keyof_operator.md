# The keyof type operator

## Table of contents

<!-- toc -->

- [Description](#description)
- [Example](#example)

<!-- tocstop -->

## Description 

The `keyof` operator takes an object type and produces a string or numeric *literal union* of its keys. 

## Example

The following type `P` is the same type as `type P = 'x' | 'y'`:

```typescript
type Point = { x: number; y: number };
type P = keyof Point;
```