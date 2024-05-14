# Data Storage

- https://www.youtube.com/watch?v=bo8YAkPi8Po
- https://developer.android.com/guide/topics/data

## Intro

What to Store:
- Preferences & Settings
- Content & Information
- User Data
- Temporary data

Storage criterias:
- Storage size
- Access speed
- Security & Protection
- Accessibility & Reliability
- Search & analysis capabilities
- Support costs

Where to store:
- Cloud Storage
	- ( + ) Storage size: better than local storage, but still limited
	- ( - ) Access speed: over internet connection
	- ( + ) Security & Protection: relies on server infrastructure
	- ( - ) Accessibility & Reliability: online access with stable connection only
	- ( + ) Search & analisys opportunities: relies on server infrastructure
	- ( +/- ) Support costs: external dependencies usually required
- File System
	- ( + ) Data size: pretty good on modern devices, but still limited
	- ( + ) Access speed: quite good
	- ( + ) Security & Protection: secure sections available, Android protection is good
	- ( + ) Accessibility & Reliability: Internal Storage is always available
	- ( - ) Search & analysis capabilities: file data aggregation is too slow to analyse
	- ( + ) Support costs: casual files manipulation
- Database
	- ( + ) Storage size: DB can store data of any size, including 
	- ( + ) Access speed: can even be increased with indexes
	- ( + ) Security & Protection: DB is usually available only for app
	- ( + ) Accessibility & Reliability: always available
	- ( + ) Search & analysis capabilities: the best solution for big sized data
	- ( - ) Support costs: DB structure has to be implemented and kept updated
- RAM
	- ( -- ) Storage size: too low to store anything big
	- ( ++ ) Access speed: only CPU registers are faster :-)
	- ( ++ ) Security & Protection: root devices only can access other apps' RAM
	- ( ++ ) Accessibility & Reliability: always accessible
	- ( ++ ) Search & analysis capabilities: any programmer implemented methods
	- ( + ) Support costs: app architecture only


## SharedPreferences

- https://developer.android.com/training/data-storage/shared-preferences
- https://developer.android.com/reference/android/content/SharedPreferences

- Key-value storage
- Primitives only (`Boolean`, `Int`, `Float`, `Long`, `String`, `Set<String>`)
- XML file in InternalStorage

### Init

```kotlin
val sharedPref = context.getSharedPreferences("filename", Context.MODE_PRIVATE)
```

### Write

```kotlin
with (sharedPref.edit()) {
	putInt("key Int", 1)
	putString("key String", "value")
	// both change the in-memory `SharedPreferences` object immediately
	apply() // write the data to disk asynchronously
	commit() // write the data to disk synchronously, do not call on UI thread
}
// or using Kotlin Extension:
sharedPref.edit(commit = false) { // false: using commit(), true: using apply()
	putInt("key Int", 1)
	putString("key String", "value")
}
```

### Read

```kotlin
val valueInt = sharedPref.getInt("key String", defaultValue)
```

## Jetpack DataStore

- https://developer.android.com/topic/libraries/architecture/datastore
- https://developer.android.com/jetpack/androidx/releases/datastore
- https://developer.android.com/codelabs/android-preferences-datastore#0

Jetpack DataStore is a data storage solution that allows you to store key-value pairs or typed objects with [protocol buffers](https://developers.google.com/protocol-buffers). DataStore uses Kotlin coroutines and Flow to store data asynchronously, consistently, and transactionally.

- **Preferences DataStore** stores and accesses data using keys. This implementation does not require a predefined schema, and it does not provide type safety.
- **Proto DataStore** stores data as instances of a custom data type. This implementation requires you to define a schema using [protocol buffers](https://developers.google.com/protocol-buffers), but it provides type safety.

### Preferences DataStore

#### Setup

`build.gradle.kts`
```kotlin
    dependencies {
        implementation("androidx.datastore:datastore-preferences:1.1.1")
    }
```

#### Init

```kotlin
// At the top level of your kotlin file:  
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```

#### Read

```kotlin
val EXAMPLE_KEY = intPreferencesKey("example_counter")  
val exampleCounterFlow: Flow<Int> = context.dataStore.data.map { prefs -> 
	// No type safety.
	prefs[EXAMPLE_KEY] ?: 0  
}
val exampleCounterVal = runBlocking { exampleCounterFlow.first() }
```

#### Write

```kotlin
suspend fun incrementCounter() { 
	context.dataStore.edit { prefs ->
		val currentCounterValue = prefs[EXAMPLE_KEY] ?: 0
		prefs[EXAMPLE_KEY] = currentCounterValue + 1
	}
}
// or
suspend fun incrementCounter() { 
	context.dataStore.updateData { prefs ->
		val snapshot = prefs.toMutablePreferences()
		snapshot[EXAMPLE_KEY] = (snapshot[EXAMPLE_KEY] ?: 0) + 1
		snapshot
	}
}
```

### Proto DataStore

#### Setup

`build.gradle.kts`
```kotlin
    dependencies {
        implementation("androidx.datastore:datastore:1.1.1")
    }
```

also adding <u>protobuf</u> lite for generating Java classes from proto:
- https://github.com/google/protobuf-gradle-plugin
- https://stackoverflow.com/questions/72210395/how-to-setup-protobuf-in-kotlin-android-studio

`app/build.gradle.kts`
```kotlin
plugins {
    ...
    id("com.google.protobuf") version "0.9.4"
}
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.19.4"
    }
    generateProtoTasks {
        all().configureEach {
            builtins {
                id("java") {
                    option("lite")
                }
            }
        }
    }
}
dependencies {
    ...
    implementation("com.google.protobuf:protobuf-javalite:3.18.0")
}
```


#### Define a schema

Proto DataStore requires a predefined schema in a proto file in the `app/src/main/proto/` directory. This schema defines the type for the objects that you persist in your Proto DataStore. To learn more about defining a proto schema, see the [protobuf language guide](https://developers.google.com/protocol-buffers/docs/proto3).

`app/src/main/proto/Settings`
```proto
syntax = "proto3";

option java_package = "com.example.application";
option java_multiple_files = true;

message Settings {
  int32 example_counter = 1;
}
```

#### Create a Proto DataStore

Define a class that implements `Serializer<T>`, where `T` is the type defined in the proto file. This serializer class tells DataStore how to read and write your data type.

```kotlin
object SettingsSerializer : Serializer<Settings> {
	override val defaultValue: Settings = Settings.getDefaultInstance()
	override suspend fun readFrom(input: InputStream): Settings {
	    try {
		    return Settings.parseFrom(input)
	    } catch (exception: InvalidProtocolBufferException) {
		    throw CorruptionException("Cannot read proto.", exception)
	    }
	}
	override suspend fun writeTo (t: Settings, output: OutputStream) = t.writeTo(output)
}
```

#### Init

```kotlin
val Context.settingsDataStore: DataStore<Settings> by dataStore(
	fileName = "settings.pb",
	serializer = SettingsSerializer
)
```

#### Read

```kotlin
val exampleCounterFlow: Flow<Int> = context.settingsDataStore.data
	.map { settings ->
	    // The exampleCounter property is generated from the proto schema.
	    settings.exampleCounter
	}
```

#### Write

```kotlin
suspend fun incrementCounter() {
	context.settingsDataStore.updateData { currentSettings ->
		currentSettings.toBuilder()
			.setExampleCounter(currentSettings.exampleCounter + 1)
		    .build()
    }
}
```




