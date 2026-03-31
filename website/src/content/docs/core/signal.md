---
title: Signal
description: كيفية انشاء و استخدام signal
sidebar:
  order: 0
---


تُنشئ دالة `signal` إشارة جديدة. الإشارة هي حاوية لقيمة يمكن أن تتغير بمرور الوقت. يمكنك قراءة قيمة الإشارة أو الاشتراك في تحديثات القيمة عن طريق الوصول إلى خاصية `.value` الخاصة بها.

```dart
import 'package:signals/signals.dart';

final counter = signal(0);

// Read value from signal, logs: 0
print(counter.value);

// Write to a signal
counter.value = 1;
```

يمكن إنشاء الإشارات بشكل عام، داخل الفئات أو الدوال. الأمر متروك لك في كيفية هيكلة تطبيقك.

لا يُنصح بإنشاء الإشارات داخل `effects` أو `computed`، حيث سيؤدي ذلك إلى إنشاء إشارة جديدة في كل مرة يتم فيها تشغيل `effect` أو `computed`. قد يؤدي هذا إلى سلوك غير متوقع.

في Flutter، لا تُنشئ إشارات داخل طرق `build`، حيث سيؤدي ذلك إلى إنشاء إشارة جديدة في كل مرة يتم فيها إعادة بناء الودجت.

## الكتابة إلى إشارة (Writing to a signal)

تتم الكتابة إلى إشارة عن طريق تعيين خاصية `.value` الخاصة بها. يؤدي تغيير قيمة الإشارة إلى تحديث متزامن لكل من `computed` و `effect` الذي يعتمد على تلك الإشارة، مما يضمن أن حالة تطبيقك متسقة دائمًا.

## .peek()

في الحالات النادرة التي يكون لديك `effect` يجب أن يكتب إلى إشارة أخرى بناءً على القيمة السابقة، ولكنك **لا** تريد أن يكون `effect` مشتركًا في تلك الإشارة، يمكنك قراءة القيمة السابقة للإشارة عبر `signal.peek()`.

```dart
final counter = signal(0);
final effectCount = signal(0);

effect(() {
	print(counter.value);

	// Whenever this effect is triggered, increase `effectCount`.
	// But we don't want this signal to react to `effectCount`
	effectCount.value = effectCount.peek() + 1;
});
```

لاحظ أنه يجب عليك استخدام `signal.peek()` فقط إذا كنت في حاجة ماسة إليه. قراءة قيمة الإشارة عبر `signal.value` هي الطريقة المفضلة في معظم السيناريوهات.

## .value

تُستخدم خاصية `.value` للإشارة للقراءة أو الكتابة إلى الإشارة. إذا تم استخدامها داخل `effect` أو `computed`، فستشترك في الإشارة وتُشغل `effect` أو `computed` كلما تغيرت قيمة الإشارة.

```dart
final counter = signal(0);

effect(() {
	print(counter.value);
});

counter.value = 1;
```

## فرض التحديث (Force Update)

إذا كنت ترغب في فرض تحديث لإشارة، يمكنك استدعاء الدالة `.set(..., force: true)`. سيؤدي هذا إلى تشغيل جميع `effects` ووضع علامة على جميع `computed` على أنها غير نظيفة (dirty).

```dart
final counter = signal(0);
counter.set(1, force: true);
```

## التخلص (Disposing)

### التخلص التلقائي (Auto Dispose)

إذا تم إنشاء إشارة مع تعيين `autoDispose` إلى `true`، فستتخلص من نفسها تلقائيًا عندما لا يكون هناك المزيد من المستمعين.

```dart
final s = signal(0, autoDispose: true);
s.onDispose(() => print('Signal destroyed'));
final dispose = s.subscribe((_) {});
dispose();
final value = s.value; // 0
// prints: Signal destroyed
```

لا تتطلب الإشارة التي تتخلص تلقائيًا أن تكون تبعياتها تتخلص تلقائيًا. عندما يتم التخلص منها، ستجمد قيمتها وتتوقف عن تتبع تبعياتها.

```dart
final s = signal(0);
s.dispose();
final c = computed(() => s.value);
// c will not react to changes in s
```

يمكنك التحقق مما إذا تم التخلص من إشارة عن طريق استدعاء الدالة `.disposed`.

```dart
final s = signal(0);
print(s.disposed); // false
s.dispose();
print(s.disposed); // true
```

### دالة رد الاتصال عند التخلص (On Dispose Callback)

يمكنك إرفاق دالة رد اتصال بإشارة سيتم استدعاؤها عند تدمير الإشارة.

```dart
final s = signal(0);
s.onDispose(() => print('Signal destroyed'));
s.dispose();
```

## إشارة مخصصة (Custom Signal)

يمكنك إنشاء إشارة مخصصة عن طريق توسيع فئة `Signal`.

```dart
class MySignal extends Signal<int> {
  MySignal(int value) : super(value);
}
```

:::tip
يمكنك تطبيق أي عدد من الـ `mixins` على إشارة مخصصة لإضافة وظائف إضافية.

الـ `Mixins`:
- `ValueListenableSignalMixin` لتطبيق `ValueListenable<T>`
- `ValueNotifierSignalMixin` لتطبيق `ValueNotifier<T>`
- `ChangeStackSignalMixin` لإضافة وظائف التراجع والإعادة
- `TrackedSignalMixin` لإضافة تتبع القيمة الأولية والسابقة
- `StreamSignalMixin` لتطبيق `Stream`
- `SinkSignalMixin` لتطبيق `Sink`
- `EventSinkSignalMixin` لتطبيق `EventSink`
- `ListSignalMixin` لأنواع قيم القائمة (List)
- `MapSignalMixin` لأنواع قيم الخريطة (Map)
- `SetSignalMixin` لأنواع قيم المجموعة (Set)
- `IterableSignalMixin` لأنواع قيم `Iterable<T>`
- `QueueSignalMixin` لأنواع قيم الطابور (Queue)
:::

## Flutter

في Flutter، إذا كنت ترغب في إنشاء إشارة تتخلص من نفسها تلقائيًا عند إزالة الودجت من شجرة الودجت وتُعيد بناء الودجت عندما تتغير الإشارة، يمكنك استخدام `createSignal` داخل ودجت ذي حالة (stateful widget).

```dart
import 'package:flutter/material.dart';
import 'package:signals/signals_flutter.dart';

class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> with SignalsMixin {
  late final counter = createSignal(0);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Counter: $counter'),
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

يمكن اختبار الإشارات عن طريق تحويل الإشارة إلى `stream` واختبارها مثل أي `stream` آخر في Dart.

```dart
test('test as stream', () {
  final s = signal(0);
  final stream = s.toStream(); // create a stream of values

  s.value = 1;
  s.value = 2;
  s.value = 3;

  expect(stream, emitsInOrder([0, 1, 2, 3]));
});
```

`emitsInOrder` هو مطابق (matcher) سيتحقق مما إذا كان الـ `stream` يُصدر القيم بالترتيب الصحيح، وهو في هذه الحالة كل قيمة بعد تحديث الإشارة.

يمكنك أيضًا تجاوز القيمة الأولية للإشارة عند الاختبار. هذا مفيد للمحاكاة واختبار تطبيقات قيم محددة.

```dart
test('test with override', () {
  final s = signal(0).overrideWith(-1);

  final stream = s.toStream();

  s.value = 1;
  s.value = 2;
  s.value = 3;

  expect(stream, emitsInOrder([-1, 1, 2, 3]));
});
```

تُرجع `overrideWith` إشارة جديدة بنفس المعرف العام وتُعيّن القيمة كما لو تم إنشاؤها بها. يمكن أن يكون هذا مفيدًا عند استخدام إشارات غير متزامنة أو إشارات عامة تُستخدم لحقن التبعية.
