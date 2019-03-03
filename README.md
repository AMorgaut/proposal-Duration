# ECMAScript `Duration` feature proposal

**Stage**: 0

**Author**: Alexandre Morgaut

**Champion**: To be found

## Problems

- Working with Dates & Durations is often tricky and error prone. Current ECMAScript state requires to either work with UTC based millisecond timestamps or with date objects. 
- Using timestamps is really not adapted to store values concepts such as number of months or years which can have different number of days among the time. Even the number of hours in a day can change to handle winter vs summer time.
- Some native APIs like `setTimeout()` & `setInterval()` are expecting durations but only support number of milliseconds
- It can be complicated to understand the result of a diff between 2 dates (how much years/months/days considering timezones, winter times, ...)
- While there is some ES 402 I18n APIs for dates, none exists for durations which are very much related, projects can sometimes keep dependencies of 3rd party libraries just for the durations part.

## Proposal

Introduce a new `Duration` native object & related additional `Date.prototype` methods:

- Providing a useful API to generate / read / edit durations using different units
- Supporting the `ISO 8601` duration standard
- Supporting **I18n** APIs
  - handling plural in `toLocaleString()`
  - in a next step adding a `Intl.DurationFormat` to the ECMA 402 API
- Integrated by JS engines as part of the `Timer`/`performance` APIs
  - use 'millisecond' as default unit like `Date.now()` & `performance.now()`
  - comply to the `privacy.reduceTimerPrecision` limitations
- Hopefully allowing **engine optimizations** for duration & date operations

For usage examples, the ISO 8601 standard is used for durations by the YouTube video API:
https://developers.google.com/youtube/v3/docs/videos#contentDetails.duration

## Examples

```javascript
const duration = new Duration('P1D2H'); // 1 day & 2 hours in ISO 8601

console.log( duration.equals(new Duration({ days: 1, hours: 2 })) ); // true

duration.add('P20H'); // ISO string operation
duration.addHours(2); // number operation
console.log(duration.toLocaleString()); // "2 days"
console.log(duration.toISOString()); // "P2D"
console.log(Number(duration)) // 172800000 (in milli-seconds for compatibility with ES Dates)

const duration2 = new Duration(1628000005);
console.log(duration2 > duration); // true -> comparison is done by valueOf() that returns the timestamp

const delay = new Duration('PT1M'); // 1 minute
setTimeout(callback, delay);
```

Of course would be awesome if it could support '+' & '-' operators, but would need too much changes in the standard right now.

## Prior Art

- Java: https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html
- Rust: https://doc.rust-lang.org/std/time/struct.Duration.html
- Dart: https://api.dartlang.org/stable/2.1.0/dart-core/Duration-class.html
- Scala: https://www.scala-lang.org/api/2.12.3/scala/concurrent/duration/Duration.html
- Ruby on Rails: https://api.rubyonrails.org/classes/ActiveSupport/Duration.html
- Go: https://golang.org/pkg/time/#Duration
- C++: http://www.cplusplus.com/reference/chrono/duration/duration/
- .NET C#: https://docs.microsoft.com/en-us/dotnet/api/system.timespan.duration
- Moment.js: https://momentjs.com/docs/#/durations/

Similar concepts

- PHP: http://php.net/manual/fr/class.dateinterval.php
- Swift/Objective C (Foundation Framework): https://developer.apple.com/documentation/foundation/timeinterval
- .NET Visual Basic/C#: https://docs.microsoft.com/en-us/dotnet/api/microsoft.visualbasic.dateinterval?view=netframework-4.7.2

## Duration Object (Draft)

Would take place in the [`Numbers & Dates` ES specification section](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-numbers-and-dates)

`Date` *abstract operations* like [DaysInYear(y)](https://www.ecma-international.org/ecma-262/9.0/index.html#eqn-DaysInYear) or [HourFromTime(t)](https://www.ecma-international.org/ecma-262/9.0/index.html#eqn-HourFromTime) should be reused for Duration objects.

### Constructor

The Duration constructor:

- is the intrinsic object *%Duration%*.
- is the initial value of the **Duration** property of the global object.
- creates and initializes a new Duration object when called as a constructor.
- returns the zero number value representing the zero duration when called as a function rather than as a constructor.
- is a single function whose behaviour is overloaded based upon the number and types of its arguments.
- is designed to be subclassable. It may be used as the value of an **extends** clause of a class definition. Subclass constructors that intend to inherit the specified **Duration** behaviour must include a **super** call to the **Duration** constructor to create and initialize the subclass instance with a [[DateValue]] internal slot.
- has a **length** property whose value is 2.

#### new Duration(string)

This description applies only if the Duration constructor is called with a string as first argument.

First try to construct a Duration object from an ISO 8601 Duration string.
If the string is not a valid duration ISO string, try to interpret it as a locale duration string.

Behave the same as `Duration.parse(string)`

#### new Duration(number[, unit])

This description applies only if the Duration constructor is called with a number as first argument.

Construct a Duration object from a number value and the associated unit represented by a string enum ('year', 'month', ...)
The `unit`string enum could support shorthand aliases (ex: 'ms' for 'millisecond")
For easier integration with JS Date timestamps, the default unit would be 'ms'

#### new Duration(options)

This description applies only if the Duration constructor is called with an object as first argument.

Construct a Duration object from an object holding the values parts)

ex: `new Duration({ week: 12 })`

### Properties of the Duration Constructor

#### Duration.between(date1, date2)

Construct a Duration object from the difference between 2 dates.

#### Duration.parse(string)

First try to construct a Duration object from an ISO 8601 Duration string.
If the string is not a valid duration ISO string, try to interpret it as a locale duration string.

#### Duration.prototype

The initial value of `Duration.prototype` is the intrinsic object `%DurationPrototype%`.

This property has the attributes { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }.

### Properties of the Duration Prototype Object

The Duration prototype object:

- is the intrinsic object *%DatePrototype%*.
- is itself an ordinary object.
- is not a Duration instance and does not have a `[[DurationValue]]` internal slot.
- has a `[[Prototype]]` internal slot whose value is the intrinsic object *%ObjectPrototype%*.

#### Duration.prototype.add{time unit}(value)

Add the specified duration to the instance value

ex: `duration.addDays(5);`

#### Duration.prototype.as{time unit}()

Returns a number representing the **full** duration **in the desired unit**

ex: `const days = duration.asDays();`

#### Duration.prototype.equals(duration)

Returns true if the current instance and the given attribute represent the same duration

ex: `duration.equals(duration2); // true or false`

#### Duration.prototype.get{time unit}()

Returns a number representing **the part** of the duration corresponding of the **desired unit**

ex: `const days = duration.getDays();`

#### Duration.prototype.set{time unit}(value)

Update **the part** of the duration corresponding of the **desired unit** with the specified value number

ex: `duration.setDays(5);`

#### Duration.prototype.subtract{time unit}(value)

Removes the specified duration to the instance value

ex: `duration.subtractDays(5);`

#### Duration.prototype.toLocaleString()

Returns a locale string representation of the duration value

ex: `duration.toLocaleString(); // "2 days"`

#### Duration.prototype.toISOString()

Returns an ISO 8601 string representation of the duration value

ex: `duration.toISOString(); // "P2D"`

#### Duration.prototype.toString()

To be defined

Might be implementation-dependent like of the toString family Date methods

#### Duration.prototype.valueOf()

Returns an "date indempentant"* timestamp representation of the duration value

ex: `duration.valueOf(); // 172800000`

> To clarify: 
> It can be complex to choose how to represent, as a number, a 3 month duration as months can have 28, 29, 30, or 31 days; and same complexity comes for years or even days.

## Additional properties to the Date Prototype Object (Draft)

#### Date.prototype.add(duration)

Update the date by adding the specified duration to the instance value

ex:
```
const date = new Date('2019-01-25T12:25:00.000Z');
const delay = new Duration('P2M');
date.add(delay);
console.log(date.toLocaleString); // "3/25/2019, 1:25:00 PM"
```

#### Date.prototype.subtract(duration)

Update the date by removing the specified duration to the instance value
`


## Intl.DurationFormat API overview

TODO in a dedicated document

based on [ECMA 402 DateTimeFormat Objects](https://www.ecma-international.org/ecma-402/4.0/#datetimeformat-objects)
