# Branded Types

Create nominal types to prevent mixing domain primitives that share the same underlying type.

## The Problem

TypeScript's structural type system allows this:

```typescript
type User = {
  id: string;
  name: string;
};

type Post = {
  id: string;
  ownerId: string;
  comments: Comment[];
};

type Comment = {
  id: string;
  authorId: string;
  body: string;
};

async function getCommentsForPost(postId: string, authorId: string) {
  const response = await api.get(`/author/${authorId}/posts/${postId}/comments`);
  return response.data;
}

const comments = await getCommentsForPost(user.id, post.id); // OK for TypeScript!
```

The arguments are swapped, but TypeScript sees only `string` for all IDs—it cannot catch this error at compile time.

## Naive Branded Types

Brand a type by intersecting with an object literal containing a `__brand` property:

```typescript
type Brand<K, T> = K & { __brand: T };

type UserId = Brand<string, "UserId">;
type PostId = Brand<string, "PostId">;

async function getCommentsForPost(postId: PostId, authorId: UserId) {
  // ...
}

const comments = getCommentsForPost(user.id, post.id);
// Error: Argument of type 'UserId' is not assignable to parameter of type 'PostId'
```

### Downsides of Naive Approach

- `__brand` is a "build-time only" property—it disappears at runtime
- The property shows up in IntelliSense, tempting developers to access it
- No protection against duplicate brands since `__brand` is just a string
- Anyone can manually create `{ __brand: "PostId" }` without validation

## Better Branded Types with Unique Symbol

Use `unique symbol` to create unreproducible brand markers:

```typescript
declare const __brand: unique symbol;

type Brand<B> = { [__brand]: B };

export type Branded<T, B> = T & Brand<B>;
```

**Why this is better:**

- `unique symbol` ensures the brand property cannot be manually created
- No string-based collision—each brand is a distinct symbol
- The property is still compile-time only (intellisense will show it less prominently)

Place this in a dedicated file to prevent read access to the symbol:

```typescript
// types/brand.ts
declare const __brand: unique symbol;

export type Brand<B> = { [__brand]: B };
export type Branded<T, B> = T & Brand<B>;
```

## Creating Brand Factories

Constructor functions enforce validation at the creation point:

```typescript
import type { Branded } from "./brand";

type UserId = Branded<string, "UserId">;
type PostId = Branded<string, "PostId">;

function createUserId(id: string): UserId {
  if (!/^[a-zA-Z0-9-]+$/.test(id)) {
    throw new Error("Invalid UserId format");
  }
  return id as UserId;
}

function createPostId(id: string): PostId {
  if (!/^POST-[a-zA-Z0-9]+$/.test(id)) {
    throw new Error("Invalid PostId format");
  }
  return id as PostId;
}
```

## Common Use Cases

### Domain Modeling

Prevent mixing primitives that share the same underlying type:

```typescript
type UserID = Branded<string, "UserID">;
type PostID = Branded<string, "PostID">;
type CommentID = Branded<string, "CommentID">;

type User = {
  id: UserID;
  name: string;
};

type Post = {
  id: PostID;
  ownerId: UserID;
  comments: Comment[];
};

type Comment = {
  id: CommentID;
  authorId: UserID;
  body: string;
};
```

### Currency

```typescript
type USD = Branded<number, "USD">;
type EUR = Branded<number, "EUR">;

function convertUSDToEUR(amount: USD, rate: number): EUR {
  return (amount * rate) as EUR;
}

const dollars = 100 as USD;
const euros = convertUSDToEUR(dollars, 0.85);
// euros is EUR—cannot accidentally mix with USD
```

### Validated Primitives

Create branded types that carry runtime validation:

```typescript
type EmailAddress = Branded<string, "EmailAddress">;

function validateEmail(email: string): EmailAddress | null {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return null;
  }
  return email as EmailAddress;
}

type Age = Branded<number, "Age">;

function createAge(n: number): Age {
  if (n < 0 || n > 125) {
    throw new Error("Age must be between 0 and 125");
  }
  return n as Age;
}

function getBirthYear(age: Age, currentYear: number): number {
  return currentYear - age;
}

const myAge = createAge(36);
const birthYear = getBirthYear(myAge, 2026); // 1990

getBirthYear(36, 2026); // Error: Argument of type 'number' is not assignable to parameter of type 'Age'
```

### API Responses

Model discriminated success/failure states:

```typescript
type ApiSuccess<T> = T & { __success: true };
type ApiFailure = {
  code: number;
  message: string;
  error: Error;
} & { __failure: true };

type ApiResponse<T> = ApiSuccess<T> | ApiFailure;

function isApiSuccess<T>(response: ApiResponse<T>): response is ApiSuccess<T> {
  return "__success" in response && response.__success === true;
}

const response = await fetchSomeEndpoint();

if (isApiSuccess(response)) {
  // response is ApiSuccess<T>
} else {
  // response is ApiFailure
}
```

## Unbranding

Extract the underlying type:

```typescript
type Unbrand<T> = T extends Branded<infer K, any> ? K : T;

type A = Unbrand<UserId>;  // string
type B = Unbrand<string>;  // string
```

## Key Takeaways

- Use for critical domain primitives where mixing would cause bugs
- Prefer `unique symbol` over string-based `__brand` for better safety
- Create factory functions that validate at the creation point
- Common uses: IDs, currency, email/URL, units of measure, API responses
- The brand property is compile-time only—it does not exist at runtime
