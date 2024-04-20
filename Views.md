# Views

## Toolbar

[Set up the app bar &nbsp;|&nbsp; Views &nbsp;|&nbsp; Android Developers](https://developer.android.com/develop/ui/views/components/appbar/setting-up)

[Toolbar &nbsp;|&nbsp; Android Developers](https://developer.android.com/reference/android/widget/Toolbar)

[android - Change Custom Toolbar Text - Stack Overflow](https://stackoverflow.com/questions/43643720/change-custom-toolbar-text)

```xml
    <androidx.appcompat.widget.Toolbar
        android:id="@+id/main_toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:elevation="@dimen/toolbar_elevation"
        android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        app:titleTextAppearance="@style/ActionBar.titleText"
        app:titleMarginStart="@dimen/title_margin_start"
        />
    <style name="ActionBar.titleText" parent="TextAppearance.AppCompat.Widget.ActionBar.Title">
<!--        <item name="android:textColor">@color/black</item>-->
        <item name="android:textSize">28sp</item>
<!--        <item name="android:textStyle">bold</item>-->
    </style>
```

```kotlin
// onCreate:
        binding.mainToolbar.title = getString(R.string.popular_title)
        setSupportActionBar(binding.mainToolbar)
```

Как спрятать toolbar при прокрутке и чтобы он появлялся при прокрутке вверх их любого места полосы прокрутки:

[How to hide/show toolbar while scrolling - Mobikul](https://mobikul.com/hideshow-toolbar-scrolling/)

```xml
    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light">

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/main_toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:titleTextAppearance="@style/ActionBar.titleText"
                app:titleMarginStart="@dimen/title_margin_start"
                app:layout_scrollFlags="scroll|enterAlways"/>

        </com.google.android.material.appbar.AppBarLayout>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/movie_list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior"/>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>
```

## CardView

[Android: CardView (Карточка)](https://developer.alexanderklimov.ru/android/views/cardview.php)

```xml
<androidx.cardview.widget.CardView
        android:id="@+id/movie_card"
        android:foreground="?attr/selectableItemBackground"
        android:clickable="true"
        android:focusable="true"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_margin="16dp"
        app:cardCornerRadius="16dp"
        app:cardElevation="10dp"
        >
```

## Shimmer progress

[Shimmer Effect on Android - DEV Community](https://dev.to/janirefdez/shimmer-effect-on-android-57ke)

[GitHub - facebookarchive/shimmer-android: An easy, flexible way to add a shimmering effect to any view in an Android app.](https://github.com/facebookarchive/shimmer-android/tree/main)

Все `CardView` внутри `ShimmerFrameLayout` будут с анимацией Shimmer:

```xml
<com.facebook.shimmer.ShimmerFrameLayout
            android:id="@+id/movie_details_placeholder"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            tools:duration="800">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">

                <androidx.cardview.widget.CardView
                    android:layout_width="300dp"
                    android:layout_height="32dp"
                    android:layout_marginStart="32dp"
                    android:layout_marginTop="16dp"
                    app:cardCornerRadius="8dp"
                    android:backgroundTint="@android:color/darker_gray" />

                <androidx.cardview.widget.CardView
                    android:layout_width="300dp"
                    android:layout_height="32dp"
                    android:layout_marginStart="32dp"
                    android:layout_marginTop="16dp"
                    app:cardCornerRadius="8dp"
                    android:backgroundTint="@android:color/darker_gray"/>
                <androidx.cardview.widget.CardView
                    android:layout_width="300dp"
                    android:layout_height="32dp"
                    android:layout_marginStart="32dp"
                    android:layout_marginTop="8dp"
                    app:cardCornerRadius="8dp"
                    android:backgroundTint="@android:color/darker_gray" />
                <androidx.cardview.widget.CardView
                    android:layout_width="300dp"
                    android:layout_height="32dp"
                    android:layout_marginStart="32dp"
                    android:layout_marginTop="8dp"
                    app:cardCornerRadius="8dp"
                    android:backgroundTint="@android:color/darker_gray" />

            </LinearLayout>

        </com.facebook.shimmer.ShimmerFrameLayout>
```

Хороший пример использования Shimmer: [GitHub - janirefdez/ShimmerEffectAndroid: Basic Android example where you can find how Shimmer Effect is implemented on Android.](https://github.com/janirefdez/ShimmerEffectAndroid/tree/master)

**Кстати это также хороший пример Retrofit, Repository, MVVM, ViewModel, CleanArchitecture, RecyclerView. Очень базовый, очень упрощенный, и поэтому качественный!**

### Shimmer Drawable

Например, в качестве placeholder'а для Glide: [android - How to use a view (shimmer) as a placeholder for an imageView (Glide) - Stack Overflow](https://stackoverflow.com/questions/61076174/how-to-use-a-view-shimmer-as-a-placeholder-for-an-imageview-glide)

```kotlin
import com.facebook.shimmer.Shimmer
import com.facebook.shimmer.ShimmerDrawable

fun getShimmerDrawable() = ShimmerDrawable().apply {
    setShimmer(
        Shimmer
            .AlphaHighlightBuilder()// The attributes for a ShimmerDrawable is set by this builder
            .setDuration(1800) // how long the shimmering animation takes to do one full sweep
            .setBaseAlpha(0.7f) //the alpha of the underlying children
            .setHighlightAlpha(0.6f) // the shimmer alpha amount
            .setDirection(Shimmer.Direction.LEFT_TO_RIGHT)
            .setAutoStart(true)
            .build()
    )
}
```

```kotlin
            Glide
                .with(movieIcon.context)
                .load(movie.iconUrl)
                .placeholder(getShimmerDrawable())
                .diskCacheStrategy(DiskCacheStrategy.ALL)
                .into(movieIcon)
```

## View Binding

[Kotlin Android Extensions deprecated. Что делать? Инструкция по миграции / Хабр](https://habr.com/ru/articles/526192/)

[View binding  |  Android Developers](https://developer.android.com/topic/libraries/view-binding)

[Долгожданный View Binding в Android / Хабр](https://habr.com/ru/articles/467295/)

```kotlin
// build.gradle.kts

    buildFeatures {
        viewBinding = true
    }
    // MainActivity:
    private lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.mainToolbar.title = getString(R.string.popular_title)
        setSupportActionBar(binding.mainToolbar)
    }
```

## Button

button color: [Android button background color - Stack Overflow](https://stackoverflow.com/questions/18070008/android-button-background-color)

```xml
        <Button
            ...
            android:backgroundTint="@color/accent"/>
```


