---
name: react-native-native-modules-turbo
description: Turbo Modules end to end — TypeScript spec, codegenConfig, type mapping, Kotlin and Objective-C++ implementations, calling Swift, promises, event emitters, constants, and registering app-local modules.
---

# Turbo Modules

A Turbo Module is a native module described by a TypeScript spec. Codegen turns the spec into a C++ JSI binding plus a Java/Kotlin abstract class and an Objective-C protocol that you implement.

## 1. Spec

```ts
// src/NativeDeviceInfo.ts  — file name must start with "Native"
import { TurboModuleRegistry, type TurboModule, type CodegenTypes } from 'react-native';

export type BatteryState = { level: number; charging: boolean };

export interface Spec extends TurboModule {
  getModel(): string;                                   // sync
  getBatteryState(): Promise<BatteryState>;             // async
  setKeepAwake(enabled: boolean): void;
  readonly onBatteryChanged: CodegenTypes.EventEmitter<BatteryState>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('DeviceInfo');
```

```ts
// src/index.ts — public API wraps the native module
import NativeDeviceInfo from './NativeDeviceInfo';
export const getModel = () => NativeDeviceInfo.getModel();
export const getBatteryState = () => NativeDeviceInfo.getBatteryState();
export const onBatteryChanged = NativeDeviceInfo.onBatteryChanged; // returns a subscription with remove()
```

Use `getEnforcing` (throws a clear error if the module isn't linked) unless the module is optional, then `get` and handle `null`.

### Type mapping

| TypeScript | Kotlin | Objective-C |
|------------|--------|-------------|
| `string` | `String` | `NSString *` |
| `number` | `Double` | `double` / `NSNumber *` |
| `CodegenTypes.Int32` | `Int` | `NSInteger` |
| `CodegenTypes.Float` | `Float` | `float` |
| `boolean` | `Boolean` | `BOOL` / `NSNumber *` |
| `T \| null` / optional | nullable type | nullable pointer |
| Object type alias | `ReadableMap` (args) / `WritableMap` (returns) | generated C++ struct / `NSDictionary *` |
| `Array<T>` | `ReadableArray` / `WritableArray` | `NSArray *` |
| `Object` (untyped) | `ReadableMap` | `NSDictionary *` |
| `Promise<T>` | `Promise` last parameter | `resolve:` / `reject:` blocks |
| `(value: T) => void` | `Callback` | `RCTResponseSenderBlock` |
| String union `'a' \| 'b'` / enum | `String` / generated enum | `NSString *` / generated enum |
| `CodegenTypes.EventEmitter<T>` | generated `emitOnX(value)` | generated `emitOnX:` |

Codegen supports a subset of TypeScript: no generics other than `Array`/`Promise`, no imported types from other packages, no default exports of types.

## 2. Codegen config

```json
// package.json (library or app)
{
  "codegenConfig": {
    "name": "DeviceInfoSpec",
    "type": "modules",
    "jsSrcsDir": "src",
    "android": { "javaPackageName": "com.example.deviceinfo" }
  }
}
```

`type` is `modules`, `components`, or `all`. Codegen runs during `pod install` and the Gradle build; run it manually with `npx @react-native-community/cli codegen` when debugging.

## 3. Android (Kotlin)

```kotlin
// android/src/main/java/com/example/deviceinfo/DeviceInfoModule.kt
package com.example.deviceinfo

import android.os.Build
import com.facebook.react.bridge.Arguments
import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.ReactApplicationContext

class DeviceInfoModule(reactContext: ReactApplicationContext) :
  NativeDeviceInfoSpec(reactContext) {

  override fun getModel(): String = Build.MODEL

  override fun getBatteryState(promise: Promise) {
    try {
      val (level, charging) = readBattery(reactApplicationContext)
      promise.resolve(Arguments.createMap().apply {
        putDouble("level", level)
        putBoolean("charging", charging)
      })
    } catch (e: Exception) {
      promise.reject("E_BATTERY", "Could not read battery state", e)
    }
  }

  override fun setKeepAwake(enabled: Boolean) {
    reactApplicationContext.currentActivity?.runOnUiThread { /* toggle FLAG_KEEP_SCREEN_ON */ }
  }

  private fun notifyBattery(level: Double, charging: Boolean) {
    emitOnBatteryChanged(Arguments.createMap().apply {
      putDouble("level", level)
      putBoolean("charging", charging)
    })
  }

  companion object { const val NAME = NativeDeviceInfoSpec.NAME }
}
```

```kotlin
// DeviceInfoPackage.kt
class DeviceInfoPackage : BaseReactPackage() {
  override fun getModule(name: String, reactContext: ReactApplicationContext): NativeModule? =
    if (name == DeviceInfoModule.NAME) DeviceInfoModule(reactContext) else null

  override fun getReactModuleInfoProvider() = ReactModuleInfoProvider {
    mapOf(DeviceInfoModule.NAME to ReactModuleInfo(
      name = DeviceInfoModule.NAME,
      className = DeviceInfoModule.NAME,
      canOverrideExistingModule = false,
      needsEagerInit = false,
      isCxxModule = false,
      isTurboModule = true,
    ))
  }
}
```

Libraries are autolinked. For a module inside the app's own `android/` folder, add `DeviceInfoPackage()` in `MainApplication`'s `getPackages()`.

## 4. iOS (Objective-C++)

```objc
// ios/DeviceInfo.h
#import <DeviceInfoSpec/DeviceInfoSpec.h>

@interface DeviceInfo : NativeDeviceInfoSpecBase <NativeDeviceInfoSpec>
@end
```

```objc
// ios/DeviceInfo.mm
#import "DeviceInfo.h"
#import <UIKit/UIKit.h>

@implementation DeviceInfo

+ (NSString *)moduleName { return @"DeviceInfo"; }

- (NSString *)getModel {
  return UIDevice.currentDevice.model;
}

- (void)getBatteryState:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject {
  dispatch_async(dispatch_get_main_queue(), ^{
    UIDevice *device = UIDevice.currentDevice;
    device.batteryMonitoringEnabled = YES;
    resolve(@{
      @"level": @(device.batteryLevel),
      @"charging": @(device.batteryState == UIDeviceBatteryStateCharging),
    });
  });
}

- (void)setKeepAwake:(BOOL)enabled {
  dispatch_async(dispatch_get_main_queue(), ^{
    UIApplication.sharedApplication.idleTimerDisabled = enabled;
  });
}

- (void)notifyBattery:(double)level charging:(BOOL)charging {
  [self emitOnBatteryChanged:@{ @"level": @(level), @"charging": @(charging) }];
}

- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
    (const facebook::react::ObjCTurboModule::InitParams &)params {
  return std::make_shared<facebook::react::NativeDeviceInfoSpecJSI>(params);
}

@end
```

Inheriting from `Native<Name>SpecBase` provides the generated `emitOn…` methods; a module without events can subclass `NSObject` and adopt the protocol only. Method selectors must match the generated protocol exactly — check the generated header in `build/generated/ios` when the compiler complains.

### Calling Swift

Turbo Modules need the Objective-C++ entry point, but the logic can live in Swift:

```swift
// ios/DeviceInfoImpl.swift
import UIKit

@objc public class DeviceInfoImpl: NSObject {
  @objc public func model() -> String { UIDevice.current.model }
}
```

```objc
// DeviceInfo.mm
#import "DeviceInfo-Swift.h"   // generated Swift header; name follows the pod/module

@implementation DeviceInfo {
  DeviceInfoImpl *_impl;
}
- (instancetype)init { if (self = [super init]) { _impl = [DeviceInfoImpl new]; } return self; }
- (NSString *)getModel { return [_impl model]; }
// ...
@end
```

If most of your code is Swift, the Expo Modules API or Nitro Modules avoid this glue.

## 5. App-local modules

With `create-react-native-library --local`, the module lives in `modules/<name>` and is linked through the app's `package.json` — autolinking and Codegen work as for a published library.

For a spec placed directly in the app:

```json
// app package.json
{
  "codegenConfig": {
    "name": "AppSpecs",
    "type": "modules",
    "jsSrcsDir": "specs",
    "android": { "javaPackageName": "com.example.app.specs" },
    "ios": { "modulesProvider": { "DeviceInfo": "DeviceInfo" } }
  }
}
```

`ios.modulesProvider` maps the JS module name to the Objective-C class so the app can find it without a library podspec. Register the Android package manually in `MainApplication`.

Expo apps with Continuous Native Generation: native folders are regenerated, so put app-local native code in a local module (`create-expo-module --local` or a local library), never directly in `ios/` or `android/`.

## Key Points

- `Native<Name>.ts` spec → `codegenConfig` → implement generated `Native<Name>Spec` (Kotlin) and protocol (Objective-C++).
- Promises for anything that blocks; `CodegenTypes.EventEmitter` for native → JS events.
- iOS entry point is Objective-C++; wrap Swift behind it.
- Local modules: `--local` scaffold, or app-level `codegenConfig` with `ios.modulesProvider` plus manual Android package registration.
