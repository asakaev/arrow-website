---
id: from-fp
title: Compose and UIs
sidebar_position: 3
---

Arrow provides several features which are very interesting when developing 
interactive applications, especially in combination with libraries with a
similar functional flavor, such as Compose.

:::info Example projects

Projects using Compose and Arrow can be found in the
[corresponding section](../../design/projects/).

:::

:::note Multiplatform-ready

All the libraries under the Arrow umbrella are 
[Multiplatform](https://kotlinlang.org/docs/multiplatform.html)-ready.
This means you can use them in your Android applications using
[Jetpack Compose](https://developer.android.com/jetpack/compose),
and in Desktop, iOS, or Web alongside
[Compose Multiplaftorm](https://www.jetbrains.com/lp/compose-multiplatform/).

:::

## Compose is functional

As opposed to other frameworks where stateful components are the norm, the
[architecture](https://developer.android.com/jetpack/compose/architecture)
promoted by Compose brings many concepts traditionally associated with a
more functional approach. For example, the UI is defined as a _function_
taking as arguments the _values_ from the current state. Updating the state
is also explicitly marked, and often separated in a `ViewModel`.

As a consequence, Arrow and Compose make great dancing partners. Below we
discuss a few feature which we think are an immediately gain for Android
(and with Compose Multiplatform, other UI) developers. In the same vein, our
[design](../design/) section readily applies to projects using Compose. 

## Simpler effectful code

Most applications don't live in a vacuum, they need to access other services
or data sources. In those cases we write _effectful_ code, where `suspend` and
coroutines become relevant.

Arrow Fx introduces
[high-level concurrency](../../coroutines/parallel/) as a way to simplify code
where different actions must happen concurrently, ensuring that all rules
of Structured Concurrency are followed, but without the hassle.

```kotlin
suspend fun loadUserData(userId: UserId): UserData =
  parZip(
    { downloadAvatar(userId) },
    { UserRepository.getById(userId) }
  ) { avatarFile, user ->
    // this code is called once both finish
  }
```

Anything outside your own application is the wilderness: connections are
down, services are unavailable. Arrow's [resilience](../../resilience/)
module provides several ready-to-use patterns to better handle those situations,
including [retry policies](../../resilience/retry-and-repeat/)
and [circuit breakers](../../resilience/circuitbreaker/).

## Updating the model

One potential drawback of using
[immutable data to model your state](../../design/domain-modeling/)
is that updating it can become quite tiresome, because Kotlin provides
no dedicated feature other than `copy` for this task.

```kotlin
class UserSettingsModel: ViewModel() {
  private val _userData = mutableStateOf<UserData?>(null)
  val userData: State<UserData?> get() = _userData

  fun updateName(
    newFirstName: String, newLastName: String
  ) {
    _userData.value = _userData.value.copy(
      details = _userData.value.details.copy(
        name = _userData.value.details.name.copy(
          firstName = newFirstName, lastName = newLastName
        )
      )
    )
  }
}
```

Arrow Optics addresses these drawbacks, providing 
[tools for manipulating and transforming _immutable_ data](../../immutable-data/intro/).
The code above can be rewritten without boring repetition.

```kotlin
class UserSettingsModel: ViewModel() {
  private val _userData = mutableStateOf<UserData?>(null)
  val userData: State<UserData?> get() = _userData

  fun updateName(
    newFirstName: String, newLastName: String
  ) {
    _userData.value = _userData.value.copy {
      inside(UserData.details.name) {
        Name.firstName set newFirstName
        Name.lastName  set newLastName
      }
    }
  }
}
```