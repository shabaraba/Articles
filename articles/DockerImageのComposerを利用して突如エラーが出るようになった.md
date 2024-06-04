# DockerImageのComposerを利用して突如エラーが出るようになった

Created: 2022年3月28日 8:53
Updated: 2023年1月1日 14:05
Published_Time: 2022年3月24日
Slug: error-in-composer-docker-image
Tags: Composer, Docker, PHP
Published: Yes
URL: https://blog.shaba.dev/posts/error-in-composer-docker-image

## 何が起きたか

laravelのプロジェクトを開発する際に、laravelのapp用のコンテナとcomposer用のコンテナを用意してdocker-composeで環境を起動するのが僕の主流になっているのですが、
つい先日久しぶりに環境を立ち上げたら

```bash
$ docker-compose run composer composer install
Creating laravel-environment_composer_run ... done
You are using Composer 1 which is deprecated. You should upgrade to Composer 2, see https://blog.packagist.com/deprecating-composer-1-support/
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
Nothing to install or update
Package swiftmailer/swiftmailer is abandoned, you should avoid using it. Use symfony/mailer instead.
Generating optimized autoload files
Deprecation Notice: preg_replace(): Passing null to parameter #3 ($subject) of type array|string is deprecated in phar:///usr/bin/composer/src/Composer/Autoload/ClassMapGenerator.php:257
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
Deprecation Notice: Return type of Illuminate\Container\Container::offsetExists($key) should either be compatible with ArrayAccess::offsetExists(mixed $offset): bool, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php:1231
Deprecation Notice: Return type of Illuminate\Container\Container::offsetGet($key) should either be compatible with ArrayAccess::offsetGet(mixed $offset): mixed, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php:1242
Deprecation Notice: Return type of Illuminate\Container\Container::offsetSet($key, $value) should either be compatible with ArrayAccess::offsetSet(mixed $offset, mixed $value): void, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php:1254
Deprecation Notice: Return type of Illuminate\Container\Container::offsetUnset($key) should either be compatible with ArrayAccess::offsetUnset(mixed $offset): void, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php:1267
> @php artisan package:discover --ansi

Deprecated: Return type of Illuminate\Container\Container::offsetExists($key) should either be compatible with ArrayAccess::offsetExists(mixed $offset): bool, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php on line 1231

Deprecated: Return type of Illuminate\Container\Container::offsetGet($key) should either be compatible with ArrayAccess::offsetGet(mixed $offset): mixed, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php on line 1242

Deprecated: Return type of Illuminate\Container\Container::offsetSet($key, $value) should either be compatible with ArrayAccess::offsetSet(mixed $offset, mixed $value): void, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php on line 1254

Deprecated: Return type of Illuminate\Container\Container::offsetUnset($key) should either be compatible with ArrayAccess::offsetUnset(mixed $offset): void, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Container/Container.php on line 1267

Deprecated: Return type of Illuminate\Config\Repository::offsetExists($key) should either be compatible with ArrayAccess::offsetExists(mixed $offset): bool, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Config/Repository.php on line 141

Deprecated: Return type of Illuminate\Config\Repository::offsetGet($key) should either be compatible with ArrayAccess::offsetGet(mixed $offset): mixed, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Config/Repository.php on line 152

Deprecated: Return type of Illuminate\Config\Repository::offsetSet($key, $value) should either be compatible with ArrayAccess::offsetSet(mixed $offset, mixed $value): void, or the #[\ReturnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Config/Repository.php on line 164

Deprecated: Return type of Illuminate\Config\Repository::offsetUnset($key) should either be compatible with ArrayAccess::offsetUnset(mixed $offset): void, or the #[\Re
turnTypeWillChange] attribute should be used to temporarily suppress the notice in /app/vendor/laravel/framework/src/Illuminate/Config/Repository.php on line 175

In Collection.php line 11:

  During inheritance of ArrayAccess: Uncaught ErrorException: Return type of
  Illuminate\Support\Collection::offsetExists($key) should either be compatib
  le with ArrayAccess::offsetExists(mixed $offset): bool, or the #[\ReturnTyp
  eWillChange] attribute should be used to temporarily suppress the notice in
   /app/vendor/laravel/framework/src/Illuminate/Support/Collection.php:1277
  Stack trace:
  #0 /app/vendor/laravel/framework/src/Illuminate/Support/Collection.php(11):
   Illuminate\Foundation\Bootstrap\HandleExceptions->handleError(8192, 'Retur
  n type of ...', '/app/vendor/lar...', 1277)
  #1 /app/vendor/composer/ClassLoader.php(444): include('/app/vendor/lar...')
  #2 /app/vendor/composer/ClassLoader.php(322): Composer\Autoload\includeFile
  ('/app/vendor/com...')
  #3 /app/vendor/laravel/framework/src/Illuminate/Support/helpers.php(109): C
  omposer\Autoload\ClassLoader->loadClass('Illuminate\\Supp...')
  #4 /app/vendor/laravel/framework/src/Illuminate/Foundation/PackageManifest.
  php(130): collect(Array)
  #5 /app/vendor/laravel/framework/src/Illuminate/Foundation/PackageManifest.
  php(106): Illuminate\Foundation\PackageManifest->build()
  #6 /app/vendor/laravel/framework/src/Illuminate/Foundation/PackageManifest.
  php(89): Illuminate\Foundation\PackageManifest->getManifest()
  #7 /app/vendor/laravel/framework/src/Illuminate/Foundation/PackageManifest.
  php(78): Illuminate\Foundation\PackageManifest->config('aliases')
  #8 /app/vendor/laravel/framework/src/Illuminate/Foundation/Bootstrap/Regist
  erFacades.php(26): Illuminate\Foundation\PackageManifest->aliases()
  #9 /app/vendor/laravel/framework/src/Illuminate/Foundation/Application.php(
  219): Illuminate\Foundation\Bootstrap\RegisterFacades->bootstrap(Object(Ill
  uminate\Foundation\Application))
  #10 /app/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.
  php(320): Illuminate\Foundation\Application->bootstrapWith(Array)
  #11 /app/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.
  php(129): Illuminate\Foundation\Console\Kernel->bootstrap()
  #12 /app/artisan(37): Illuminate\Foundation\Console\Kernel->handle(Object(S
  ymfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console
  \Output\ConsoleOutput))
  #13 {main}

Script @php artisan package:discover --ansi handling the post-autoload-dump event returned with error code 255
ERROR: 255
```

え？そんな怒る？

## ComposerイメージのPHPバージョンアップが原因

調べていくと下記qiita記事に出くわしました。

[https://qiita.com/t_arase/items/817baa417d05590a4f38](https://qiita.com/t_arase/items/817baa417d05590a4f38)

ComposerイメージはPHPを内包してくれているので、単体で動作できるようになっているスグレモノなのですが、
最新のComposerイメージはPHP8.1を利用するようになっていました。

一方で今回立ち上げようとしていたlaravelのバージョンはlaravel6で、PHP8.1に対応していません。

バージョンのミスマッチに依るエラーでした。

## 利用するComposerイメージを下げることで解決

上記記事の解決策に倣い、Composerイメージを`composer:2.1.11`に下げました。

```yaml
composer:
      image: composer:2.1.11 # <- これ
      container_name: composer
      command: "composer install"
      volumes:
        - ./src:/app
```

改めて。

```bash
$ docker-compose run composer
Pulling composer (composer:2.1.11)...
2.1.11: Pulling from library/composer
a0d0a0d46f8b: Pull complete
153eea49496a: Pull complete
11efd0df1fcb: Pull complete
b3f3214c344d: Pull complete
b030db4cf62f: Pull complete
5add8d0d1091: Pull complete
4061c20798a1: Pull complete
af9c648d2e05: Pull complete
fdb1a2e635da: Pull complete
352470c7dadc: Pull complete
326449c1eb9e: Pull complete
2ca1e8fa6534: Pull complete
dc2a785bae4d: Pull complete
03e7e39b8e98: Pull complete
Digest: sha256:cf977c927084dbb563ea8def55ffd317af13c52b23152b2247c13a67824d729f
Status: Downloaded newer image for composer:2.1.11
Creating laravel-environment_composer_run ... done
Installing dependencies from lock file (including require-dev)
Verifying lock file contents can be installed on current platform.
Nothing to install, update or remove
Package swiftmailer/swiftmailer is abandoned, you should avoid using it. Use symfony/mailer instead.
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
68 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

環境構築は難しいですね。