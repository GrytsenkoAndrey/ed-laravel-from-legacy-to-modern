# From Legacy to Modern: Migrating PHP 7.2 to 8.2 and Laravel 6 to 11

## Why Upgrade?

### Before we get into the migration process, let’s briefly discuss why the update is useful:

1. Performance Improvements: PHP 8.2 features significant performance improvements, including the Just-In-Time (JIT) compiler, which can speed up the execution of your PHP code.
2. Security improvements: The new versions of PHP and Laravel include important security patches and features that protect your application from vulnerabilities.
3. Modern Features: PHP 8.2 and Laravel 11 come with a number of new features and enhancements that can make your code more efficient and easier to maintain.
4. Community and Support: Using the latest versions provides better community support and access to up-to-date documentation and resources.

### Preparing for the Migration

1. Backup Your Application: Before making any changes, ensure you have a complete backup of your application and database. This allows you to revert to the previous state if anything goes wrong.
2. Review the Official Documentation: Familiarize yourself with the official migration guides for PHP and Laravel. These documents provide detailed information on changes, deprecated features, and new functionalities.
3. Review Packages Documentation: Check all third-party packages and dependencies to versions compatible with PHP 8.2 and Laravel 11.

## Upgrading PHP

Install PHP 8.2 on your development environment. Using Docker and its images you can do this very easily and quickly. I’ll take a ready-made PHP 8.2 image as an example.
```
FROM php:8.2-fpm-alpine
```

## Enable JIT Compilation

JIT compilation is not enabled by default. You need to configure PHP to use it. This is typically done through the php.ini configuration file.

Look for the opcache.jit_buffer_size and opcache.jit directives. Uncomment them if they exist, otherwise add them:

```
opcache.jit_buffer_size=100M
opcache.jit=tracing
```

## Dependencies and pain

After changing the PHP version, if you try to run the composer install command you will get VERY many errors. Follow these steps first:

1. Delete the vendor folder
2. Delete composer.lock file
3. Run composer install

After that, you will see a message about a package version conflict that you need to fix. This can be done manually or you can use Laravel Shift (which unfortunately costs $29-39).

I will not write about package upgrades because each project uses different packages, you need to follow the upgrade guides from the official pages of these packages.

### It’s important not to forget about these packages:

1. laravel/framework ^11.0
2. laravel/passport ^12.0
3. laravel/helpers ^1.5
4. guzzlehttp/guzzle ^7.0
5. phpunit/phpunit ^10
6. league/flysystem-aws-s3-v3 ^3.0
7. league/flysystem-sftp-v3 ^3.0

### Refactoring deprecated code

Upgrading your PHP application to PHP 8.2 involves updating the code base to handle legacy functions. Doing this manually can be tedious and error-prone. Rector, a powerful automated refactoring tool, can help you efficiently upgrade your code to PHP 8.2 standards. In this chapter, you’ll learn how to use Rector to refactor legacy code, making the migration process smoother and more reliable.

## Install Rector

https://github.com/rectorphp/rector

Use composer to install it.
```
composer require rector/rector --dev
```

After that, create a rector.php file and add the main customization code to it. In the Rector setup file, you can add the rules you need and remove the rules that are not suitable for you.

```
se Rector\Config\RectorConfig;
use Rector\TypeDeclaration\Rector\Property\TypedPropertyFromStrictConstructorRector;

return RectorConfig::configure()
    ->withPaths([
        __DIR__ . '/app',
        __DIR__ . '/bootstrap',
        __DIR__ . '/config',
        __DIR__ . '/public',
        __DIR__ . '/resources',
        __DIR__ . '/routes',
        __DIR__ . '/tests',
    ])
    // register rules for php8.2
    ->withPhpSets(php82: true)
    // register single rule
    ->withRules([
        TypedPropertyFromStrictConstructorRector::class
    ])
    // here we can define, what prepared sets of rules will be applied
    ->withPreparedSets(
        deadCode: true,
        codeQuality: true,
        typeDeclarations: true,
        privatization: true,
    );
```

After customization, we run Rector, which will automatically fix all the code to make it compatible with PHP 8.2.

```
vendor/bin/rector process app
```

You will see a progress bar showing how many files have already been refactored.

## Laravel Upgrade

To make a refactor for laravel we use the same Rector, but we need to install the package that extends it.
```
composer require driftingly/rector-laravel --dev
```

To add a set of Laravel rules to your config, use the RectorLaravel\Set\LaravelSetList class and select one of the constants:
```
->withSets([
  LaravelSetList::LARAVEL_110
]);
```

Run the command again and you will get refactored code.
```
vendor/bin/rector process app
```

## Migrating from Laravel Mix to Vite

Migrating from Laravel Mix to Laravel Vite can significantly improve productivity and optimize your development workflow.

In this case, we will use Laravel Shift as it is free for this migration (for now).

**https://laravelshift.com/convert-laravel-mix-to-vite**

Go to this link and follow the instructions, it will generate a vite configuration based on the mix configuration and refactor your components to be compatible with your version of vue/react.

Unfortunately, it makes mistakes in imports in blade files, so be careful and change it manually if you need to.

Switching to vite is worth the time, as it speeds up javascript builds by several dozen times.

## PHPUnit Upgrade

To upgrade PHPUnit to version 10 you can use the same Laravel Shift, this migration is free, or you can do it manually.
PHPUnit 10 Shift - Upgrade your tests to PHPUnit 10
The PHPUnit 10 Shift reviews your tests for any backward incompatible changes and automates the upgrade process to…

**laravelshift.com**

For manual migration you will need to:

- Remove deprecated features.
- Update packages for compatibility with PHPUnit 10

## Conclusion

Migrating from PHP 7.2 to 8.2 and from Laravel 6 to 11 may seem like a major undertaking, but the benefits far outweigh the challenges. By following this guide and utilizing the latest features and enhancements, you can ensure your application is secure, performant, and ready for future developments. Embrace the journey from legacy to modern and unlock the full potential of your web application.

