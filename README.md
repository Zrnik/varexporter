# Brick\VarExporter

<img src="https://raw.githubusercontent.com/brick/brick/master/logo.png" alt="" align="left" height="64">

A PHP library that offers a pretty and powerful alternative to `var_export()`.

[![Build Status](https://secure.travis-ci.org/brick/varexporter.svg?branch=master)](http://travis-ci.org/brick/varexporter)
[![Coverage Status](https://coveralls.io/repos/brick/varexporter/badge.svg?branch=master)](https://coveralls.io/r/brick/varexporter?branch=master)
[![Latest Stable Version](https://poser.pugx.org/brick/varexporter/v/stable)](https://packagist.org/packages/brick/varexporter)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](http://opensource.org/licenses/MIT)

## Introduction

PHP's [var_export()](https://www.php.net/manual/en/function.var-export.php) function is a handy way to dump a variable as executable PHP code.
 
It is particularly useful to store data that can be cached by OPCache, and later retrieved very fast, much faster than unserializing data using `unserialize()` or `json_decode()`.

It also suffers from several drawbacks:

- It outputs arrays as `array()` notation, instead of the short `[]` notation
- It outputs numeric arrays with explicit and unnecessary `0 => ...` key => value syntax
- It outputs invalid PHP code for `stdClass` objects, using `stdClass::__set_state()` which doesn't exist
- It cannot handle objects with public properties, without implementing `__set_state()` explicitly
- It does not complain when exporting an object with overridden private properties, and outputs and array with duplicate keys

This library aims to provide a prettier, safer, and more complete alternative to `var_export()`.

### Installation

This library is installable via [Composer](https://getcomposer.org/):

```bash
composer require brick/varexporter
```

### Requirements

This library requires PHP 7.1 or later.

### Project status & release process

While this library is still under development, it is well tested and should be stable enough to use in production environments.

The current releases are numbered `0.x.y`. When a non-breaking change is introduced (adding new methods, optimizing existing code, etc.), `y` is incremented.

**When a breaking change is introduced, a new `0.x` version cycle is always started.**

It is therefore safe to lock your project to a given release cycle, such as `0.1.*`.

If you need to upgrade to a newer release cycle, check the [release history](https://github.com/brick/varexporter/releases) for a list of changes introduced by each further `0.x.0` version.

## Quickstart

Just grab an instance of `VarExporter`, and call its `export()` method:

```php
use Brick\VarExporter\VarExporter;

$exporter = new VarExporter;
echo $exporter->export([1, 2, ['foo' => 'bar', 'baz' => []]]);
```

This code will output:

```php
[
    1,
    2,
    [
        'foo' => 'bar',
        'baz' => []
    ]
]
```

Compare this to the `var_export` output:

```php
array (
  0 => 1,
  1 => 2,
  2 => 
  array (
    'foo' => 'bar',
    'baz' => 
    array (
    ),
  ),
)
```

Note: unlike `var_export()`, `export()` always returns the exported variable, and never outputs it.

Note: `export()` is not a static method, as the exporter keeps an internal cache of class information, which speeds up the process of exporting multiple objects of the same class. As such, reusing a `VarExporter` instance several times will be slightly faster than creating a new instance every time.

### Exporting stdClass objects

You come across a `stdClass` object every time you cast an array to an object, or use `json_decode()` with the second argument set to `false` (which is the default).

While the output of `var_export()` for `stdClass` is syntactically valid PHP code:

```php
var_export(json_decode('
    {
        "foo": "bar",
        "baz": {
            "hello": "world"
        }
    }
'));
```

```
stdClass::__set_state(array(
   'foo' => 'bar',
   'baz' => 
  stdClass::__set_state(array(
     'hello' => 'world',
  )),
))
```

it is totally useless as it assumes that `stdClass` has a static `__set_state()` method, when it doesn't:

> Error: Call to undefined method stdClass::__set_state()

What does `VarExporter::export()` do instead? Well, it outputs an array to object cast, which is both syntactically valid **and** executable:

```php
echo $exporter->export(json_decode('
    {
        "foo": "bar",
        "baz": {
            "hello": "world"
        }
    }
'));
```

```php
(object) [
    'foo' => 'bar',
    'baz' => (object) [
        'hello' => 'world'
    ]
]
```
