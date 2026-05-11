---
name: react-native-testing-patterns
description: RNTL core patterns — query priority, render/screen, userEvent vs fireEvent, waitFor, Jest matchers, 11 rules.
---

# RNTL Core Patterns

## Query Priority

Use in this order (most to least preferred):

1. `getByRole` — most accessibility-aware
2. `getByLabelText` — form inputs with labels
3. `getByPlaceholderText` — inputs without labels
4. `getByText` — non-interactive text
5. `getByDisplayValue` — current value of inputs
6. `getByTestId` — last resort only

## Query Variants

| Variant | Use case | Returns | Async |
|---------|----------|---------|-------|
| `getBy*` | Element must exist | element instance (throws) | No |
| `getAllBy*` | Multiple must exist | element instance[] (throws) | No |
| `queryBy*` | Check non-existence ONLY | element instance \| null | No |
| `queryAllBy*` | Count elements | element instance[] | No |
| `findBy*` | Wait for element | `Promise<element instance>` | Yes |
| `findAllBy*` | Wait for multiple | `Promise<element instance[]>` | Yes |

## Interactions: userEvent (Preferred)

`userEvent` is **always async** — always `await`:

```tsx
const user = userEvent.setup();

await user.press(element);                              // full press sequence
await user.longPress(element, { duration: 800 });       // long press
await user.type(textInput, 'Hello');                    // char-by-char typing
await user.clear(textInput);                            // clear TextInput
await user.paste(textInput, 'pasted text');             // paste
await user.scrollTo(scrollView, { y: 100 });            // scroll
```

## fireEvent (Fallback)

Use only when `userEvent` doesn't support the event. Sync in v13, async in v14:

```tsx
fireEvent.press(element);                  // v13 sync, v14 must await
fireEvent.changeText(textInput, 'new');    // v13 sync, v14 must await
fireEvent(element, 'blur');                // arbitrary event
```

## Assertions (Jest Matchers)

Available automatically with any `@testing-library/react-native` import.

| Matcher | Use for |
|---------|---------|
| `toBeOnTheScreen()` | Element exists in tree |
| `toBeVisible()` | Element visible (not hidden / display:none) |
| `toBeEnabled()` / `toBeDisabled()` | Disabled state via `aria-disabled` |
| `toBeChecked()` / `toBePartiallyChecked()` | Checked state |
| `toBeSelected()` | Selected state |
| `toBeExpanded()` / `toBeCollapsed()` | Expanded state |
| `toBeBusy()` | Busy state |
| `toHaveTextContent(text)` | Text content match |
| `toHaveDisplayValue(value)` | TextInput display value |
| `toHaveAccessibleName(name)` | Accessible name |
| `toHaveAccessibilityValue(val)` | Accessibility value |
| `toHaveStyle(style)` | Style match |
| `toHaveProp(name, value?)` | Prop check (last resort) |
| `toContainElement(el)` | Contains child element |
| `toBeEmptyElement()` | No children |

## 11 Core Rules

1. **Use `screen`** for queries — never destructure from `render()`
2. **Use `getByRole` first** with `{ name: '...' }` option
3. **Use `queryBy*` ONLY** for `.not.toBeOnTheScreen()` checks
4. **Use `findBy*`** for async elements — NOT `waitFor` + `getBy*`
5. **Never put side-effects in `waitFor`** (no `fireEvent`/`userEvent` inside)
6. **One assertion per `waitFor`**
7. **Never pass empty callbacks to `waitFor`**
8. **Don't wrap in `act()`** — `render`, `fireEvent`, `userEvent` handle it
9. **Don't call `cleanup()`** — automatic after each test
10. **Prefer ARIA props** (`role`, `aria-label`, `aria-disabled`) over legacy `accessibility*` props
11. **Use RNTL matchers** over raw prop assertions

## `*ByRole` Quick Reference

Common roles: `button`, `text`, `heading` (alias: `header`), `searchbox`, `switch`, `checkbox`, `radio`, `img`, `link`, `alert`, `menu`, `menuitem`, `tab`, `tablist`, `progressbar`, `slider`, `spinbutton`, `timer`, `toolbar`.

`getByRole` options: `{ name, disabled, selected, checked, busy, expanded, value: { min, max, now, text } }`.

For `*ByRole` to match, the element must be an accessibility element:
- `Text`, `TextInput`, `Switch` are by default
- `View` needs `accessible={true}` (or use `Pressable` / `TouchableOpacity`)

## waitFor

```tsx
// Correct: action first, then wait for result
fireEvent.press(button);
await waitFor(() => {
  expect(screen.getByText('Result')).toBeOnTheScreen();
});

// Better: use findBy* instead
fireEvent.press(button);
expect(await screen.findByText('Result')).toBeOnTheScreen();
```

Options: `waitFor(cb, { timeout: 1000, interval: 50 })`. Works with Jest fake timers automatically.

## Fake Timers

Recommended with `userEvent` (press/longPress involve real durations):

```tsx
jest.useFakeTimers();

test('with fake timers', async () => {
  const user = userEvent.setup();
  render(<Component />);
  await user.press(screen.getByRole('button'));
  // ...
});
```

## Custom Render (Provider Wrappers)

```tsx
function renderWithProviders(ui: React.ReactElement) {
  return render(ui, {
    wrapper: ({ children }) => (
      <ThemeProvider>
        <AuthProvider>{children}</AuthProvider>
      </ThemeProvider>
    ),
  });
}
```

## Key Points

- Query priority: `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > `getByDisplayValue` > `getByTestId`
- `userEvent` is always async — `fireEvent` is sync in v13, async in v14
- For async UI updates, prefer `findBy*` over `waitFor` + `getBy*`
- Always use `screen` for queries; don't destructure from `render()`

<!--
Source references:
- https://github.com/callstack/react-native-testing-library/blob/main/skills/react-native-testing/SKILL.md
- https://callstack.github.io/react-native-testing-library/docs/api/queries
-->
