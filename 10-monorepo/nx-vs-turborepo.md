# Nx vs Turborepo

Choose the right monorepo tool for your project.

## Comparison

| Feature | Turborepo | Nx |
|---------|-----------|-----|
| Learning curve | Low | Medium-High |
| Speed | Fast | Fast (often faster) |
| Visualization | Basic (build graph) | Rich (project graph, affected) |
| Caching | Local + Remote | Local + Remote + Computation |
| Plugins | Minimal (use existing tools) | Rich plugin ecosystem |
| Generators | No | Yes (code generation) |
| CI complexity | Low | Higher |

## When to Choose Turborepo

- Simple structure with < 20 packages
- Need fast setup
- Already using Vercel or GitHub Actions
- Want minimal configuration
- Pure TypeScript/JavaScript project

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "lint": {}
  }
}
```

## When to Choose Nx

- Large monorepo with > 50 packages
- Complex dependency graphs
- Need computation caching (not just file caching)
- Want code generators
- Enterprise with multiple teams
- Need visualization of project dependencies

```bash
# Create Nx workspace
npx create-nx-workspace@latest myorg --preset=ts

# Generate library
npx nx generate @nx/js:lib mylib

# Run affected tests
npx nx affected --target=test
```

## Nx Features Worth the Complexity

### Affected Commands

```bash
# Only build what changed + dependencies
npx nx affected --target=build

# Only test affected
npx nx affected --target=test
```

### Project Graph

```bash
npx nx graph
# Opens interactive visualization
```

### Remote Caching

```bash
# Connect toNx Cloud
npx nx connect
```

## Performance

For large monorepos:

| Repo Size | Turborepo | Nx |
|-----------|-----------|-----|
| < 10 packages | Excellent | Excellent |
| 10-50 packages | Good | Excellent |
| > 50 packages | May struggle | Excellent |

## Migration

### Lerna to Turborepo

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    }
  }
}
```

### Lerna to Nx

```bash
# Install Nx
npm install --save-dev nx

# Generate workspace
npx nx init

# Run migration
npx nx migrate latest
```

## Key Takeaways

- Turborepo: simplicity and speed
- Nx: features and visualization for complex projects
- Both support remote caching
- Nx has higher learning curve but more power
