# Utility Types

Built-in TypeScript transformations for common type operations.

## Property Transformation

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Make all properties optional
type Partial<T> = { [P in keyof T]?: T[P] };
type PartialUser = Partial<User>;

// Make all properties required
type Required<T> = { [P in keyof T]-?: T[P] };

// Make all properties readonly
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type ReadonlyUser = Readonly<User>;
```

## Property Selection

```typescript
// Select specific properties
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type UserName = Pick<User, "name" | "email">;
// { name: string; email: string; }

// Remove specific properties
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
type UserWithoutPassword = Omit<User, "id">;
// { name: string; email: string; }
```

## Union Helpers

```typescript
// Exclude types from union
type Exclude<T, U> = T extends U ? never : T;
type T1 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"

// Extract types from union
type Extract<T, U> = T extends U ? T : never;
type T2 = Extract<"a" | "b" | "c", "a" | "b">; // "a" | "b"
```

## Nullability

```typescript
// Exclude null and undefined
type NonNullable<T> = T extends null | undefined ? never : T;
type T3 = NonNullable<string | null | undefined>; // string

// Extract non-nullable keys
type NonNullableKeys<T, K extends keyof T> = {
  [P in K]-?: T[P] extends null | undefined ? never : P;
}[K];
```

## Record

```typescript
// Create object type with keys K and values T
type Record<K extends keyof any, T> = { [P in K]: T };

type PageInfo = Record<"home" | "about", { title: string; url: string }>;
// { home: { title: string; url: string }; about: { title: string; url: string } }
```

## Parameters and Return Type

```typescript
// Extract function parameters as tuple
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

// Extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

// Extract instance type from constructor
type InstanceType<T extends new (...args: any[]) => any> =
  T extends new (...args: any[]) => infer R ? R : any;
```

## Constructor Types

```typescript
class UserClass {
  constructor(public name: string, public age: number) {}
}

type UserInstance = InstanceType<typeof UserClass>;
// UserClass

type UserParams = ConstructorParameters<typeof UserClass>;
// [name: string, age: number]
```

## ThisType

```typescript
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>;
};

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  const data: D = desc.data ?? ({} as D);
  const methods: M = (desc.methods ?? {}) as M & ThisType<D & M>;
  return { ...data, ...methods };
}

const obj = makeObject({
  data: { x: 10, y: 20 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // Type: number
      this.y += dy; // Type: number
    },
  },
});
```

## Template Literal Utilities

```typescript
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;
```

## Combining Utility Types

```typescript
interface ApiResponse {
  id: string;
  name: string;
  createdAt: string;
  updatedAt: string;
}

// Create update type (all optional except id)
type UpdateUser = Partial<Omit<ApiResponse, "id" | "createdAt" | "updatedAt">> & { id: string };

// Create response without timestamps
type UserPreview = Omit<ApiResponse, "createdAt" | "updatedAt">;
```

## Real-World Examples

### Example 1: React Component Props

When building a component library:

```typescript
interface BaseComponentProps {
  className?: string;
  id?: string;
  "data-testid"?: string;
}

interface ButtonProps extends BaseComponentProps {
  variant: "primary" | "secondary" | "danger";
  size: "sm" | "md" | "lg";
  disabled?: boolean;
  onClick?: () => void;
}

// Extract only the event-related props for event handler typing
type EventProps = Pick<ButtonProps, "onClick">;
// { onClick?: () => void }

// Extract style-related props
type StyleProps = Pick<ButtonProps, "className" | "variant" | "size">;
// { className?: string; variant: ...; size: ... }
```

### Example 2: API Request/Response Types

When building an API client:

```typescript
interface ApiEndpoint<TResponse, TRequest = never> {
  response: TResponse;
  request?: TRequest extends never ? unknown : TRequest;
}

// Define endpoints
interface Endpoints {
  getUser: ApiEndpoint<User>;
  createPost: ApiEndpoint<Post, { title: string; content: string }>;
  updateUser: ApiEndpoint<User, Partial<Pick<User, "name" | "email">>>;
  deletePost: ApiEndpoint<void, { id: string }>;
}

// Extract all response types
type ApiResponses = {
  [K in keyof Endpoints]: Endpoints[K]["response"];
};
// { getUser: User; createPost: Post; updateUser: User; deletePost: void }

// Extract all request types (non-void)
type ApiRequests = {
  [K in keyof Endpoints]: Endpoints[K] extends { request: infer R }
    ? R extends unknown
      ? never
      : R
    : never;
};
// { createPost: { title: string; content: string }; updateUser: ...; deletePost: ... }
```

### Example 3: Database Model Mutations

When building an ORM:

```typescript
interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  deletedAt: Date | null;
}

interface User extends BaseEntity {
  email: string;
  name: string;
  role: "admin" | "user" | "guest";
}

// Types for different operations
type UserCreateInput = Omit<User, "id" | "createdAt" | "updatedAt" | "deletedAt">;
// { email: string; name: string; role: "admin" | "user" | "guest" }

type UserUpdateInput = Partial<Omit<User, "id" | "createdAt" | "updatedAt" | "deletedAt">>;
// { email?: string; name?: string; role?: "admin" | "user" | "guest" }

// Types for what clients can safely see (no soft-delete flags)
type UserPublic = Omit<User, "deletedAt">;
// { id: string; email: string; name: string; role: ...; createdAt: Date; updatedAt: Date }

// Query filters - only allow filtering on certain fields
type UserFilterFields = Pick<User, "email" | "role">;
// { email?: string; role?: "admin" | "user" | "guest" }
```

### Example 4: Form Validation Schema

When building a form validation library:

```typescript
interface FormField<T = any> {
  value: T;
  error?: string;
  touched: boolean;
  dirty: boolean;
}

interface FormSchema<T> {
  fields: {
    [K in keyof T]: FormField<T[K]>;
  };
  isValid: boolean;
  isDirty: boolean;
}

// Extract field types from form schema
type FormValues<T> = {
  [K in keyof T]: T[K] extends FormField<infer V> ? V : T[K];
};

// Extract only dirty (modified) fields
type DirtyFields<T> = {
  [K in keyof T as T[K] extends FormField<any> ? (T[K]["dirty"] extends true ? K : never) : never]: T[K] extends FormField<infer V> ? V : never;
};

// Usage
interface LoginForm {
  email: FormField<string>;
  password: FormField<string>;
}

type LoginValues = FormValues<LoginForm>;
// { email: string; password: string }

type ModifiedFields = DirtyFields<LoginForm>;
// { email?: string; password?: string } (only if dirty)
```

### Example 5: Plugin/Extension System

When building an extensible application:

```typescript
interface Plugin {
  name: string;
  version: string;
  dependencies?: string[];
}

interface CoreApp {
  plugins: Plugin[];
  config: Record<string, any>;
  services: Record<string, any>;
}

// Extract only the plugin names
type PluginNames = Extract<keyof Plugin, string>;
// "name" | "version" | "dependencies"

// Make certain properties required in a derived type
type RequiredPlugin = Required<Pick<Plugin, "name" | "version">> & Partial<Pick<Plugin, "dependencies">>;
// { name: string; version: string; dependencies?: string[] }

// Create a type for enabled plugins only (non-null)
type EnabledPlugins = NonNullable<Plugin>[];
// Plugin[] (since Plugin is already non-nullable)
```

### Example 6: Route Types for SPA Router

When building a single-page application router:

```typescript
interface RouteDefinition {
  path: string;
  component: any;
  guards?: any[];
  meta?: Record<string, any>;
}

interface AppRoutes {
  home: RouteDefinition & { path: "/"; meta: { title: string } };
  users: RouteDefinition & { path: "/users"; children?: any };
  userDetail: RouteDefinition & { path: "/users/:id"; meta: { requiresAuth: boolean } };
  posts: RouteDefinition & { path: "/posts"; children?: any };
}

// Extract all valid paths
type RoutePaths = {
  [K in keyof AppRoutes]: AppRoutes[K]["path"];
}[keyof AppRoutes];
// "/" | "/users" | "/users/:id" | "/posts"

// Extract all paths with auth requirements
type ProtectedRoutes = {
  [K in keyof AppRoutes]: AppRoutes[K]["meta"] extends { requiresAuth: true } ? K : never;
}[keyof AppRoutes];
// "userDetail"

// Extract route metadata
type RouteMeta<T extends keyof AppRoutes> = AppRoutes[T]["meta"];
```

## When to Use Utility Types

| Scenario | Utility Types |
|----------|---------------|
| Component props (some required, some optional) | `Partial`, `Pick`, `Omit` |
| API request/response separation | `Pick`, `Omit` |
| Database mutations (create vs update) | `Omit` for create, `Partial<Omit>` for update |
| Form values vs form schema | `FormValues<T>` pattern |
| Plugin configurations | `Required`, `Partial`, `Pick` |
| Route types | `Pick`, `Omit`, mapped types |

## Key Takeaways

- Utility types transform existing types without redefining them
- `Partial`, `Required`, `Readonly`, `Pick`, `Omit` are most common
- Combine utilities to create precise type transformations
- Most built-in utilities are themselves mapped types or conditional types
- Essential for building reusable component libraries and APIs
