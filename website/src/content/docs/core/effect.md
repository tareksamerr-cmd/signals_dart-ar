---
title: Effect
description: تُستخدم المؤثرات (Effects) لتنفيذ الآثار الجانبية من الإشارات
sidebar:
  order: 2
---

دالة `effect` هي القطعة الأخيرة التي تجعل كل شيء تفاعليًا. عندما تصل إلى إشارة داخل دالة رد الاتصال الخاصة بها، سيتم تنشيط تلك الإشارة وكل تبعية لتلك الإشارة والاشتراك فيها. في هذا الصدد، هي مشابهة جدًا لـ [`computed(fn)`](/core/computed). بشكل افتراضي، تكون جميع التحديثات كسولة (lazy)، لذلك لن يتم تحديث أي شيء حتى تصل إلى إشارة داخل `effect`.

```dart
import 'package:signals/signals.dart';

final name = signal("Jane");
final surname = signal("Doe");
final fullName = computed(() => name.value + " " + surname.value);

// Logs: "Jane Doe"
effect(() => print(fullName.value));

// Updating one of its dependencies will automatically trigger
// the effect above, and will print "John Doe" to the console.
name.value = "John";
```

يمكنك تدمير `effect` وإلغاء الاشتراك من جميع الإشارات التي كان مشتركًا فيها، عن طريق استدعاء الدالة المعادة.

```dart
import 'package:signals/signals.dart';

final name = signal("Jane");
final surname = signal("Doe");
final fullName = computed(() => name.value + " " + surname.value);

// Logs: "Jane Doe"
final dispose = effect(() => print(fullName.value));

// Destroy effect and subscriptions
dispose();

// Update does nothing, because no one is subscribed anymore.
// Even the computed `fullName` signal won't change, because it knows
// that no one listens to it.
surname.value = "Doe 2";
```

## دالة رد الاتصال للتنظيف (Cleanup Callback)

يمكنك أيضًا إعادة دالة تنظيف من `effect`. سيتم استدعاء هذه الدالة عند تدمير `effect`.

```dart
import 'package:signals/signals.dart';

final s = signal(0);

final dispose = effect(() {
  print(s.value);
  return () => print('Effect destroyed');
});

// Destroy effect and subscriptions
dispose();
```

## دالة رد الاتصال عند التخلص (On Dispose Callback)

يمكنك أيضًا تمرير دالة رد اتصال إلى `effect` سيتم استدعاؤها عند تدمير `effect`.

```dart
import 'package:signals/signals.dart';

final s = signal(0);

final dispose = effect(() {
  print(s.value);
}, onDispose: () => print('Effect destroyed'));

// Destroy effect and subscriptions
dispose();
```

## منع الحلقات (Preventing Cycles)

:::danger
سيؤدي تغيير إشارة داخل `effect` إلى حلقة لا نهائية، لأن `effect` سيتم تشغيله مرة أخرى. لمنع هذا، يمكنك استخدام [`untracked(fn)`](/core/untracked) لقراءة إشارة دون الاشتراك فيها.

```dart
import 'dart:async';

import 'package:signals/signals.dart';

Future<void> main() async {
  final completer = Completer<void>();
  final age = signal(0);

  effect(() {
    print('You are ${age.value} years old');
    age.value++; // <-- This will throw a cycle error
  });

  await completer.future;
}
```

:::
