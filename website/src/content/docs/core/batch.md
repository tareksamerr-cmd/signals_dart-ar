---
title: Batch
description: تجميع عمليات كتابة الإشارات المتعددة في تحديث واحد
sidebar:
  order: 4
---

تسمح لك دالة `batch` بدمج عمليات كتابة إشارات متعددة في تحديث واحد يتم تشغيله في النهاية عند اكتمال دالة رد الاتصال.

```dart
import 'package:signals/signals.dart';

final name = signal("Jane");
final surname = signal("Doe");
final fullName = computed(() => name.value + " " + surname.value);

// Logs: "Jane Doe"
effect(() => print(fullName.value));

// Combines both signal writes into one update. Once the callback
// returns the `effect` will trigger and we'll log "Foo Bar"
batch(() {
	name.value = "Foo";
	surname.value = "Bar";
});
```

عندما تصل إلى إشارة قمت بالكتابة إليها سابقًا داخل دالة رد الاتصال، أو تصل إلى إشارة `computed` تم إبطالها بواسطة إشارة أخرى، سنقوم فقط بتحديث التبعيات الضرورية للحصول على القيمة الحالية للإشارة التي قرأت منها. سيتم تحديث جميع الإشارات الأخرى التي تم إبطالها في نهاية دالة رد الاتصال.

```dart
import 'package:signals/signals.dart';

final counter = signal(0);
final _double = computed(() => counter.value * 2);
final _triple = computed(() => counter.value * 3);

effect(() => print(_double.value, _triple.value));

batch(() {
	counter.value = 1;
	// Logs: 2, despite being inside batch, but `triple`
	// will only update once the callback is complete
	print(_double.value);
});
// Now we reached the end of the batch and call the effect
```

يمكن تداخل عمليات `Batch` وسيتم تفريغ التحديثات عند اكتمال استدعاء `batch` الخارجي.

```dart
import 'package:signals/signals.dart';

final counter = signal(0);
effect(() => print(counter.value));

batch(() {
	batch(() {
		// Signal is invalidated, but update is not flushed because
		// we're still inside another batch
		counter.value = 1;
	});

	// Still not updated...
});
// Now the callback completed and we'll trigger the effect.
```
