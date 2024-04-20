# res

## layout

horizontal layout [How to Create Landscape Layout in Android Studio? - GeeksforGeeks](https://www.geeksforgeeks.org/how-to-create-landscape-layout-in-android-studio/)

Просто копируем layout.xml в папку res-land и перестраиваем его как хотим! В разметке должны быть те же id, чтобы с ними можно было работать из Kotlin!

## Styles / Themes

### Full screen

Сдалать приложение полноэкранным, чтобы статусбар и кнопки навигации стали полупрозрачными и под ними был виден контент приложения.

Перепробовал кучу всего, в т.ч. офф.гайд: [Hide the status bar  |  Android Developers](https://developer.android.com/training/system-ui/status)

Идеально сработало и не deprecated только:

```kotlin
        window.setFlags(
            WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
            WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS
        )
```

это вставляем в onCreate Activity после `super.onCreate(savedInstanceState)`, но до `setContentView(binding.root)`

Вот здесь вроде еще тоже что-то рабочее было, но не совсем: https://proandroiddev.com/android-full-screen-ui-with-transparent-status-bar-ef52f3adde63

### App background color

[Android find default background color - Stack Overflow](https://stackoverflow.com/questions/41000209/android-find-default-background-color)

```xml
<item name="android:colorBackground">@color/white</item>
```

И на любую View (например на root view фрагмента, которая прозрачная по умолчанию) его можно установить при необходимости вот так:

```xml
android:background="?android:colorBackground"
```
