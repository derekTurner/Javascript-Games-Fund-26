# Functional Programming in TypeScript

Essential elements of functional programming are:

-   [First Class Functions](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function)
-   Higher order Functions
-   Declarative rather than Imperative code
-   Purity
-   Composition

## First Class

A first class function is one which can be treated as a variable.

```typescript
const firstClass = (): string => {
    return "first Class function";
};

let answer: string = firstClass();
```

Just like a variable, a function can be passed as a parameter to a second function. In this case, the returned value from the first function is passed into the second function.

```typescript
const cube = (z: number): number => {
    return z * z * z;
};

const statement = (volume: number): string => {
    return "The volume of a cube is " + volume;
};

let answer: string = statement(cube(4));
```

## Higher order

A function which receives a function as an argument (like `statement` above), or a function which returns a function, is called a higher order function.

```typescript
const power = (z: number): ((x: number) => string) => {
    return (x: number): string => {
        return "The value of " + x + " to the power " + z + " is " + Math.pow(x, z);
    };
};

const volume = power(3);
const area = power(2);

let c: string = area(4);
```

Edit the last line of this code to display `volume(4)`.

## Declarative

Declarative code avoids counters and uses iterators so that the code expresses what is to be done rather than the steps to do an action. The `map` method of an array is an example of this.

```typescript
// Declarative
// Find volumes from numbers in a list

const power = (z: number): ((x: number) => number) => {
    return (x: number): number => {
        return Math.pow(x, z);
    };
};

const volume = power(3);
const area = power(2);

const nums: number[] = [2, 3, 4, 5];
const numVolume: number[] = nums.map(volume);

let a: number[] = numVolume;
```

## Purity

A pure function is one which returns a value which depends only on its arguments. That means that a pure function should not refer to any global variables or data values outside its definition.

Pure functions are very portable as they have no external dependencies.

## Composition

A function can be written which uses two or more functions as arguments and then utilises both to produce a result.

Note that the order of execution for the `effect` function runs function `g` first and passes the result to function `f`. Evaluating the inner brackets first is the normal way of evaluating an expression.

In the code which follows, try editing `a` to use the `compose1`–`compose4` functions in turn.

```typescript
type NumberFunction = (value: number) => number;

const effect = (f: NumberFunction, g: NumberFunction): NumberFunction => {
    return (x: number): number => {
        return f(g(x));
    };
};

const bigEffect = (f: NumberFunction, g: NumberFunction, h: NumberFunction): NumberFunction => {
    return (x: number): number => {
        return f(g(x) + h(x));
    };
};

const add1 = (y: number): number => y + 1;
const times2 = (z: number): number => z * 2;
const negate = (w: number): number => w * -1;

const compose1 = effect(add1, times2);
const compose2 = effect(add1, negate);
const compose3 = effect(negate, times2);
const compose4 = bigEffect(add1, times2, negate);

let a: number = compose1(2);
```
