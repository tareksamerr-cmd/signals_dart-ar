---
title: Untracked
description: يمنع الاشتراكات من الحدوث
sidebar:
  order: 3
---

في الحالة التي تتلقى فيها دالة رد اتصال يمكنها قراءة بعض الإشارات، ولكنك لا ترغب في الاشتراك فيها، يمكنك استخدام `untracked` لمنع حدوث أي اشتراكات.

```dart
final counter = signal(0);
final effectCount = signal(0);
final fn = () => effectCount.value + 1;

effect(() {
	print(counter.value);

	// Whenever this effect is triggered, run `fn` that gives new value
	effectCount.value = untracked(fn);
});
```
