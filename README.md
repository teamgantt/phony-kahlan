 Phony for Kahlan

[![Current version image][version-image]][current version]

[current version]: https://packagist.org/packages/teamgantt/phony-kahlan
[version-image]: https://img.shields.io/packagist/v/teamgantt/phony-kahlan.svg?style=flat-square "This project uses semantic versioning"

## Installation

    composer require --dev teamgantt/phony-kahlan

## See also

- Read the [documentation].
- Read the [Kahlan documentation].
- Visit the [main Phony repository].

[documentation]: http://eloquent-software.com/phony/latest/
[kahlan documentation]: https://kahlan.github.io/docs/
[main phony repository]: https://github.com/teamgantt/phony

## What is *Phony for Kahlan*?

*Phony for Kahlan* is a plugin for the [Kahlan] testing framework that provides
general integration with the [Phony] mocking framework, as well as optional
auto-wired test dependencies.

In other words, if a [Kahlan] test (or suite) requires a mock object, it can be
defined to have a parameter with an appropriate [type declaration], and it will
automatically receive a mock of that type as an argument when run.

[Stubs] for `callable` types, and "empty" values for other type declarations are
also [supported].

## Plugin installation

Installation is only required when using the [dependency injection] features of
the plugin. The plugin can be installed in the [Kahlan configuration file] like
so:

```php
<?php // kahlan-config.php

// disable monkey-patching for Phony classes
$this->commandLine()->set('exclude', ['Eloquent\Phony']);

// install the plugin once autoloading is available
Kahlan\Filter\Filters::apply($this, 'run', function (callable $chain) {
    Eloquent\Phony\Kahlan\install();

    return $chain();
});
```

## Dependency injection

Once the plugin is installed, any tests or suites that are defined with
parameters will be supplied with matching arguments when run:

```php
describe('Phony for Kahlan', function () {
    it('Auto-wires test dependencies', function (PDO $db) {
        expect($db)->toBeAnInstanceOf(PDO::class);
    });
});
```

### Injected mock objects

*Phony for Kahlan* supports automatic injection of [mock objects]. Because
[Phony] doesn't alter the interface of mocked objects, it is necessary to use
[`on()`] to retrieve the [mock handle] in order to perform [stubbing] and
[verification]:

```php
use function Eloquent\Phony\Kahlan\on;

describe('Phony for Kahlan', function () {
    it('Supports stubbing', function (PDO $db) {
        on($db)->exec->with('DELETE FROM users')->returns(111);

        expect($db->exec('DELETE FROM users'))->toBe(111);
    });

    it('Supports verification', function (PDO $db) {
        $db->exec('DROP TABLE users');

        on($db)->exec->calledWith('DROP TABLE users');
    });
});
```

### Injected stubs

*Phony for Kahlan* supports automatic injection of [stubs] for parameters with
a `callable` type declaration:

```php
describe('Phony for Kahlan', function () {
    it('Supports callable stubs', function (callable $stub) {
        $stub->with('a', 'b')->returns('c');
        $stub('a', 'b');

        $stub->calledWith('a', 'b')->firstCall()->returned('c');
    });
});
```

## Supported types

The following table lists the supported type declarations, and the value
supplied for each:

Parameter type | Supplied value
---------------|---------------
*(none)*       | `null`
`bool`         | `false`
`int`          | `0`
`float`        | `.0`
`string`       | `''`
`array`        | `[]`
`iterable`     | `[]`
`object`       | `(object) []`
`stdClass`     | `(object) []`
`callable`     | [`stub()`]
`Closure`      | `function () {}`
`Generator`    | `(function () {return; yield;})()`

When using a [type declaration] that is not listed above, the supplied value
will be a [mock] of the specified type.

By necessity, the supplied value will not be wrapped in a [mock handle]. In
order to obtain a handle, simply use [`on()`]:

```php
use function Eloquent\Phony\Kahlan\on;

it('Example mock handle retrieval', function (ClassA $mock) {
    $handle = on($mock);
});
```

## License

For the full copyright and license information, please view the [LICENSE file].

## Note

This is a fork of the now unmaintained [eloquent/phony-kahlan]. The original purpose of this fork
was to allow for compatibility with PHP 8.3. Be aware that we do not intend to regularly 
maintain this other than to keep it up to date with PHP as needed.

<!-- References -->

[`on()`]: http://eloquent-software.com/phony/latest/#facade.on
[`stub()`]: http://eloquent-software.com/phony/latest/#facade.stub
[dependency injection]: #dependency-injection
[kahlan configuration file]: https://kahlan.github.io/docs/config-file.html
[kahlan]: https://kahlan.github.io/docs/
[license file]: LICENSE
[mock handle]: http://eloquent-software.com/phony/latest/#mock-handles
[mock objects]: http://eloquent-software.com/phony/latest/#mocks
[mock]: http://eloquent-software.com/phony/latest/#mocks
[phony]: http://eloquent-software.com/phony/latest/
[stubbing]: http://eloquent-software.com/phony/latest/#stubs
[stubs]: http://eloquent-software.com/phony/latest/#stubs
[supported]: #supported-types
[type declaration]: http://php.net/functions.arguments#functions.arguments.type-declaration
[verification]: http://eloquent-software.com/phony/latest/#verification
[eloquent/phony-kahlan]: https://packagist.org/packages/eloquent/phony-kahlan