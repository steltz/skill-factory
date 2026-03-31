# Tracing Patterns Reference

Language-specific field-access patterns for Phase 2 (Trace). Consult this reference when encountering unfamiliar consumer code patterns.

## JavaScript / TypeScript

### Hook-Based Consumers (React)

```typescript
// Apollo Client
const { data, loading } = useQuery(GET_USER);
// trace: data

// Apollo useSuspenseQuery
const { data } = useSuspenseQuery(GET_USER);
// trace: data

// urql
const [result] = useQuery({ query: GET_USER });
// trace: result.data

// Relay
const data = useLazyLoadQuery(GetUserQuery, {});
// trace: data
// Relay also uses useFragment — trace the fragment ref
const user = useFragment(UserFragment, userRef);
// trace: user (all fragment fields potentially used)
```

### Render Props and HOCs

```typescript
// Apollo render prop (legacy)
<Query query={GET_USER}>
  {({ data }) => <Profile user={data.user} />}
</Query>
// trace: data in the render function, follow into <Profile>

// graphql() HOC (legacy)
const Enhanced = graphql(GET_USER)(Component);
// trace: props.data in Component
```

### Direct Client Calls

```typescript
// Apollo client.query
const { data } = await client.query({ query: GET_USER });
// trace: data

// graphql-request
const data = await request(endpoint, GET_USER);
// trace: data (top-level fields are query root fields)

// fetch + manual parsing
const res = await fetch('/graphql', { body: JSON.stringify({ query }) });
const { data } = await res.json();
// trace: data
```

### Fragment Patterns (JS/TS)

```typescript
// Colocated fragment with useFragment (Relay, Apollo 3.8+)
const user = useFragment(UserFieldsFragmentDoc, data.user);
// trace: user — all fields of UserFields are potentially accessed

// Fragment spread in query definition
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      ...UserFields
      extraField
    }
  }
  ${USER_FIELDS_FRAGMENT}
`;
// trace: data.user.extraField directly, UserFields fields via fragment tracing
```

## Python

### Client Libraries

```python
# graphql-core / Ariadne
result = graphql_sync(schema, query)
# trace: result.data

# gql (graphql-transport)
result = client.execute(query)
# trace: result (dict — field access via result["user"]["name"])

# Strawberry (async)
result = await schema.execute(query)
# trace: result.data

# sgqlc
op = Operation(Query)
op.user(id=1)
result = endpoint(op)
# trace: result.user (typed access)
```

### Field Access Patterns

```python
# Dict access
data["user"]["name"]  # → user.name used

# .get() with default
data.get("user", {}).get("name")  # → user.name used

# Unpacking
user = data["user"]
name, email = user["name"], user["email"]
# → user.name, user.email used

# Dataclass mapping
@dataclass
class User:
    name: str
    email: str
user = User(**data["user"])
# → if only name and email in dataclass, mark other user fields as unused
# → if **kwargs or extra fields accepted, mark as indeterminate
```

## Go

### Client Libraries

```go
// shurcooL/graphql
var query struct {
    User struct {
        Name  string
        Email string
    } `graphql:"user(id: $id)"`
}
err := client.Query(ctx, &query, vars)
// trace: query.User.Name, query.User.Email
// Go struct fields map directly to GraphQL fields — unused struct fields = unused query fields

// Khan/genqlient
resp, err := getUser(ctx, client, id)
// trace: resp.User.Name, resp.User.Email (generated types)

// machinebox/graphql
req := graphql.NewRequest(query)
var resp struct { User UserData }
err := client.Run(ctx, req, &resp)
// trace: resp.User fields
```

### Struct-Based Tracing

In Go, the struct definition IS the usage map. Fields defined in the struct are "used" by definition (Go will not compile with unused struct fields if they are unexported and accessed). Check which struct fields are actually accessed in subsequent code — a field present in the struct but never read after unmarshaling is still fetched but unused.

## General Patterns

### Spread Operators

```typescript
// Typed spread — all fields used
const profile: UserProfile = { ...data.user };
// → all fields of UserProfile type are used

// Untyped spread — indeterminate
const obj = { ...data.user };
// → mark all user fields as indeterminate (can't determine which are accessed)

// Partial spread with override — indeterminate
const merged = { ...data.user, name: "override" };
// → mark all user fields as indeterminate (spread is opaque)
```

### Serialization Sinks (Always Indeterminate)

These patterns make ALL fields on the path indeterminate:

- `JSON.stringify(data)` / `json.dumps(data)` / `json.Marshal(data)`
- `console.log(data)` / `print(data)` / `fmt.Println(data)`
- `localStorage.setItem("cache", JSON.stringify(data))`
- `sendToAnalytics(data)` (any opaque function call)
- `Object.keys(data.user)` / `Object.values(data.user)`
- `for (const key in data.user)` / `for key in data["user"]:`

### Cross-File Import Tracing

When the query consumer imports a function or component and passes data to it:

1. Follow the import to the target file
2. Read the function/component signature
3. Trace field access within that function (one level deep only)
4. If the function re-exports or passes data further, mark remaining fields as indeterminate
