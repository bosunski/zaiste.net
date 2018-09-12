---
created_at: 2018-09-11
kind: article
publish: true
title: "How to use multiple callbacks in Laravel Socialite"
image: "nodejs-elasticsearch.jpg"
tags:
- Laravel
- Laravel Socialite
- PHP
- OAuth 2
sitemap:
  priority: 0.8
abstract: >
 Laravel Socialite examples and tutorials, most times, shows how to create a login using just a a callback,
 In this article I will show you how you can have multiple callbacks that are dynamic.
---
![How to use multiple callbacks in Laravel Socialite][1]
## The Problem
[Laravel Socialite][2] examples and tutorials, most times, shows how to create a login using just a a callback.
As much as this is good, it comes with some limitations, for example, if you needed to created different callbacks to process information differently.
A good analogy is when I want to have a different callback to handle registration and a different one to handle login. Your own reason can be different but the goal
is to be able to use as many callback URIs as possible.

## What you'll need
I believe whoever is reading this tutorial will know a think or two about what Laravel Socialite is and might have used it one way or the other before.
I Assume you have Laravel Socialite set up and ready to use in your laravel installation. I'm also assuming you already have a sample implementation of Laravel Socialite
which is what we want to decouple to be able to use dynamic callbacks.

## An example of the problem
I will like to start by showing the popular way we use Laravel Socialite and then we can see how to improve this to use dynamic callbacks.

We have our credentials like this inside `.env`:
```dotenv
FACEBOOK_CLIENT_ID=693724864165765
FACEBOOK_CLIENT_SECRET=90d9ddf7dd76aa7a3dfac90b433c4273
FACEBOOK_CLIENT_CALLBACK=/login/facebook/callback
```

Then we'll configure `config/services.php` like this:

```js
return [
	//...
	'facebook' => [
			'client	_id' => env('FACEBOOK_CLIENT_ID'),
			'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
			'redirect' => env('FACEBOOK_CLIENT_CALLBACK'),
		],
	// ...
];
```

And somewhere in our code we'll have to do this to redirect to provider:

```js
// SocialLoginController.php
...
public function redirectToProvider($provider)
{
	Socialite::driver($provider)->redirect();
}
...
```

And then in our callback we'll have something like this to retrieve the user:

```js
// SocialLoginController.php
...
public function handleProviderCallback()
{
	$user = Socialite::driver($provider)->user();
	
	return $user;
}	
...
```

And lastly we'll have our routes set up like this:

```js
// routes.php
Route::get('login/{provider}', 'SocialLoginController@redirectToProvider')->name('socialLogin');
Route::get('login/{provider}/callback', 'SocialLoginController@handleProviderCallback')->name('socialLoginCallback');
```
At this point our callback URI can only be used by the Login route and the limitation actually comes when what we want have a different callback for Registration,
this might be because you want to perform some other things that are different from a login process.

For example you might want to verify the user'd email after registration which means you need a different call back when registering.

## The little solution
In order for us to work around this in a neat way, we'll start by updating the routes:

```js

// routes.php
Route::get('login/{provider}', 'SocialLoginController@redirectToProvider')->name('socialLogin');
Route::get('login/{provider}/callback', 'SocialLoginController@handleProviderCallback')->name('socialLoginCallback');

Route::get('register/{provider}', 'SocialRegisterationController@redirectToProvider')->name('socialRegister');
Route::get('register/{provider}/callback', 'SocialRegisterationController@handleProviderCallback')->name('socialRegisterCallback');
```

In above, I've added 2 more routes similar to that of the login routes. So, how do we make this happen?

#### 1. I removed the individual configuration that I have inside
This means I removed this:
```js
	// config/services.php
	'facebook' => [
			'client	_id' => env('FACEBOOK_CLIENT_ID'),
			'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
			'redirect' => env('FACEBOOK_CLIENT_CALLBACK'),
		],
	// ...
```


#### 2. Build Providers dynamically


The technique here is that instead of we allowing `Socialite` to build the provider with the default configuration, we will build them ourselves and pass our own configuration to it.

For me to do this in a clean way I created a `CanBuildSocialProvider` [trait][3] that can be used anywhere you intend to use socialite to make changing of already existing code easy.
Here's the contents:

```js
namespace App\Traits;

use Laravel\Socialite\Facades\Socialite;
use Laravel\Socialite\Two\ProviderInterface;

trait CanBuildSocialProvider {

	/**
	 * Creates an Instance of a Provider with config
	 *
	 * @param string|null $redirectUrl
	 *
	 * @return ProviderInterface
	 */
	public function buildSocialProvider(string $redirectUrl = null): ProviderInterface
	{
		$providerClass = ucfirst($this->provider);
		$provider = strtoupper($this->provider);
		$config = [
			'client_id' => env($provider . '_CLIENT_ID'),
			'client_secret' => env($provider. '_CLIENT_SECRET'),
			'redirect' => $redirectUrl,
		];

		return Socialite::buildProvider('\Laravel\Socialite\Two\\' . $providerClass . 'Provider', $config);
	}
}
```

In above we use the `buildProvider` method to create the provider and pass the `$config` that holds our configuration to it. Dynamically we'll be passing our call back URIs to the method `buildSocialProvider` to include in the `$config` array.

#### 3. Use the trait

The next thing is to use the trait anywhere I to use socialite and updated the codes to this:

```js
// SocialLoginController.php
namespace App\Http\Controller;

use App\Traits\CanBuildSocialProvider;

class SocialLoginController extends Controller
{
    use CanbuildSocialProvider;
    
    public function redirectToProvider($provider)
    {
        $this->provider = $provider;
        $redirectUrl = '/login/facebook/callback';
    	return $this->buildSocialProvider($redirectUrl)->redirect();
    }
    
    public function handleProviderCallback($provider)
	{
		$this->provider = $provider;
		$redirectUrl = '/login/facebook/callback';
		$user = $this->buildSocialProvider($redirectUrl)->user();
		
		// Do login processing
	}
	

}
```

And then I can also repeat same for the the registration like this:

```js
// SocialRegistrationController.php
namespace App\Http\Controller;

use App\Traits\CanBuildSocialProvider;

class SocialLoginController extends Controller
{
    use CanbuildSocialProvider;
    
    public function redirectToProvider($provider)
    {
        $this->provider = $provider;
        $redirectUrl = '/register/facebook/callback';
    	return $this->buildSocialProvider($redirectUrl)->redirect();
    }
    
    public function handleProviderCallback($provider)
	{
		$this->provider = $provider;
		$redirectUrl = '/register/facebook/callback';
		$user = $this->buildSocialProvider($redirectUrl)->user();
		
		// Do registration processing
	}

```

You can observe the different `$redirectUrl` passed to the `buildSocialProvider` method.

#### 4. Add new callback URIs to the Facebook App settings


I went to the Facebook App I created on the developer website and added the new callback URL so tht it'll be authorized.

## Conclusion

That's the little way I used to solve that problem, If you have a better way you've used to solve the problem, please share in the comment.

[1]: http://res.cloudinary.com/xeviant/image/upload/v1536701422/laravel-socialite_qkeyl1.png
[2]: https://laravel.com/docs/5.7/socialite
[3]: http://php.net/manual/en/language.oop5.traits.php