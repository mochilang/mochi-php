# mochi-php

[![CI](https://github.com/mochilang/mochi-php/actions/workflows/ci.yml/badge.svg)](https://github.com/mochilang/mochi-php/actions/workflows/ci.yml)

Bidirectional PHP/Composer bridge for [Mochi](https://github.com/mochilang/mochi) (MEP-75). Lets Mochi code consume any PHP Composer package and publish Mochi modules as Composer libraries.

## Pipeline (phases 0-14)

```
mochi.toml [php-dependencies]
   |  pkgmanifest.Parse + pkgsolver.Solve (MEP-57)
   v
resolved PHP dep tree (vendor/package + version + Packagist dist URL)
   |  packagist.Fetch (Packagist v2 sparse API)
   v
dist zips in ~/.cache/mochi/php-deps/<sha256-hex>/
   |  reflect.Run (php reflect.php <pkg-path> -> JSON)
   v
ReflectionSurface JSON document per package
   |  typemap.Translate (closed PHP-to-Mochi table)
   v
TranslatedSurface + SkipReport per package
   |  externemit.Emit (Mochi extern fn / extern type)
   v
synthesised .mochi shim file per package
   |  glue.Emit (PHP-side use + forwarding stubs)
   v
vendor/ injection into MEP-55 build sandbox
   |  MEP-55 Driver.Build (TargetPhpSource / TargetPhpLibrary / ...)
   v
PHP program or Composer library
```

## Installation

```bash
go get github.com/mochilang/mochi-php
```

Requires Go 1.24+.

## Packages

| Package | Phase | Description |
|---|---|---|
| errors | 0 | SkipReason constants, SkipReport, BridgeError |
| build | 0 | Driver, Options, Workspace scaffold |
| packagist | 1 | Packagist v2 sparse-index client |
| cache | 2 | Content-addressed Composer dist fetcher |
| reflect | 3 | PHP CLI reflection invoker + JSON surface parser |
| typemap | 4 | Closed PHP-to-Mochi type translation table |
| externemit | 5 | Mochi extern fn / extern type emitter + hierarchy bridge |
| glue | 6 | PHP-side use + forwarding stubs |
| autoload | 7 | vendor/autoload.php generator (PSR-4, no composer install) |
| lock | 8 | mochi.lock [[php-package]] read/write |
| library | 9 | TargetPhpLibrary: PSR-4 src/ + composer.json emit |
| publish | 10 | Packagist publish: GPG tag + Update API + wait for index |
| asyncemit | 12 | async extern fn emitter for ReactPHP / Amp / Revolt promises |
| pharemit | 13 | Phar stub + build script generator |
| corpus | 14 | 24-package fixture corpus + integration tests |

## PHP-to-Mochi type table (closed set)

| PHP type     | Mochi type  | Notes                                      |
|--------------|-------------|--------------------------------------------|
| int          | int         |                                            |
| float        | float       |                                            |
| string       | string      |                                            |
| bool         | bool        |                                            |
| ?T           | T\|nil      | nullable wrapper                           |
| array (typed)| list/map    | heuristic from docblock or typed property  |
| class        | record/handle |                                          |
| interface    | protocol handle |                                        |
| void         | unit        |                                            |
| never        | panic boundary | SkipNever in v1                         |
| mixed        | SKIP        | SkipMixed                                  |
| object       | SKIP        | SkipObject                                 |
| callable     | SKIP        | SkipCallable                               |
| resource     | SKIP        | SkipResource                               |

## See also

- [MEP-75 spec](https://github.com/mochilang/mochi/blob/main/website/docs/mep/mep-0075.md)
- [Mochi monorepo](https://github.com/mochilang/mochi)
