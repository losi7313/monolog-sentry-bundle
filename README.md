# Monolog Sentry Bundle

[![Latest Stable Version](https://poser.pugx.org/dziki/monolog-sentry-bundle/v/stable)](https://packagist.org/packages/dziki/monolog-sentry-bundle)
[![Build Status](https://travis-ci.org/mleczakm/monolog-sentry-bundle.svg?branch=master)](https://travis-ci.org/mleczakm/monolog-sentry-bundle)
[![Coverage Status](https://coveralls.io/repos/github/mleczakm/monolog-sentry-bundle/badge.svg?branch=master)](https://coveralls.io/github/mleczakm/monolog-sentry-bundle?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/mleczakm/monolog-sentry-bundle/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/mleczakm/monolog-sentry-bundle/?branch=master)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/539b5154-ad2a-4417-bbea-dc13a6f69c0c/mini.png)](https://insight.sensiolabs.com/projects/539b5154-ad2a-4417-bbea-dc13a6f69c0c)
[![License](https://poser.pugx.org/dziki/monolog-sentry-bundle/license)](https://packagist.org/packages/dziki/monolog-sentry-bundle)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fmleczakm%2Fmonolog-sentry-bundle.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fmleczakm%2Fmonolog-sentry-bundle?ref=badge_shield)

Bundle for appending useful data to [Monolog](https://github.com/Seldaek/monolog) log records like username, 
parsed user-agent header, host name, Symfony version, commit hash and a lot more - you can provide custom tags 
to be added to all your logs.

## Installation

Install bundle with `composer require dziki/monolog-sentry-bundle` command.

## TL;DR

Comparison of exactly same error handled by default [monolog raven handler](#hints) with `sentry/sentry` package client with bundle
turned off and on with some [basic config](#full-basic-config). As you can see - after turning bundle on - browser, user, 
breadcrumbs and some valuable tags showed up, making your error logs much easier to read.

### Before
![before](https://user-images.githubusercontent.com/3474636/45269343-d8c4d700-b48c-11e8-89b3-8a6a0e602c12.png)

## After
![after](https://user-images.githubusercontent.com/3474636/45269349-e1b5a880-b48c-11e8-9143-53058e67e757.png)

## Enable the Bundle

Add entry to `config/bundles.php`:

```php
return [
    // ...
    Dziki\MonologSentryBundle\MonologSentryBundle::class => ['all' => true],
];
```

or to `app/AppKernel.php`

```php
<?php // app/AppKernel.php

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Dziki\MonologSentryBundle\MonologSentryBundle(),
        );
        // ...
    }
    // ...
}
```

## Configuration

Default configuration does nothing, You need to adjust it manually according to Your needs:

```yaml
monolog_sentry:
    user_context: false # append username from TokenStorage to log
    user_agent_parser: false # set to 'phpuseragent' to parse browser name, version and platform from user agent
``` 

You can turn on logging user context by setting value to `true` - it requires `symfony/security-bundle` package.
`user_agent_parser` requires valid user agent header parser service as value. 
Parsing user agent takes about 0.1ms (up to 1ms using native parser) for every request, so...

## Caching once parsed User Agents

Caching is supported when service implementing `Psr\SimpleCache\CacheInterface` is provided under `cache` config entry.
Starting from version 4.1 of Symfony there is default simple cache service `cache.app.simple`, in previous versions you 
need to define own service:

```yaml
monolog_sentry:
    cache: cache.app.simple # service implementing "Psr\SimpleCache\CacheInterface" interface
``` 

## Custom tags

You can extend amount of logged data by adding custom tags. For example, for logging Symfony version, setting 
useful [Sentry environment](https://docs.sentry.io/learn/environments/) and server name you should modify config to this:

```yaml
monolog_sentry:
    ...
    tags:
        symfony_version: !php/const Symfony\Component\HttpKernel\Kernel::VERSION # useful for regression check
        commit: '%env(APP_REVISION)%' # for example hash of commit, set your own environment variable or parameter
        environment: '%env(SERVER_NAME)%' # Sentry environment discriminator, much more useful than default `prod`
```

## Full basic config

```yaml
monolog_sentry:
    user_context: true # append username from TokenStorage to log
    user_agent_parser: phpuseragent # parse browser name, version and platform from user agent
    cache: cache.app.simple # service implementing "Psr\SimpleCache\CacheInterface" interface, since SF 4.1
    tags:
        symfony_version: !php/const Symfony\Component\HttpKernel\Kernel::VERSION # useful for regression check
        commit: '%env(APP_REVISION)%' # for example hash of commit, set your own environment variable or parameter
        environment: '%env(SERVER_NAME)%' # Sentry environment discriminator, much more useful than default `prod`
```

## User Agent parser

Bundle supports two parsers:
- `phpuseragent` ([github.com/donatj/PhpUserAgent](https://github.com/donatj/PhpUserAgent)) as default, no config needed
- `native` ([get_browser()](https://php.net/manual/en/function.get-browser.php)) - browscap configuration setting in php.ini 
must point to the correct location of the [browscap.ini](https://browscap.org/)

Configurable through `user_agent_parser` value, respectively `phpuseragent` or `native`. You can also add own, by providing
name of service implementing [ParserInterface](https://github.com/mleczakm/monolog-sentry-bundle/blob/master/UserAgent/ParserInterface.php).

## Hints

- Add `stop_buffering: false` to your `fingers_crossed` handler to keep low level messages notifications as breadcrumbs:

```yaml
monolog:
    handlers:
        main:
            type:           fingers_crossed
            action_level:   error
            handler:        buffered
            stop_buffering: false
        sentry:
            type:    raven
            dsn:     '%env(SENTRY_DSN)%'
            level:   info # logs which will be shown as breadcrumbs in Sentry issue
            release: 1.0.0
```

- Add Sentry handler `release` option to monolog config for easy regression seeking:

```yaml
monolog:
    handlers:
        ...
        sentry:
            ...
            release: '%env(APP_VERSION)%' # version tag or any release ID
```

## License

MonologSentryBundle is released under the [MIT license](https://github.com/mleczakm/monolog-sentry-bundle/blob/master/LICENSE.md).


[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fmleczakm%2Fmonolog-sentry-bundle.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fmleczakm%2Fmonolog-sentry-bundle?ref=badge_large)