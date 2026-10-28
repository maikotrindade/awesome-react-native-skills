---
name: react-native-native-modules-fabric
description: Fabric native components — codegenNativeComponent props spec, WithDefault, direct and bubbling events, codegenNativeCommands, Android SimpleViewManager with generated delegate, iOS RCTViewComponentView, and wrapping the component in JS.
---

# Fabric Native Components

A Fabric component exposes a native view to React. The props/events spec generates C++ props and event emitters, an Android `ViewManagerInterface` + delegate, and iOS helpers.

## 1. Spec

```ts
// src/RatingViewNativeComponent.ts — file name must end in "NativeComponent"
import {
  codegenNativeComponent,
  codegenNativeCommands,
  type CodegenTypes,
  type ColorValue,
  type HostComponent,
  type ViewProps,
} from 'react-native';

type ChangeEvent = Readonly<{ value: CodegenTypes.Double }>;

export interface NativeProps extends ViewProps {
  value?: CodegenTypes.Double;
  max?: CodegenTypes.WithDefault<CodegenTypes.Int32, 5>;
  tint?: ColorValue;
  readOnly?: CodegenTypes.WithDefault<boolean, false>;
  onValueChange?: CodegenTypes.DirectEventHandler<ChangeEvent>;
}

interface NativeCommands {
  reset: (viewRef: React.ElementRef<HostComponent<NativeProps>>) => void;
}

export const Commands = codegenNativeCommands<NativeCommands>({
  supportedCommands: ['reset'],
});

export default codegenNativeComponent<NativeProps>('RatingView');
```

| Spec type | Meaning |
|-----------|---------|
| `CodegenTypes.WithDefault<T, V>` | Prop with a native default value |
| `CodegenTypes.DirectEventHandler<T>` | Event delivered only to this component |
| `CodegenTypes.BubblingEventHandler<T>` | Event that bubbles through ancestors |
| `ColorValue`, `ImageSource`, `PointValue`, `EdgeInsetsValue` | Converted to native types for you |
| String union with `WithDefault` | Generated native enum |

`package.json` → `"codegenConfig": { "type": "components" }` (or `"all"` with modules); on iOS map the component to its class under `ios.components`.

## 2. JS wrapper

```tsx
// src/RatingView.tsx
import { useRef, useImperativeHandle, type Ref } from 'react';
import NativeRatingView, { Commands, type NativeProps } from './RatingViewNativeComponent';

export type RatingViewHandle = { reset(): void };

export function RatingView({ ref, onChange, ...props }: Omit<NativeProps, 'onValueChange'> & {
  ref?: Ref<RatingViewHandle>;
  onChange?: (value: number) => void;
}) {
  const nativeRef = useRef<React.ElementRef<typeof NativeRatingView>>(null);

  useImperativeHandle(ref, () => ({
    reset: () => nativeRef.current && Commands.reset(nativeRef.current),
  }));

  return (
    <NativeRatingView
      ref={nativeRef}
      {...props}
      onValueChange={(e) => onChange?.(e.nativeEvent.value)}
    />
  );
}
```

Keep the generated component private and export a friendly wrapper (unwrapped `nativeEvent`, normalized props).

## 3. Android

```kotlin
// RatingViewManager.kt
@ReactModule(name = RatingViewManager.NAME)
class RatingViewManager : SimpleViewManager<RatingView>(),
  RatingViewManagerInterface<RatingView> {

  private val delegate = RatingViewManagerDelegate(this)
  override fun getDelegate(): ViewManagerDelegate<RatingView> = delegate
  override fun getName() = NAME

  override fun createViewInstance(context: ThemedReactContext) = RatingView(context)

  @ReactProp(name = "value")
  override fun setValue(view: RatingView, value: Double) { view.rating = value }

  @ReactProp(name = "max", defaultInt = 5)
  override fun setMax(view: RatingView, value: Int) { view.max = value }

  @ReactProp(name = "tint", customType = "Color")
  override fun setTint(view: RatingView, value: Int?) { view.setTint(value) }

  @ReactProp(name = "readOnly")
  override fun setReadOnly(view: RatingView, value: Boolean) { view.isEnabled = !value }

  override fun reset(view: RatingView) { view.rating = 0.0 } // command

  override fun getExportedCustomDirectEventTypeConstants(): Map<String, Any> =
    mapOf("topValueChange" to mapOf("registrationName" to "onValueChange"))

  companion object { const val NAME = "RatingView" }
}
```

```kotlin
// Dispatching the event from the view
class ValueChangeEvent(surfaceId: Int, viewId: Int, private val value: Double) :
  Event<ValueChangeEvent>(surfaceId, viewId) {
  override fun getEventName() = "topValueChange"
  override fun getEventData(): WritableMap = Arguments.createMap().apply { putDouble("value", value) }
}

private fun emitChange(value: Double) {
  val reactContext = context as ReactContext
  val surfaceId = UIManagerHelper.getSurfaceId(reactContext)
  UIManagerHelper.getEventDispatcherForReactTag(reactContext, id)
    ?.dispatchEvent(ValueChangeEvent(surfaceId, id, value))
}
```

Register the manager in the package's `createViewManagers` (`BaseReactPackage` / `ReactPackage`).

## 4. iOS

```objc
// RatingView.mm
#import "RatingView.h"
#import <React/RCTConversions.h>
#import <react/renderer/components/RatingViewSpec/ComponentDescriptors.h>
#import <react/renderer/components/RatingViewSpec/EventEmitters.h>
#import <react/renderer/components/RatingViewSpec/Props.h>
#import <react/renderer/components/RatingViewSpec/RCTComponentViewHelpers.h>

using namespace facebook::react;

@interface RatingView () <RCTRatingViewViewProtocol>
@end

@implementation RatingView {
  MyRatingControl *_control;
}

+ (ComponentDescriptorProvider)componentDescriptorProvider {
  return concreteComponentDescriptorProvider<RatingViewComponentDescriptor>();
}

- (instancetype)initWithFrame:(CGRect)frame {
  if (self = [super initWithFrame:frame]) {
    static const auto defaultProps = std::make_shared<const RatingViewProps>();
    _props = defaultProps;
    _control = [MyRatingControl new];
    [_control addTarget:self action:@selector(valueChanged) forControlEvents:UIControlEventValueChanged];
    self.contentView = _control;
  }
  return self;
}

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps {
  const auto &oldP = *std::static_pointer_cast<RatingViewProps const>(_props);
  const auto &newP = *std::static_pointer_cast<RatingViewProps const>(props);

  if (oldP.value != newP.value) _control.rating = newP.value;
  if (oldP.max != newP.max) _control.max = newP.max;
  if (oldP.tint != newP.tint) _control.tintColor = RCTUIColorFromSharedColor(newP.tint);
  if (oldP.readOnly != newP.readOnly) _control.enabled = !newP.readOnly;

  [super updateProps:props oldProps:oldProps];
}

- (void)valueChanged {
  if (!_eventEmitter) return;
  std::static_pointer_cast<const RatingViewEventEmitter>(_eventEmitter)
    ->onValueChange(RatingViewEventEmitter::OnValueChange{ .value = _control.rating });
}

- (void)handleCommand:(const NSString *)commandName args:(const NSArray *)args {
  RCTRatingViewHandleCommand(self, commandName, args);
}

- (void)reset { _control.rating = 0; }

@end
```

`RatingView.h` declares `@interface RatingView : RCTViewComponentView`. Map it in `codegenConfig.ios.components` (`"RatingView": { "className": "RatingView" }`).

## Pitfalls

| Problem | Cause / fix |
|---------|-------------|
| Prop changes ignored on iOS | Compare old vs new props in `updateProps` and always call `super` |
| Event never fires on Android | Event name must be `top` + PascalCase of the prop minus `on`, and exported in `getExportedCustomDirectEventTypeConstants` |
| View sized 0×0 | Fabric sizes the view from Yoga; give it a style, or implement a custom shadow node in C++ for intrinsic size |
| Recycled iOS view shows stale state | Override `prepareForRecycle` and reset native state |
| "Unimplemented component" in red | Codegen didn't run or the component isn't linked — clean build, `pod install`, check `codegenConfig` |

## Key Points

- Spec in `*NativeComponent.ts` with `codegenNativeComponent` and optional `codegenNativeCommands`.
- Android: implement the generated `ViewManagerInterface`, use the generated delegate, dispatch `top*` events.
- iOS: subclass `RCTViewComponentView`, diff props in `updateProps`, emit through the generated event emitter, route commands with the generated `RCT<Name>HandleCommand` helper.
- Wrap the native component in a small JS component for a clean public API.
