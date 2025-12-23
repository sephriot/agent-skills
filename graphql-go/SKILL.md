---
name: graphql-go
description: GraphQL development in Go. Use when working with GraphQL schemas, resolvers, mutations, or queries in Go projects. Enforces schema-first approach, structured mutation inputs, and resolver consistency.
---

# GraphQL Go Development

Schema-first GraphQL development in Go with consistent resolvers and structured inputs.

## Core Principles

1. **Schema is source of truth** - Update schema before implementing resolvers
2. **Single schema file** - Keep entire GraphQL schema in one file
3. **Resolvers must match schema** - Every schema field needs a corresponding resolver
4. **Use input structs** - Never raw parameters in mutations

## Mutation Input Pattern (Mandatory)

```go
// Create inputs: required fields are non-pointer, optional are pointers
type createUserArgs struct {
    Input createUserInput
}
type createUserInput struct {
    Name  string   // Required
    Email string   // Required
    Role  *string  // Optional
}

// Update inputs: all fields are pointers (only update if provided)
type updateUserArgs struct {
    ID    graphql.ID
    Input updateUserInput
}
type updateUserInput struct {
    Name  *string  // Optional
    Email *string  // Optional
}
```

## Schema Update Checklist

**New mutation**: Add to schema → Define input type → Create Go input/args structs → Implement resolver
**New query**: Add to schema → Create args struct if needed → Implement resolver
**Modify type**: Update schema → Update Go struct → Update all affected resolvers

## Best Practices

- Use globally unique IDs (base64 encoded `Type:ID`)
- Delegate authorization to business logic layer, not resolvers
- Evolve schema without versions (deprecate instead of remove)
- Non-null for always-available fields, nullable for fields that might fail
- Use cursor-based pagination (Connection pattern)

## Clarification Protocol

**NEVER assume.** Ask about: schema file location, nullability, error handling pattern, pagination needs, authorization requirements.
