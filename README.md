![img](https://i.imgur.com/3rNrEjC.png)

<h3 align="center"><i>Persistent limits. Effortlessly automated.</i></h3>
<p align="center">
        <img src="https://img.shields.io/codefactor/grade/github/jozzdart/limit?style=flat-square">
        <img src="https://img.shields.io/github/license/jozzdart/limit?style=flat-square">
        <img src="https://img.shields.io/pub/points/limit?style=flat-square">
        <img src="https://img.shields.io/pub/v/limit?style=flat-square">
        
</p>
<p align="center">
  <a href="https://buymeacoffee.com/yosefd99v" target="https://buymeacoffee.com/yosefd99v">
    <img src="https://img.shields.io/badge/Buy%20me%20a%20coffee-Support (:-blue?logo=buymeacoffee&style=flat-square" />
  </a>
</p>

One line. No boilerplate. No setup. The **limit** package gives you instant, persistent control over cooldowns and rate limits — across sessions, isolates, and app restarts. Define once, automate forever.

#### Table of Contents

- ⏲ [**Cooldown**](#-cooldown-persistent-cooldown-service) — automatically manage cooldown periods (e.g. daily rewards, retry delays)
- 📊 [**Rate Limiter**](#-ratelimiter-token-bucket-rate-limiter) — control rates using a token bucket (e.g. 1000 actions per 15 minutes)
- 📦 [**More Packages**](#-more-jozz-packages) — other packages by Jozz

---

### Why Use `limit`?

Working with cooldowns and rate limits usually means:

- Manual `DateTime` comparisons
- Writing timers or expiration logic
- Saving timestamps or counters
- Handling null, casting, and cleanup

**limit** removes all that:
you just define, call, and trust it.

- ✅ Lets you **define, control, and forget** — the system handles everything in the background
- ✅ One-line setup, no manual storage or timers
- ✅ Persisted across app restarts and isolates
- ✅ Async-safe and cache-friendly
- ✅ Works great for **daily rewards**, **retry delays**, **API limits**, **chat quotas**, and more

---

### Choosing the Right Limiter

Each limiter is tailored for a specific pattern of time-based control.

| Goal                            | Use                                                      |
| ------------------------------- | -------------------------------------------------------- |
| "Only once every X time"        | [`Cooldown`](#-cooldown-persistent-cooldown-service)     |
| "Allow N actions per Y minutes" | [`RateLimiter`](#-ratelimiter-token-bucket-rate-limiter) |

[**⏲ `Cooldown`**](#-cooldown-persistent-cooldown-service)

> _"Only once every 24 hours"_  
> → Fixed cooldown timer from last activation  
> → Great for claim buttons, retry delays, or cooldown locks

[**📊 `RateLimiter`**](#-ratelimiter-token-bucket-rate-limiter)

> _"Allow 100 actions per 15 minutes (rolling refill)"_  
> → Token bucket algorithm  
> → Replenishes tokens over time (not per action)  
> → Great for APIs, messaging, or hard quota control

---

# ⏲ `Cooldown` Persistent Cooldown Service

_[⤴️ Back](#table-of-contents) -> Table of Contents_

`Cooldown` is a plug-and-play utility service for managing **cooldown windows** (e.g. daily rewards, button lockouts, retry delays) that persist across sessions and isolates — no timers, no manual bookkeeping, no re-implementation every time.

It handles:

- Cooldown timing (`DateTime.now()` + duration)
- Persistent storage (with caching and async-safety)
- Activation tracking and expiration logic
- Usage statistics (activation count, expiry progress, etc.)

---

### 🔧 How to Use

- `isCooldownActive()` — Returns `true` if the cooldown is still active
- `isExpired()` — Returns `true` if the cooldown has expired or was never started
- `activateCooldown()` — Starts the cooldown using the configured duration
- `tryActivate()` — Starts cooldown only if it's not active — returns whether it was triggered
- `reset()` — Clears the cooldown timer, but keeps the activation count
- `completeReset()` — Fully resets both the cooldown and its usage counter
- `timeRemaining()` — Returns remaining time as a `Duration`
- `secondsRemaining()` — Same as above, in seconds
- `percentRemaining()` — Progress indicator between `0.0` and `1.0`
- `getLastActivationTime()` — Returns `DateTime?` of last activation
- `getEndTime()` — Returns when the cooldown will end
- `whenExpires()` — Returns a `Future` that completes when the cooldown ends
- `getActivationCount()` — Returns the total number of activations
- `removeAll()` — Deletes all stored values (for testing/debugging)
- `anyStateExists()` — Returns `true` if any cooldown data exists in storage

---

#### ✅ Define a Cooldown

```dart
final cooldown = Cooldown('daily_reward', duration: Duration(hours: 24));
```

This creates a persistent cooldown that lasts 24 hours. It uses the prefix `'daily_reward'` to store:

- Last activation timestamp
- Activation count

---

#### 🔍 Check If Cooldown Is Active

```dart
if (await cooldown.isCooldownActive()) {
  print('Wait before trying again!');
}
```

---

#### ⏱ Activate the Cooldown

```dart
await cooldown.activateCooldown();
```

This sets the cooldown to now and begins the countdown. The activation count is automatically incremented.

---

#### ⚡ Try Activating Only If Expired

```dart
if (await cooldown.tryActivate()) {
  print('Action allowed and cooldown started');
} else {
  print('Still cooling down...');
}
```

Use this for one-line cooldown triggers (e.g. claiming a daily gift or retrying a network call).

---

#### 🧼 Reset or Fully Clear Cooldown

```dart
await cooldown.reset();         // Clears only the time
await cooldown.completeReset(); // Clears time and resets usage counter
```

---

#### 🕓 Check Time Remaining

```dart
final remaining = await cooldown.timeRemaining();
print('Still ${remaining.inMinutes} minutes left');
```

You can also use:

```dart
await cooldown.secondsRemaining();   // int
await cooldown.percentRemaining();   // double between 0.0–1.0
```

---

#### 📅 View Timing Info

```dart
final lastUsed = await cooldown.getLastActivationTime();
final endsAt = await cooldown.getEndTime();
```

---

#### ⏳ Wait for Expiry (e.g. for auto-retry)

```dart
await cooldown.whenExpires(); // Completes only when cooldown is over
```

---

#### 📊 Get Activation Count

```dart
final count = await cooldown.getActivationCount();
print('Used $count times');
```

---

#### 🧪 Test Utilities

```dart
await cooldown.removeAll();                     // Clears all stored cooldown state
final exists = await cooldown.anyStateExists(); // Returns true if anything is stored
```

> You can create as many cooldowns as you need — each with a unique prefix.
> All state is persisted, isolate-safe, and instantly reusable.

---

### ⚡ Optional `useCache` Parameter

Each limiter accepts a `useCache` flag:

```dart
final cooldown = Cooldown(
  'name_key',
  duration: Duration(minutes: 5),
  useCache: true // false by default
);
```

- `useCache: false` (default):

  - Fully **isolate-safe**
  - Reads directly from storage every time
  - Best when multiple isolates might read/write the same data

- `useCache: true`:
  - Uses **memory caching** for faster access
  - **Not isolate-safe** — may lead to stale or out-of-sync data across isolates
  - Best when used in single-isolate environments (most apps)

> ⚠️ **Warning**: Enabling `useCache` disables isolate safety. Use only when you're sure no other isolate accesses the same key.

---

# 📊 `RateLimiter` Token Bucket Rate Limiter

[⤴️ Back](#table-of-contents) -> Table of Contents

`RateLimiter` is a high-performance, plug-and-play utility that implements a **token bucket** algorithm to enforce rate limits — like “100 actions per 15 minutes” — across sessions, isolates, and app restarts.

It handles:

- Token-based rate limiting
- Automatic time-based token refill
- Persistent state using `prf` types (`PrfIso<double>`, `PrfIso<DateTime>`)
- Async-safe, isolate-compatible behavior

Perfect for chat limits, API quotas, retry windows, or any action frequency cap — all stored locally.

---

### 🔧 How to Use

- `tryConsume()` — Tries to use 1 token; returns `true` if allowed, or `false` if rate-limited
- `isLimitedNow()` — Returns `true` if no tokens are currently available
- `isReady()` — Returns `true` if at least one token is available
- `getAvailableTokens()` — Returns the current number of usable tokens (calculated live)
- `timeUntilNextToken()` — Returns a `Duration` until at least one token will be available
- `nextAllowedTime()` — Returns the exact `DateTime` when a token will be available
- `reset()` — Resets to full token count and updates last refill to now
- `removeAll()` — Deletes all limiter state (for testing/debugging)
- `anyStateExists()` — Returns `true` if limiter data exists in storage
- `runIfAllowed(action)` — Runs a callback if allowed, otherwise returns `null`
- `debugStats()` — Returns detailed internal stats for logging and debugging

The limiter uses fractional tokens internally to maintain precise refill rates, even across app restarts. No timers or background services required — it just works.

---

#### ✅ `RateLimiter` Basic Setup

Create a limiter with a key, a maximum number of actions, and a refill duration:

```dart
final limiter = RateLimiter(
  'chat_send',
  maxTokens: 100,
  refillDuration: Duration(minutes: 15),
);
```

This example allows up to **100 actions per 15 minutes**. The token count is automatically replenished over time — even after app restarts.

---

#### 🚀 Check & Consume

To attempt an action:

```dart
final canSend = await limiter.tryConsume();

if (canSend) {
  // Allowed – proceed with the action
} else {
  // Blocked – too many actions, rate limit hit
}
```

Returns `true` if a token was available and consumed, or `false` if the limit was exceeded.

---

#### 🧮 Get Available Tokens

To check how many tokens are usable at the moment:

```dart
final tokens = await limiter.getAvailableTokens();
print('Tokens left: ${tokens.toStringAsFixed(2)}');
```

Useful for debugging, showing rate limit progress, or enabling/disabling UI actions.

---

#### ⏳ Time Until Next Token

To wait or show feedback until the next token becomes available:

```dart
final waitTime = await limiter.timeUntilNextToken();
print('Try again in: ${waitTime.inSeconds}s');
```

You can also get the actual time point:

```dart
final nextTime = await limiter.nextAllowedTime();
```

---

#### 🔁 Reset the Limiter

To fully refill the bucket and reset the refill clock:

```dart
await limiter.reset();
```

Use this after manual overrides, feature unlocks, or privileged user actions.

---

#### 🧼 Clear All Stored State

To wipe all saved token/refill data (for debugging or tests):

```dart
await limiter.removeAll();
```

To check if the limiter has any stored state:

```dart
final exists = await limiter.anyStateExists();
```

---

### ⚡ Optional `useCache` Parameter

Each limiter accepts a `useCache` flag:

```dart
final limiter = RateLimiter(
  'key',
  maxTokens: 10,
  refillDuration: Duration(minutes: 5),
  useCache: true // false by default
);
```

- `useCache: false` (default):

  - Fully **isolate-safe**
  - Reads directly from storage every time
  - Best when multiple isolates might read/write the same data

- `useCache: true`:
  - Uses **memory caching** for faster access
  - **Not isolate-safe** — may lead to stale or out-of-sync data across isolates
  - Best when used in single-isolate environments (most apps)

> ⚠️ **Warning**: Enabling `useCache` disables isolate safety. Use only when you're sure no other isolate accesses the same key.

_[⤴️ Back](#table-of-contents) -> Table of Contents_

---

# 📦 More `jozz` Packages

_[⤴️ Back](#table-of-contents) → Table of Contents_

I’m Jozz — and my packages share a simple philosophy: **developer experience first**.
I try to avoid boilerplate wherever possible, and most of these packages were born out of real needs in my own projects. Each one comes with clear documentation, minimal setup, and APIs that are easy to pick up without surprises.

They’re built to be lightweight, reliable, and ready for production, always with simplicity in mind. There are more packages in the works, following the same approach.
If you find them useful and feel like supporting, you’re welcome to do so (:

<p>
  <a href="https://buymeacoffee.com/yosefd99v" target="https://buymeacoffee.com/yosefd99v">
    ☕ Buy me a coffee
  </a>
</p>

- [shrink](#-shrink--compress-anything-in-one-line) – Compress Anything in One Line
- [track](#-track--persistent-streaks-counters--records) – Persistent Streaks, Counters & Records
- [hivez](#-hivez--hive-but-safer--smarter) – Hive, but Safer & Smarter
- [time_plus](#-time_plus--smarter-datetime--duration-extensions) – Smarter DateTime & Duration Extensions
- [prf](#-prf--sharedpreferences-without-the-pain) – SharedPreferences, Without the Pain
- [exui](#-exui--supercharge-your-flutter-ui) – Supercharge Your Flutter UI
- [jozz_events](#-jozz_events--strongly-typed-events-for-clean-architecture) – Strongly-Typed Events for Clean Architecture

### 🔽 [shrink](https://pub.dev/packages/shrink) – Compress Anything in One Line

Because every byte counts. `shrink` makes data compression effortless with a **one-line API** and fully lossless results. It auto-detects the best method, often cutting size by **5× to 40×** (and up to **1,000×+** for structured data). Perfect for **Firestore, local storage, or bandwidth-sensitive apps**. Backed by clear docs and real-world benchmarks.

### 📊 [track](https://pub.dev/packages/track) – Persistent Streaks, Counters & Records

Define once, track forever. `track` gives you plug-and-play tools for **streaks, counters, activity logs, and records** — all persisted safely across sessions and isolates. From **daily streaks** to **rolling counters** to **best-ever records**, it handles resets, history, and storage automatically. Clean APIs, zero boilerplate, and deeply detailed documentation.

### 🐝 [hivez](https://pub.dev/packages/hivez) – Hive, but Safer & Smarter

`hivez` is a production-ready layer on top of Hive CE that keeps its raw speed but makes it safer and easier to use. It auto-initializes boxes, enforces type safety, and gives you a single unified API for Box, LazyBox, and IsolatedBox. Concurrency issues are handled with built-in locks, and you also get extras like backup/restore, search, and crash recovery. Backed by clear, detailed documentation, `hivez` is designed for real-world apps where you want Hive’s performance without the boilerplate or pitfalls.

### ⏱ [time_plus](https://pub.dev/packages/time_plus) – Smarter DateTime & Duration Extensions

Stop wrestling with `DateTime` and `Duration`. `time_plus` adds the missing tools you wish Dart had built in: **add and subtract time units**, **start/end of day/week/month**, **compare by precision**, **yesterday/tomorrow**, **fractional durations**, and more. Built with **128+ extensions**, **700+ tests**, and **zero dependencies**, it’s faster, more precise, and more reliable than the classic `time` package — while keeping APIs clear and intuitive. Ideal for **scheduling, analytics, or any app where every microsecond counts**.

### ⚡ [prf](https://pub.dev/packages/prf) – SharedPreferences, Without the Pain

No strings, no boilerplate, no setup. `prf` lets you define variables once, then `get()` and `set()` them anywhere with a **type-safe API**. It fully replaces raw `SharedPreferences` with support for **20+ built-in types** (including `DateTime`, `Duration`, `Uint8List`, JSON, and enums). Every variable is cached, test-friendly, and isolate-safe with a `.isolated` mode. Designed for **clarity, scale, and zero friction**, with docs that make local persistence finally headache-free.

### 🎨 [exui](https://pub.dev/packages/exui) – Supercharge Your Flutter UI

Everything your widgets wish they had. `exui` is a **zero-dependency extension library** for Flutter with **200+ chainable utilities** for padding, margin, centering, gaps, visibility, constraints, gestures, buttons, text styling, and more — all while keeping your widget tree fully native.

No wrappers. No boilerplate. Just concise, expressive methods that feel built into Flutter itself. Backed by **hundreds of unit tests** and **exceptional documentation**, `exui` makes UI code cleaner, faster, and easier to maintain.

### 📢 [jozz_events](https://pub.dev/packages/jozz_events) – Strongly-Typed Events for Clean Architecture

A **domain-first, framework-agnostic event bus** built for scalable apps. `jozz_events` enables **decoupled, strongly-typed communication** between features and layers — without the spaghetti. It’s lightweight, dependency-free, lifecycle-aware, and integrates naturally with **Clean Architecture**. Ideal for Flutter or pure Dart projects where modularity, testability, and clarity matter most.

## 🔗 License MIT © Jozz

<p align="center">
  <a href="https://buymeacoffee.com/yosefd99v" target="https://buymeacoffee.com/yosefd99v">
    ☕ Enjoying this package? You can support it here.
  </a>
</p>
