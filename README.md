# ECMAScript `Duration` feature proposal

**Stage**: 0

**Author**: Alexandre Morgaut

**Champion**: To be found

## Problems

- Working with Dates & Durations is often tricky and error prone. Current ECMAScript state requires to either work with timestamps or with date objects. 
- Using timestamps is really not adapted to store values concepts such as number of months or years which can have different number of days among the time. even the number of hours in a day can change to handle winter vs summer time.
- Some native APIs like `setTimeout()` & `setInterval()` are expecting durations but only support number of milliseconds
- It can be complicated to understand the result of a diff between 2 dates (how much years/months/days considering timazones, winter times, ...)

## Proposal

Introduce a `Duration` native object:
- Providing a useful API to read / edit a duration using different units
- Allowing **engine optimizations** for duration & date operations
- supporting the `ISO 8601` duration standard
- that Web engines & ssjs engines could support within Timer APIs

For usage examples, the ISO 8601 standard is used for durations by the YouTube video API:
https://developers.google.com/youtube/v3/docs/videos#contentDetails.duration

## Example

```javascript
const duration = new Duration('P1D2H'); // 1 day & 2 hours in ISO 8601
duration.add('P20H'); // ISO string operation
duration.addHours(2); // number operation
console.log(duration.toLocaleString()); // 2 days
console.log(duration.toISOString()); // P2D
console.log(Number(duration)) // 172800000

const delay = new Duration('PT1M'); // 1 minute
setTimeout(callback, delay);
```

Of course would be awesome if it could support '+' & '-' operators, but would need too much changes in the standard right now.

## Prior Art

- Java: https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html
- Rust: https://doc.rust-lang.org/std/time/struct.Duration.html
- Dart: https://api.dartlang.org/stable/2.1.0/dart-core/Duration-class.html
- Scala: https://www.scala-lang.org/api/2.12.3/scala/concurrent/duration/Duration.html
- Ruby: https://api.rubyonrails.org/classes/ActiveSupport/Duration.html
- Go: https://golang.org/pkg/time/#Duration
- Moment.js: https://momentjs.com/docs/#/durations/
