# ViewModel

ViewModel - это реализация VM от Google Android, которая связана с жизненным циклом Activity/Fragment. **Объект класса `ViewModel `жив на протяжении всего жизненного цикла**. Необходимо отнаследоваться от него, предоставить `companion object` с `Factory` и инициализировать экземпляр особым образом с помощью делегата `viewModels()`. **Все `properties` экземпляра класса `ViewModel` будут жить вместе с самим объектом не зависимо от того `LiveData` они или нет!**

Гайд от Google Android очень хорош: [ViewModel overview &nbsp;|&nbsp; Android Developers](https://developer.android.com/topic/libraries/architecture/viewmodel)

Как создать ViewModel: [Create ViewModels with dependencies &nbsp;|&nbsp; Android Developers](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-factories)

```kotlin
class MainViewModel (
    private val repository: MovieRepository
) : ViewModel() {

    @Suppress("UNCHECKED_CAST")
    companion object {
        val Factory: ViewModelProvider.Factory = object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(
                modelClass: Class<T>,
                extras: CreationExtras
            ): T {
                val application = checkNotNull(extras[APPLICATION_KEY])
                return MainViewModel((application as MyApplication).repository) as T
            }
        }
    }
```

Как инициализировать инстанс `ViewModel` в `Activity`:

[android - how to get viewModel by viewModels? (fragment-ktx) - Stack Overflow](https://stackoverflow.com/questions/56748334/how-to-get-viewmodel-by-viewmodels-fragment-ktx)

Чтобы использовать делегаты:

```kotlin
    implementation ("androidx.activity:activity-ktx:1.8.2")
    implementation ("androidx.fragment:fragment-ktx:1.6.2")
```

В `Activity` или `Fragment`:

```kotlin
private val viewModel: MainViewModel by viewModels{MainViewModel.Factory}
```

Вот еще гайд: [Урок 4. ViewModel](https://startandroid.ru/ru/courses/dagger-2/27-course/architecture-components/527-urok-4-viewmodel.html)

И статья: [Android Architecture Components. Часть 4. ViewModel / Хабр](https://habr.com/ru/articles/334942/)

## LiveData

`LiveData` - это observable на который можно подписаться из UI. Холодный поток, который хранит в себе значение для View. `LiveData` будет жить в `ViewModel`:

```kotlin
class MainViewModel (
    private val repository: MovieRepository
) : ViewModel() {
    val loadingStateFlow = MutableLiveData(LoadingState.IDLE)
    val moviesList = MutableLiveData<MutableList<MovieListItem>>(mutableListOf())
```

Подписываемся в `Activity`/`Fragment`:

```kotlin
        viewModel.loadingStateFlow.observe(this) {
            if(it == null) return@observe
            ...
        }
        viewModel.moviesList.observe(this) {
            if(it == null) return@observe
            movieListAdapter.submitList(it)
        }
```

Гайд от Google Android: [LiveData overview &nbsp;|&nbsp; Android Developers](https://developer.android.com/topic/libraries/architecture/livedata)

Чтобы использовать `LiveData` нужна библиотека. Но если подключить только `androidx.activity:activity-ktx` и `androidx.fragment:fragment-kt`, то можно обойтись без нее:

```kotlin
    implementation ("androidx.lifecycle:lifecycle-extensions:2.2.0")
```

Еще гайд по LiveData: [Урок 2. LiveData](https://startandroid.ru/ru/courses/dagger-2/27-course/architecture-components/525-urok-2-livedata.html)

Особенности работы с LiveData [kotlin - MutableLiveData: Cannot invoke setValue on a background thread from Coroutine - Stack Overflow](https://stackoverflow.com/questions/53304347/mutablelivedata-cannot-invoke-setvalue-on-a-background-thread-from-coroutine): ее нужно обновлять из потока UI или использовать `postValue()`:

```kotlin
private suspend fun onListScrolledToBottom() {
    ...
    loadingStateFlow.postValue(LoadingState.LOADING)
    ...
    loadingStateFlow.postValue(LoadingState.IDLE)
```

Еще про особенности: [android - Calling postValue() multiple times on a Livedata will result only the last one is dispatched. Is there any alternative solution for that? - Stack Overflow](https://stackoverflow.com/questions/66724421/calling-postvalue-multiple-times-on-a-livedata-will-result-only-the-last-one-i)

И еще: [android - Updating a RecyclerView by a new LiveData&lt;List&gt; return from Room dynamically - Stack Overflow](https://stackoverflow.com/questions/58730425/updating-a-recyclerview-by-a-new-livedatalist-return-from-room-dynamically)



## Flows в ViewModel

Вместо LiveData можно использовать обычнве потоки Kotlin. Они гораздо более гибкие и удобные в использовании и предоставляют больше функционала: [Android, Kotlin Flow во ViewModel — все сложно / Хабр](https://habr.com/ru/articles/581914/)

Собственно на проекте в Сириусе так и делали: https://github.com/JumpyWizardEni/SiriusProject/blob/master/app/src/main/java/com/siriusproject/coshelek/wallet_list/ui/view/view_models/WalletListViewModel.kt
