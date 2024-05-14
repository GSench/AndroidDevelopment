# Coroutine

Лекция по корутинам от Yandex: [Корутины - YouTube](https://www.youtube.com/watch?v=ITLe4FIrrTg)

гайд от JetBrains: [Coroutines guide | Kotlin Documentation](https://kotlinlang.org/docs/coroutines-guide.html)

Реализация ожидания завершения корутины: [android - How to wait for end of a coroutine - Stack Overflow](https://stackoverflow.com/questions/59491707/how-to-wait-for-end-of-a-coroutine)

Пример реализации запуска Job из фрагмента (использовал такое, все ок): [kotlin - Launch coroutine from click event in fragment - Stack Overflow](https://stackoverflow.com/questions/59608923/launch-coroutine-from-click-event-in-fragment)



### Callback to coroutine

Если нужно выполнить единичный запрос с помощью какого-нибудь API и получить результат, Callback можно превратить в suspend функцию: [Callback to Coroutines in Kotlin](https://amitshekhar.me/blog/callback-to-coroutines-in-kotlin)

Или вот еще хороший гайд: [Котлин корутины. Часть 4. Переход callback API на корутины - Fandroid.info](https://www.fandroid.info/converting-existing-callback-apis-with-coroutines/)

В итоге не пригодилось, т.к. чаще все-таки результат не один, а нужно именно реагировать на callback'и, а для этого хорошо подходит Flow.

# Flow

## Callbacks

### Callback to flow

Очень круто, что любой Callback можно превратить в flow:

Базовый гайд от Kotlin: [callbackFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/callback-flow.html)

И пример реализации: поток событий UI

```kotlin
        uiEventFlow = callbackFlow {
            binding.movieListView.addOnScrollListener(object: OnScrollListener(){
                override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                    super.onScrolled(recyclerView, dx, dy)
                    val lastVisibleItem = (binding.movieListView.layoutManager as LinearLayoutManager).findLastVisibleItemPosition()
                    if(lastVisibleItem > movieListAdapter.itemCount - 6) {
                        trySendBlocking(UIEvent.SCROLLED_TO_BOTTOM)
                    }
                }
            })
            binding.retryButton.setOnClickListener { trySendBlocking(UIEvent.RETRY) }
            trySendBlocking(UIEvent.POPULAR_LIST_OPENED)
            awaitClose()
        }


    fun subscribeUI(uiEventFlow: Flow<UIEvent>) = uiEventFlow
        .onEach {
            when(it) {
                UIEvent.SCROLLED_TO_BOTTOM -> {/* react on scroll*/}
                UIEvent.POPULAR_LIST_OPENED -> { /* react on button*/ }
                UIEvent.RETRY -> { /* react on retry button*/ }
            }
        }
        .launchIn(CoroutineScope(Dispatchers.IO))
```

Офф.гайд от гугла: [Kotlin flows on Android &nbsp;|&nbsp; Android Developers](https://developer.android.com/kotlin/flow)

В итоге везде использовал `callbackFlow`. А есть еще `flow`, `channelFlow`:

[Android Kotlin Coroutines: what is the difference between flow, callbackFlow, channelFlow,... other flow constructors - Stack Overflow](https://stackoverflow.com/questions/61865744/android-kotlin-coroutines-what-is-the-difference-between-flow-callbackflow-ch)

Вот хорошая статья - объяснение: https://medium.com/mobile-app-development-publication/kotlins-flow-channelflow-and-callbackflow-made-easy-5e82ce2e27c0



# 
