# Socialite Login with Laravel project

Chúng ta đều biết, chúng ta đang sống trong thời đại "Cách mang công nghệ 4.0", thời đại mà internet ngày càng phát triển, con người có thể kết nối với nhau dễ ràng và tiện lợi. Thời đại mà khái niệm mạng xã hội không còn quá xa lạ với mỗi người dân nữa. Facebook, twitter, instagram, youtube, ... Trở lên quá đỗi thân thuộc với chúng ta hàng ngày, hàng giờ. 

Là một nhà lập trình, đặc biệt là lập trình web, để bắt kịp xu hướng của thời đại, thì trang web của chúng ta làm ra cũng phải sống động, thì mới thu hút được user, đặc biệt, trang web của chúng ta cũng nên có chức năng liên kết với các mạng xã hội. Các bạn có thể thấy, vào bất cứ các trang web nào hiện nay, bên cạnh chức năng đăng ký đăng nhập cổ điển (nhập username, password, ...) thì đều hỗ trợ việc đăng nhập nhanh thông qua facebook, twitter google, ...

Nếu bạn sử dụng framework Laravel, thì trong Laravel cũng đã hỗ trợ chúng ta package Socialite để giúp bạn nhanh chóng tạo được chức năng đăng nhập thông qua các mạng xã hội. Trong bài viết này, tôi xin tổng hợp lại cách bạn có thể tích hợp chức năng đăng nhập thông qua Facebook, twitter. Mong quý bạn đọc gần xa, có thể tham khảo và góp ý cho bài viết này. 

# Setup

## Step 1:

Trước tiên, để bắt đầu project, chúng ta cần dựng framework Laravel lên môi trường phát triển của mình

```
composer create-project laravel/laravel socialLogin --prefer-dist
```

Câu lệnh trên, sẽ giúp bạn tạo một project Laravel với tên 'socialLogin'. Sau khi câu lệnh chạy xong, config database trong file .env của bạn và chạy

```
php artisan migrate
```
Mọi việc hoàn thành, thì hãy thử start server bằng lệnh

```
php artisan serve
```

Để kiểm tra việc cài đặt, hãy vào đường dẫn (http://localhost:8000)[http://localhost:8000]
Nếu việc cài đặt hoàn tất, thì bạn sẽ thấy trang welcome của Laravel hiện lên, nếu bạn k thấy trang thì hãy kiểm tra lại nhé

## Step 2: Create a Laravel Authentication

Laravel cung cấp sẵn cho chúng ta package authentication, để sử dụng nó, bạn chỉ cần chạy lệnh

```
php artisan make:auth
```

Trong file routes > web.php, các bạn thêm route của package auth vào như sau

```
<?php 

// web.php

Auth::routes();
```

Để vào page longin, truy cập vào URL (http://localhost:8000/login)[http://localhost:8000/login]

## Step 3: Install laravel/socialite package

Để tích hợp được package trên vào trong project, các bạn làm theo step sau:

```
composer require laravel/socialite
```
Với việc sử dụng công cụ quản lý các thư viện bằng composer, chúng ta có thể dễ dàng thêm mới 1 package vào project. Sau khi thêm thành công. Hãy khai báo trong providers của file config/app.php

```
<?php

// app.php

'providers' => [
    // Other service providers...

    Laravel\Socialite\SocialiteServiceProvider::class,
],
```

Nhớ là update alias array nữa nhé

```
<?php

// app.php

'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

# Facebook Login

## Step 1: Create Facebook app

Để tạo được 1 facebook app, bạn truy cập vào đường link sau (https://developers.facebook.com/)[https://developers.facebook.com/]. Login thông qua tài khoản facebook của bạn, trên màn hình hiện lên hãy điền vào 'Display Name' và 'Contack Email' sau đó ấn 'Create App ID'. Sau khi hoàn tất, các bạn sẽ thể được màn Dashboard. Vào phần 'Cài đặt' để lấy được App ID và App Sercret

Tiếp theo, hãy thêm vào file config  >>  services.php

```
<?php

// services.php

'facebook' => [
    'client_id' => 'xxxxxxx',
    'client_secret' => 'xxxxxxxx',
    'redirect' => '',
],
```

## Step 2: Generage Controller and router

```
php artisan make:controller SocialAuthFacebookController
```

Câu lệnh trên, sẽ tạo cho chúng ta 1 controller

```
<?php

// SocialAuthFacebookController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Socialite;

class SocialAuthFacebookController extends Controller
{
  /**
   * Create a redirect method to facebook api.
   *
   * @return void
   */
    public function redirect()
    {
        return Socialite::driver('facebook')->redirect();
    }

    /**
     * Return a callback method from facebook api.
     *
     * @return callback URL from facebook
     */
    public function callback()
    {
       
    }
}
```
Ở đây, chúng ta có định nghĩa thêm 2 method

- redirect(): Redirect user tới Facebook
- callback(): Handle callback từ Facebook gọi về 

Tiếp theo, vào file routes >> web.php, và khai báo routes

```
<?php

//web.php

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});

Auth::routes();

Route::get('/redirect', 'SocialAuthFacebookController@redirect');
Route::get('/callback', 'SocialAuthFacebookController@callback');

Route::get('/home', 'HomeController@index')->name('home');
```

Cuối cùng là vào file config  >>  services.php, và định nghĩa url để callback

```
<?php

// services.php

'facebook' => [
    'client_id' => 'xxxxxxx',
    'client_secret' => 'xxxxxxx',
    'redirect' => 'http://localhost:8000/callback',
],
```

## Step 3: Create template and view
Vào file resources  >>  views  >>  auth  >>  login.blade.php và sửa code thành nh

```
<!-- login.blade.php -->

@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-8 col-md-offset-2">
            <div class="panel panel-default">
                <div class="panel-heading">Login</div>
                <div class="panel-body">
                    <form class="form-horizontal" method="POST" action="{{ route('login') }}">
                        {{ csrf_field() }}

                        <div class="form-group{{ $errors->has('email') ? ' has-error' : '' }}">
                            <label for="email" class="col-md-4 control-label">E-Mail Address</label>

                            <div class="col-md-6">
                                <input id="email" type="email" class="form-control" name="email" value="{{ old('email') }}" required autofocus>

                                @if ($errors->has('email'))
                                    <span class="help-block">
                                        <strong>{{ $errors->first('email') }}</strong>
                                    </span>
                                @endif
                            </div>
                        </div>

                        <div class="form-group{{ $errors->has('password') ? ' has-error' : '' }}">
                            <label for="password" class="col-md-4 control-label">Password</label>

                            <div class="col-md-6">
                                <input id="password" type="password" class="form-control" name="password" required>

                                @if ($errors->has('password'))
                                    <span class="help-block">
                                        <strong>{{ $errors->first('password') }}</strong>
                                    </span>
                                @endif
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-md-6 col-md-offset-4">
                                <div class="checkbox">
                                    <label>
                                        <input type="checkbox" name="remember" {{ old('remember') ? 'checked' : '' }}> Remember Me
                                    </label>
                                </div>
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-md-8 col-md-offset-4">
                                <button type="submit" class="btn btn-primary">
                                    Login
                                </button>

                                <a class="btn btn-link" href="{{ route('password.request') }}">
                                    Forgot Your Password?
                                </a>
                            </div>
                        </div>
                        <br />
                        <p style="margin-left:265px">OR</p>
                        <br />
                        <div class="form-group">
                            <div class="col-md-8 col-md-offset-4">
                              <a href="{{url('/redirect')}}" class="btn btn-primary">Login with Facebook</a>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

## Step 4: Tạo modal cho Social Account

Run câu lệnh sau

```
php artisan make:model SocialFacebookAccount -m
```
Thêm đoạn code  sau vào file migrate vừa được sinh ra, create_social_facebook_accounts_table.php

```
<?php

// create_social_facebook_accounts.php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateSocialFacebookAccountsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('social_facebook_accounts', function (Blueprint $table) {
          $table->integer('user_id');
          $table->string('provider_user_id');
          $table->string('provider');
          $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('social_facebook_accounts');
    }
}
```
Chạy lệnh migrate để tạo tables mới

```
php artisan migrate
```

## Step 5: Khai báo relation cho SocialFacebookAccount vs User

```
<?php

// SocialFacebookAccount.php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SocialFacebookAccount extends Model
{
  protected $fillable = ['user_id', 'provider_user_id', 'provider'];

  public function user()
  {
      return $this->belongsTo(User::class);
  }
}
```

Tiếp theo, tạo 1 service để xử lý logic

```
<?php

namespace App\Services;
use App\SocialFacebookAccount;
use App\User;
use Laravel\Socialite\Contracts\User as ProviderUser;

class SocialFacebookAccountService
{
    public function createOrGetUser(ProviderUser $providerUser)
    {
        $account = SocialFacebookAccount::whereProvider('facebook')
            ->whereProviderUserId($providerUser->getId())
            ->first();

        if ($account) {
            return $account->user;
        } else {

            $account = new SocialFacebookAccount([
                'provider_user_id' => $providerUser->getId(),
                'provider' => 'facebook'
            ]);

            $user = User::whereEmail($providerUser->getEmail())->first();

            if (!$user) {

                $user = User::create([
                    'email' => $providerUser->getEmail(),
                    'name' => $providerUser->getName(),
                    'password' => md5(rand(1,10000)),
                ]);
            }

            $account->user()->associate($user);
            $account->save();

            return $user;
        }
    }
}
```

## Step 5: Update callback function

```
<?php

// SocialAuthFacebookController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Socialite;
use App\Services\SocialFacebookAccountService;

class SocialAuthFacebookController extends Controller
{
  /**
   * Create a redirect method to facebook api.
   *
   * @return void
   */
    public function redirect()
    {
        return Socialite::driver('facebook')->redirect();
    }

    /**
     * Return a callback method from facebook api.
     *
     * @return callback URL from facebook
     */
    public function callback(SocialFacebookAccountService $service)
    {
        $user = $service->createOrGetUser(Socialite::driver('facebook')->user());
        auth()->login($user);
        return redirect()->to('/home');
    }
}
```

Đến đây, công việc đã hoàn thành, hãy thử test lại trên page login v
