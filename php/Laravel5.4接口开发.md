## 1.使用composer安装dingo/api和tymon/jwt-auth

    composer require "dingo/api:dev-master" //目前低版本的dingo/api不能支持laravel5.4，使用dev版本的dingo/api
    composer require "tymon/jwt-authi:dev-master"

## 2.文档：这里将会用dingo api和json web token来开发后台API

    （1）dingo api文档，https://github.com/dingo/api/wiki
    （2）jwt-auth文档，https://github.com/tymondesigns/jwt-auth/wiki

## 3.后端API配置

在config/app.php中加入

    'providers' => [
        //自己添加的laravel拓展
        Dingo\Api\Provider\LaravelServiceProvider::class,
        Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class
    ],
    'aliases' => [
        'APIRoute'=>Dingo\Api\Facade\Route::class,                //dingo
        'API'=>Dingo\Api\Facade\API::class,                       //dingo
        'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class,        //jwt认证
        'JWTFactory' => Tymon\JWTAuth\Facades\JWTFactory::class,  //jwt认证
    ],

执行

    php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"

执行完后config目录下就会自动生成api.php和jwt.php，接下来执行php artisan jwt:generate生成JWT_SECRET，最后在.env文件中写入

    API_STANDARDS_TREE=vnd
    API_SUBTYPE=myapp
    API_PREFIX=api
    #API_DOMAIN=api.cms.com
    API_VERSION=v1
    API_DEBUG=true

在app/Http/Kernel.php中加入

    protected $routeMiddleware = [
        //自己添加的路由中间件
        'jwt.auth'=> \Tymon\JWTAuth\Middleware\GetUserFromToken::class,
        'jwt.refresh'=>\Tymon\JWTAuth\Middleware\RefreshToken::class,
    ];

jwt-auth依赖于user表，也就是database/migrations/2014_10_12_000000_create_users_table.php，执行php artisan migrate生成user表，并确保app/User.php存在，然后类中加入protected $table = 'user';，不然声明使用user表，不然会默认使用users表.

打开/routes/api.php文件

    （1）引入dingo api，$api = app('Dingo\Api\Routing\Router');
    （2）创建API版本
    //开始定义api的接口路由
    $api = app('Dingo\Api\Routing\Router');
    $api->version('v1',function ($api){
        $api->group(['namespace' => 'App\Http\Controllers\Api','middleware'=>'jwt.auth'], function($api){
            $api->get('test','TestController@index');
        });
    });

创建相应的controller和action

    //文件路径：app/Http/Controllers/ApiController.php
    /* 定义一个api使用的公共模块 */
    namespace App\Http\Controllers;
    use Dingo\Api\Routing\Helpers;
    use Illuminate\Routing\Controller;

    class ApiController extends Controller{
        use Helpers;
    }


//文件路径：app/Http/Controllers/Api/TestController.php

    namespace App\Http\Controllers\Api;   
    use App\Http\Controllers\ApiController; //引用自己定义的api公共类
    use Illuminate\Http\Request;            //接收传参使用 function(Request $request,$id=1)
    use Illuminate\Support\Facades\Input;   //接收传参使用 $someInput = Input::all();

    class TestController extends ApiController {

        //获取get过来的用户id
        public function index(Request $request,$id=1){
            // $users = DB::select('select * from m_userinfo where id = ?', [$id]);
            //print_r($users);
            //$someInput = $request->input('id');
            $someInput = Input::all();
            var_dump($someInput);
        }
    }

执行php aritsan api:routes