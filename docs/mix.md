# Laravel Mix

- [Wprowadzenie](#introduction)

<a name="introduction"></a>
## Wprowadzenie

> [!WARNING]
> Laravel Mix jest starszym pakietem, który nie jest już aktywnie rozwijany. [Vite](/docs/{{version}}/vite) może być używany jako nowoczesna alternatywa.

[Laravel Mix](https://github.com/laravel-mix/laravel-mix), pakiet stworzony przez Jeffreya Way'a, twórcę [Laracasts](https://laracasts.com), zapewnia płynne API do definiowania kroków budowania [webpack](https://webpack.js.org) dla Twojej aplikacji Laravel przy użyciu kilku popularnych preprocesorów CSS i JavaScript.

Innymi słowy, Mix sprawia, że kompilacja i minimalizacja plików CSS i JavaScript Twojej aplikacji staje się dziecinnie prosta. Dzięki prostemu łańcuchowaniu metod możesz płynnie zdefiniować swój pipeline zasobów. Na przykład:

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

Jeśli kiedykolwiek byłeś zdezorientowany i przytłoczony rozpoczynaniem pracy z webpack i kompilacją zasobów, pokochasz Laravel Mix. Jednak nie jesteś zobowiązany do korzystania z niego podczas rozwijania swojej aplikacji; możesz swobodnie używać dowolnego narzędzia do obsługi zasobów lub nawet żadnego.

> [!NOTE]
> Vite zastąpił Laravel Mix w nowych instalacjach Laravel. Aby zapoznać się z dokumentacją Mix, odwiedź oficjalną stronę [Laravel Mix](https://laravel-mix.com/). Jeśli chcesz przejść na Vite, zapoznaj się z naszym [przewodnikiem migracji do Vite](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite).
