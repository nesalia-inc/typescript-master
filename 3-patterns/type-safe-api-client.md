# Type-Safe API Client

End-to-end typed HTTP client where the type system enforces correct request/response shapes.

## Type-Safe Endpoint Definition

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";

type User = { id: string; name: string; email: string };
type Post = { id: string; title: string; body: string; authorId: string };

type EndpointConfig = {
  "/users": {
    GET: { response: User[] };
    POST: { body: Omit<User, "id">; response: User };
  };
  "/users/:id": {
    GET: { params: { id: string }; response: User };
    PUT: { params: { id: string }; body: Partial<Omit<User, "id">>; response: User };
    DELETE: { params: { id: string }; response: void };
  };
  "/posts": {
    GET: { query: { page?: number; limit?: number }; response: Post[] };
    POST: { body: Omit<Post, "id">; response: Post };
  };
  "/posts/:id": {
    GET: { params: { id: string }; response: Post };
    DELETE: { params: { id: string }; response: void };
  };
};
```

## Extracting Types from Config

```typescript
type ExtractParams<T> = T extends { params: infer P } ? P : Record<string, string>;
type ExtractBody<T> = T extends { body: infer B } ? B : never;
type ExtractQuery<T> = T extends { query: infer Q } ? Q : never;
type ExtractResponse<T> = T extends { response: infer R } ? R : never;
```

## The Typed API Client

```typescript
class APIClient<Config extends Record<string, Record<HTTPMethod, any>>> {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async request<
    Path extends keyof Config,
    Method extends keyof Config[Path]
  >(
    path: Path,
    method: Method,
    options?: {
      params?: ExtractParams<Config[Path][Method]>;
      query?: ExtractQuery<Config[Path][Method]>;
      body?: ExtractBody<Config[Path][Method]>;
    }
  ): Promise<ExtractResponse<Config[Path][Method]>> {
    let url = `${this.baseUrl}${path}`;

    // Interpolate params
    if (options?.params) {
      Object.entries(options.params).forEach(([key, value]) => {
        url = url.replace(`:${key}`, encodeURIComponent(String(value)));
      });
    }

    // Add query string
    if (options?.query) {
      const query = new URLSearchParams(
        options.query as Record<string, string>
      ).toString();
      url += `?${query}`;
    }

    const response = await fetch(url, {
      method,
      body: options?.body ? JSON.stringify(options.body) : undefined,
      headers: {
        "Content-Type": "application/json",
        ...(options?.body ? { "Content-Type": "application/json" } : {}),
      },
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return response.json() as Promise<ExtractResponse<Config[Path][Method]>>;
  }
}
```

## Usage

```typescript
const api = new APIClient<EndpointConfig>("https://api.example.com");

// GET /users — returns User[]
const users = await api.request("/users", "GET");

// POST /users — body is { name, email }, returns User
const newUser = await api.request("/users", "POST", {
  body: { name: "John", email: "john@example.com" },
});

// GET /users/:id — params: { id }, returns User
const user = await api.request("/users/:id", "GET", {
  params: { id: "123" },
});

// PUT /users/:id — params + body, returns User
const updated = await api.request("/users/:id", "PUT", {
  params: { id: "123" },
  body: { name: "Jane" },
});

// GET /posts with query params
const posts = await api.request("/posts", "GET", {
  query: { page: "1", limit: "10" },
});

// Error: 'email' is not in the POST body type
api.request("/users", "POST", {
  body: { name: "John" }, // Missing required 'email'
});
```

## Key Techniques

1. **Nested type config** — `Record<string, Record<HTTPMethod, HandlerConfig>>`
2. **Conditional type extraction** — `T extends { params: infer P } ? P : never`
3. **Template literal paths** — `"/users/:id"` with param interpolation
4. **Generic method signature** — `<Path, Method>` preserves exact types

## Benefits

- Compile-time checking of all request/response shapes
- IDE autocomplete for endpoints and fields
- Refactoring safety — rename a field and TypeScript catches mismatches
- Documentation through types
