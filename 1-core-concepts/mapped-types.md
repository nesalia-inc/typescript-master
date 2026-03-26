# Mapped Types

Transform existing types by iterating over their properties.

## Basic Mapped Type

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
// Type: { readonly id: number; readonly name: string; }
```

## Optional Properties

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type PartialUser = Partial<User>;
// Type: { id?: number; name?: string; }
```

## Required Properties

```typescript
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```

The `-?` removes the optional modifier.

## Key Remapping

Transform property names using `as`:

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// Type: { getName: () => string; getAge: () => number; }
```

## Filtering Properties

```typescript
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Mixed {
  id: number;
  name: string;
  age: number;
  active: boolean;
}

type OnlyNumbers = PickByType<Mixed, number>;
// Type: { id: number; age: number; }
```

## Exclude by Property Type

```typescript
type ExcludeByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K];
};

type NoBooleans = ExcludeByType<Mixed, boolean>;
// Type: { id: number; name: string; age: number; }
```

## Mapped Type Modifiers

| Modifier | Syntax | Effect |
|----------|--------|--------|
| Optional | `?` | Makes property optional |
| Required | `-?` | Removes optional |
| Readonly | `readonly` | Makes property readonly |
| Mutable | `-readonly` | Removes readonly |

## Real-World Example: Deep Partial

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? T[P] extends Array<infer U>
      ? Array<DeepPartial<U>>
      : DeepPartial<T[P]>
    : T[P];
};

interface Config {
  server: {
    host: string;
    port: number;
  };
}

type PartialConfig = DeepPartial<Config>;
// { server?: { host?: string; port?: number } }
```

## Real-World Example: Required Keys Only

```typescript
type RequiredKeys<T, K extends keyof T> = {
  [P in K]-?: T[P];
} & {
  [P in Exclude<keyof T, K>]?: T[P];
};

interface User {
  id: string;
  name: string;
  email?: string;
}

type UserRegistration = RequiredKeys<User, "id" | "name">;
// { id: string; name: string; email?: string; }
```

## Template Literals in Mapped Types

```typescript
type EventMap = {
  user: { id: string };
  post: { id: string; title: string };
};

type EventHandlers<T> = {
  [K in keyof T as `on${Capitalize<string & K>}`]: (data: T[K]) => void;
};

type UserEventHandlers = EventHandlers<EventMap>;
// { onUser: (data: { id: string }) => void; onPost: (data: { id: string; title: string }) => void; }
```

## Real-World Examples

### Example 1: Database Model to API Schema Transformer

When building an ORM that generates API schemas from database models:

```typescript
interface DatabaseModel {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  deletedAt: Date | null;
}

interface ApiSchema<T> {
  [K in keyof T]: T[K] extends Date | null ? string : T[K];
}

// Transform database timestamps to ISO strings for JSON API
type UserApiSchema = ApiSchema<DatabaseModel>;
// { id: string; createdAt: string; updatedAt: string; deletedAt: string | null }
```

### Example 2: Type-Safe Form Fields

When building a form generation library:

```typescript
interface FormField {
  type: "text" | "number" | "select";
  label: string;
  required: boolean;
  options?: string[];
}

interface FormConfig {
  name: FormField;
  email: FormField;
  age: FormField;
}

// Generate form field props from form config
type FormFieldProps<T> = {
  [K in keyof T]: {
    name: K;
    label: T[K]["label"];
    required: T[K]["required"];
    type: T[K]["type"];
    options?: T[K]["options"];
  };
};

type UserFormProps = FormFieldProps<FormConfig>;
// { name: { name: "name"; label: string; required: boolean; type: "text" ... }
//   email: { name: "email"; ... }
```

### Example 3: Observable State Proxy

When building a reactive state management system:

```typescript
type Observable<T> = {
  get<K extends keyof T>(key: K): T[K];
  set<K extends keyof T>(key: K, value: T[K]): void;
  subscribe<K extends keyof T>(key: K, callback: (value: T[K]) => void): () => void;
};

type CreateObservable<T> = {
  [K in keyof T]: {
    get: () => T[K];
    set: (value: T[K]) => void;
    subscribe: (callback: (value: T[K]) => void) => () => void;
  };
};

interface UserState {
  name: string;
  age: number;
}

declare function createObservable<T>(initial: T): CreateObservable<T>;

const userState = createObservable({ name: "John", age: 30 });
userState.name.set("Jane");                    // OK
userState.name.subscribe((name) => console.log(name)); // Returns unsubscribe
```

### Example 4: GraphQL Schema to Type Generator

When building a GraphQL-to-TypeScript code generator:

```typescript
interface GraphQLField {
  type: string;
  isRequired: boolean;
  isList: boolean;
}

interface GraphQLObject {
  name: string;
  fields: Record<string, GraphQLField>;
}

// Transform GraphQL schema to TypeScript interface
type ToTypeScriptInterface<T extends GraphQLObject> = {
  [K in keyof T["fields"] as `${string & K}`: T["fields"][K]["isRequired"] extends true
    ? T["fields"][K]["isList"] extends true
      ? `${string & K}`: T["fields"][K]["type"][]`
      : `${string & K}`: T["fields"][K]["type"]`
    : T["fields"][K]["isList"] extends true
      ? `${string & K}?`: `${string & K}?`]: T["fields"][K]["type"] | null;
};
```

### Example 5: Immutable Update Utilities

When building a state management library with immutable updates:

```typescript
type Immutable<T> = {
  readonly [K in keyof T]: T[K] extends object ? Immutable<T[K]> : T[K];
};

type WithImmutability<T> = {
  set<K extends keyof T>(key: K, value: T[K]): WithImmutability<T>;
  setIn<K extends keyof T>(key: K, updater: (value: T[K]) => T[K]): WithImmutability<T>;
  delete<K extends keyof T>(key: K): WithImmutability<T>;
};

interface State {
  user: { name: string; address: { city: string } };
  posts: string[];
}

const state: WithImmutability<Immutable<State>> = {} as any;

// Chained immutable updates
const updated = state
  .setIn("user", { name: "John", address: { city: "NYC" } })
  .setIn("posts", (posts) => [...posts, "New Post"]);
```

### Example 6: Database Column Types

When building a database migration system:

```typescript
type ColumnTypes = {
  string: "VARCHAR";
  number: "INTEGER";
  boolean: "BOOLEAN";
  date: "TIMESTAMP";
};

type ToDatabaseColumn<T> = {
  [K in keyof T]: {
    name: K;
    type: ColumnTypes[T[K] extends string ? "string" : T[K] extends number ? "number" : T[K] extends boolean ? "boolean" : "date"];
    nullable: false;
    primaryKey: K extends "id" ? true : false;
  };
};

interface User {
  id: string;
  name: string;
  age: number;
}

type UserColumns = ToDatabaseColumn<User>;
// { id: { name: "id"; type: "VARCHAR"; nullable: false; primaryKey: true }
//   name: { name: "name"; type: "VARCHAR"; nullable: false; primaryKey: false }
//   age: { name: "age"; type: "INTEGER"; nullable: false; primaryKey: false }
```

## When to Use Mapped Types

| Scenario | Example |
|----------|---------|
| Transforming API responses | `ApiSchema<T>` (Date → string) |
| Creating form systems | `FormFieldProps<T>` |
| Building reactive state | `Observable<T>` |
| Generating GraphQL types | `ToTypeScriptInterface<T>` |
| Immutable update helpers | `WithImmutability<T>` |
| Database migrations | `ToDatabaseColumn<T>` |

## Key Takeaways

- Mapped types iterate over `keyof T`
- Use `as` to remap keys with template literals
- Use conditional types to filter properties
- Combine with template literals for dynamic property names
- Essential for code generation and type transformation utilities
