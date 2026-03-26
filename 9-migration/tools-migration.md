# Tools Migration

Transitioning between TypeScript-related tools.

## Biome vs ESLint

### When to Choose Biome

- Speed is critical (Biome is often 10-100x faster)
- Want single tool for lint + format
- TypeScript-first project
- Okay with fewer rules (64 TS rules vs 100+ in typescript-eslint)

### When to Stay with ESLint

- Need specific rules or plugins
- Have complex custom rules
- Working with Vue/Angular (limited Biome support)
- Need type-aware linting (Biome doesn't have this yet)

### Migration to Biome

```bash
# Install Biome
npm install --save-dev @biomejs/biome

# Initialize config
npx biome init

# Convert ESLint config (optional)
npx biome migrate eslint

# Run
npx biome check .
npx biome format .
```

### Biome Configuration

```json
// biome.json
{
  "$schema": "https://biomejs.dev/schemas/1.0.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedVariables": "warn"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  }
}
```

## Lerna to Turborepo

### When to Choose Turborepo

- Simple structure (< 20 packages)
- Need speed
- GitHub Actions focused

### When to Choose Nx

- Complex dependencies
- Need visualization
- Want plugins for custom tooling
- Large monorepo (> 50 packages)

### Migration to Turborepo

```bash
# Install
npm install --save-dev turbo

# Create turbo.json
npx turbo init

# Structure
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": []
    },
    "lint": {
      "outputs": []
    }
  }
}
```

## TSC Linting to Type-Check Only

If using TSC for linting and it's slow:

```json
// Remove from tsconfig.json
{
  "compilerOptions": {
    // "noUnusedLocals": true,  // Remove
    // "noUnusedParameters": true  // Remove
  }
}
```

Use Biome or ESLint for linting instead:

```bash
# Biome is fastest for pure linting
npm install --save-dev @biomejs/biome
npx biome check .
```

## Key Takeaways

- Biome for speed, ESLint for features
- Turborepo for simple monorepos, Nx for complex ones
- Separate linting from type checking
- Migration is usually 1 day to 1 week
