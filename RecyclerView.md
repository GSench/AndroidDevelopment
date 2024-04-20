# RecyclerView

[RecyclerView для начинающего Android-разработчика / Хабр](https://habr.com/ru/articles/705064/)

[Create dynamic lists with RecyclerView &nbsp;|&nbsp; Android Developers](https://developer.android.com/develop/ui/views/layout/recyclerview)

[Kotlin: RecyclerView](https://developer.alexanderklimov.ru/android/views/recyclerview-kot.php)

По сути самое полезное, да и вообще все что можно рассказать про RecyclerView:

[Ускоряем работу RecyclerView. Лучшие практики оптимизации - YouTube](https://www.youtube.com/watch?v=o8rzzQPOo2U)

[GitHub - elvisfromsouth/RecyclerViewTipsAndTricks](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks)

## Моя базовая имплементация:

```xml
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/movie_list_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/main_toolbar"
        />
```

```kotlin
class MovieListAdapter(val context: Context) : RecyclerView.Adapter<MovieListAdapter.MovieListItemViewHolder>() {

    var movieList: List<MovieListItem> = emptyList()
        set(newList) {
            field = newList
            notifyDataSetChanged()
        }
    class MovieListItemViewHolder (val binding: MovieListItemBinding): RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MovieListItemViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val binding = MovieListItemBinding.inflate(inflater, parent, false)
        return MovieListItemViewHolder(binding)
    }

    override fun getItemCount() = movieList.size

    override fun onBindViewHolder(holder: MovieListItemViewHolder, position: Int) {
        with(holder.binding) {
            movieTitle.text = movieList[position].title
            movieSubtitle.text = context.getString(
                R.string.movie_item_subtitle_format,
                movieList[position].genre,
                movieList[position].year
            )
            Glide
                .with(context)
                .load(movieList[position].iconUrl)
                .into(movieIcon)
        }
    }
}
```

```kotlin
data class MovieListItem (
    val title: String,
    val genre: String,
    val year: String,
    val iconUrl: String,
)
```

```kotlin
        val movieListAdapter = MovieListAdapter(this)
        with(binding.movieListView) {
            layoutManager = LinearLayoutManager(this@MainActivity)
            adapter = movieListAdapter
        }
        movieListAdapter.movieList = listOf(
            MovieListItem("The Avengers", "Fantasy", "2012", "http://kinopoiskapiunofficial.tech/images/posters/kp/263531.jpg")
        )
```

Все-таки содержимое onBindViewHolder лучше перенести в класс самого ViewHolder

```kotlin
class MovieListItemViewHolder (val binding: MovieListItemBinding): RecyclerView.ViewHolder(binding.root) {
        fun bind(movie: MovieListItem, onMovieClick: (movieID: Int) -> Unit) = with(binding) {
            movieTitle.text = movie.title
            movieSubtitle.text = movieSubtitle.context.getString(
                R.string.movie_item_subtitle_format,
                movie.genre,
                movie.year
            )
            Glide
                .with(movieIcon.context)
                .load(movie.iconUrl)
                .placeholder(getShimmerDrawable())
                .diskCacheStrategy(DiskCacheStrategy.ALL)
                .into(movieIcon)
            movieCard.setOnClickListener { onMovieClick(movie.id) }
        }
    }
```

```kotlin
    override fun onBindViewHolder(holder: MovieListItemViewHolder, position: Int) =
        holder.bind(currentList[position], onMovieClick)
```

awdawd

## По видео:

### 1. Абстрактный Item

Для создания разных видов элементов внутри RV можно создать абстрактный [Item ](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/Item.kt) и абстрактный элемент списка [Fingerprint](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/ItemFingerprint.kt) с абстрактным [ViewHolder](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/BaseViewHolder.kt) и использовать его в [адаптере](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/FingerprintAdapter.kt), а передавать разные реализации [1](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/fingerprints/PostFingerprint.kt) [2](https://github.com/elvisfromsouth/RecyclerViewTipsAndTricks/blob/01-fingerprint/app/src/main/java/com/broadcast/myapplication/adapter/fingerprints/TitleFingerprint.kt) (Инкапсуляция, Полиморфизм). Так не придется везде проверять тип элемента списка.

*Это один из способов сделать Header и Footer*: [android - How to add a footer view in recyclerview that is get data from DB and in a reverse layout - Stack Overflow](https://stackoverflow.com/a/52224183)

### 2. Оптимизация bind

Данные для ViewHolder нужно готовить заранее, подавать их уже в готовом виде.

### 3. Decorations

*Тоже вариант сделать header и footer, но запарный, decoration вообще не для этого* [android - How to add a footer view in recyclerview that is get data from DB and in a reverse layout - Stack Overflow](https://stackoverflow.com/a/52223668)

[RecyclerView.ItemDecoration: используем по максимуму / Хабр](https://habr.com/ru/companies/surfstudio/articles/513038/)

### 4. DiffUtil

notifyDataSetChanged() - это зло. т.к. = полная перезагрузка данных (в т.ч. видимой части, а это лаги)

Вместо этого нужно использовать отдельные notifyItemInserted/Changed/RangeInserted... (они могут даже не затронуть текущий отображаемый список), но это сложная задача, т.к. нужно сравнивать 2 списка, отлавливать изменения, готовить оптимальную стратегию наката. Для решения этой проблемы и существует DiffUtil, где данный процесс уже написан.

DiffUtil можно использовать с любым адаптером.

*ИМХО если элементы в список должны просто падать кучей в конец, то можно в целом не запариваться с DiffUtil, и использовать просто notifyItemRangeInserted - это проще и работает быстрее.*

### 5. ListAdapter

А если RV - обычный список с обычным адаптером, то можно использовать ListAdapter, где DiffUtil уже встроен. Пример простейшей реализации:

```kotlin
class MovieListAdapter(
    private val onMovieClick: (movieID: Int) -> Unit
) : ListAdapter<MovieListItem, MovieListAdapter.MovieListItemViewHolder>(MovieListDiffUtil()) {

    class MovieListItemViewHolder (val binding: MovieListItemBinding): RecyclerView.ViewHolder(binding.root) {
        fun bind(movie: MovieListItem, onMovieClick: (movieID: Int) -> Unit) = with(binding) {
            movieTitle.text = movie.title
            movieSubtitle.text = movieSubtitle.context.getString(
                R.string.movie_item_subtitle_format,
                movie.genre,
                movie.year
            )
            Glide
                .with(movieIcon.context)
                .load(movie.iconUrl)
                .placeholder(getShimmerDrawable())
                .diskCacheStrategy(DiskCacheStrategy.ALL)
                .into(movieIcon)
            movieCard.setOnClickListener { onMovieClick(movie.id) }
        }
    }
    class MovieListDiffUtil : DiffUtil.ItemCallback<MovieListItem>() {
        override fun areItemsTheSame(oldItem: MovieListItem, newItem: MovieListItem) = oldItem.id == newItem.id
        override fun areContentsTheSame(oldItem: MovieListItem, newItem: MovieListItem) = oldItem.id == newItem.id
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MovieListItemViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val binding = MovieListItemBinding.inflate(inflater, parent, false)
        return MovieListItemViewHolder(binding)
    }

    override fun onBindViewHolder(holder: MovieListItemViewHolder, position: Int) =
        holder.bind(currentList[position], onMovieClick)

    override fun submitList(list: List<MovieListItem>?) {
        super.submitList(list?.let { ArrayList(it) })
    }

}
```

Замена списка:

```kotlin
movieListAdapter.submitList(it)
```

**ВАЖНАЯ ОСОБЕННОСТЬ**:

Выполняем `submitList`, а ничего не происходит:

[android - ListAdapter not updating item in RecyclerView - Stack Overflow](https://stackoverflow.com/questions/49726385/listadapter-not-updating-item-in-recyclerview)

> Because apparently I am calling the `submitList(...)` function for a reason. I am pretty sure people are trying to figure out what went wrong for hours until they figure out the submitList() ignores silently the call.
> 
> This is because of `Google`s weird logic. So if you pass the same list to the adapter it does not even call the `DiffUtil`.
> 
> ```kotlin
> public void submitList(final List<T> newList) {
>     if (newList == mList) {
>         // nothing to do
>         return;
>     }
> ....
> }
> ```
> 
> I really don't understand the whole point of this `ListAdapter` if it can't handle changes on the same list. If you want to change the items on the list you pass to the `ListAdapter` and see the changes then either you need to create a deep copy of the list or you need to use regular `RecyclerView` with your own `DiffUtill` class.

Решение:

> What you can do in case you're not using any such libraries is:
> 
> ```kotlin
> submitList(null);
> submitList(myList);
> ```
> 
> Another solution would be to override submitList (which doesn't cause that quick blink) as such:
> 
> ```kotlin
> override fun submitList(list: List<CatItem>?) {
>     super.submitList(list?.let { ArrayList(it) })
> }
> ```
> 
> Questionable logic but works perfectly. My preferred method is the second one because it doesn't cause each row to get an onBind call.

*В моем примере выше это уже реализовано!*

## ScrollListener

[java - RecyclerView scrolled UP/DOWN listener - Stack Overflow](https://stackoverflow.com/questions/29024058/recyclerview-scrolled-up-down-listener)

```kotlin
            binding.movieListView.addOnScrollListener(object: OnScrollListener(){
                override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                    super.onScrolled(recyclerView, dx, dy)
                    val lastVisibleItem = (binding.movieListView.layoutManager as LinearLayoutManager).findLastVisibleItemPosition()
                    if(lastVisibleItem > movieListAdapter.itemCount - 6) {
                        // Прослистали вниз, осталось 5 элементов
                    }
                }
            })
```

## ConcatAdapter

Последовательно соединяет несколько адаптеров в одном RV

[ConcatAdapter &nbsp;|&nbsp; Android Developers](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter)

Эффективный способ сделать Header, Footer и вообще добавлять разные кастомные элементы в RV, если они имеют постоянное положение в общем списке.

[android - RecyclerView header and footer - Stack Overflow](https://stackoverflow.com/a/61008527)

```kotlin
implementation("androidx.recyclerview:recyclerview:1.3.2")
```

```kotlin
        movieListAdapter = MovieListAdapter(::onMovieClick)
        val movieLayoutManager = LinearLayoutManager(this)

        movieListFooterAdapter = MovieListFooterAdapter()
        val concatAdapter = ConcatAdapter(movieListAdapter, movieListFooterAdapter)

        with(binding.movieListView) {
            layoutManager = movieLayoutManager
            adapter = concatAdapter
        }
```

Другой похожий адаптер (хз чем отличается):

[Делаем код в адаптере чище с помощью MergeAdapter / Хабр](https://habr.com/ru/articles/523840/)

## Paging Library

[Paging library overview &nbsp;|&nbsp; Android Developers](https://developer.android.com/topic/libraries/architecture/paging/v3-overview)

[Android Paging Advanced codelab  |  Android Developers](https://developer.android.com/codelabs/android-paging#0)

***см. AppDevTools.md***
