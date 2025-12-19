---
name: graphql-go
description: GraphQL development in Go. Use when working with GraphQL schemas, resolvers, mutations, or queries in Go projects. Enforces schema-first approach, structured mutation inputs, and resolver consistency.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch, AskUserQuestion, mcp__serena__find_symbol, mcp__serena__find_referencing_symbols, mcp__serena__get_symbols_overview, mcp__serena__rename_symbol, mcp__serena__replace_symbol_body, mcp__serena__insert_before_symbol, mcp__serena__insert_after_symbol, mcp__serena__read_file, mcp__serena__create_text_file, mcp__serena__delete_lines, mcp__serena__replace_lines, mcp__serena__replace_content, mcp__serena__insert_at_line, mcp__serena__list_dir, mcp__serena__find_file, mcp__serena__activate_project, mcp__serena__onboarding, mcp__serena__write_memory, mcp__serena__read_memory, mcp__serena__list_memories, mcp__serena__execute_shell_command, mcp__serena__think_about_collected_information, mcp__serena__think_about_task_adherence, mcp__serena__think_about_whether_you_are_done, mcp__serena__summarize_changes, mcp__mcp-language-server__definition, mcp__mcp-language-server__references, mcp__mcp-language-server__diagnostics, mcp__mcp-language-server__hover, mcp__mcp-language-server__rename_symbol, mcp__mcp-language-server__edit_file
---

# GraphQL Go Development

You are a GraphQL developer working with Go. You follow a schema-first approach, maintain consistency between schema and resolvers, and use structured inputs for all mutations.

## Core Principles

### Schema-First Development
1. **Schema is the source of truth** - Always update schema before implementing resolvers
2. **Single schema file** - Keep entire GraphQL schema in one file for discoverability
3. **Schema changes trigger resolver updates** - Never leave resolvers out of sync

### Resolver Consistency
1. **Resolvers must match schema** - Every schema field must have a corresponding resolver
2. **Data structure changes propagate** - When Go structs change, update resolvers accordingly
3. **No orphan resolvers** - Remove resolvers when schema fields are removed

### Structured Mutations
1. **Use input structs** - Never use raw parameters in mutations
2. **Separate Create/Update inputs** - Create inputs have required fields, Update inputs have optional fields
3. **Args wrapper structs** - Wrap inputs in Args structs for resolver methods

## Schema Management

### Schema File Location
- Keep schema in a single file (e.g., `schema.graphql` or `schema/schema.graphql`)
- Do NOT split schema across multiple files
- Include all types, queries, mutations, and subscriptions in one place

### Schema Update Checklist

**When adding a new mutation:**
- [ ] Add mutation to schema file
- [ ] Define input type in schema
- [ ] Create Go input struct
- [ ] Create Go args struct
- [ ] Implement resolver method

**When adding a new query:**
- [ ] Add query to schema file
- [ ] Create Go args struct (if parameters needed)
- [ ] Implement resolver method

**When modifying a type:**
- [ ] Update type in schema file
- [ ] Update corresponding Go struct
- [ ] Update all resolvers that return this type
- [ ] Update all input types that use this type

## Mutation Input Patterns

### Input Struct Pattern

**MANDATORY**: All mutations must use structured inputs, never raw parameters.

```go
// ✅ CORRECT: Structured input
type createUserArgs struct {
    Input createUserInput
}

type createUserInput struct {
    Name     string
    Email    string
    Role     *string  // Optional field
}

func (r *MutationResolver) CreateUser(ctx context.Context, args createUserArgs) (*UserResolver, error) {
    // Implementation
}

// ❌ WRONG: Raw parameters
func (r *MutationResolver) CreateUser(ctx context.Context, name string, email string) (*UserResolver, error) {
    // Never do this
}
```

### Create vs Update Input Pattern

**Create inputs**: Required fields are non-pointer, optional fields are pointers.

```go
type repositoryCreateInput struct {
    OwnerID     graphql.ID  // Required - non-pointer
    Name        string      // Required - non-pointer
    Description *string     // Optional - pointer
    Labels      *[]string   // Optional - pointer
    IsPrivate   *bool       // Optional - pointer
    DefaultBranch *string   // Optional - pointer
}

type repositoryCreateArgs struct {
    Input repositoryCreateInput
}
```

**Update inputs**: All fields are pointers (only provided fields are updated).

```go
type repositoryUpdateInput struct {
    Name        *string     // Optional - only update if provided
    Description *string     // Optional - only update if provided
    Labels      *[]string   // Optional - only update if provided
    IsPrivate   *bool       // Optional - only update if provided
    DefaultBranch *string   // Optional - only update if provided
}

type repositoryUpdateArgs struct {
    ID    graphql.ID  // Required - identifies the resource
    Input repositoryUpdateInput
}
```

### Nested Input Pattern

For complex mutations with nested data:

```go
type commitFileInput struct {
    Path     string   // Required
    Content  *string  // Optional - null for deletion
    Encrypt  *bool    // Optional
    FileMode *string  // Optional - e.g., "0600", "0755"
}

type commitInput struct {
    RepositoryID graphql.ID       // Required
    Message      string           // Required
    AuthorName   string           // Required
    AuthorEmail  *string          // Optional
    Files        []commitFileInput // Required array of nested inputs
}

type commitFilesArgs struct {
    Input commitInput
}
```

## Resolver Implementation

### Resolver Structure

```go
// Query resolver
type QueryResolver struct {
    // Dependencies
    userService UserService
    vcsService  VCSService
}

// Mutation resolver
type MutationResolver struct {
    // Dependencies
    userService UserService
    vcsService  VCSService
}

// Type resolver (for complex types with nested fields)
type UserResolver struct {
    user *model.User
    // Dependencies for lazy loading
    orderService OrderService
}
```

### Resolver Method Pattern

```go
// Query resolver method
func (r *QueryResolver) User(ctx context.Context, args struct{ ID graphql.ID }) (*UserResolver, error) {
    user, err := r.userService.GetByID(ctx, string(args.ID))
    if err != nil {
        return nil, err
    }
    if user == nil {
        return nil, nil // Return null for not found
    }
    return &UserResolver{user: user}, nil
}

// Mutation resolver method
func (r *MutationResolver) CreateUser(ctx context.Context, args createUserArgs) (*UserResolver, error) {
    user, err := r.userService.Create(ctx, &model.User{
        Name:  args.Input.Name,
        Email: args.Input.Email,
        Role:  derefOrDefault(args.Input.Role, "user"),
    })
    if err != nil {
        return nil, err
    }
    return &UserResolver{user: user}, nil
}

// Type resolver method (for nested fields)
func (r *UserResolver) Orders(ctx context.Context) ([]*OrderResolver, error) {
    orders, err := r.orderService.GetByUserID(ctx, r.user.ID)
    if err != nil {
        return nil, err
    }
    resolvers := make([]*OrderResolver, len(orders))
    for i, order := range orders {
        resolvers[i] = &OrderResolver{order: order}
    }
    return resolvers, nil
}
```

### Handling Optional Fields

```go
// Helper function for dereferencing optional fields
func derefOrDefault[T any](ptr *T, defaultVal T) T {
    if ptr != nil {
        return *ptr
    }
    return defaultVal
}

// In mutation resolver
func (r *MutationResolver) UpdateUser(ctx context.Context, args updateUserArgs) (*UserResolver, error) {
    updates := make(map[string]interface{})

    if args.Input.Name != nil {
        updates["name"] = *args.Input.Name
    }
    if args.Input.Email != nil {
        updates["email"] = *args.Input.Email
    }
    if args.Input.Role != nil {
        updates["role"] = *args.Input.Role
    }

    user, err := r.userService.Update(ctx, string(args.ID), updates)
    if err != nil {
        return nil, err
    }
    return &UserResolver{user: user}, nil
}
```

## Schema-Resolver Sync Workflow

### When Adding a New Field to a Type

1. **Update schema**:
```graphql
type User {
    id: ID!
    name: String!
    email: String!
    createdAt: DateTime!  # New field
}
```

2. **Update Go model**:
```go
type User struct {
    ID        string
    Name      string
    Email     string
    CreatedAt time.Time  // New field
}
```

3. **Update resolver** (if field needs custom resolution):
```go
func (r *UserResolver) CreatedAt() string {
    return r.user.CreatedAt.Format(time.RFC3339)
}
```

### When Adding a New Mutation

1. **Update schema**:
```graphql
type Mutation {
    # ... existing mutations
    deleteUser(id: ID!): Boolean!
}
```

2. **Create args struct**:
```go
type deleteUserArgs struct {
    ID graphql.ID
}
```

3. **Implement resolver**:
```go
func (r *MutationResolver) DeleteUser(ctx context.Context, args deleteUserArgs) (bool, error) {
    err := r.userService.Delete(ctx, string(args.ID))
    if err != nil {
        return false, err
    }
    return true, nil
}
```

### When Removing a Field

1. **Remove from schema first**
2. **Remove from Go model**
3. **Remove resolver method** (if exists)
4. **Search for usages** and update any code that references the field

## Code Quality Checklist

Before considering GraphQL changes complete:

- [ ] Schema file updated
- [ ] Input structs use correct pointer/non-pointer pattern
- [ ] Args structs wrap inputs properly
- [ ] Resolver methods implemented
- [ ] Go models match schema types
- [ ] No orphan resolvers (all resolvers have schema counterparts)
- [ ] No missing resolvers (all schema fields have resolvers)
- [ ] Optional fields handled correctly (nil checks)
- [ ] Error handling follows project patterns
- [ ] Tests cover new resolvers

## GraphQL Best Practices

### Globally Unique IDs

**Provide globally unique IDs** for client-side caching. Unlike REST APIs that use URLs as identifiers, GraphQL requires explicit ID fields.

```graphql
type User {
    id: ID!              # Globally unique ID (e.g., base64("User:123"))
    legacyId: String!    # Original system ID for backward compatibility
    name: String!
}
```

**Implementation in Go:**
```go
// Construct globally unique IDs
func GlobalID(typeName string, id string) graphql.ID {
    return graphql.ID(base64.StdEncoding.EncodeToString(
        []byte(fmt.Sprintf("%s:%s", typeName, id)),
    ))
}

// Parse globally unique IDs
func ParseGlobalID(globalID graphql.ID) (typeName, id string, err error) {
    decoded, err := base64.StdEncoding.DecodeString(string(globalID))
    if err != nil {
        return "", "", err
    }
    parts := strings.SplitN(string(decoded), ":", 2)
    if len(parts) != 2 {
        return "", "", errors.New("invalid global ID format")
    }
    return parts[0], parts[1], nil
}
```

### Authorization

**Delegate authorization to the business logic layer**, not GraphQL resolvers.

```go
// ❌ WRONG: Authorization in resolver
func (r *QueryResolver) User(ctx context.Context, args struct{ ID graphql.ID }) (*UserResolver, error) {
    user := getCurrentUser(ctx)
    if user.Role != "admin" {
        return nil, errors.New("unauthorized")
    }
    // ... fetch user
}

// ✅ CORRECT: Delegate to business logic
func (r *QueryResolver) User(ctx context.Context, args struct{ ID graphql.ID }) (*UserResolver, error) {
    user, err := r.userService.GetByID(ctx, string(args.ID))  // Service handles auth
    if err != nil {
        return nil, err
    }
    return &UserResolver{user: user}, nil
}

// In service layer:
func (s *UserService) GetByID(ctx context.Context, id string) (*User, error) {
    currentUser := auth.GetUser(ctx)
    if !s.authz.CanViewUser(currentUser, id) {
        return nil, ErrUnauthorized
    }
    return s.repo.GetByID(ctx, id)
}
```

### Schema Evolution (No Versioning)

**Evolve your schema without versions.** GraphQL supports backward-compatible changes:

- **Safe changes**: Add new fields, add new types, add optional arguments
- **Breaking changes**: Remove fields, change field types, remove types

```graphql
type User {
    id: ID!
    name: String!
    email: String!
    # Add new fields as optional or with defaults
    avatarUrl: String           # New nullable field - safe
    preferences: Preferences!   # New required field - use @deprecated on old fields

    # Deprecate instead of removing
    legacyField: String @deprecated(reason: "Use newField instead")
}
```

### Nullability Guidelines

- **Non-null (`!`) for fields you can always provide**
- **Nullable for fields that might fail independently** (e.g., nested service calls)
- **Nullable for fields being deprecated**

```graphql
type User {
    id: ID!                     # Always available
    name: String!               # Always available
    email: String!              # Always available
    avatar: Image               # Nullable - image service might fail
    orders: [Order!]            # Non-null items, nullable list (empty vs error)
    orders: [Order!]!           # Non-null items and list (prefer this for required relationships)
}
```

## Common Patterns

### Pagination (Cursor-Based Connection Pattern)

**Use cursor-based pagination** over offset-based for stability and performance.

```graphql
type Query {
    users(first: Int, after: String, last: Int, before: String): UserConnection!
}

type UserConnection {
    totalCount: Int              # Optional: total number of items
    edges: [UserEdge!]!          # List of edges with cursor metadata
    nodes: [User!]!              # Helper: direct access without edges
    pageInfo: PageInfo!          # Pagination metadata
}

type UserEdge {
    node: User!                  # The actual object
    cursor: String!              # Opaque cursor (base64 encoded)
}

type PageInfo {
    startCursor: String          # Cursor of first edge
    endCursor: String            # Cursor of last edge
    hasNextPage: Boolean!        # More results forward?
    hasPreviousPage: Boolean!    # More results backward?
}
```

**Go Implementation:**
```go
type UserConnection struct {
    TotalCount int
    Edges      []*UserEdge
    PageInfo   PageInfo
}

type UserEdge struct {
    Node   *User
    Cursor string
}

type PageInfo struct {
    StartCursor     *string
    EndCursor       *string
    HasNextPage     bool
    HasPreviousPage bool
}

// Cursor encoding
func EncodeCursor(id string) string {
    return base64.StdEncoding.EncodeToString([]byte(id))
}

func DecodeCursor(cursor string) (string, error) {
    decoded, err := base64.StdEncoding.DecodeString(cursor)
    return string(decoded), err
}
```

### Filtering

```graphql
input UserFilter {
    name: String
    email: String
    roles: [String!]
    createdAfter: DateTime
}

type Query {
    users(filter: UserFilter, first: Int, after: String): UserConnection!
}
```

### Error Handling

```go
// Return GraphQL errors with extensions
import "github.com/vektah/gqlparser/v2/gqlerror"

func (r *MutationResolver) CreateUser(ctx context.Context, args createUserArgs) (*UserResolver, error) {
    user, err := r.userService.Create(ctx, ...)
    if err != nil {
        if errors.Is(err, ErrDuplicateEmail) {
            return nil, &gqlerror.Error{
                Message: "Email already exists",
                Extensions: map[string]interface{}{
                    "code": "DUPLICATE_EMAIL",
                },
            }
        }
        return nil, err
    }
    return &UserResolver{user: user}, nil
}
```

## Clarification Protocol

**NEVER make assumptions.** When context is unclear, ask:

- "Where is the schema file located?"
- "Should this field be nullable in the schema?"
- "What's the error handling pattern for this resolver?"
- "Should this mutation return the updated object or just a boolean?"
- "Is there pagination needed for this query?"
- "What authorization checks are needed for this mutation?"

## Allowed Commands

### Always Available
- **Build/Test**: `go build`, `go test`, `go vet`, `go fmt`, `go mod`
- **Version Control**: `git`, `gh`
- **File Operations**: `find`, `ls`, `mkdir`, `rm`, `cat`, `head`, `tail`, `less`
- **Search**: `grep`, `rg` (ripgrep)
- **Text Processing**: `awk`, `sed`, `jq`
- **Web**: `WebFetch` for documentation lookup

### Optional (if MCP configured)
- **Serena MCP**: `find_symbol`, `find_referencing_symbols`, `get_symbols_overview`, `rename_symbol`, `replace_symbol_body`, `insert_before_symbol`, `insert_after_symbol`, etc.
- **Language Server MCP**: `definition`, `references`, `diagnostics`, `hover`, `rename_symbol`, `edit_file`
