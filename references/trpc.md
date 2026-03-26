# tRPC Reference

End-to-end type safety between client and server.

## Setup

### Server (index.ts)

```typescript
import { initTRPC } from "@trpc/server";

const t = initTRPC.create();

export const router = t.router;
export const publicProcedure = t.procedure;
```

### Router Definition

```typescript
export const appRouter = router({
  user: router({
    getById: publicProcedure
      .input(z.string())
      .query(async ({ input }) => {
        return db.user.findUnique({ where: { id: input } });
      }),

    create: publicProcedure
      .input(z.object({
        name: z.string(),
        email: z.string().email(),
      }))
      .mutation(async ({ input }) => {
        return db.user.create({ data: input });
      }),
  }),
});

export type AppRouter = typeof appRouter;
```

## Client

### Setup

```typescript
import { createTRPCProxyClient } from "@trpc/client";
import type { AppRouter } from "../server";

const client = createTRPCProxyClient<AppRouter>({
  links: [httpBatchLink({ url: "/api/trpc" })],
});
```

### Usage

```typescript
// Fully typed
const user = await client.user.getById.query("123");

// Mutation
const newUser = await client.user.create.mutate({
  name: "John",
  email: "john@example.com",
});
```

## Input Validation

```typescript
import { z } from "zod";

publicProcedure
  .input(
    z.object({
      id: z.string().uuid(),
      page: z.number().int().positive().default(1),
    })
  )
  .query(({ input }) => {
    // input.id is string
    // input.page is number, defaults to 1
  });
```

## Protected Procedures

```typescript
const t = initTRPC.context<Context>().create();

const router = t.router({
  protected: t.router({
    me: t.procedure.use(({ ctx, next }) => {
      if (!ctx.session) {
        throw new TRPCError({ code: "UNAUTHORIZED" });
      }
      return next({ ctx: { ...ctx, user: ctx.session.user } });
    }),

    updateProfile: t.protected
      .input(z.object({ name: z.string() }))
      .mutation(({ ctx, input }) => {
        return db.user.update({
          where: { id: ctx.user.id },
          data: input,
        });
      }),
  }),
});
```

## Error Handling

```typescript
import { TRPCError } from "@trpc/server";

throw new TRPCError({
  code: "NOT_FOUND",
  message: "User not found",
  cause: error,
});
```

Client-side:

```typescript
try {
  await client.user.getById.query("123");
} catch (error) {
  if (error instanceof TRPCClientError) {
    console.log(error.data.code); // "NOT_FOUND"
  }
}
```

## React Integration

```typescript
import { trpc } from "../utils/trpc";

function UserProfile({ id }: { id: string }) {
  const { data: user, isLoading } = trpc.user.getById.useQuery(id);

  const utils = trpc.useContext();
  const mutation = trpc.user.update.useMutation({
    onSuccess: () => utils.user.getById.invalidate(id),
  });

  if (isLoading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

## Best Practices

- Use Zod for all input validation
- Create a separate `AppRouter` type export
- Use protected procedures for authenticated routes
- Use `httpBatchLink` for production
- Invalidate queries after mutations
- Handle errors with TRPCError for type safety
