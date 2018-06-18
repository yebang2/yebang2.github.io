开始之前我们先理清楚相关的概念信息。

相关的项目Demo可以参考[Laravel-Passport-API-Server-Client-Demo](https://github.com/LaravelDaily/Laravel-Passport-API-Server-Client-Demo)

# Github Oauth App
以github为例子，假如我们的应用支持Github登录，第一步去github的Settings/Developer settings 里面新建一个New Oauth App。

Oauth App，需要填写的信息有Application name，Homepage URL，Application description,
Authorization callback URL
- Homepage 就是你网页的URL，例如：http://laravel.work
- Authorization callback URL 例如：http://larabbs.work/oauth/github/callback 这是客户端授权成功后要返回的页面。我们在客户端要新建这样一个路由。

这样子我们在github上已经把Server端建立好了，现在我们需要在网页的客户端进行相关的操作。
- 第一我们需要向github的发送一个请求授权的路由

 Route::get('/oauth/github', 'Auth\LoginController@redirectToProvider')->name('github.provider');
路由会把我们重定向到github的授权页面，我们同意授权然后输入用户名和密码
```
public function redirectToProvider()
{
    $config = config('services');

    $socialite = new SocialiteManager($config);;
    
    // 重定向到callback页面
    return  $socialite->driver('github')->redirect();
}
```

- 授权结束后，会返回带http://larabbs.work/oauth/github/callback 这个路由，我们这时候会返回github上获取的用户信息，我们将其获取出来然后将其注册到当前的应用的用户表内
```
public function handleProviderCallback()
{
    $config = config('services');

    $socialite = new SocialiteManager($config);

    $user = $socialite->driver('github')->user();

    $user_info = [
        'name' => $user->getUsername(),
        'email'=> $user->getEmail(),
        'avatar' => $user->getAvatar(),
        'password' => bcrypt(str_random(16)),
    ];

    $is_user_exit = User::query()->where('email',$user_info['email'])->first();

    if ($is_user_exit){
        try{
            \Auth::login($is_user_exit);
            return redirect()->route('root')->with('success','登录成功');
        }catch (\Exception $e){
            return  redirect()->route('login')->with('danger','登录失败');
        }
    }else{
        $user = User::create($user_info);
        try{
            \Auth::login($user);
            return redirect()->route('root')->with('success','登录成功');
        }catch (\Exception $e){
            return back()->with('danger','登录失败');
        }

    }

    return $user;
}
```
这样一个Oauth的注册流程就完成了。

在Server服务端记录了client_id和client_secret和Authorization callback URL这样的参数，Authorization callback URL 是返回到当前客户端的路由当中进行相关逻辑的处理。

在Client需要记录client_id,client_secret和Authorization callback URL这三个参数。

---

# Laravel Passport
Laravel Server中新建Client，会获取到client_id和Client_secret和callback。这三个参数。

Laravel Passport 提供了JSON API，我们可以用他来操作Client。

这些api也有中间件web，所以不能在别postman里面进行操作
```
GET /oauth/clients

POST /oauth/clients

PUT /oauth/clients/{client-id}

DELETE /oauth/clients/{client-id}

```
### 1. 授权码模式
#### 在服务端创建client_id和client_secret和callback
创建客户端最简单的方式是使用 Artisan 命令 passport:client，你可以使用此命令创建自己的客户端，用于测试你的 OAuth2 的功能。在你执行 client 命令时，Passport 会提示你输入有关客户端的信息，最终会给你提供客户端的 ID 和 密钥：

php artisan passport:client

#### 在客户端调用

其中API_URL是服务端的IP地址
```
Route::get('/redirect', function () {
    if (!env('API_CLIENT_ID') || !env('API_URL') || !env('API_CLIENT_SECRET')) {
        return "Please fill API fields in env file";
    }
    $query = http_build_query([
        'client_id' => env('API_CLIENT_ID'),
        'response_type' => 'code',
        'scope' => '',
    ]);

    return redirect(env('API_URL').'/oauth/authorize?'.$query);
});
```

> return redirect(env('API_URL').'/oauth/authorize?'.$query); 

这一个会跳转到server的视图中进行验证，用用户确定是否授权，如果用户没有登录就进行登录操作。操作结束之后，就会定向到客户端的callback的路由上。也就是下面的代码，进行相关的逻辑处理。



#### 将授权码转换为访问令牌

用户批准授权请求后，会被重定向回接入的应用程序。然后接入应用应该将通过 POST 请求向你的应用程序申请访问令牌。请求应该包括当用户批准授权请求时由应用程序发出的授权码。在下面的例子中，我们使用 Guzzle HTTP 库来实现这次 POST 请求：

> request()->code
这个code是/redirect 返回之后获取的。

callback通过code获取了access_token，然后将access_token保存在sesion中，当然也可以保存在数据库当中
```
// Callback获得了accessToken
Route::get('/callback', function () {
    $http = new GuzzleHttp\Client;

    $response = $http->post(env('API_URL').'/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => env('API_CLIENT_ID'),
            'client_secret' => env('API_CLIENT_SECRET'),
            'code' => request()->code,
        ],
    ]);

    $apiResponse = json_decode((string) $response->getBody(), true);
    session(['api'=> $apiResponse]);
    session(['api-token'=> $apiResponse['access_token']]);
    return redirect('/');
});
```
##### 刷新令牌
如果你的应用程序发放了短期的访问令牌，用户将需要通过在发出访问令牌时提供给他们的刷新令牌来刷新其访问令牌。在下面的例子中，我们使用 Guzzle HTTP 库来刷新令牌：

```
$http = new GuzzleHttp\Client;

$response = $http->post('http://API_URL/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```
---

这是相关的操作的路由，中间件是auth:api，使用grant是client模式生成的token是没有办法访问到的，因为他需要用户信息，client模式是没有用户信息的
```
Route::group(['prefix' => '/v1', 'namespace' => 'Api\V1', 'as' => 'api.', 'middleware' => ['auth:api']], function () {

    Route::resource('projects', 'ProjectsController', ['except' => ['create', 'edit']]);
});
```
Oauth2.0的授权模式获取的code只能使用一次，第二次就不能使用了。
为了保证安全,所以code要重新获取。

```
{
    "error": "invalid_request",
    "message": "The request is missing a required parameter, includes an invalid parameter value, includes a parameter more than once, or is otherwise malformed.",
    // 显示code已经失效了
    "hint": "Authorization code has expired"
}
```

有了access_token 之后，就可以对操作api了，进行增删改查

---

# 密码授权令牌
OAuth2 密码授权机制可以让你自己的客户端（如移动应用程序）邮箱地址或者用户名和密码获取访问令牌。如此一来你就可以安全地向自己的客户端发出访问令牌，而不需要遍历整个 OAuth2 授权代码重定向流程。


创建密码授权客户端
在应用程序通过密码授权机制来发布令牌之前，在 passport:client 命令后加上 --password 参数来创建密码授权的客户端。如果你已经运行了 passport:install 命令，则不需要再运行此命令：

```
php artisan passport:client --password
```
#### 请求令牌

```
$http = new GuzzleHttp\Client;

$response = $http->post('http://API_URL/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```
获取到了access_token之后就可以进行一些列的操作了
在header里面里面加上
```
Accept application/json

Authorization Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6ImU0NmU4ZDRjMmZiNTMxNjExMjk1NWNmYmVhYjNkZWZkZWFkMDk4ZTZmOWE5YmNmNmM2M2YzOWI1YzE0NjY1NmVhMWNiZTM5ZmRmMjJhZGFlIn0.eyJhdWQiOiI2IiwianRpIjoiZTQ2ZThkNGMyZmI1MzE2MTEyOTU1Y2ZiZWFiM2RlZmRlYWQwOThlNmY5YTliY2Y2YzYzZjM5YjVjMTQ2NjU2ZWExY2JlMzlmZGYyMmFkYWUiLCJpYXQiOjE1MjkzMjUzNDEsIm5iZiI6MTUyOTMyNTM0MSwiZXhwIjoxNTYwODYxMzQxLCJzdWIiOiIyIiwic2NvcGVzIjpbXX0.L7IIhy02jcctOugcXqqThQKVSQ19Rou9PCEgsSFuTKv9VByBS-gDIHpoitvonFfFouj2qZiTkqgDgbjYG4tLaaMPS_U0Fu33m-bfjAMlZYFDdMSkPJbrhXykB2SFz9M-pv-laXN7flMfUtZ0W19a_FUvUNvea-_VwwyWYNmsoT-xBvEaGxrFNQ_ZQry50T12RDDBQz3fxCyTbQUHifvM-pIDByu8DMcq27GwpZRpet9LbbHNkKKX2awHcGyY2GQmiBQC7MK6NEVTD47008KVmau38VW6B1rBk4HdvIZMim4P8J4LvJHGiz9-PqOjcaKlUFbLufFNSAXsBLvPy3td3rHMFWYrwWkw5XQkkLZLbpTNXBpkRNZLC_IQmoIQvBWWWbxGlkg2GM7Hj0Y5I_-B-5j3l_NPlAOMffeqIRsEgIlxiYwKenaVyaH-CZ8wvXFEqPPUINsNCzdLaM07Dyu4TPcMUH1Ikjtq5QWbLjA3gUaW3MtllFsi1eqnjAs6PtUXgyYYCcbsYuV5SzGzbPaV7pPX74xNRHLsaeiM0o8VAvLIlNiV92EsjUKggZ7EwcAw6BzXaBOexTd450VfC39pt6WKOA8QUdY8jIb9ytkV7pJM1w0tB6cyG7L6d9CdJwlqTOg7hUDzHEPk9mN-R1ybEyndMwuFXqyRH2wdlqbtR-4
```

# 客户端凭据授权令牌
客户端凭据授权适用于机器到机器的认证。例如，你可以在通过 API 执行维护任务中使用此授权。要使用这种授权，你首先需要在 app/Http/Kernel.php 的 $routeMiddleware 变量中添加新的中间件：
```
use Laravel\Passport\Http\Middleware\CheckClientCredentials::class;

protected $routeMiddleware = [
    'client' => CheckClientCredentials::class,
];
```
然后在路由上追加这个中间件：
```
Route::get('/user', function(Request $request) {
    ...
})->middleware('client');
```
接下来通过向 oauth/token 接口发出请求来获取令牌:
```
$guzzle = new GuzzleHttp\Client;

$response = $guzzle->post('http://API_URL/oauth/token', [
    'form_params' => [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ],
]);

echo json_decode((string) $response->getBody(), true);
```
---
```
传递访问令牌
当调用 Passport 保护下的路由时，接入的 API 应用需要将访问令牌作为 Bearer 令牌放在请求头 Authorization 中。例如，使用 Guzzle HTTP 库时：

$response = $client->request('GET', '/api/user', [
    'headers' => [
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ],
]);
```

# 令牌作用域
定义作用域

作用域可以让 API 客户端在请求账户授权时请求特定的权限。例如，如果你正在构建电子商务应用程序，并不是所有接入的 API 应用都需要下订单的功能。你可以让接入的 API 应用只被允许授权访问订单发货状态。换句话说，作用域允许应用程序的用户限制第三方应用程序执行的操作。

你可以在 AuthServiceProvider 的 boot 方法中使用 Passport::tokensCan 方法来定义 API 的作用域。tokensCan 方法接受一个作用域名称、描述的数组作为参数。作用域描述将会在授权确认页中直接展示给用户，你可以将其定义为任何你需要的内容：
```
use Laravel\Passport\Passport;

// 现在定义scope 数据的第一个参数是scope 第二个是描述

Passport::tokensCan([
    'place-orders' => 'Place orders',
    'check-status' => 'Check order status',
]);
```

# 检查作用域
Passport 包含两个中间件，可用于验证传入的请求是否已被授予给定作用域的令牌进行身份验证。使用之前，需要将下面的中间件添加到 app/Http/Kernel.php 文件的 $routeMiddleware 属性中：
```
'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
```
#### 检查所有作用域

路由可以使用 scopes 中间件来检查当前请求是否拥有指定的 所有 作用域：
```
Route::get('/orders', function () {
    // 访问令牌具有 "check-status" and "place-orders" 的作用域...
})->middleware('scopes:check-status,place-orders');
```
#### 检查任意作用域

路由可以使用 scope 中间件来检查当前请求是否拥有指定的 任意 作用域：
```
Route::get('/orders', function () {
    // Access token has either "check-status" or "place-orders" scope...
})->middleware('scope:check-status,place-orders');
```
#### 检查令牌实例上的作用域

就算访问令牌验证的请求已经通过应用程序的验证，你仍然可以使用当前授权 User 实例上的 tokenCan 方法来验证令牌是否拥有指定的作用域：
```
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('place-orders')) {
        //
    }
});
```
在client模式下不能通过auth:api中间件，而code和password模式可以通过client中间件,
所以在client模式下是没有用户信息的，在code和password模式下是有用户信息的。