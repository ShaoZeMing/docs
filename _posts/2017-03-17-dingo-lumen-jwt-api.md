---
layout: blog
title: Git SSH gitHub 秘钥连接配置
category: blogs
---

Lumen轻量级的框架，用来做API很合适，起码本人这么想。

做API，可以配合Dingo/Api来搞。

## Dingo/Api可以使用的用户认证方式有：

- HTTP Basic (Dingo\Api\Auth\Provider\Basic)
- JSON Web Tokens (Dingo\Api\Auth\Provider\JWT)
- OAuth 2.0 (Dingo\Api\Auth\Provider\OAuth2)

这里选了JWT来实现。

## 原文链接
- Laravel-lumen 配置JWT<http://blog.csdn.net/hooloo/article/details/49714259>
- Lumen上使用Dingo/Api做API开发时用JWT-Auth做认证的实现<http://blog.csdn.net/jimmy_zrone/article/details/50667743>

# Tymon/JWT-Auth安装

- 下载包

`composer require tymon/jwt-auth`

- 在"bootstrap/app.PHP" 文件中，找到Register Service Providers一节，添加：

`$app->register('Tymon\JWTAuth\Providers\JWTAuthServiceProvider');`

- 在“app”目录下创建“helpers.php”文件。

```
<?php

if ( ! function_exists('config_path'))
{
    /**
     * Get the configuration path.
     *
     * @param  string $path
     * @return string
     */
    function config_path($path = '')
    {
        return app()->basePath() . '/config' . ($path ? '/' . $path : $path);
    }
}
```

- 修改composer.json文件。添加：
```
"autoload": {
    ...
    "files": [
      "app/helpers.php"
    ]
  },
```
- 执行：`composer dump-autoload`

## 生成jwt-auth的配置文件

最简单的办法是从 `/vendor/tymon/jwt-auth/src/config/config.php` 复制一份到`config/jwt.php`没有config创建一个。

- 运行  `composer require basicit/lumen-vendor-publish `

- 在 app/Console/Kernel.php 文件内 添加(**此处是lumen5.4,如果提示没有这个类，可以在vendor下查看对应的命名空间路径**):
```
protected $commands = [
     'Laravelista\LumenVendorPublish\VendorPublishCommand'   //新版"basicit/lumen-vendor-publish": "^2.0"
    // 'BasicIT\LumenVendorPublish\VendorPublishCommand'  //老版"basicit/lumen-vendor-publish": "^1.0"
];

```

- 运行 ：`php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider`

此时生成了config/jwt.php

- 执行`php artisan jwt:generate`生成secret。

例：`g3JjVnSwj82ncqckvguMDP7Em1z9zjUa`


## 添加 facades

- bootstrap/app.php文件中去掉"$app->withFacades();"前的注释。

- 在这行下面添加“$app->configure('jwt');”，不然调用不到jwt的配置文件。

- 紧接着是facades。

- 去 bootstrap/app.php 内, 找到 $app->withFacades(); 并去掉注释,在下面写上

```
$app->configure('jwt');
class_alias('Tymon\JWTAuth\Facades\JWTAuth', 'JWTAuth');
class_alias('Tymon\JWTAuth\Facades\JWTFactory', 'JWTFactory');
```
jwt的配置文件里保持默认就可以。想知道具体含义可以参考它的文档。secret是必须设的。前面已经设过了。

- 在bootstrap/app.php文件中，查找Register Middleware 小节。去掉"routeMiddleware"的注释，修改成下面这样：

```
$app->routeMiddleware([
    'jwt.auth'    => Tymon\JWTAuth\Middleware\GetUserFromToken::class,
    'jwt.refresh' => Tymon\JWTAuth\Middleware\RefreshToken::class,
]);
```
- 此时，可以使用路由中间件了。下面举个例子。

```
<pre name="code" class="php">// store, update, destory这些需要权限的操作全部需要经过认证。
$app->group(['prefix' => 'projects', 'middleware' => 'jwt.auth'], function($app) { $app->post('/', 'App\Http\Controllers\ProjectsController@store'); $app->put('/{projectId}', 'App\Http\Controllers\ProjectsController@update'); $app->delete('/{projectId}', 'App\Http\Controllers\ProjectsController@destroy');}); // index, show这些则不需要$app->group(['prefix' => 'projects'], function ($app){ $app->get('/', 'App\Http\Controllers\ProjectsController@index'); $app->get('/{projectId}', 'App\Http\Controllers\ProjectsController@show');});

```
- 登录。下面要做的就是写一个登录的程序，因为lumen没有，要自己写。
建一个路由“`$app->post('auth/login', 'App\Http\Controllers\Auth\AuthController@postLogin');`”
在“`app/Http/Controllers/Auth/AuthController.php`”中创建我们的登录代码。

示例如下：

```
<?php

namespace App\Http\Controllers\Auth;

use Illuminate\Http\Exception\HttpResponseException;
use JWTAuth;
use Tymon\JWTAuth\Exceptions\JWTException;
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\Response as IlluminateResponse;

class AuthController extends Controller {

    /**
     * Handle a login request to the application.
     *
     * @param \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function postLogin(Request $request)
    {
        try
        {
            $this->validate($request, [
                'email' => 'required|email|max:255', 'password' => 'required',
            ]);
        }
        catch (HttpResponseException $e)
        {
            return response()->json([
                'error' => [
                    'message'     => 'Invalid auth',
                    'status_code' => IlluminateResponse::HTTP_BAD_REQUEST
                ]],
                IlluminateResponse::HTTP_BAD_REQUEST,
                $headers = []
            );
        }

        $credentials = $this->getCredentials($request);

        try
        {
            // attempt to verify the credentials and create a token for the user
            if ( ! $token = JWTAuth::attempt($credentials))
            {
                return response()->json(['error' => 'invalid_credentials'], 401);
            }
        }
        catch (JWTException $e)
        {
            // something went wrong whilst attempting to encode the token
            return response()->json(['error' => 'could_not_create_token'], 500);
        }

        // all good so return the token
        return response()->json(compact('token'));
    }

    /**
     * Get the needed authorization credentials from the request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    protected function getCredentials(Request $request)
    {
        return $request->only('email', 'password');
    }
}

```
此时，用post方式提交一个登录请求会得到json格式的返回值，里面就是登录后获得的token.

```
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjEsImlzcyI6Imh0dHA6XC9cLzEyNy4wLjAuMTo4NFwvYXV0aFwvbG9naW4iLCJpYXQiOiIxNDQ2OTE4MjM3IiwiZXhwIjoiMTQ0NjkyMTgzNyIsIm5iZiI6IjE0NDY5MTgyMzciLCJqdGkiOiI1ZGY1Njk5OGYxMTc3MzlhMjQ4ZjgzNzUyZmQ2MTA1MiJ9.1FP8yXwlw_KHrx9NAFqQWPaq2c2LLq_vyuJgwI_EX9k"
}
```

- 还要提一下，jwt-auth默认使用Users表做为登录认证的表，这个表跟laravel是一样的。所以可以直接从laravel复制过来。
也可以另外指定。具体请看jwt.php中"User Model namespace" 的设置。

也就是，创建"User" model，生成数据库表users。再插入几条用户记录，到此项目完成。

但是！！！这里有个问题，User Model有些内容必须要有。

如下：
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Auth\Authenticatable;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

class User extends Model implements AuthenticatableContract, CanResetPasswordContract
{
    use Authenticatable, CanResetPassword;

    //use EntrustUserTrait; // add this trait to your user model

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'users';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'email', 'password'];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token'];
}

```
请跟自己的User Model对比一下。
出现如下错误一般都是User Model有问题。
```
[plain] view plain copy
Argument 1 passed to Illuminate\Auth\EloquentUserProvider::validateCredentials() must be an instance of Illuminate\Contracts\Auth\Authenticatable, instance of App\User given, called in C:\xampp\htdocs\APIs\vendor\illuminate\auth\Guard.php on line 390
```

另外，有条件的可以装个Postman。这个工具非常非常棒。

