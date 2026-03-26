# JavaScript to TypeScript Migration

Incremental migration strategy for existing JavaScript codebases.

## Phase 1: Enable TypeScript

```json
// tsconfig.json - add to existing config
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "outDir": "./dist"
  }
}
```

Now TypeScript will:
- Type-check existing JS files
- Provide autocomplete for JS
- Not break the build

## Phase 2: Rename Files Incrementally

```bash
# Rename .js → .ts gradually
mv src/utils.js src/utils.ts

# For JSX files
mv src/Component.jsx src/Component.tsx
```

## Phase 3: Add Types

Start with the boundaries:

```javascript
// Before: utils.js
export function processData(data) {
  return data.map(item => item.value);
}
```

```typescript
// After: utils.ts
interface DataItem {
  value: string;
}

export function processData(data: DataItem[]): string[] {
  return data.map((item) => item.value);
}
```

## Phase 4: Enable Strict Mode

Enable features one at a time:

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true
  }
}
```

Then eventually:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

## Automated Helpers

```bash
# Install missing @types packages
npx typesync

# Automated migration (limited)
npx ts-migrate migrate . --sources 'src/**/*.js'
```

## TypeScript in Existing Frameworks

### React

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

Rename `.jsx` to `.tsx` as you convert components.

### Node.js

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext"
  }
}
```

### Vue

Use Volar extension and `vue-tsc`:

```bash
npx vue-tsc --noEmit
```

## Common Patterns

### any to unknown

```javascript
// Before
function parseJSON(json) {
  return JSON.parse(json);
}

// After
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}
```

### null/undefined handling

```typescript
// Before
function getName(user) {
  return user && user.profile && user.profile.name;
}

// After
function getName(user: User | null | undefined): string | undefined {
  return user?.profile?.name;
}
```

## Key Takeaways

- Start with `allowJs: true`
- Rename incrementally, don't do big bang
- Add types at boundaries first
- Enable strict mode gradually
- Use automated tools for @types
