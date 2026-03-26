# Biome vs ESLint

Choose the right linting tool for your TypeScript project.

## Comparison

| Feature | Biome | ESLint + typescript-eslint |
|---------|-------|---------------------------|
| Speed | 10-100x faster | Slower |
| Rules | ~64 TS rules | ~100+ TS rules |
| Type-aware | No | Yes |
| Formatting | Yes | Separate (Prettier) |
| Plugins | Limited | Rich ecosystem |
| Vue/Angular | Limited | Full support |

## When to Use Biome

### Simple TypeScript Project

```bash
npm install --save-dev @biomejs/biome
npx biome init
```

```json
// biome.json
{
  "organizeImports": { "enabled": true },
  "linter": { "enabled": true, "rules": { "recommended": true } },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "lineWidth": 100
  }
}
```

### CI Configuration

```yaml
- name: Lint with Biome
  run: npx biome ci src
  # CI mode fails on warnings
```

## When to Use ESLint

### Type-Aware Linting Required

```bash
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

```javascript
// .eslintrc.js
module.exports = {
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn"
  }
};
```

### Custom Rules or Plugins

ESLint's plugin ecosystem is much larger. If you need:
- Vue-specific linting → eslint-plugin-vue
- Angular-specific → eslint-plugin-angular
- Custom domain rules → Write ESLint plugin

## Migration from ESLint to Biome

```bash
# Install Biome
npm install --save-dev @biomejs/biome

# Migrate config (optional)
npx biome migrate eslint biome.json

# Run ESLint to find issues Biome doesn't catch
npx biome check .
```

## Biome Limitations

```typescript
// Biome won't catch this (no type-aware linting)
function process(value: any) {
  return value.nonExistentMethod();
}
```

For type-aware checks, you'll still need `tsc --noEmit` or ESLint.

## Combining Tools

```yaml
# CI: Fast check with Biome, type check with tsc
- name: Lint (fast)
  run: npx biome check src

- name: Type check
  run: npx tsc --noEmit
```

## Key Takeaways

- Biome: speed, simplicity, combined lint + format
- ESLint: features, type-awareness, ecosystem
- For pure TS speed: Biome
- For complex projects: ESLint
- Use both: Biome for fast feedback, ESLint for deep analysis
