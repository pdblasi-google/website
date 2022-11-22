---
title: Added microsecond precision support to `AutomatedTestWidgetsBinding.pump`
description: One method (`WidgetTester.pumpFrames`) is replaced and others could see differences in non-default behavior to support a change from millisecond to microsecond precision for `AutomatedTestWidgetsBinding.pump`.
---

## Summary

One method (`WidgetTester.pumpFrames`) is replaced and others could see
differences in non-default behavior to support a change from millisecond to
microsecond precision for `AutomatedTestWidgetsBinding.pump`. This may affect
tests with highly specific timing needs such as golden tests for animations or
using `AnimationSheetBuilder.collate`. Other possibly affected methods include:
`WidgetTester.pumpAndSettle`, `WidgetTester.pump`, `WidgetTester.pumpWidget`.

## Context

On of the primary uses of `AutomatedTestWidgetsBinding` is to imitate changes in time, allowing for tests to be run without needing to actually elapse the total amount of time required for animations or other time sensitive UI operations. Previously, changes in time with this class were limited to millisecond precision even though microsecond precision was available.

The millisecond precision is inconsistent with many parts of the system, including:
* The platform engines, which support up to nanosecond precision
* Integration testing using `LiveTestWidgetsFlutterBinding` which supports microsecond precision
* `WidgetTester.pumpFrames` which assumes microsecond precision support for the default value of the `interval` parameter

With microsecond precision, these inconsistencies are resolved, allowing for timing during testing to be more in line with testing on an actual device.

## Description of change

The `pump` method of `AutomatedTestWidgetsBinding` has been updated to no longer truncate the microseconds off of the `duration` parameter when passing the timestamp to `handleBeginFrame`. This affects not only the `AutomatedTestWidgetsBinding.pump` method itself, but also various other methods that make use of `AutomatedTestWidgetsBinding.pump` internally. 

### Replacing `WidgetTester.pumpFrames`

The `WidgetTester.pumpFrames` method would see a change to default behavior, so has been replaced with `WidgetTester.pumpFramesFor` which has the same functionality, but supporting microsecond precision in all cases.

### Other affected methods

The following methods may also see differences in behavior. Unlike `WidgetTester.pumpFrames`, these methods don't exhibit differences in their default behavior, so they haven't been replaced and deprecated in order to minimize the impact on the overall `WidgetTester` API.

* `WidgetTester.pumpAndSettle`
* `WidgetTester.pump`
* `WidgetTester.pumpWidget`

## Migration guide

### `WidgetTester.pumpFrames`

Code before migration:

```dart
main() {
  testWidgets('example test', (tester) {
    final AnimationSheetBuilder animationSheet = AnimationSheetBuilder(frameSize: const Size(50, 50));

    // The default value of `interval` here includes microsecond precision
    // that previously was truncated in AutomatedTestWidgetsBinding.
    await tester.pumpFrames(animationSheet.record(
      const RefreshProgressIndicator(),
    ), const Duration(seconds: 3)); 
  });
}
```

Code after migration:

<!-- skip -->
```dart
main() {
  testWidgets('example test', (tester) {
    final AnimationSheetBuilder animationSheet = AnimationSheetBuilder(frameSize: const Size(50, 50));

    await tester.pumpFramesFor(animationSheet.record(
      const RefreshProgressIndicator(),
    ), const Duration(seconds: 3));
  });
}
```

### Other affected methods

Code before migration:

```dart
main() {
  testWidgets('example test', (tester) {
    // These calls are fine, as the Durations do not include microseconds
    await tester.pumpWidget(const Container(), Duration(milliseconds:16));
    await tester.pump(Duration(milliseconds:16));
    await tester.pumpAndSettle(Duration(milliseconds: 160));


    // These calls will see a behavior change due to the microseconds in the
    // Durations that are eventually passed to `pump`. Any changes to golden
    // files after these methods with microseconds in the Durations are
    // expected and safe to accept.
    await tester.pumpWidget(
      const Container(),
      Duration(milliseconds:16, microseconds: 683));
    await tester.pump(Duration(milliseconds:16, microseconds: 683));
    await tester.pumpAndSettle(Duration(milliseconds: 166, microseconds: 830));
  });
}
```

## Timeline

Landed in version: xxx<br>
In stable release: not yet

## References

{% comment %}
  These links are commented out because they
  cause the GitHubActions (GHA) linkcheck to fail.
  Remove the comment tags once you fill this in with
  real links. Only use the "master-api" include if
  you link to "master-api.flutter.dev".

{% include docs/master-api.md %}

API documentation:

* [`AutomatedTestWidgetsFlutterBinding`][{{site.api}}/flutter/flutter_test/AutomatedTestWidgetsFlutterBinding/pump.html]
* [`WidgetTester.pump`]({{site.api}}/flutter/flutter_test/WidgetTester/pump.html)
* [`WidgetTester.pumpAndSettle`]({{site.api}}/flutter/flutter_test/WidgetTester/pumpAndSettle.html)
* [`WidgetTester.pumpFrames`]({{site.api}}/flutter/flutter_test/WidgetTester/pumpFrames.html)
* [`WidgetTester.pumpWidget`]({{site.api}}/flutter/flutter_test/WidgetTester/pumpWidget.html)

Relevant issues:

* [Issue 112610][{{site.repo.flutter}}/issues/112610]

Relevant PRs:

* [PR title #1][]
{% endcomment %}
