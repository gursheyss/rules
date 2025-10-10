**Always use Zod v4 syntax. Deprecated v3 APIs should not be used.**

## Error Customization

- **`message`** → **`error`**
  ```typescript
  // ❌ v3
  z.string().min(5, { message: "Too short" });
  
  // ✅ v4
  z.string().min(5, { error: "Too short" });
  ```

- **`invalid_type_error` / `required_error`** → **`error` function**
  ```typescript
  // ❌ v3
  z.string({ required_error: "Required", invalid_type_error: "Not a string" });
  
  // ✅ v4
  z.string({ error: (issue) => issue.input === undefined ? "Required" : "Not a string" });
  ```

- **`errorMap`** → **`error`**
  ```typescript
  // ❌ v3
  z.string({ errorMap: (issue, ctx) => ({ message: "Custom error" }) });
  
  // ✅ v4
  z.string({ error: (issue) => "Custom error" });
  ```

## String Formats

Use top-level functions instead of `.string()` methods:

```typescript
// ❌ v3 (deprecated)
z.string().email()
z.string().uuid()
z.string().url()
z.string().emoji()
z.string().base64()
z.string().base64url()
z.string().nanoid()
z.string().cuid()
z.string().cuid2()
z.string().ulid()
z.string().datetime()
z.string().ip()
z.string().cidr()

// ✅ v4
z.email()
z.uuid()
z.url()
z.emoji()
z.base64()
z.base64url()
z.nanoid()
z.cuid()
z.cuid2()
z.ulid()
z.iso.datetime()
z.ipv4() // or z.ipv6()
z.cidrv4() // or z.cidrv6()
```

## Objects

- **`.strict()`** → **`z.strictObject()`**
  ```typescript
  // ❌ v3
  z.object({ name: z.string() }).strict()
  
  // ✅ v4
  z.strictObject({ name: z.string() })
  ```

- **`.passthrough()`** → **`z.looseObject()`**
  ```typescript
  // ❌ v3
  z.object({ name: z.string() }).passthrough()
  
  // ✅ v4
  z.looseObject({ name: z.string() })
  ```

- **`.merge()`** → **`.extend()`**
  ```typescript
  // ❌ v3
  BaseSchema.merge(AdditionalSchema)
  
  // ✅ v4
  BaseSchema.extend(AdditionalSchema.shape)
  // or
  z.object({ ...BaseSchema.shape, ...AdditionalSchema.shape })
  ```

## Native Enums

- **`z.nativeEnum()`** → **`z.enum()`**
  ```typescript
  enum Color { Red = "red", Green = "green" }
  
  // ❌ v3
  z.nativeEnum(Color)
  
  // ✅ v4
  z.enum(Color)
  ```

## Records

Single argument no longer supported:

```typescript
// ❌ v3
z.record(z.string())

// ✅ v4
z.record(z.string(), z.string())
```

## Functions

New API with upfront `input` and `output`:

```typescript
// ❌ v3
z.function()
  .args(z.string(), z.number())
  .returns(z.boolean())

// ✅ v4
z.function({
  input: [z.string(), z.number()],
  output: z.boolean()
})
```

For async functions, use `.implementAsync()` instead of `.implement()`.

## Error Formatting

- **`error.format()`** → **`z.treeifyError()`**
- **`error.flatten()`** → **`z.treeifyError()`**

## Promises

**`z.promise()`** is deprecated. Just `await` the promise before parsing.

