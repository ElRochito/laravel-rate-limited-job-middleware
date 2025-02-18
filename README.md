# A job middleware to rate limit jobs

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/laravel-rate-limited-job-middleware.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-rate-limited-job-middleware)
[![Build Status](https://img.shields.io/travis/spatie/laravel-rate-limited-job-middleware/master.svg?style=flat-square)](https://travis-ci.org/spatie/laravel-rate-limited-job-middleware)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/laravel-rate-limited-job-middleware.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/laravel-rate-limited-job-middleware)
[![StyleCI](https://github.styleci.io/repos/211561705/shield?branch=master)](https://github.styleci.io/repos/211561705)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/laravel-rate-limited-job-middleware.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-rate-limited-job-middleware)

This package contains a [job middleware](https://laravel.com/docs/master/queues#job-middleware) that can rate limit jobs in Laravel apps.

## Installation

You can install the package via composer:

```bash
composer require spatie/laravel-rate-limited-job-middleware
```

This package requires Redis to be set up in your Laravel app.

## Usage

By default, the middleware will only allow 5 jobs to be executed per second. Any jobs that are not allowed will be released for 5 seconds. 

To apply the middleware just add the `Spatie\RateLimitedMiddleware\RateLimited` to the middlewares of your job.

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Spatie\RateLimitedMiddleware\RateLimited;

class TestJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable;

    public function handle()
    {
        // your job logic
    }

    public function middleware()
    {
        return [new RateLimited()];
    }
}
```

### Configuring attempts

When using rate limiting, the number of attempts of your job may be hard to predict. Instead of using a fixed number of attempts, it's better to use [time based attempts](https://laravel.com/docs/master/queues#time-based-attempts).

You can add this to your job class:

```php
/*
 * Determine the time at which the job should timeout.
 *
 */
public function retryUntil() :  \DateTime
{
    return now()->addDay();
}
```

### Customizing the behaviour

You can customize all the behaviour. Here's an example where the middleware allows a maximum of 30 jobs to performed in a timespan off 60 seconds. Jobs that are not allowed will be released for 90 seconds.

```php
// in your job

public function middleware()
{
    $rateLimitedMiddleware = (new RateLimited())
        ->allow(30)
        ->everySeconds(60)
        ->releaseAfterSeconds(90);

    return [$rateLimitedMiddleware];
}
```

### Customizing Redis

By default, the middleware will use the default Redis connection. 

The default key that will be used in redis will be the name of the class that created the instance of the middleware. In most cases this will be name of job in which the middleware is applied. If this is not what you expect, you can use the `key` method to customize it. 
 


Here's an example where a custom connection and custom key is used.

```php
// in your job

public function middleware()
{
    $rateLimitedMiddleware = (new RateLimited())
        ->connection('my-custom-connection')
        ->key('my-custm-key');

    return [$rateLimitedMiddleware];
}
```

### Conditionally applying the middleware

If you want to conditionally apply the middleware you can use the `enabled` method. If accepts a boolean that determines if the middleware should rate limit your job or not.

You can also pass a `Closure` to `enabled`. If it evaluates to a truthy value the middleware will be enable.

Here's a silly example where the rate limiting is only activated in January.

```php
// in your job

public function middleware()
{
    $shouldRateLimitJobs = Carbon::now()->month === 1;

    $rateLimitedMiddleware = (new RateLimited())
        ->enabled($shouldRateLimitJobs);

    return [$rateLimitedMiddleware];
}
```

### Available methods.

These methods are available to be called on the middleware. Their names should be self-explanatory.

- `allow(int $allowedNumberOfJobsInTimeSpan)`
- `everySecond(int $timespanInSeconds = 1)`
- `everySeconds(int $timespanInSeconds)`
- `everyMinute(int $timespanInMinutes = 1)`
- `everyMinutes(int $timespanInMinutes)`
- `releaseAfterOneSecond()`
- `releaseAfterSeconds(int $releaseInSeconds)`
- `releaseAfterOneMinute()`
- `releaseAfterMinutes(int $releaseInMinutes)`
- `releaseAfter(callable $releaseAfter)`

### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)

This code is heavily based on [the rate limiting example](https://laravel.com/docs/master/queues#job-middleware) found in the Laravel docs.

## Support us

Spatie is a webdesign agency based in Antwerp, Belgium. You'll find an overview of all our open source projects [on our website](https://spatie.be/opensource).

Does your business depend on our contributions? Reach out and support us on [Patreon](https://www.patreon.com/spatie). 
All pledges will be dedicated to allocating workforce on maintenance and new awesome stuff.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
