# Type Guards

Runtime checks that narrow types for compile-time safety.

## Basic Type Guard

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: unknown) {
  if (isString(value)) {
    // TypeScript knows value is string here
    console.log(value.toUpperCase()); // OK
  }
}
```

## Custom Type Guards

```typescript
interface Cat {
  kind: "cat";
  meow(): void;
}

interface Dog {
  kind: "dog";
  bark(): void;
}

type Animal = Cat | Dog;

function isCat(animal: Animal): animal is Cat {
  return animal.kind === "cat";
}

function makeSound(animal: Animal) {
  if (isCat(animal)) {
    animal.meow(); // Type: Cat
  } else {
    animal.bark(); // Type: Dog
  }
}
```

## Guard with Additional Checks

```typescript
interface User {
  type: "user";
  name: string;
  email: string;
}

interface Admin {
  type: "admin";
  name: string;
  privileges: string[];
}

type Person = User | Admin;

function isAdmin(person: Person): person is Admin {
  return person.type === "admin" && "privileges" in person;
}

function getName(person: Person): string {
  return person.name; // Works for both
}

function getPrivileges(person: Person): string[] | undefined {
  if (isAdmin(person)) {
    return person.privileges;
  }
  return undefined;
}
```

## Type Predicate Shorthand

```typescript
// Same as above but shorter
function isNonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}

// Usage
const maybeName: string | null = getName();
if (isNonNullable(maybeName)) {
  console.log(maybeName.toUpperCase()); // string
}
```

## Guard Arrays

```typescript
function isArrayOf<T>(
  value: unknown,
  guard: (item: unknown) => item is T
): value is T[] {
  return Array.isArray(value) && value.every(guard);
}

function isString(value: unknown): value is string {
  return typeof value === "string";
}

const data: unknown = ["a", "b", "c"];

if (isArrayOf(data, isString)) {
  data.forEach((s) => {
    s.toUpperCase(); // Type: string
  });
}
```

## Guard for Object Shapes

```typescript
function hasProperty<K extends string>(
  obj: unknown,
  key: K
): obj is Record<K, unknown> {
  return typeof obj === "object" && obj !== null && key in obj;
}

function parseResponse(data: unknown) {
  if (hasProperty(data, "users")) {
    // data.users exists
    if (hasProperty(data, "total")) {
      // data.total also exists
      console.log(data.total); // unknown
    }
  }
}
```

## in Operator Guard

```typescript
interface Admin {
  adminOnly: true;
  privileges: string[];
}

interface User {
  name: string;
  email: string;
}

function isAdmin(person: User | Admin): person is Admin {
  return "adminOnly" in person;
}
```

## instanceof Guard

```typescript
class JsonError extends Error {
  constructor(public statusCode: number) {
    super();
  }
}

class NetworkError extends Error {
  constructor(public retryable: boolean) {
    super();
  }
}

function handleError(error: Error) {
  if (error instanceof JsonError) {
    console.log(error.statusCode); // JsonError
  } else if (error instanceof NetworkError) {
    console.log(error.retryable); // NetworkError
  }
}
```

## Combining Guards

```typescript
function isValidUser(obj: unknown): obj is User {
  if (!isNonNullable(obj) || typeof obj !== "object") return false;
  const user = obj as Record<string, unknown>;
  return (
    typeof user.name === "string" &&
    typeof user.email === "string" &&
    user.name.length > 0 &&
    user.email.includes("@")
  );
}
```

## Real-World Examples

### Example 1: API Response Validation

```typescript
interface ApiSuccess<T> {
  status: "success";
  data: T;
}

interface ApiError {
  status: "error";
  code: number;
  message: string;
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

function isSuccess<T>(response: ApiResponse<T>): response is ApiSuccess<T> {
  return response.status === "success";
}

async function fetchUser(): Promise<User> {
  const response: ApiResponse<User> = await fetch("/api/user").then(r => r.json());

  if (isSuccess(response)) {
    return response.data; // TypeScript knows this is User
  }

  throw new Error(`API Error ${response.code}: ${response.message}`);
}
```

### Example 2: Form Data Processing

```typescript
interface TextField { type: "text"; name: string; value: string; }
interface NumberField { type: "number"; name: string; value: number; }
interface SelectField { type: "select"; name: string; options: string[]; selected: string; }

type FormField = TextField | NumberField | SelectField;

function isTextField(field: FormField): field is TextField {
  return field.type === "text";
}

function processForm(fields: FormField[]) {
  for (const field of fields) {
    if (isTextField(field)) {
      console.log(`Text: ${field.value.toUpperCase()}`); // value is string
    } else if (field.type === "number") {
      console.log(`Number: ${field.value.toFixed(2)}`); // value is number
    } else {
      console.log(`Select: ${field.selected} from ${field.options.join(", ")}`);
    }
  }
}
```

### Example 3: Event Handler Registration

```typescript
type ClickEvent = { type: "click"; x: number; y: number };
type KeyEvent = { type: "key"; key: string; code: string };
type FocusEvent = { type: "focus"; target: HTMLElement };

type UIEvent = ClickEvent | KeyEvent | FocusEvent;

function isClickEvent(event: UIEvent): event is ClickEvent {
  return event.type === "click";
}

function handleEvent(event: UIEvent) {
  if (isClickEvent(event)) {
    console.log(`Clicked at ${event.x}, ${event.y}`); // x, y available
  } else if (event.type === "key") {
    console.log(`Pressed: ${event.key}`); // key available
  } else {
    console.log("Focused element"); // FocusEvent
  }
}
```

### Example 4: Database Row Parsing

```typescript
interface UserRow { id: string; table: "users"; name: string; email: string; }
interface PostRow { id: string; table: "posts"; title: string; authorId: string; }
interface CommentRow { id: string; table: "comments"; body: string; postId: string; }

type DatabaseRow = UserRow | PostRow | CommentRow;

function processRow(row: DatabaseRow) {
  switch (row.table) {
    case "users":
      return row.email.toLowerCase(); // Only UserRow here
    case "posts":
      return row.title.length; // Only PostRow here
    case "comments":
      return row.body.split(" ").length; // Only CommentRow here
  }
}
```

### Example 5: Filtering Union Types

```typescript
interface Admin { role: "admin"; permissions: string[]; }
interface User { role: "user"; email: string; }
interface Guest { role: "guest"; expiresAt: Date; }

type Person = Admin | User | Guest;

// Extract only admins
function isAdmin(person: Person): person is Admin {
  return person.role === "admin";
}

const people: Person[] = getPeople();
const admins = people.filter(isAdmin);
// admins is Admin[]

// Extract non-guests
function isNotGuest(person: Person): boolean {
  return person.role !== "guest";
}

const registered = people.filter(isNotGuest);
// registered is (Admin | User)[]
```

## Best Practices

1. Return type must be `value is T` (type predicate)
2. Keep guards pure — no side effects
3. Test single responsibility — one guard per type
4. Use discriminated unions when possible — avoids complex guards
5. Use `switch` on discriminator for multiple cases
6. Chain guards with `filter` to extract union subsets
