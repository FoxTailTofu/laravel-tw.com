# 路由

- [基本路由](#basic-routing)
    - [路由重導向](#redirect-routes)
    - [視圖路由](#view-routes)
- [路由參數](#route-parameters)
    - [必要參數](#required-parameters)
    - [選擇性參數](#parameters-optional-parameters)
    - [正規表達式限制](#parameters-regular-expression-constraints)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Subdomain Routing](#route-group-subdomain-routing)
    - [Route Prefixes](#route-group-prefixes)
    - [Route Name Prefixes](#route-group-name-prefixes)
- [Route Model Binding](#route-model-binding)
    - [Implicit Binding](#implicit-binding)
    - [Explicit Binding](#explicit-binding)
- [Fallback Routes](#fallback-routes)
- [Rate Limiting](#rate-limiting)
- [Form Method Spoofing](#form-method-spoofing)
- [Accessing The Current Route](#accessing-the-current-route)
- [Cross-Origin Resource Sharing (CORS)](#cors)

<a name="basic-routing"></a>
## 基本路由

最基本的 Laravel 路由會接收一個 URI 和一個 `Closure`, 提供非常簡單的路由表達方式：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 預設路由檔案

所有Laravel的路由都透過您 `route` 資料夾中的檔案來定義的。這些檔案會由框架自動載入。 `routes/web.php` 定義了您網路介面的路由。這些路由會被分配至 `web` 中介層組，讓它提供像是 session 狀態和 CSRF 保護的功能。在 `routes/api.php` 裡面的路由則是是無狀態的並且被分配至 `api` 中介層組。

在大部分情況下，您會從您的 `routes/web.php` 開始設置路由。在 `routes/web.php` 中被定義的路由可能會透過輸入您所設置的路由網址來訪問。舉例來說，您可以透過瀏覽 `http://your-app.test/user` 取得下面所指定的路由設定：

    Route::get('/user', 'UserController@index');

被定義在 `routes/api.php` 中的路由會在一個路由群組中透過 `RouteServiceProvider` 變成巢狀結構。在這個群組裡面， `/api` URI前綴會被自動套用好，您不需要再手動幫每個路由加上它。您也可以透過修改 `RouteServiceProvider` 類別來調整前綴和其他路由群組選項。

#### 可用的路由器選項
路由器可以讓您登記任何路由來回應任何 HTTP 動作 (HTTP verb)：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有時候您需要登記一個路由讓他回應多種 HTTP 動作。您可能需要使用 `match` 方法。或是您也可以使用 `any` 來回應所有的 HTTP 動作：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

#### CSRF 保護

任何使用 `POST`、`PUT`、`PATCH` 或 `DELETE` 的 HTML 表單，在 `web` 路由設定中都會被要求要有 CSRF token field。不然將會拒絕請求。有關 CSRF 保護的功能您可以閱讀 [CSRF 文件資料](/docs/{{version}}/csrf)：

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### 路由重導向

如果您正在定義一個會導向其他的 URI 的路由，您可以使用 `Route::redirect` 方法。這個方法提供了簡單的捷徑讓您不用定義一整個新的路由或是控制器：

    Route::redirect('/here', '/there');

預設情況下 `Route::redirect` 會回傳一個 `302` 狀態碼。您可以透過第三個參數定義它：

    Route::redirect('/here', '/there', 301);

您也可以用 `Route::permanentRedirect` 方法回傳一個 `301` 狀態碼：

    Route::permanentRedirect('/here', '/there');

<a name="view-routes"></a>
### 視圖路由

如果您的路由只需要回傳一個視圖(View)，您可以使用 `Route::view` 方法。就像 `redirect`，這個方法提供了簡單的捷徑讓您不用定義一整個新的路由或是控制器。`view` 接受一個 URI 作為第一個參數 和一個View Name 做為第二個參數。另外，您也可以用陣列傳送資料給視圖：

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## 路由參數

<a name="required-parameters"></a>
### 必要參數

有時候您會需要取得 URI 上面的參數提供給路由使用。舉例來說，您需要透過 URL 取得使用者的 ID。您可以透過下面的方法定義：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

您可以依照需求定義多個參數來使用：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由參數都會包裝在一組 `{}` 內並且只由英文字母組成，裡面不能包含 `-` 字元，如果有需要請改用下劃線 `_` 替代。路由的參數會依據順序傳回路由或控制器，回傳和控制器的引數 (argument) 並不會影響。

<a name="parameters-optional-parameters"></a>
### 選擇性參數

偶爾您會需要定義一個路由參數，但是不要求這個參數一定存在。您可以透過在參數名稱後面加上一個 `?` 字元來做這件事。請確定您給路由一組對應預設數值避免問題：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正規表達式限制

您可以透過 `where` 方法來限制路由參數的格式。`where` 透過一組正規表達式限制參數的格式：

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全域限制

如果您希望一個路由的所有特定參數都被一組正規表達式限制，您可以使用 `pattern` 方法。您應該在 `RouteServiceProvider` 中的 `boot` 方法裡面定義：

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

當 pattern 被定義後，它會自動套用到所有使用到對應參數名稱的參數上：

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### 編碼正斜線 (Encoded Forward Slashes)

Laravel 的路由組件接受除了 `/` 以外的所有字元。您需要使用 `where` 加上一組正規表達式明確的宣告 `/` 是參數值：

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {備註} 編碼正斜線 只在路由的最後一段可以使用

<a name="named-routes"></a>
## Named Routes

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:

    Route::get('user/profile', function () {
        //
    })->name('profile');

You may also specify route names for controller actions:

    Route::get('user/profile', 'UserProfileController@show')->name('profile');

> {note} Route names should always be unique.

#### Generating URLs To Named Routes

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via the global `route` function:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the URL in their correct positions:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

If you pass additional parameters in the array, those key / value pairs will automatically be added to the generated URL's query string:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> {tip} Sometimes, you may wish to specify request-wide default values for URL parameters, such as the current locale. To accomplish this, you may use the [`URL::defaults` method](/docs/{{version}}/urls#default-values).

#### Inspecting The Current Route

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Route Groups

Route groups allow you to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. Shared attributes are specified in an array format as the first parameter to the `Route::group` method.

Nested groups attempt to intelligently "merge" attributes with their parent group. Middleware and `where` conditions are merged while names, namespaces, and prefixes are appended. Namespace delimiters and slashes in URI prefixes are automatically added where appropriate.

<a name="route-group-middleware"></a>
### Middleware

To assign middleware to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces

Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` method:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.

<a name="route-group-subdomain-routing"></a>
### Subdomain Routing

Route groups may also be used to handle subdomain routing. Subdomains may be assigned route parameters just like route URIs, allowing you to capture a portion of the subdomain for usage in your route or controller. The subdomain may be specified by calling the `domain` method before defining the group:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> {note} In order to ensure your subdomain routes are reachable, you should register subdomain routes before registering root domain routes. This will prevent root domain routes from overwriting subdomain routes which have the same URI path.

<a name="route-group-prefixes"></a>
### Route Prefixes

The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Route Name Prefixes

The `name` method may be used to prefix each route name in the group with a given string. For example, you may want to prefix all of the grouped route's names with `admin`. The given string is prefixed to the route name exactly as it is specified, so we will be sure to provide the trailing `.` character in the prefix:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Route Model Binding

When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Implicit Binding

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Since the `$user` variable is type-hinted as the `App\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

#### Customizing The Key

Sometimes you may wish to resolve Eloquent models using a column other than `id`. To do so, you may specify the column in the route parameter definition:

    Route::get('api/posts/{post:slug}', function (App\Post $post) {
        return $post;
    });

#### Custom Keys & Scoping

Sometimes, when implicitly binding multiple Eloquent models in a single route definition, you may wish to scope the second Eloquent model such that it must be a child of the first Eloquent model. For example, consider this situation that retrieves a blog post by slug for a specific user:

    use App\Post;
    use App\User;

    Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

When using a custom keyed implicit binding as a nested route parameter, Laravel will automatically scope the query to retrieve the nested model by its parent using conventions to guess the relationship name on the parent. In this case, it will be assumed that the `User` model has a relationship named `posts` (the plural of the route parameter name) which can be used to retrieve the `Post` model.

#### Customizing The Default Key Name

If you would like model binding to use a default database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Explicit Binding

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Next, define a route that contains a `{user}` parameter:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### Customizing The Resolution Logic

If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->firstOrFail();
        });
    }

Alternatively, you may override the `resolveRouteBinding` method on your Eloquent model. This method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    /**
     * Retrieve the model for a bound value.
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }

<a name="fallback-routes"></a>
## Fallback Routes

Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you may define the `fallback` route within your `routes/web.php` file, all middleware in the `web` middleware group will apply to the route. You are free to add additional middleware to this route as needed:

    Route::fallback(function () {
        //
    });

> {note} The fallback route should always be the last route registered by your application.

<a name="rate-limiting"></a>
## Rate Limiting

Laravel includes a [middleware](/docs/{{version}}/middleware) to rate limit access to routes within your application. To get started, assign the `throttle` middleware to a route or a group of routes. The `throttle` middleware accepts two parameters that determine the maximum number of requests that can be made in a given number of minutes. For example, let's specify that an authenticated user may access the following group of routes 60 times per minute:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Dynamic Rate Limiting

You may specify a dynamic request maximum based on an attribute of the authenticated `User` model. For example, if your `User` model contains a `rate_limit` attribute, you may pass the name of the attribute to the `throttle` middleware so that it is used to calculate the maximum request count:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Distinct Guest & Authenticated User Rate Limits

You may specify different rate limits for guest and authenticated users. For example, you may specify a maximum of `10` requests per minute for guests `60` for authenticated users:

    Route::middleware('throttle:10|60,1')->group(function () {
        //
    });

You may also combine this functionality with dynamic rate limits. For example, if your `User` model contains a `rate_limit` attribute, you may pass the name of the attribute to the `throttle` middleware so that it is used to calculate the maximum request count for authenticated users:

    Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Rate Limit Segments

Typically, you will probably specify one rate limit for your entire API. However, your application may require different rate limits for different segments of your API. If this is the case, you will need to pass a segment name as the third argument to the `throttle` middleware:

    Route::middleware('auth:api')->group(function () {
        Route::middleware('throttle:60,1,default')->group(function () {
            Route::get('/servers', function () {
                //
            });
        });

        Route::middleware('throttle:60,1,deletes')->group(function () {
            Route::delete('/servers/{id}', function () {
                //
            });
        });
    });

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

You may use the `@method` Blade directive to generate the `_method` input:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel can automatically respond to CORS OPTIONS requests with values that you configure. All CORS settings may be configured in your `cors` configuration file and OPTIONS requests will automatically be handled by the `HandleCors` middleware that is included by default in your global middleware stack.

> {tip} For more information on CORS and CORS headers, please consult the [MDN web documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).
