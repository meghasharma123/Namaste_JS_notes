# Basics

## Difference between type and interface

- interface is primarily used to describe the shape of objects and supports declaration merging.
- type can define objects, unions, intersections, primitives, tuples, function types, etc.

And both can be extended:

```
interface User {
    name: string;
}

interface Employee extends User {
    salary: number;
}

type User = {
    name: string;
};

type Employee = User & {
    salary: number;
};
```

`Declaration Merging` - interface can be defined multiple time the properties will get merge. However, duplicate properties must have compatible types.

| Feature             | Type    | Interface     |
| ------------------- | ------- | ------------- |
| Object definition   | Yes     | Yes           |
| Union types         | Yes     | No            |
| Intersection        | Yes (&) | Yes (extends) |
| Declaration merging | No      | Yes           |
| Extend another type | Yes (&) | Yes (extends) |
| Primitive aliases   | Yes     | No            |

## Generics

```
function identity<T>(value: T): T {
    return value;
}

const result1 = identity<string>("Hello");
const result2 = identity<number>(100);
```

#### What is T ?

- T is a generic type parameter. It acts as a placeholder for a type that will be determined when the function is called.

#### Why generics instead of any?

```
function getValue(value: any): any {
    return value;
}

const result = getValue("Hello");
result.toFixed(); // No TypeScript error
```

`Generic is more type safe than any.` if we have used generic in the above function we must have got `type error`.

#### Real-world use case

Imagine your Node.js application returns different types of data from APIs.

```
interface ApiResponse<T> {
    success: boolean;
    data: T;
    message: string;
}
```

#### queestion

This code produces a TypeScript error.

```
function getLength<T>(value: T): number {
    return value.length;
}
```

Questions:

1. Why does this error occur?
2. How can we fix it using extends?
3. What is the difference between T extends string and T extends { length: number }?

Ans:

1. TypeScript doesn't know whether T has a length property.
2. fix using extend

```
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}
```

3. Difference between the constraints:

- T extends string: T must be assignable to string.
- T extends { length: number }: T can be any type that has a numeric length property.

## Utility Types

What is the difference between Partial, Required, and Pick in TypeScript?

```
interface User {
  name: string;
  age: number;
  email?: string;
}
```

1. What does Partial<User> do?
2. What does Required<User> do?
3. How would you create a type containing only name and age using Pick?

Ans:

1. Makes all properties optional.
2. Makes all properties required, including previously optional properties like email.
3. `type UserBasicInfo = Pick<User, "name" | "age">;`
   T

```
{
  name: string;
  age: number;
}
```

Remember:

- Partial<T> -> all optional
- Required<T> -> all required
- Pick<T, K> -> select specific properties
- Omit<T, K> -> exclude specific properties

=>`type NoSalary = Omit<Employee, "salary">;` // remove salary field from employee interface

## any vs unknown vs never

| Type    | Meaning                                                        |
| ------- | -------------------------------------------------------------- |
| any     | Accepts any value and disables type checking.                  |
| unknown | Accepts any value, but requires type checking before using it. |
| never   | Represents a value that can never occur.                       |

```
let a: any = "hello";
let b: unknown = "hello";

console.log(a.toUpperCase()); // Compiles
console.log(b.toUpperCase()); // TypeScript error
```

- any: TypeScript allows calling any method without checking.
- unknown: TypeScript doesn't know whether the value is a string, number, etc.

To fix the second line:

```
if (typeof b === "string") {
  console.log(b.toUpperCase());
}
```

What is never? This function never successfully returns a value. Another common use is exhaustive checking in TypeScript.

```
function throwError(message: string): never {
  throw new Error(message);
}
```

## Type narrowing

Type narrowing means reducing a broader type into a more specific type using checks like typeof, instanceof, or in.

```
function printValue(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is a string here.
  } else {
    // TypeScript knows value is a number here.
  }
}
```

## keyof

```
interface User {
  name: string;
  age: number;
  email: string;
}

type UserKeys = keyof User;
```

It creates a union of property names: `type UserKeys = "name" | "age" | "email";`

It helps us safely access object properties while maintaining type safety.

```
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Megha", age: 27 };

getProperty(user, "name"); // string
getProperty(user, "age");  // number
getProperty(user, "salary") // gives an error as its not available
```

| Operator     | Purpose                             |
| ------------ | ----------------------------------- |
| typeof       | Gets the type of a variable         |
| keyof        | Gets the keys of a type             |
| keyof typeof | Gets the keys of an existing object |

```
type A = typeof user;
=> type A = {
  name: string;
  age: number;
};

type B = keyof typeof user;
=> type B = "name" | "age";
```

# Complex problems

1. using generic type as array of any element:

```
function getFirstElement<T>(value: T[]): T | undefined {
  if (value.length === 0) return undefined;
  return value[0];
}
```

2. generic type for API:
```
interface ApiResponse<T> {
  success: boolean;
  data: T;
}
```