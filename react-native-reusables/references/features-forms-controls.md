---
name: react-native-reusables-forms-controls
description: Form controls — Label, Input, Select, Checkbox, RadioGroup; compound components, primitives, and react-hook-form integration.
---

# Form Controls & Compound Components

React Native Reusables provides form-style components built on RN Primitives: Label, Input, Textarea, Select, Checkbox, RadioGroup. They follow the same compound-component pattern as shadcn/ui and require the matching `@rn-primitives/*` dependency when added manually.

## Usage

**Label + Input:** Use `Label` with `nativeID` and link via `accessibilityLabelledBy` for accessibility:

```tsx
import { Label } from '@/components/ui/label';
import { Input } from '@/components/ui/input';

<View>
  <Label nativeID="email" className="text-sm font-medium">Email</Label>
  <Input className="mt-2" accessibilityLabelledBy="email" />
</View>
```

**Select:** Compound component: `Select`, `SelectTrigger`, `SelectValue`, `SelectContent`, `SelectGroup`, `SelectLabel`, `SelectItem`. Use `SelectContent` with `contentInsets` for safe area:

```tsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from '@/components/ui/select';

function MySelect() {
  const insets = useSafeAreaInsets();
  const contentInsets = { top: insets.top, bottom: insets.bottom, left: 12, right: 12 };

  return (
    <Select>
      <SelectTrigger className="w-[180px]">
        <SelectValue placeholder="Select a fruit" />
      </SelectTrigger>
      <SelectContent insets={contentInsets} className="w-[180px]">
        <SelectItem label="Apple" value="apple">Apple</SelectItem>
        <SelectItem label="Banana" value="banana">Banana</SelectItem>
      </SelectContent>
    </Select>
  );
}
```

**Checkbox / RadioGroup:** Controlled via `checked`/`onCheckedChange` (checkbox) or `value`/`onValueChange` (radio group):

```tsx
import { Checkbox } from '@/components/ui/checkbox';
import { Label } from '@/components/ui/label';

<View className="flex-row items-center gap-2">
  <Checkbox
    id="terms"
    checked={accepted}
    onCheckedChange={setAccepted}
  />
  <Label nativeID="terms">Accept terms</Label>
</View>
```

**Installation:** Prefer CLI — it installs `@rn-primitives/*` automatically:

```bash
npx react-native-reusables add input label select checkbox radio-group textarea
```

Manual: `npx expo install @rn-primitives/select`, then copy the component from the registry.

## react-hook-form Integration

Pair RNR inputs with `react-hook-form` via `Controller`:

```bash
npm install react-hook-form
```

```tsx
import { useForm, Controller } from 'react-hook-form';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Button } from '@/components/ui/button';
import { Text } from '@/components/ui/text';

interface LoginForm { email: string; password: string; }

export function LoginForm() {
  const { control, handleSubmit, formState: { errors } } = useForm<LoginForm>();

  const onSubmit = (data: LoginForm) => console.log(data);

  return (
    <View className="gap-4">
      <View>
        <Label nativeID="email">Email</Label>
        <Controller
          control={control}
          name="email"
          rules={{ required: 'Email is required', pattern: { value: /\S+@\S+\.\S+/, message: 'Invalid email' } }}
          render={({ field: { onChange, onBlur, value } }) => (
            <Input
              accessibilityLabelledBy="email"
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              keyboardType="email-address"
              autoCapitalize="none"
              className={errors.email ? 'border-destructive' : ''}
            />
          )}
        />
        {errors.email && <Text className="text-sm text-destructive">{errors.email.message}</Text>}
      </View>

      <Button onPress={handleSubmit(onSubmit)}>
        <Text>Sign In</Text>
      </Button>
    </View>
  );
}
```

For **Select** with react-hook-form:

```tsx
<Controller
  control={control}
  name="role"
  render={({ field: { onChange, value } }) => (
    <Select onValueChange={(opt) => onChange(opt?.value)} value={{ value, label: value }}>
      {/* ... */}
    </Select>
  )}
/>
```

## Key Points

- Each form component has a corresponding RN Primitives package; CLI installs it automatically
- Use `Text` from `@/components/ui/text` for labels and descriptions so `TextClassContext` applies styles correctly
- Overlay parts (SelectContent, DialogContent) require a root `PortalHost` — see [features-overlays-portals](references/features-overlays-portals.md)
- Pair with `react-hook-form` + `Controller` for validation; apply error classes via `cn()` on the input

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (apps/docs/content/docs/components/input.mdx, select.mdx, checkbox.mdx, label.mdx, radio-group.mdx)
- https://reactnativereusables.com/docs/components/input
- https://reactnativereusables.com/docs/components/select
-->
