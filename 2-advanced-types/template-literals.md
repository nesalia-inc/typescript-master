# Template Literal Types

String manipulation at the type level.

## Basic Template Literals

```typescript
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"
```

## Built-in String Manipulations

```typescript
type A = Uppercase<"hello">;           // "HELLO"
type B = Lowercase<"HELLO">;           // "hello"
type C = Capitalize<"hello">;         // "Hello"
type D = Uncapitalize<"Hello">;       // "hello"
```

## Mapped Types with Template Literals

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  name: string;
  age: number;
}

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }
```

## Complex Patterns

### Type-Safe CSS Classes

```typescript
type Size = "sm" | "md" | "lg";
type Color = "red" | "blue" | "green";
type State = "primary" | "secondary";

type ButtonClass = `${Size}-${Color}-${State}`;
// "sm-red-primary" | "sm-red-secondary" | ... (18 combinations)
```

### API Path Builder

```typescript
type ExtractPathParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractPathParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
      ? Param
      : never;

type Params = ExtractPathParams<"/users/:id/posts/:postId">;
// "id" | "postId"
```

### Event System

```typescript
type EventMap = {
  user: { id: string; name: string };
  post: { id: string; title: string };
};

type EventHandlers<T> = {
  [K in keyof T as `on${Capitalize<string & K>}`]: (data: T[K]) => void;
};

type Handlers = EventHandlers<EventMap>;
// { onUser: (data: { id: string; name: string }) => void;
//   onPost: (data: { id: string; title: string }) => void; }
```

## Parsing with Infer

```typescript
type ParseQuery<P extends string> =
  P extends `${infer Key}=${infer Value}&${infer Rest}`
    ? { [K in Key]: Value } & ParseQuery<Rest>
    : P extends `${infer Key}=${infer Value}`
      ? { [K in Key]: Value }
      : {};

type Query = ParseQuery<"name=john&age=30">;
// { name: "john"; age: "30" }
```

## Combining with Conditional Types

```typescript
type RouteParams<T extends string> =
  T extends `${infer Prefix}:${infer Param}/${infer Rest}`
    ? Param | RouteParams<`/${Rest}`>
    : T extends `${infer Prefix}:${infer Param}`
      ? Param
      : never;

function getParam<T extends string>(route: T, param: RouteParams<T>): string {
  // Implementation
  return "" as any;
}

getParam("/users/:id", "id");    // OK
getParam("/users/:id", "name");   // Error
```

## Real-World: Form Field Names

```typescript
type FormPrefix = "user" | "admin";
type FormField = "name" | "email";

type FullField = `${FormPrefix}.${FormField}`;
// "user.name" | "user.email" | "admin.name" | "admin.email"
```

## Key Takeaways

- Template literals create precise string types
- Combine with `infer` to extract parts of strings
- Use with mapped types for dynamic property names
- Distributes over union types automatically
- Enables compile-time checking of string patterns
