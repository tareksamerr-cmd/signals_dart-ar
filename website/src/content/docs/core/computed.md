---
title: Computed
description: إنشاء إشارة تستمد قيمتها من إشارات أخرى
sidebar:
  order: 1
---

غالبًا ما تُشتق البيانات من أجزاء أخرى من البيانات الموجودة. تتيح لك دالة `computed` دمج قيم إشارات متعددة في إشارة جديدة يمكن التفاعل معها، أو حتى استخدامها بواسطة `computed` إضافية. عندما تتغير الإشارات التي يتم الوصول إليها من داخل دالة رد الاتصال الخاصة بـ `computed`، يتم إعادة تنفيذ دالة رد الاتصال الخاصة بـ `computed` وتصبح قيمتها المرتجعة الجديدة هي قيمة إشارة `computed`.

> فئة `Computed` توسع فئة `Signal`، لذا يمكنك استخدامها في أي مكان تستخدم فيه إشارة.

```dart
import 'package:signals/signals.dart';

final name = signal("Jane");
final surname = signal("Doe");

final fullName = computed(() => name.value + " " + surname.value);

// Logs: "Jane Doe"
print(fullName.value);

// Updates flow through computed, but only if someone
// subscribes to it. More on that later.
name.value = "John";
// Logs: "John Doe"
print(fullName.value);
```

أي إشارة يتم الوصول إليها داخل دالة رد الاتصال الخاصة بـ `computed` سيتم الاشتراك فيها وتتبعها تلقائيًا كاعتماد لإشارة `computed`.

> إشارات `Computed` يتم تقييمها بشكل كسول (lazily evaluated) وتخزينها مؤقتًا (memoized).

## فرض إعادة التقييم (Force Re-evaluation)

يمكنك فرض إعادة تقييم إشارة `computed` عن طريق استدعاء دالة `.recompute` الخاصة بها. سيؤدي هذا إلى إعادة تشغيل دالة رد الاتصال الخاصة بـ `computed` وتحديث قيمة إشارة `computed`.

```dart
final name = signal("Jane");
final surname = signal("Doe");
final fullName = computed(() => name.value + " " + surname.value);

fullName.recompute(); // Re-runs the computed callback
```

## التخلص (Disposing)

### التخلص التلقائي (Auto Dispose)

إذا تم إنشاء إشارة `computed` مع تعيين `autoDispose` إلى `true`، فستتخلص من نفسها تلقائيًا عندما لا يكون هناك المزيد من المستمعين.

```dart
final s = computed(() => 0, autoDispose: true);
s.onDispose(() => print('Signal destroyed'));
final dispose = s.subscribe((_) {});
dispose();
final value = s.value; // 0
// prints: Signal destroyed
```

لا تتطلب الإشارة التي تتخلص تلقائيًا أن تكون تبعياتها تتخلص تلقائيًا. عندما يتم التخلص منها، ستجمد قيمتها وتتوقف عن تتبع تبعياتها.

هذا يعني أنها لن تتفاعل بعد الآن مع التغييرات في تبعياتها.

```dart
final s = computed(() => 0);
s.dispose();
final value = s.value; // 0
final b = computed(() => s.value); // 0
// b will not react to changes in s
```

يمكنك التحقق مما إذا تم التخلص من إشارة عن طريق استدعاء دالة `.disposed`.

```dart
final s = computed(() => 0);
print(s.disposed); // false
s.dispose();
print(s.disposed); // true
```

### دالة رد الاتصال عند التخلص (On Dispose Callback)

يمكنك إرفاق دالة رد اتصال بإشارة سيتم استدعاؤها عند تدمير الإشارة.

```dart
final s = computed(() => 0);
s.onDispose(() => print('Signal destroyed'));
s.dispose();
```

## إشارة Computed مخصصة (Custom Computed)

يمكنك إنشاء إشارة `computed` مخصصة عن طريق توسيع فئة `Computed`.

```dart
class MyComputed extends Computed<int> {
  MyComputed() : super(() => 0);
}
```

## Flutter

في Flutter، إذا كنت ترغب في إنشاء إشارة تتخلص من نفسها تلقائيًا عند إزالة الودجت من شجرة الودجت وتُعيد بناء الودجت عندما تتغير الإشارة، يمكنك استخدام `createComputed` داخل ودجت ذي حالة (stateful widget).

```dart
import 'package:flutter/material.dart';
import 'package:signals/signals_flutter.dart';

class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> with SignalsMixin {
  late final counter = createSignal(0);
  late final isEven = createComputed(() => counter.value.isEven);
  late final isOdd = createComputed(() => counter.value.isOdd);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Counter: even=$isEven, odd=$isOdd'),
            ElevatedButton(
              onPressed: () => counter.value++,
              child: Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}
```

لا توجد حاجة إلى ودجت `Watch` أو امتداد، ستتخلص الإشارة من نفسها تلقائيًا عند إزالة الودجت من شجرة الودجت.

`SignalsMixin` هو `mixin` يتخلص تلقائيًا من جميع الإشارات التي تم إنشاؤها في الحالة عندما يتم إزالة الودجت من شجرة الودجت.

## الاختبار (Testing)

يمكن اختبار إشارات `computed` عن طريق تحويل إشارة `computed` إلى `stream` واختبارها مثل أي `stream` آخر في Dart.

```dart
test('test as stream', () {
    final a = signal(0);
    final s = computed(() => a());
    final stream = s.toStream();

    a.value = 1;
    a.value = 2;
    a.value = 3;

    expect(stream, emitsInOrder([0, 1, 2, 3]));
});
```

`emitsInOrder` هو مطابق (matcher) سيتحقق مما إذا كان الـ `stream` يُصدر القيم بالترتيب الصحيح، وهو في هذه الحالة كل قيمة بعد تحديث الإشارة.

يمكنك أيضًا تجاوز القيمة الأولية لإشارة `computed` عند الاختبار. هذا مفيد للمحاكاة واختبار تطبيقات قيم محددة.

```dart
test('test with override', () {
    final a = signal(0);
    final s = computed(() => a()).overrideWith(-1);

    final stream = s.toStream();

    a.value = 1;
    a.value = 2;
    a.value = 2; // check if skipped
    a.value = 3;

    expect(stream, emitsInOrder([-1, 1, 2, 3]));
});
```

تُرجع `overrideWith` إشارة `computed` جديدة بنفس المعرف العام وتُعيّن القيمة كما لو كانت دالة رد الاتصال الخاصة بـ `computed` قد أعادتها.
