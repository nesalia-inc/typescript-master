# Discriminated Unions

Type narrowing through a shared literal property. Essential for state machines and sum types.

## Basic Pattern

```typescript
type Success<T> = {
  status: "success";
  data: T;
};

type Error = {
  status: "error";
  error: string;
};

type Loading = {
  status: "loading";
};

type AsyncState<T> = Success<T> | Error | Loading;

function handleState<T>(state: AsyncState<T>): void {
  switch (state.status) {
    case "success":
      console.log(state.data); // Type: T
      break;
    case "error":
      console.log(state.error); // Type: string
      break;
    case "loading":
      console.log("Loading..."); // No data access
      break;
  }
}
```

## State Machine Pattern

```typescript
type State =
  | { type: "idle" }
  | { type: "fetching"; requestId: string }
  | { type: "success"; data: any }
  | { type: "error"; error: Error };

type Event =
  | { type: "FETCH"; requestId: string }
  | { type: "SUCCESS"; data: any }
  | { type: "ERROR"; error: Error }
  | { type: "RESET" };

function reducer(state: State, event: Event): State {
  switch (state.type) {
    case "idle":
      if (event.type === "FETCH") {
        return { type: "fetching", requestId: event.requestId };
      }
      return state;

    case "fetching":
      if (event.type === "SUCCESS") {
        return { type: "success", data: event.data };
      }
      if (event.type === "ERROR") {
        return { type: "error", error: event.error };
      }
      return state;

    case "success":
    case "error":
      if (event.type === "RESET") {
        return { type: "idle" };
      }
      return state;
  }
}
```

## Exhaustive Checking

```typescript
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(x)}`);
}

function processState(state: AsyncState<string>): string {
  switch (state.status) {
    case "success":
      return `Data: ${state.data}`;
    case "error":
      return `Error: ${state.error}`;
    case "loading":
      return "Loading...";
    default:
      return assertNever(state); // Compile error if a case is missed!
  }
}
```

## Real-World: API Response

```typescript
type ApiResponse<T> =
  | { success: true; data: T; timestamp: number }
  | { success: false; error: { code: string; message: string } };

function handleResponse<T>(response: ApiResponse<T>): T {
  if (response.success) {
    return response.data;
  } else {
    throw new Error(`${response.error.code}: ${response.error.message}`);
  }
}

// Usage
const response: ApiResponse<User[]> = await fetchUsers();
const users = handleResponse(response);
```

## Real-World: Tree Structure

```typescript
type TreeNode<T> =
  | { type: "leaf"; value: T }
  | { type: "branch"; children: TreeNode<T>[] };

function sum(tree: TreeNode<number>): number {
  if (tree.type === "leaf") {
    return tree.value;
  }
  return tree.children.reduce((acc, child) => acc + sum(child), 0);
}

function mapTree<T, U>(tree: TreeNode<T>, fn: (t: T) => U): TreeNode<U> {
  if (tree.type === "leaf") {
    return { type: "leaf", value: fn(tree.value) };
  }
  return {
    type: "branch",
    children: tree.children.map((child) => mapTree(child, fn)),
  };
}
```

## Combining with Mapped Types

```typescript
type Result<T, E = string> =
  | { ok: true; value: T }
  | { ok: false; error: E };

type Results<T> = {
  [K in keyof T]: Result<T[K]>;
};

type FormResults = Results<{ name: string; email: string; age: number }>;
// {
//   name: Result<string>;
//   email: Result<string>;
//   age: Result<number>;
// }

function getFirstError(results: FormResults): string | undefined {
  for (const key in results) {
    if (!results[key].ok) {
      return results[key].error;
    }
  }
  return undefined;
}
```

## Key Techniques

1. **Discriminant property** — Literal type (`status: "success"`) shared across all variants
2. **Exhaustive narrowing** — Switch on discriminant, `default` asserts `never`
3. **Never for dead code** — `assertNever(x: never)` causes compile error on unhandled cases
4. **Pattern matching via switch** — Each branch narrows the type automatically

## Benefits

- Impossible states are unrepresentable
- No null checks needed when discriminated properly
- IDE autocomplete within each branch
- Refactoring safety — adding a new variant causes compile errors everywhere not handling it
