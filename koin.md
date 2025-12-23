
# 1️⃣ What is Koin?

**Koin is a Dependency Injection (DI) framework for Kotlin.**

In simple words:

> **Koin creates objects for you and gives them to the classes that need them.**

Instead of doing this everywhere:

```kotlin
val client = HttpClient(...)
val api = ApiServiceImpl(client)
val repo = NewsRepository(api)
val vm = NewsViewModel(repo, scope)
```

You tell Koin:

> “Here is how to create these things.
> When someone asks for one, give it to them.”

---

# 2️⃣ Why do we need Dependency Injection at all?

Without DI:

* Classes create their own dependencies
* Code becomes tightly coupled
* Testing is painful
* Multiplatform becomes messy

Example ❌:

```kotlin
class NewsViewModel {
    private val api = ApiServiceImpl(HttpClient())
}
```

Problems:

* Can’t replace API with fake
* Can’t reuse HttpClient
* Android/iOS logic leaks
* Hard to scale

DI **inverts control**:

> Classes *receive* what they need instead of *creating* it.

---

# 3️⃣ Why Koin specifically (especially for KMP)?

There are many DI frameworks:

| Framework | KMP support    | Complexity  |
| --------- | -------------- | ----------- |
| Dagger    | ❌ No           | Very high   |
| Hilt      | ❌ Android only | Medium      |
| Koin      | ✅ Yes          | Low         |
| Manual DI | ✅ Yes          | Error-prone |

### Why Koin fits KMP perfectly

* Pure Kotlin (no annotation processing)
* Runtime DI (works on iOS)
* Very small learning curve
* No code generation
* Works in `commonMain`

That’s why **Koin is the de-facto DI for KMP**.

---

# 4️⃣ Core concepts of Koin (VERY IMPORTANT)

There are only **4 core ideas** you must understand:

1. Module
2. Definition (`single`, `factory`)
3. Koin container
4. Injection (`get()`)

Let’s go one by one.

---

## 4.1️⃣ Module

A **module** is just a **list of instructions** for Koin.

```kotlin
val networkModule = module {
    // definitions go here
}
```

It answers:

> “How do I create objects?”

---

## 4.2️⃣ Definitions: `single` vs `factory`

### 🔹 `single`

> Create **one instance** and reuse it everywhere.

```kotlin
single { ApiClient().client }
```

Same instance every time.

Use for:

* HttpClient
* Repository
* Database
* CoroutineScope

---

### 🔹 `factory`

> Create a **new instance every time**.

```kotlin
factory { NewsViewModel(get(), get()) }
```

Use for:

* ViewModels
* UseCases
* Short-lived objects

---

### 🔑 Rule to remember

| Object type    | Koin definition |
| -------------- | --------------- |
| HttpClient     | `single`        |
| ApiService     | `single`        |
| Repository     | `single`        |
| ViewModel      | `factory`       |
| CoroutineScope | `single`        |

---

## 4.3️⃣ The Koin Container

When you start Koin, it creates a **container**.

```kotlin
startKoin {
    modules(networkModule)
}
```

This container:

* Stores all definitions
* Creates objects lazily
* Resolves dependencies

Think of it as a **map of object creators**.

---

## 4.4️⃣ Injection with `get()`

Inside a definition:

```kotlin
single {
    NewsRepository(get())
}
```

`get()` means:

> “Give me whatever you know how to create for this type.”

Koin looks at the container and finds:

```kotlin
single { ApiServiceImpl(...) }
```

And injects it.

---

# 5️⃣ How dependency resolution works (important mental model)

When this runs:

```kotlin
val vm = getKoin().get<NewsViewModel>()
```

Koin does:

1. Looks for `NewsViewModel` definition
2. Sees it needs `NewsRepository` + `CoroutineScope`
3. Resolves those
4. Resolves *their* dependencies
5. Returns a fully built object

This is called **dependency graph resolution**.

---

# 6️⃣ Koin in KMP (how it really works)

### Shared (`commonMain`)

* Modules
* Definitions
* Interfaces
* Business logic

### Platform (`androidMain`, `iosMain`)

* Start Koin
* Provide platform-specific things

Example (Android):

```kotlin
startKoin {
    androidContext(this@MainActivity)
    modules(networkModule)
}
```

Example (iOS):

```swift
KoinKt.startKoin()
```

Same shared logic, different bootstrap.

---

# 7️⃣ Why Koin is runtime DI (and why that’s OK)

Unlike Hilt/Dagger:

* Koin resolves dependencies **at runtime**
* No code generation
* No compile-time graph

### Is this bad?

❌ No — for KMP it’s actually **necessary**

Reasons:

* iOS has no annotation processing
* KMP needs runtime flexibility
* Errors are easy to catch early

Tradeoff:

* Slight runtime overhead (negligible for apps)

---

# 8️⃣ Koin + Compose (critical rule you already learned)

❌ Wrong:

```kotlin
val vm = getKoin().get<NewsViewModel>()
```

Why?

* Compose recomposes
* `factory` creates new VM
* State resets

✅ Correct:

```kotlin
val vm = remember {
    getKoin().get<NewsViewModel>()
}
```

Or scope it properly.

---

# 9️⃣ Koin scopes (advanced but important)

Scopes let you control **lifetime**.

Example:

```kotlin
scope<NewsScreen> {
    scoped { NewsViewModel(get(), get()) }
}
```

Now:

* ViewModel exists only while screen exists
* Automatically cleaned up

Useful for:

* Navigation
* Large apps

---

# 🔟 Koin vs Hilt (clear comparison)

| Feature             | Koin | Hilt  |
| ------------------- | ---- | ----- |
| KMP support         | ✅    | ❌     |
| Setup               | Easy | Heavy |
| Compile-time safety | ❌    | ✅     |
| Runtime flexibility | ✅    | ❌     |
| Learning curve      | Low  | High  |
| iOS support         | ✅    | ❌     |

**For Android-only → Hilt is great**
**For KMP → Koin is the correct choice**

---

# 1️⃣1️⃣ Testing with Koin (huge advantage)

You can replace dependencies easily:

```kotlin
startKoin {
    modules(
        module {
            single<ApiService> { FakeApiService() }
        }
    )
}
```

No annotations. No mocks. Very clean.

---

# 1️⃣2️⃣ Common Koin mistakes (avoid these)

❌ Creating ViewModel without `remember`
❌ Using `factory` for HttpClient
❌ Starting Koin multiple times
❌ Putting Android code in `commonMain`
❌ Forgetting lifecycle of CoroutineScope

You avoided most of these already 👍

---

# 🧠 FINAL MENTAL MODEL (THIS IS KEY)

```
Koin = Object Factory + Object Store
```

* `module` → instructions
* `single` → one object
* `factory` → many objects
* `get()` → request object
* `startKoin` → start engine

---

# ✅ FINAL TAKEAWAYS

1. Koin is a **DI container**
2. It manages object creation & lifecycle
3. `single` vs `factory` is crucial
4. Perfect fit for KMP
5. Runtime DI is intentional
6. Compose requires `remember`
7. Testing is extremely easy
8. This is production-ready DI

---

If you want next:

* 🔥 Koin scopes with navigation
* 🔥 Koin vs manual DI in KMP
* 🔥 Advanced testing patterns
* 🔥 Performance considerations


