# Type-Safe Form Validation

A validation system where rules and error types are fully inferred.

## Validation Rule Definition

```typescript
type ValidationRule<T> = {
  validate: (value: T) => boolean;
  message: string;
};

type FieldValidation<T> = {
  [K in keyof T]?: ValidationRule<T[K]>[];
};

type ValidationErrors<T> = {
  [K in keyof T]?: string[];
};
```

## The Form Validator

```typescript
class FormValidator<T extends Record<string, any>> {
  constructor(private rules: FieldValidation<T>) {}

  validate(data: T): ValidationErrors<T> | null {
    const errors: ValidationErrors<T> = {};
    let hasErrors = false;

    for (const key in this.rules) {
      const fieldRules = this.rules[key];
      const value = data[key];

      if (fieldRules) {
        const fieldErrors: string[] = [];

        for (const rule of fieldRules) {
          if (!rule.validate(value)) {
            fieldErrors.push(rule.message);
          }
        }

        if (fieldErrors.length > 0) {
          errors[key] = fieldErrors;
          hasErrors = true;
        }
      }
    }

    return hasErrors ? errors : null;
  }

  validateField<K extends keyof T>(
    key: K,
    value: T[K]
  ): string[] | undefined {
    const rules = this.rules[key];
    if (!rules) return undefined;

    return rules
      .filter((rule) => !rule.validate(value))
      .map((rule) => rule.message);
  }
}
```

## Usage

```typescript
interface LoginForm {
  email: string;
  password: string;
}

const validator = new FormValidator<LoginForm>({
  email: [
    {
      validate: (v) => typeof v === "string" && v.length > 0,
      message: "Email is required",
    },
    {
      validate: (v) => typeof v === "string" && v.includes("@"),
      message: "Email must contain @",
    },
    {
      validate: (v) =>
        typeof v === "string" &&
        /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
      message: "Invalid email format",
    },
  ],
  password: [
    {
      validate: (v) => typeof v === "string" && v.length >= 8,
      message: "Password must be at least 8 characters",
    },
    {
      validate: (v) =>
        typeof v === "string" && /[A-Z]/.test(v),
      message: "Password must contain uppercase letter",
    },
    {
      validate: (v) =>
        typeof v === "string" && /[0-9]/.test(v),
      message: "Password must contain number",
    },
  ],
});

// Validate entire form
const result = validator.validate({
  email: "invalid",
  password: "short",
});

if (result) {
  console.log(result.email);   // ["Email must contain @"]
  console.log(result.password); // ["Password must be at least 8 characters"]
}

// Validate single field
const emailErrors = validator.validateField("email", "valid@example.com");
// undefined (valid)
```

## Complex Nested Forms

```typescript
interface Address {
  street: string;
  city: string;
  zip: string;
}

interface RegistrationForm {
  name: string;
  email: string;
  address: Address;
}

const registrationValidator = new FormValidator<RegistrationForm>({
  name: [
    {
      validate: (v) => typeof v === "string" && v.length >= 2,
      message: "Name must be at least 2 characters",
    },
  ],
  email: [
    {
      validate: (v) => typeof v === "string" && v.includes("@"),
      message: "Invalid email",
    },
  ],
  address: [
    {
      validate: (v) =>
        typeof v === "object" &&
        v.street.length > 0 &&
        v.city.length > 0 &&
        v.zip.length > 0,
      message: "Address is incomplete",
    },
  ],
});
```

## Async Validation

```typescript
class AsyncFormValidator<T extends Record<string, any>> {
  constructor(private rules: FieldValidation<T>) {}

  async validateAsync(
    data: T
  ): Promise<ValidationErrors<T> | null> {
    const errors: ValidationErrors<T> = {};
    let hasErrors = false;

    for (const key in this.rules) {
      const fieldRules = this.rules[key];
      const value = data[key];

      if (fieldRules) {
        const fieldErrors: string[] = [];

        for (const rule of fieldRules) {
          const result = await rule.validate(value);
          if (!result) {
            fieldErrors.push(rule.message);
          }
        }

        if (fieldErrors.length > 0) {
          errors[key] = fieldErrors;
          hasErrors = true;
        }
      }
    }

    return hasErrors ? errors : null;
  }
}

// Usage with async validation
const asyncValidator = new AsyncFormValidator<{ email: string }>({
  email: [
    {
      validate: async (v) => {
        const response = await fetch(`/api/check-email?email=${v}`);
        const { available } = await response.json();
        return available;
      },
      message: "Email is already taken",
    },
  ],
});
```

## Key Techniques

1. **Generic constraint** — `T extends Record<string, any>`
2. **Mapped type for rules** — `{ [K in keyof T]?: ValidationRule<T[K]>[] }`
3. **Error type mirrors structure** — `{ [K in keyof T]?: string[] }`
4. **Rule composition** — Multiple rules per field, first failure stops evaluation

## Benefits

- Type-safe rules — validator knows exact form shape
- Inferred error types — no manual error type maintenance
- Composable rules — combine simple rules into complex validation
- Testable — each rule is a pure function
