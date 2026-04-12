# Tiering Rules

Detailed tier classification heuristics for `audit-ui-handlers`. SKILL.md references this file by name for scan-time lookup.

## Tier Definitions

### Tier 1 — Definitely Unfinished

Propose a diff. Confidence: high.

An element or handler is Tier 1 if ANY of these apply:

1. **Orphaned element** — an interactive element (`button`, `a`, custom `*Button`, `*Link`, etc.) has no interaction prop (`onClick`, `onSubmit`, `onChange`, `href`) and is not inside a `<form>` with its own `onSubmit` when `type="submit"` would apply.
2. **Empty body** — handler is literally `() => {}` or `function() {}` with no statements.
3. **Log-only body** — handler contains only `console.log(...)`, `console.warn(...)`, `console.error(...)`, or `alert(...)` and nothing else.
4. **TODO comment** — handler contains a comment matching `/stub|placeholder|not\s+implemented|fixme|todo/i` as its only substantive content.

### Tier 2 — Likely Unfinished

Propose a diff. Confidence: medium. Include a `Rationale` field explaining the signal.

An element or handler is Tier 2 if ANY of these apply:

1. **Action-verb label + state-only handler** — button text matches `/save|submit|delete|create|update|send|publish|confirm/i` AND the handler contains only `setState`/`useState` setter calls (no network call, no side effect).
2. **Form onSubmit missing an available API** — a `<form onSubmit={handler}>` where `handler` doesn't reference any function imported from `api/*`, `services/*`, `lib/api/*`, or `graphql/*`, AND such imports exist in the target file.
3. **One-level indirection into a Tier 1 body** — handler calls a named function that itself matches Tier 1.

### Tier 3 — Ambiguous

Flag only. No proposal. Include a `Note` field explaining why no proposal.

An element or handler is Tier 3 if ANY of these apply:

1. **Toggle-y label** — button text matches `/toggle|show|hide|close|open|menu|expand|collapse|filter/i` AND handler is state-only. Likely correct as-is.
2. **Untraceable handler** — handler is prop-drilled from a parent and exceeds the 2-level trace depth cap.
3. **Unknown callee** — handler calls a function that isn't defined in the target file and isn't imported with a traceable source.

### Clean

Do not touch. Do not report (except in the optional "not touching" summary list).

- Handler is fully traced within the 2-level cap
- Handler performs a non-trivial side effect (network call, store dispatch, navigation, etc.) OR an expected local-only operation for a toggle-y label

## Downgrade Rule

If a Tier 1 or Tier 2 finding has NO usable imports in the target file to build a proposal from, downgrade it to Tier 3 with this note:

> `Note: no in-file imports match the expected side effect. Could look wider for context; re-run after widening scope.`

This is the only way Tier 3 findings escape the strict imports-only scope. The skill still does not grep outside the target file.

## Examples

### Tier 1 — orphaned button

```tsx
<button>Delete Account</button>
```
Result: Tier 1. No onClick, no `type="submit"`. Propose `onClick={handleDelete}` and a `handleDelete` function body using any imported delete function.

### Tier 1 — log-only body

```tsx
const handleSave = () => { console.log('saved'); };
```
Result: Tier 1. Body is only a log. Propose a replacement using imported mutations and toast utilities.

### Tier 2 — state-only with action label

```tsx
const handleSubmit = (e) => {
  e.preventDefault();
  setForm(f => ({ ...f, saved: true }));
};
// button: <button type="submit">Save Settings</button>
```
Result: Tier 2. Label is an action verb; handler only updates state. If `updateUser` is imported, propose wrapping state update with a real call.

### Tier 3 — toggle label

```tsx
const toggleMenu = () => setOpen(v => !v);
// button: <button onClick={toggleMenu}>Toggle Filters</button>
```
Result: Tier 3. Toggle-y label + state-only handler = matches local UI mutation pattern. Flag, don't propose.

### Tier 3 — untraceable

```tsx
export function UserRow({ onDelete }: { onDelete: () => void }) {
  return <button onClick={onDelete}>Delete</button>;
}
```
Result: Tier 3 if `onDelete` is prop-drilled from a grandparent. Note the trace depth and stop.

### Clean — real side effect

```tsx
const handleRefresh = async () => {
  try { await updateUser(id, body); toast.success('ok'); }
  catch (e) { toast.error('fail'); }
};
```
Result: Clean. Do not touch.
