# Route Resolution Patterns Reference

Framework-specific route-to-component mapping patterns for Preamble Step 1.2 (Resolve Route Scope). Consult this reference when resolving a target route to its component set.

## Next.js App Router

### Directory Conventions

```
app/
├── layout.tsx              # Root layout — always included
├── page.tsx                # / route
├── users/
│   ├── layout.tsx          # /users layout — included for all /users/* routes
│   ├── page.tsx            # /users route
│   └── [id]/
│       ├── layout.tsx      # /users/:id layout
│       ├── page.tsx        # /users/:id route
│       ├── loading.tsx     # Suspense boundary — include, may contain queries
│       └── error.tsx       # Error boundary — typically no queries, but check
```

**Entry point resolution:**
1. Match the target route to a directory path under `app/`
2. The entry component is `page.tsx` (or `page.js`, `page.jsx`) in the matched directory
3. Include all `layout.tsx` files from root to the matched directory (they wrap the page)
4. Include `loading.tsx` and `template.tsx` if present — they may contain data fetching
5. Ignore `not-found.tsx`, `error.tsx` unless the user specifically asks

### Route Groups and Parallel Routes

```
app/
├── (auth)/                 # Route group — no URL segment, but shares layout
│   ├── layout.tsx
│   ├── login/page.tsx
│   └── register/page.tsx
├── @sidebar/               # Parallel route — rendered alongside page
│   └── default.tsx
├── @main/
│   └── users/[id]/page.tsx
```

- **Route groups** `(name)`: Match by looking inside the group directory; the group name is NOT a URL segment
- **Parallel routes** `@name`: Include ALL parallel route segments for the matched path — they render simultaneously

### Data Fetching (App Router)

Components fetch data via:
- `async` server components with `fetch()` or direct DB/ORM calls
- Client components with `useQuery()`, `useSuspenseQuery()`, etc.
- Only trace GraphQL queries — ignore REST `fetch()` calls and direct DB access

## Next.js Pages Router

### Directory Conventions

```
pages/
├── _app.tsx                # Custom App — always included (wraps every page)
├── index.tsx               # / route
├── users/
│   ├── index.tsx           # /users route
│   └── [id].tsx            # /users/:id route
```

**Entry point resolution:**
1. Match the target route to a file under `pages/`
2. The entry component is the default export of the matched file
3. Always include `_app.tsx` — it wraps every page and may pass GraphQL providers/data
4. Include `_document.tsx` only if it contains GraphQL queries (rare)

### Server-Side Data Functions

```typescript
// getServerSideProps — runs on every request
export async function getServerSideProps(context) {
  const { data } = await client.query({ query: GET_USER });
  return { props: { user: data.user } };
}

// getStaticProps — runs at build time
export async function getStaticProps() {
  const { data } = await client.query({ query: GET_POSTS });
  return { props: { posts: data.posts } };
}
```

- Include GraphQL queries in `getServerSideProps` and `getStaticProps` — they feed data to the page component via props
- Trace the returned `props` object into the page component's prop usage

## React Router v5 / v6

### v6 Route Config (JSX)

```tsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="users" element={<UserList />} />
    <Route path="users/:id" element={<UserProfile />} />
  </Route>
</Routes>
```

**Entry point resolution:**
1. Search for route configuration: `<Routes>`, `<Route>`, `createBrowserRouter`, `createRoutesFromElements`
2. Match the target route path to a `<Route path="...">` element
3. The entry component is the `element` prop (v6) or `component`/`render` prop (v5)
4. Include parent route components (they render `<Outlet />` and may fetch data)
5. Check for `loader` functions (v6.4+) — they pre-fetch data

### v6 Data Router (Object Config)

```tsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    children: [
      { path: "users/:id", element: <UserProfile />, loader: userLoader },
    ],
  },
]);

async function userLoader({ params }) {
  const { data } = await client.query({ query: GET_USER, variables: { id: params.id } });
  return data;
}
```

- `loader` functions are entry points for data fetching — include their GraphQL queries
- Trace loader return values via `useLoaderData()` in the route component

### v5 Route Config

```tsx
<Switch>
  <Route exact path="/users/:id" component={UserProfile} />
  <Route path="/users" render={() => <UserList />} />
</Switch>
```

- `component` prop: the component itself is the entry point
- `render` prop: follow the function to find the rendered component

## Vue Router

### Route Config

```javascript
const routes = [
  {
    path: '/',
    component: Layout,
    children: [
      { path: '', component: Home },
      { path: 'users/:id', component: UserProfile },
    ],
  },
];

// Lazy loading
const routes = [
  {
    path: '/users/:id',
    component: () => import('./views/UserProfile.vue'),
  },
];
```

**Entry point resolution:**
1. Search for `createRouter`, `new VueRouter`, or route config arrays
2. Match the target route path to a route object's `path` property
3. The entry component is the `component` property (resolve lazy imports)
4. Include parent route components (they render `<router-view>`)
5. Check for navigation guards (`beforeEnter`) that may pre-fetch data

### Vue Data Fetching

```vue
<script setup>
import { useQuery } from '@vue/apollo-composable';
const { result } = useQuery(GET_USER);
// trace: result.value
</script>

<script>
export default {
  apollo: {
    user: { query: GET_USER },
    // trace: this.user (Smart Query)
  },
};
</script>
```

- Composition API: `useQuery` from `@vue/apollo-composable` or `@urql/vue`
- Options API: `apollo` option object (Vue Apollo Smart Queries)
- `asyncData` (Nuxt.js): runs before component render, return value merged into component data

## Angular Router

### Route Config

```typescript
const routes: Routes = [
  {
    path: '',
    component: LayoutComponent,
    children: [
      { path: 'users/:id', component: UserProfileComponent },
    ],
  },
];

// Lazy loading
const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./user-profile.component').then(m => m.UserProfileComponent),
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.adminRoutes),
  },
];
```

**Entry point resolution:**
1. Search for `RouterModule.forRoot`, `provideRouter`, or route config arrays
2. Match the target route path
3. The entry component is the `component` property (resolve `loadComponent` / `loadChildren`)
4. Include parent route components (they render `<router-outlet>`)
5. Check for route resolvers — they pre-fetch data before component activation

### Angular Data Fetching

```typescript
@Component({ ... })
export class UserProfileComponent {
  constructor(private apollo: Apollo) {}

  user$ = this.apollo.watchQuery({
    query: GET_USER,
  }).valueChanges;
  // trace: subscribe callback or async pipe usage
}

// Resolver
@Injectable()
export class UserResolver implements Resolve<User> {
  resolve(route: ActivatedRouteSnapshot) {
    return this.apollo.query({ query: GET_USER, variables: { id: route.params['id'] } });
  }
}
```

- `Apollo.watchQuery` / `Apollo.query` — trace the observable subscription
- Resolvers inject data via `ActivatedRoute.data` in the component

## General Resolution Steps

Regardless of framework, follow this process:

1. **Find the router config** — search for framework-specific route definition patterns
2. **Match the target route** — find the route entry whose path matches the user's target
3. **Resolve the entry component** — follow lazy imports if needed
4. **Collect ancestor components** — layouts, wrappers, parent routes that render the matched component
5. **Traverse one level of imports** — from each entry/ancestor component, follow imported child components one level deep
6. **Record the component set** — all files discovered in steps 3-5 form the scope boundary
