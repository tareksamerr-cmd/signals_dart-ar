---
title: نظرة عامة
description: نظرة عامة على مكتبة الإشارات (signals)
sidebar:
  order: 2
---

# نظرة عامة على الإشارات (Signals)

ليست الإشارات بجديدة، بل هي موجودة منذ زمن طويل. تُعرف أيضًا باسم [الإشارات](https://en.wikipedia.org/wiki/Signals_and_slots) أو [الكائنات القابلة للملاحظة (Observables)](https://en.wikipedia.org/wiki/Observable_pattern).

أصبحت العديد من أطر JavaScript الشهيرة تتضمن الإشارات كجزء من مكتبتها الأساسية. لكل تطبيق من هذه التطبيقات ميزاته الفريدة وواجهات برمجة التطبيقات (APIs) الخاصة به. `Signals.dart` هو نسخة مأخوذة من [مكتبة إشارات Preact](https://preactjs.com/blog/introducing-signals/) وقد صُمم ليكون قريبًا قدر الإمكان من واجهة برمجة التطبيقات الأصلية في واجهته الأساسية.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Jp7QBjY5K34?si=qYs2Harl0NogWtqk" title="مشغل فيديو YouTube" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

بدأت الإشارات في Preact بتنفيذ يعتمد على تتبع التبعيات باستخدام مجموعة (Set)، ولكن تم تغيير ذلك لاحقًا إلى استخدام قائمة مرتبطة (linked list). تطبيق القائمة المرتبطة هو الأكثر أداءً بفضل الاستفادة من [تعزيز الإشارات (signal boosting)](https://preactjs.com/blog/signal-boosting/)، وهذا هو التطبيق المستخدم في `Signals.dart`.

يوجد أيضًا [ملعب DartPad](https://dartpad.dev/?id=d5f16f6be22e716d90419e41d10f281a) يحتوي على بعض الطرق الأساسية التي يمكنك استخدامها للتجربة!

:::note
إذا كنت قادمًا من عالم JavaScript وترتاح مع الإشارات، فستجد هذا مألوفًا جدًا. وإذا كنت تبحث عن مكتبة لإدارة الحالة في Flutter يمكن استخدامها في عالم JS وخارج Dart أيضًا، فلا تبحث أكثر!
:::

## التحديثات الصغرى (Minimal Updates)

الميزة الكبرى للإشارات هي التوفير في الحسابات. **إذا لم تقم مطلقًا بقراءة إشارة، فلن يتم حسابها أبدًا.** وهذا يعني أنه إذا كان لديك سلسلة من القيم المحسوبة (computed) ولم تقرأ قيمة آخرها، فلن يتم استدعاء أي من دوال الاستدعاء (callbacks).

```dart
import 'package:signals/signals.dart';

final a = signal(0);
final b = computed(() => a.value + 1);
final c = computed(() => b.value + 1);
final d = computed(() => c.value + 1);

// إذا لم تقرأ `d` مطلقًا، فلن يتم استدعاء أي من دوال الاستدعاء

// الآن سيتم استدعاء كل دوال الاستدعاء
print(d.value); // 3

// لن يتم استدعاء أي من دوال الاستدعاء لأن القيمة مخزنة في كل عقدة
print(d.value); // 3
```

الإشارات هي أيضًا مكتبة لإدارة الحالة تعتمد على أسلوب السحب (pull)، على عكس معظم الأنظمة التي تعتمد على أسلوب الدفع (push). هذا يعني أن مجرد تحديث قيمة إشارة لا يعني بالضرورة انتشار هذا التحديث (أي إخطار المستمعين) إلى الأهداف المرتبطة.

computed هي إشارة خاصة تتتبع تبعياتها وتخزّن قيمتها مؤقتًا، بحيث لا تعيد الحساب إلا عند قراءتها وعندما تتغير التبعيات. يمكن أن يكون هذا متقدمًا جدًا، حيث يمكن أن يكون لديك سلسلة من الإشارات المحسوبة وكل واحدة منها محسّنة لأقل عدد ممكن من التحديثات.

```dart
import 'package:signals/signals.dart';

final a = signal(0);
final b = signal(0);

final c = computed(() => a.value + b.value);
final d = computed(() => c.value + 1);
final e = computed(() => d.value + 1);

// سيتم استدعاء كل دوال الاستدعاء
print(e.value); // 2

// لن يتم استدعاء أي من دوال الاستدعاء لأن القيمة مخزنة مؤقتًا في كل عقدة
print(e.value); // 2

// سيتم استدعاء دوال الاستدعاء التي تحتاج إلى تحديث فقط
b.value = 1;
print(e.value); // 3
```

قراءات إضافية

· https://signia.tldraw.dev/docs/what-are-signals
· https://www.solidjs.com/guides/reactivity
· https://angular.io/guide/signals
· https://vuejs.org/guide/extras/reactivity-in-depth.html

```
