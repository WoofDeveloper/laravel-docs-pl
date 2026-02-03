# Eloquent: Zasoby API

- [Wprowadzenie](#introduction)
- [Generowanie Zasobów](#generating-resources)
- [Przegląd Koncepcji](#concept-overview)
    - [Kolekcje Zasobów](#resource-collections)
- [Tworzenie Zasobów](#writing-resources)
    - [Opakowywanie Danych](#data-wrapping)
    - [Paginacja](#pagination)
    - [Warunkowe Atrybuty](#conditional-attributes)
    - [Warunkowe Relacje](#conditional-relationships)
    - [Dodawanie Meta Danych](#adding-meta-data)
- [Odpowiedzi Zasobów](#resource-responses)

<a name="introduction"></a>
## Wprowadzenie

Podczas tworzenia API możesz potrzebować warstwy transformacji, która znajduje się pomiędzy Twoimi modelami Eloquent a odpowiedziami JSON, które są faktycznie zwracane użytkownikom Twojej aplikacji. Na przykład, możesz chcieć wyświetlać określone atrybuty dla podzbioru użytkowników, a nie dla innych, lub możesz chcieć zawsze uwzględniać określone relacje w reprezentacji JSON Twoich modeli. Klasy zasobów Eloquent pozwalają na ekspresyjną i łatwą transformację Twoich modeli i kolekcji modeli do JSON.

Oczywiście zawsze możesz konwertować modele Eloquent lub kolekcje do JSON używając ich metod `toJson`; jednak zasoby Eloquent zapewniają bardziej szczegółową i solidną kontrolę nad serializacją JSON Twoich modeli i ich relacji.

<a name="generating-resources"></a>
## Generowanie Zasobów

Aby wygenerować klasę zasobu, możesz użyć polecenia Artisan `make:resource`. Domyślnie zasoby zostaną umieszczone w katalogu `app/Http/Resources` Twojej aplikacji. Zasoby dziedziczą po klasie `Illuminate\Http\Resources\Json\JsonResource`:

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### Kolekcje Zasobów

Oprócz generowania zasobów, które transformują pojedyncze modele, możesz generować zasoby odpowiedzialne za transformację kolekcji modeli. Pozwala to na uwzględnienie w odpowiedziach JSON linków i innych meta informacji, które są istotne dla całej kolekcji danego zasobu.

Aby utworzyć kolekcję zasobów, powinieneś użyć flagi `--collection` podczas tworzenia zasobu. Lub, umieszczenie słowa `Collection` w nazwie zasobu wskaże Laravel, że powinien utworzyć zasób kolekcji. Zasoby kolekcji dziedziczą po klasie `Illuminate\Http\Resources\Json\ResourceCollection`:

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## Przegląd Koncepcji

> [!NOTE]
> To jest ogólny przegląd zasobów i kolekcji zasobów. Glubąco zachęcamy do przeczytania pozostałych sekcji tej dokumentacji, aby uzyskać głębsze zrozumienie dostosowań i możliwości oferowanych przez zasoby.

Zanim zagłębimy się we wszystkie dostępne opcje podczas pisania zasobów, przyjrzyjmy się najpierw ogólnie, jak zasoby są używane w Laravel. Klasa zasobu reprezentuje pojedynczy model, który musi zostać przekształcony w strukturę JSON. Na przykład, oto prosta klasa zasobu `UserResource`:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Przekształć zasób w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Każda klasa zasobu definiuje metodę `toArray`, która zwraca tablicę atrybutów, które powinny zostać przekonwertowane na JSON, gdy zasób jest zwracany jako odpowiedź z trasy lub metody kontrolera.

Zauważ, że możemy uzyskać dostęp do właściwości modelu bezpośrednio ze zmiennej `$this`. Dzieje się tak, ponieważ klasa zasobu automatycznie przekazuje dostęp do właściwości i metod do bazowego modelu dla wygodnego dostępu. Po zdefiniowaniu zasobu, może on zostać zwrócony z trasy lub kontrolera. Zasób akceptuje instancję bazowego modelu przez swój konstruktor:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

Dla wygody możesz użyć metody `toResource` modelu, która będzie używać konwencji frameworka do automatycznego wykrycia bazowego zasobu modelu:

```php
return User::findOrFail($id)->toResource();
```

Podczas wywoływania metody `toResource` Laravel spróbuje zlokalizować zasób, który pasuje do nazwy modelu i jest opcjonalnie zakończony `Resource` w przestrzeni nazw `Http\Resources` najbliższej przestrzeni nazw modelu.

Jeśli Twoja klasa zasobu nie jest zgodna z tą konwencją nazewnictwa lub znajduje się w innej przestrzeni nazw, możesz określić domyślny zasób dla modelu używając atrybutu `UseResource`: 

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserResource;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResource;

#[UseResource(CustomUserResource::class)]
class User extends Model
{
    // ...
}
```

Alternatywnie, możesz określić klasę zasobu przekazując ją do metody `toResource`: 

```php
return User::findOrFail($id)->toResource(CustomUserResource::class);
```

<a name="resource-collections"></a>
### Kolekcje Zasobów

Jeśli zwracasz kolekcję zasobów lub odpowiedź z paginacją, powinieneś użyć metody `collection` udostępnionej przez Twoją klasę zasobu podczas tworzenia instancji zasobu w trasie lub kontrolerze:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

Lub, dla wygody, możesz użyć metody kolekcji Eloquent `toResourceCollection` która będzie używać konwencji frameworka do automatycznego wykrycia bazowej kolekcji zasobów modelu:

```php
return User::all()->toResourceCollection();
```

Podczas wywoływania metody `toResourceCollection`, Laravel spróbuje zlokalizować kolekcję zasobów, która pasuje do nazwy modelu i jest zakończona sufiksem `Collection` w przestrzeni nazw `Http\Resources` najbliższej przestrzeni nazw modelu.

Jeśli Twoja klasa kolekcji zasobów nie jest zgodna z tą konwencją nazewnictwa lub znajduje się w innej przestrzeni nazw, możesz określić domyślną kolekcję zasobów dla modelu używając atrybutu `UseResourceCollection`:

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserCollection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResourceCollection;

#[UseResourceCollection(CustomUserCollection::class)]
class User extends Model
{
    // ...
}
```

Alternatywnie, możesz określić klasę kolekcji zasobów przekazując ją do metody `toResourceCollection`: 

```php
return User::all()->toResourceCollection(CustomUserCollection::class);
```

<a name="custom-resource-collections"></a>
#### Niestandardowe Kolekcje Zasobów

Domyślnie kolekcje zasobów nie pozwalają na dodawanie niestandardowych meta danych, które mogą być potrzebne do zwrócenia z kolekcją. Jeśli chcesz dostosować odpowiedź kolekcji zasobów, możesz utworzyć dedykowany zasób do reprezentacji kolekcji:

```shell
php artisan make:resource UserCollection
```

Po wygenerowaniu klasy kolekcji zasobów, możesz łatwo zdefiniować dowolne meta dane, które powinny zostać uwzględnione w odpowiedzi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Przekształć kolekcję zasobów w tablicę.
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

Po zdefiniowaniu kolekcji zasobów, może ona zostać zwrócona z trasy lub kontrolera:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Lub, dla wygody, możesz użyć metody kolekcji Eloquent `toResourceCollection` która będzie używać konwencji frameworka do automatycznego wykrycia bazowej kolekcji zasobów modelu:

```php
return User::all()->toResourceCollection();
```

Podczas wywoływania metody `toResourceCollection`, Laravel spróbuje zlokalizować kolekcję zasobów, która pasuje do nazwy modelu i jest zakończona sufiksem `Collection` w przestrzeni nazw `Http\Resources` najbliższej przestrzeni nazw modelu.

<a name="preserving-collection-keys"></a>
#### Zachowywanie Kluczy Kolekcji

Podczas zwracania kolekcji zasobów z trasy, Laravel resetuje klucze kolekcji tak, aby były w porządku numerycznym. Jednak możesz dodać właściwość `preserveKeys` do Twojej klasy zasobu wskazującą, czy oryginalne klucze kolekcji powinny zostać zachowane:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Określa, czy klucze kolekcji zasobu powinny zostać zachowane.
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```

Gdy właściwość `preserveKeys` jest ustawiona na `true`, klucze kolekcji zostaną zachowane, gdy kolekcja jest zwracana z trasy lub kontrolera:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

<a name="customizing-the-underlying-resource-class"></a>
#### Dostosowywanie Bazowej Klasy Zasobu

Zazwyczaj właściwość `$this->collection` kolekcji zasobów jest automatycznie wypełniana wynikiem mapowania każdego elementu kolekcji do jego pojedynczej klasy zasobu. Przyjmuje się, że pojedyncza klasa zasobu to nazwa klasy kolekcji bez końcowej części `Collection` nazwy klasy. Dodatkowo, w zależności od Twoich preferencji, pojedyncza klasa zasobu może, ale nie musi być zakończona `Resource`.

Na przykład `UserCollection` spróbuje zmapować podane instancje użytkowników do zasobu `UserResource`. Aby dostosować to zachowanie, możesz nadpisać właściwość `$collects` Twojej kolekcji zasobów:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Zasób, który ten zasób zbiera.
     *
     * @var string
     */
    public $collects = Member::class;
}
```

<a name="writing-resources"></a>
## Tworzenie Zasobów

> [!NOTE]
> Jeśli nie przeczytałeś [przeglądu koncepcji](#concept-overview), gorąco zachęcamy do zrobienia tego przed kontynuowaniem lektury tej dokumentacji.

Zasoby muszą tylko przekształcić dany model w tablicę. Więc każdy zasób zawiera metodę `toArray` która tłumaczy atrybuty Twojego modelu na przyjazną dla API tablicę, która może zostać zwrócona z tras lub kontrolerów Twojej aplikacji:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Przekształć zasób w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Po zdefiniowaniu zasobu, może on zostać zwrócony bezpośrednio z trasy lub kontrolera:

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toUserResource();
});
```

<a name="relationships"></a>
#### Relacje

Jeśli chcesz uwzględnić powiązane zasoby w odpowiedzi, możesz dodać je do tablicy zwróconej przez metodę `toArray` Twojego zasobu. W tym przykładzie użyjemy metody `PostResource` zasobu `collection` aby dodać posty bloga użytkownika do odpowiedzi zasobu:

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

> [!NOTE]
> Jeśli chcesz uwzględnić relacje tylko wtedy, gdy zostały już załadowane, sprawdź dokumentację dotyczącą [warunkowych relacji](#conditional-relationships).

<a name="writing-resource-collections"></a>
#### Kolekcje Zasobów

Podczas gdy zasoby transformują pojedynczy model w tablicę, kolekcje zasobów transformują kolekcję modeli w tablicę. Jednak nie jest absolutnie konieczne definiowanie klasy kolekcji zasobów dla każdego z Twoich modeli, ponieważ wszystkie kolekcje modeli Eloquent udostępniają metodę `toResourceCollection` do generowania "ad-hoc" kolekcji zasobów w locie: 

```php
use App\Models\User;

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```

Jednak jeśli musisz dostosować meta dane zwracane z kolekcją, konieczne jest zdefiniowanie własnej kolekcji zasobów:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Przekształć kolekcję zasobów w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

Podobnie jak pojedyncze zasoby, kolekcje zasobów mogą być zwracane bezpośrednio z tras lub kontrolerów:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Lub, dla wygody, możesz użyć metody kolekcji Eloquent `toResourceCollection` która będzie używać konwencji frameworka do automatycznego wykrycia bazowej kolekcji zasobów modelu:

```php
return User::all()->toResourceCollection();
```

Podczas wywoływania metody `toResourceCollection`, Laravel spróbuje zlokalizować kolekcję zasobów, która pasuje do nazwy modelu i jest zakończona sufiksem `Collection` w przestrzeni nazw `Http\Resources` najbliższej przestrzeni nazw modelu.

<a name="data-wrapping"></a>
### Opakowywanie Danych

Domyślnie Twój najbardziej zewnętrzny zasób jest opakowany w klucz `data` gdy odpowiedź zasobu jest konwertowana na JSON. Więc na przykład, typowa odpowiedź kolekcji zasobów wygląda następująco:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "eviervlubt@example.com"
        }
    ]
}
```

Jeśli chcesz wyłączyć opakowywanie najbardziej zewnętrznego zasobu, powinieneś wywołać metodę `withoutWrapping` na bazowej klasie `Illuminate\Http\Resources\Json\JsonResource` Zazwyczaj powinieneś wywołać tę metodę z Twojego `AppServiceProvider` lub innego [dostawcy usług](/docs/{{version}}/providers), który jest ładowany przy każdym żądaniu do Twojej aplikacji:

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Supplubt\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Rejestruj dowolne usługi aplikacji.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Uruchom dowolne usługi aplikacji.
     */
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}
```

> [!WARNING]
> Metoda `withoutWrapping` wpływa tylko na najbardziej zewnętrzną odpowiedź i nie usunie kluczy `data` które ręcznie dodajesz do własnych kolekcji zasobów.

<a name="wrapping-nested-resources"></a>
#### Opakowywanie Zagnieżdżonych Zasobów

Masz całkowitą swobodę w określaniu, jak są opakowane relacje Twojego zasobu. Jeśli chcesz, aby wszystkie kolekcje zasobów były opakowane w klucz `data` niezależnie od ich zagnieżdżenia, powinieneś zdefiniować klasę kolekcji zasobów dla każdego zasobu i zwrócić kolekcję w kluczu `data`. 

Możesz się zastanawiać, czy spowoduje to opakowanie Twojego najbardziej zewnętrznego zasobu w dwa klucze `data`. Nie martw się, Laravel nigdy nie pozwoli na przypadkowe podwójne opakowanie Twoich zasobów, więc nie musisz martwić się o poziom zagnieżdżenia kolekcji zasobów, którą transformujesz:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * Przekształć kolekcję zasobów w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }
}
```

<a name="data-wrapping-i-pagination"></a>
#### Opakowywanie Danych i Pagination

Podczas zwracania kolekcji z paginacją przez odpowiedź zasobu, Laravel opakuje Twoje dane zasobu w klucz `data` nawet jeśli metoda `withoutWrapping` została wywołana. Dzieje się tak, ponieważ odpowiedzi z paginacją zawsze zawierają klucze `meta` i `links` z informacjami o stanie paginatora:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "eviervlubt@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### Paginacja

Możesz przekazać instancję paginatora Laravel do metody `collection` zasobu lub do niestandardowej kolekcji zasobów:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

Lub, dla wygody, możesz użyć metody paginatora `toResourceCollection` która będzie używać konwencji frameworka do automatycznego wykrycia bazowej kolekcji zasobów paginowanego modelu:

```php
return User::paginate()->toResourceCollection();
```

Odpowiedzi z paginacją zawsze zawierają klucze `meta` i `links` z informacjami o stanie paginatora:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "eviervlubt@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="customizing-the-pagination-influbmation"></a>
#### Dostosowywanie Informacji o Paginacji

Jeśli chcesz dostosować informacje zawarte w kluczach `links` lub `meta` odpowiedzi paginacji, możesz zdefiniować metodę `paginationInformation` w zasobie. Ta metoda otrzyma dane `$paginated` oraz tablicę `$default` informacji, która jest tablicą zawierającą klucze `links` i `meta`: 

```php
/**
 * Dostosuj informacje o paginacji dla zasobu.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  array  $paginated
 * @param  array  $default
 * @return array
 */
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';

    return $default;
}
```

<a name="conditional-attributes"></a>
### Warunkowe Atrybuty

Czasami możesz chcieć uwzględnić atrybut w odpowiedzi zasobu tylko wtedy, gdy spełniony jest dany warunek. Na przykład możesz chcieć uwzględnić wartość tylko wtedy, gdy bieżący użytkownik jest "administratorem". Laravel zapewnia różne metody pomocnicze, aby pomóc Ci w tej sytuacji. Metoda `when` może być użyta do warunkowego dodania atrybutu do odpowiedzi zasobu:

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

W tym przykładzie klucz `secret` zostanie zwrócony w końcowej odpowiedzi zasobu tylko wtedy, gdy metoda uwierzytelnionego użytkownika `isAdmin` zwraca `true`. Jeśli metoda zwraca `false`, klucz `secret` zostanie usunięty z odpowiedzi zasobu zanim zostanie wysłany do klienta. Metoda `when` pozwala na ekspresyjne definiowanie zasobów bez uciekania się do instrukcji warunkowych podczas budowania tablicy.

Metoda `when` akceptuje również domknięcie jako drugi argument, pozwalając obliczyć wynikową wartość tylko wtedy, gdy dany warunek jest `true`:

```php
'secret' => $this->when($request->user()->isAdmin(), function () {
    return 'secret-value';
}),
```

Metoda `whenHas` może być użyta do uwzględnienia atrybutu, jeśli faktycznie istnieje w bazowym modelu:

```php
'name' => $this->whenHas('name'),
```

Dodatkowo metoda `whenNotNull` może być użyta do uwzględnienia atrybutu w odpowiedzi zasobu, jeśli atrybut nie jest null:

```php
'name' => $this->whenNotNull($this->name),
```

<a name="merging-conditional-attributes"></a>
#### Łączenie Warunkowych Atrybutów

Czasami możesz mieć kilka atrybutów, które powinny zostać uwzględnione w odpowiedzi zasobu na podstawie tego samego warunku. W tym przypadku możesz użyć metody `mergeWhen` aby uwzględnić atrybuty w odpowiedzi tylko wtedy, gdy dany warunek jest `true`:

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Ponownie, jeśli dany warunek jest `false`, te atrybuty zostaną usunięte z odpowiedzi zasobu zanim zostanie wysłany do klienta.

> [!WARNING]
> Metoda `mergeWhen` nie powinna być używana w tablicach, które mieszają klucze tekstowe i numeryczne. Ponadto nie powinna być używana w tablicach z kluczami numerycznymi, które nie są uporządkowane sekwencyjnie.

<a name="conditional-relationships"></a>
### Warunkowe Relacje

Oprócz warunkowego ładowania atrybutów, możesz warunkowo uwzględniać relacje w odpowiedziach zasobów w oparciu o to, czy relacja została już załadowana w modelu. Pozwala to kontrolerowi decydować, które relacje powinny być załadowane w modelu, a zasób może łatwo uwzględnić je tylko wtedy, gdy zostały faktycznie załadowane. Ostatecznie ułatwia to unikanie problemów z zapytaniami "N+1" w Twoich zasobach.

Metoda `whenLoaded` może być użyta do warunkowego załadowania relacji. Aby uniknąć niepotrzebnego ładowania relacji, ta metoda akceptuje nazwę relacji zamiast samej relacji:

```php
use App\Http\Resources\PostResource;

/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

W tym przykładzie, jeśli relacja nie została załadowana, klucz `posts` zostanie usunięty z odpowiedzi zasobu zanim zostanie wysłany do klienta.

<a name="conditional-relationship-counts"></a>
#### Warunkowe Liczniki Relacji

Oprócz warunkowego uwzględniania relacji, możesz warunkowo uwzględniać "liczniki" relacji "counts" w odpowiedziach zasobów w oparciu o to, czy licznik relacji został załadowany w modelu:

```php
new UserResource($user->loadCount('posts'));
```

Metoda `whenCounted` może być użyta do warunkowego uwzględnienia licznika relacji w odpowiedzi zasobu. Ta metoda unika niepotrzebnego uwzględniania atrybutu, jeśli licznik relacji nie jest obecny:

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts_count' => $this->whenCounted('posts'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

W tym przykładzie, jeśli licznik relacji `posts` nie został załadowany, klucz `posts_count` zostanie usunięty z odpowiedzi zasobu zanim zostanie wysłany do klienta.

Inne typy agregatów, takie jak `avg`, `sum`, `min`, i `max` mogą być również warunkowo załadowane za pomocą metody `whenAggregated`:

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-influbmation"></a>
#### Warunkowe Informacje Pivot

Oprócz warunkowego uwzględniania informacji o relacji w odpowiedziach zasobów, możesz warunkowo uwzględniać dane z tabel pośrednich relacji wiele-do-wielu używając metody `whenPivotLoaded`. Metoda `whenPivotLoaded` akceptuje nazwę tabeli pivot jako pierwszy argument. Drugi argument powinien być domknięciem, które zwraca wartość do zwrócenia, jeśli informacje pivot są dostępne w modelu:

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

Jeśli Twoja relacja używa [niestandardowego modelu tabeli pośredniej](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models), możesz przekazać instancję modelu tabeli pośredniej jako pierwszy argument do metody `whenPivotLoaded` 

```php
'expires_at' => $this->whenPivotLoaded(new Membership, function () {
    return $this->pivot->expires_at;
}),
```

Jeśli Twoja tabela pośrednia używa akcesu innego niż `pivot`, możesz użyć metody `whenPivotLoadedAs`: 

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

<a name="adding-meta-data"></a>
### Dodawanie Meta Danych

Niektóre standardy JSON API wymagają dodania meta danych do odpowiedzi zasobów i kolekcji zasobów. Często obejmuje to takie rzeczy jak `links` do zasobu lub powiązanych zasobów, lub meta dane o samym zasobie. Jeśli musisz zwrócić dodatkowe meta dane o zasobie, uwzględnij je w swojej metodzie `toArray`. Na przykład możesz uwzględnić informacje `links` podczas transformowania kolekcji zasobów:

```php
/**
 * Przekształć zasób w tablicę.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

Podczas zwracania dodatkowych meta danych z zasobów, nigdy nie musisz martwić się o przypadkowe nadpisanie kluczy `links` lub `meta`, które są automatycznie dodawane przez Laravel podczas zwracania odpowiedzi z paginacją. Wszelkie dodatkowe `links`, które zdefiniujesz, zostaną scalone z linkami dostarczonymi przez paginator.

<a name="top-level-meta-data"></a>
#### Meta Dane Najwyższego Poziomu

Czasami możesz chcieć uwzględnić określone meta dane w odpowiedzi zasobu tylko wtedy, gdy zasób jest najbardziej zewnętrznym zwracanym zasobem. Zazwyczaj obejmuje to meta informacje o całej odpowiedzi. Aby zdefiniować te meta dane, dodaj metodę `with` do swojej klasy zasobu. Ta metoda powinna zwrócić tablicę meta danych, które mają być uwzględnione w odpowiedzi zasobu tylko wtedy, gdy zasób jest najbardziej zewnętrznym transformowanym zasobem:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Przekształć kolekcję zasobów w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * Pobierz dodatkowe dane, które powinny zostać zwrócone z tablicą zasobu.
     *
     * @return array<string, mixed>
     */
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

<a name="adding-meta-data-when-constructing-resources"></a>
#### Dodawanie Meta Danych Podczas Konstruowania Zasobów

Możesz również dodać dane najwyższego poziomu podczas konstruowania instancji zasobów w trasie lub kontrolerze. Metoda `additional`, która jest dostępna we wszystkich zasobach, akceptuje tablicę danych, które powinny zostać dodane do odpowiedzi zasobu:

```php
return User::all()
    ->load('roles')
    ->toResourceCollection()
    ->additional(['meta' => [
        'key' => 'value',
    ]]);
```

<a name="resource-responses"></a>
## Odpowiedzi Zasobów

Jak już przeczytałeś, zasoby mogą być zwracane bezpośrednio z tras i kontrolerów:

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toResource();
});
```

Jednak czasami może być konieczne dostosowanie wychodzącą odpowiedź HTTP zanim zostanie wysłana do klienta. Istnieją dwa sposoby, aby to osiągnąć. Po pierwsze, możesz połączyć w łańcuch metodę `response` do zasobu. Ta metoda zwróci instancję `Illuminate\Http\JsonResponse` dając Ci pełną kontrolę nad nagłówkami odpowiedzi:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user', function () {
    return User::find(1)
        ->toResource()
        ->response()
        ->header('X-Value', 'True');
});
```

Alternatywnie, możesz zdefiniować metodę `withResponse` w samym zasobie. Ta metoda zostanie wywołana, gdy zasób zostanie zwrócony jako najbardziej zewnętrzny zasób w odpowiedzi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Przekształć zasób w tablicę.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * Dostosuj wychodzącą odpowiedź dla zasobu.
     */
    public function withResponse(Request $request, JsonResponse $response): void
    {
        $response->header('X-Value', 'True');
    }
}
```
