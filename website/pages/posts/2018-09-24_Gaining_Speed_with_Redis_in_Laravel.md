---
created_at: 2018-09-24
kind: article
publish: true
title: "Gaining Speed with Redis in Laravel"
image: ""
tags:
- Redis
- Laravel
- Caching
- Speed
- Performance
- Optimization
sitemap:
  priority: 0.8
abstract: >
  In this tutorial I will be showing how to build a JavaScript application on top of Elasticsearch. Its core will be written in Node.js followed by Vue.js on the frontend. We will be using modern JavaScript specifications (ES6+) with features such as `async/await`, spread operator or destructuring assignment.
---
## Initial State
This is my initial state of the app:
![Initial State][2]
## Installing Redis on the Machine
First we'll install redis on our machine. 
For an Ubuntu Machine we can do:
```
sudo apt-get install redis-server
```
On MAC  you can follow this [article][1] to install and configure Redis through Homebrew. 

## Installing Predis in your Laravel Project
```
composer require predis/predis
```

## Caching Technique
The basic technique behind caching is that we'll load the Cached version of a data if its available otherwise load
a fresh one from the database, when this happens it helps in reducing database queries upon requests and views rendered.
The last part of the technique is that the Cached version of the data is updated whether by scheduling or based on events within the app.

Based on scheduling we can hourly or daily, be updating the cached posts and based on events, we can update cached posts immediately a new post is added.

## Configuring Redis with Laravel
At this point we need to update the `.env` file to use Redis as the default caching driver, like this:
```
[...]
CACHE_DRIVER=redis
[...]
```
## Confirming if Redis is active
If all things are set up for redis, running this in the terminal:
```
$ redis-benchmark -q -n 1000 -c 10 -P 5
```
Will produce something like this:
```                         
PING_INLINE: 27777.78 requests per second
PING_BULK: 249999.98 requests per second
SET: 142857.14 requests per second
GET: 166666.67 requests per second
INCR: 142857.14 requests per second
LPUSH: 124999.99 requests per second
RPUSH: 111111.12 requests per second
LPOP: 166666.67 requests per second
RPOP: 90909.09 requests per second
SADD: 166666.67 requests per second
HSET: 90909.09 requests per second
SPOP: 166666.67 requests per second
LPUSH (needed to benchmark LRANGE): 166666.67 requests per second
LRANGE_100 (first 100 elements): 19230.77 requests per second
LRANGE_300 (first 300 elements): 7751.94 requests per second
LRANGE_500 (first 450 elements): 4524.89 requests per second
LRANGE_600 (first 600 elements): 3861.00 requests per second
MSET (10 keys): 66666.67 requests per second
```
 So, if you get something like that, it means you're good to go.
 
 If you run `redis-cli ping` in your terminal, you should get `PONG` as response!
 
 ## Using Redis
 Here is an Example implementation on how Redis can be used:
 ```js
 <?php
 namespace App\Services\Client\Features;
 
 use App\Domains\Budget\Jobs\GetBudgetHistoryDataJob;
 use App\Domains\Http\Jobs\RespondWithViewJob;
 use Illuminate\Support\Facades\Auth;
 use Illuminate\Support\Facades\Cache;
 use Illuminate\Support\Facades\Redis;
 use Lucid\Foundation\Feature;
 use Illuminate\Http\Request;
 
 class DisplayHistoryFeature extends Feature
 {
     public function handle(Request $request)
     {
 		// We check if current request is cached
     	$view = Redis::get(Auth::user()->id . '-history');
 
     	if ($view != null) {
    		return $view;
 	    }
 
 	    // If it's not, we retrieve Data from Database
 	    $pageData = $this->run(GetBudgetHistoryDataJob::class);
 
     	// We'll render the HTML View
 	    $view = $this->run(new RespondWithViewJob('client::dashboard.budget-history', $pageData))->getContent();
 
 	    // We'll Cache the HTML View
 	    $this->putInCache(Auth::user()->id . '-history', $view, 'budget-history');
 
 	    // We'll display it
 	    return $view;
     }
 
 	private function putInCache($key, $content)
 	{
 		Redis::set($key, $content);
 	}
 }
```

And here's the final Result here:
![Final Result][2]


[1]: https://medium.com/@petehouston/install-and-config-redis-on-mac-os-x-via-homebrew-eb8df9a4f298
[2]: https://res.cloudinary.com/xeviant/image/upload/v1537829735/Screenshot_2018-09-24_at_11.46.57_PM_pztdxg.png
[3]: https://res.cloudinary.com/xeviant/image/upload/v1537829734/Screenshot_2018-09-24_at_11.50.01_PM_cmc9px.png