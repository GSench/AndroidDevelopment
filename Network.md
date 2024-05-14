# Network

## Retrofit

Работа с сетью, с API.

Суть этой библиотеки: мы создаем интерфейс, методы которого - это и есть наши запросы в сеть. Их return - сразу DTO, который мы хотим получить из JSON.

Параметры запросов передаются с помощью аннотаций.

Реализацию данного интерфейса за нас сделает Retrofit.

Хороший гайд: [Курс по Retrofit в Android Studio - YouTube](https://www.youtube.com/playlist?list=PLmjT2NFTgg1cHUclGx5L9c92FG4pCN6lC)

[GitHub - square/retrofit: A type-safe HTTP client for Android and the JVM](https://github.com/square/retrofit)

[Retrofit](https://square.github.io/retrofit/)

Вроде проблем с имплементацией не было

```kotlin
    // Базовая библиотека
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    // Если хотим получать JSON и автоматически преобразовывать в DTO
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    // HTTP клиент вообще говоря не обязателен,
    // это если нужно как-то по особому перехватывать и обрабатывать запросы
    implementation("com.squareup.okhttp3:okhttp:4.7.2")
    // Например, чтобы логировать запросы, помоимо okhttp, нужен еще:
    implementation("com.squareup.okhttp3:logging-interceptor:4.7.2")
```

```kotlin
object KinopoiskApiValues {
    const val KINOPOISK_API_URL = "https://kinopoiskapiunofficial.tech"
    const val INITIAL_PAGE = 1
}
interface KinopoiskApi {
    @GET("api/v2.2/films/top?type=TOP_100_POPULAR_FILMS")
    suspend fun getTop100Movies(
        @Header("x-api-key") token: String,
        @Query("page") page: Int = 1
    ): KinopoiskMoviesList

    @GET("api/v2.2/films/{id}")
    suspend fun getMovieDetails(
        @Header("x-api-key") token: String,
        @Path("id") id: Int
    ): KinopoiskMovieDetails

    companion object {
        fun instantiateKinopoiskApi(httpClient: OkHttpClient) = Retrofit
            .Builder()
            .client(httpClient) // это не обязательно
            .baseUrl(KinopoiskApiValues.KINOPOISK_API_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

Нельзя в качестве URL параметров использовать `@Path`, именно `@Query`: [android - IllegalArgumentException in Retrofit / must not have replace block - Stack Overflow](https://stackoverflow.com/questions/35964147/illegalargumentexception-in-retrofit-must-not-have-replace-block)

Дальше полученный объект можно уже использовать, запускать нужные методы интерфейса, реализацию которых за нас выполнил Retrofit!

### JSON

Как тестировать запросы к API и получать JSON заранее:

[HTTP Request Online | Tools | Base64](https://base64.guru/tools/http-request-online)

Преобразовать json к красивому виду:

https://jsonformatter.org/

Создать `data class` на основе JSON:

[JSON To Kotlin Class (JsonToKotlinClass) - IntelliJ IDEs Plugin | Marketplace](https://plugins.jetbrains.com/plugin/9960-json-to-kotlin-class-jsontokotlinclass-)

Лучше этот плагин ставить через встроенный Plugin Manager в IDEA, а не скачиать отдельно.

**При создании `data class` на основе JSON все поля должны быть nullable!!!**

```kotlin
data class KinopoiskMovie(
    val countries: List<Country?>?,
    val filmId: Int?,
    val filmLength: String?,
    val genres: List<Genre?>?,
    val isAfisha: Int?,
    val isRatingUp: String?,
    val nameEn: String?,
    val nameRu: String?,
    val posterUrl: String?,
    val posterUrlPreview: String?,
    val rating: String?,
    val ratingChange: String?,
    val ratingVoteCount: Int?,
    val year: String?
)
```

Иногда в сгенерированном классе не распознаются типы значений (проставляется `Any?` в случае `null` в JSON), за этим нужно следить и править вручную (проще всего на `String?`)!

## OkHttpClient

[Android Retrofit2, OkHttpClient, HttpLoggingInterceptor в Android Studio (Kotlin) - YouTube](https://www.youtube.com/watch?v=UCAX0hPKEPE)

Как создать http клиент с логами:

```kotlin
val httpClient = OkHttpClient
        .Builder()
        .addInterceptor(HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BASIC
        })
        .build()
```

Используется 1 HttpClient на все приложение:

[Best way to use HttpClient in Android &#8211; Craig Andrews](https://candrews.integralblue.com/2011/09/best-way-to-use-httpclient-in-android/):

> - Always use one HttpClient instance for your entire application. HttpClient is not free to instantiate – each additional instance takes time to create and uses more memory. However, more importantly, using one instance allows HttpClient to pool and reuse connections along with other optimizations that can make big differences in how your application performs.

```kotlin
class MyApplication: Application() {

    lateinit var network: Network
    lateinit var repository: MovieRepository
    lateinit var kinopoiskApi: KinopoiskApi
    override fun onCreate() {
        super.onCreate()
        network = Network(this) // Здесь появляется HttpClient
        kinopoiskApi = KinopoiskApi.instantiateKinopoiskApi(network.httpClient)
        repository = MovieRepository(kinopoiskApi)
    }

}
```

Manifest:

```xml
    <application
        ...
        android:name=".MyApplication">
```



### Caching

[Caching with OkHttp Interceptor and Retrofit](https://amitshekhar.me/blog/caching-with-okhttp-interceptor-and-retrofit)

> **OkHttp is designed in such a way that it returns the cached response only when the Internet is available.**

Вот есть какой-то ответ на это: [java - Can Retrofit with OKHttp use cache data when offline - Stack Overflow](https://stackoverflow.com/questions/23429046/can-retrofit-with-okhttp-use-cache-data-when-offline)

Моя релизация:

```kotlin
    private companion object HttpClientConstants {
        const val HTTP_CACHE_PATH = "http-cache"
        const val HTTP_CACHE_SIZE = 10L * 1024L * 1024L
    }

    private val cacheInterceptor = object: Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response {
            val response: Response = chain.proceed(chain.request())
            val cacheControl = CacheControl.Builder()
                .maxAge(1, TimeUnit.HOURS)
                .build()
            return response.newBuilder()
                .header("Cache-Control", cacheControl.toString())
                .build()
        }
    }

    private val forceCacheInterceptor = object: Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response {
            val builder: Request.Builder = chain.request().newBuilder()
            if (networkState != NetworkState.AVAILABLE) {
                builder.cacheControl(CacheControl.FORCE_CACHE)
            }
            return chain.proceed(builder.build())
        }
    }

    val httpClient = OkHttpClient
        .Builder()
        .cache(
            Cache(
                File(context.cacheDir, HTTP_CACHE_PATH),
                HTTP_CACHE_SIZE
            )
        )
        .addNetworkInterceptor(cacheInterceptor)
        .addInterceptor(forceCacheInterceptor)
        .addInterceptor(HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BASIC
        })
        .build()
```

## Network State

Правильный способ узнавать состояние подключения к сети по гуглу: [Monitor connectivity status and connection metering &nbsp;|&nbsp; Connectivity &nbsp;|&nbsp; Android Developers](https://developer.android.com/training/monitoring-device-state/connectivity-status-type)

Заворачиваем в Flow:

```kotlin
enum class NetworkState {
    UNDEFINED,
    AVAILABLE,
    UNAVAILABLE,
}

private val networkRequest = NetworkRequest
    .Builder()
    .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
    .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
    .addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
    .build()

fun networkStateFlow(context: Context) : Flow<NetworkState> {
    val connectivityManager = context.getSystemService(ConnectivityManager::class.java) as ConnectivityManager
    return callbackFlow {
        trySendBlocking(NetworkState.UNDEFINED)
        connectivityManager.requestNetwork(networkRequest, object : ConnectivityManager.NetworkCallback() {
            override fun onAvailable(network: Network) {
                super.onAvailable(network)
                trySendBlocking(NetworkState.AVAILABLE)
            }
            override fun onUnavailable() {
                super.onUnavailable()
                trySendBlocking(NetworkState.UNAVAILABLE)
            }
            override fun onLost(network: Network) {
                super.onLost(network)
                trySendBlocking(NetworkState.UNAVAILABLE)
            }
        })
        awaitClose()
    }
}
```

Подписываемся:

```kotlin
    var networkState = NetworkState.UNDEFINED
        private set

    init {
        networkStateFlow(context)
            .onEach { networkState = it }
            .launchIn(CoroutineScope(Dispatchers.IO))
    }
```

Люди еще предлагают просто проверять подключение запросом в сеть: [java - How to check internet access on Android? InetAddress never times out - Stack Overflow](https://stackoverflow.com/questions/1560788/how-to-check-internet-access-on-android-inetaddress-never-times-out)

[networking - Android check internet connection - Stack Overflow](https://stackoverflow.com/questions/9570237/android-check-internet-connection)

лучше так не делать
