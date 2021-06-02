**Larastan** focuses on finding errors in your code without actually running it. It catches whole classes of bugs even before you write tests for the code.

- Adds static typing to Laravel to improve developer productivity and code quality
- Supports most of Laravel's beautiful magic
- Discovers bugs in your code without running it

1: First, you may use Composer to install **Larastan** as a development dependency into your Laravel project:

```
composer require --dev nunomaduro/larastan
```

2: Then, create a **phpstan.neon** or **phpstan.neon.dist** file in the root of your application. It might look like this:

```
includes:
    - ./vendor/nunomaduro/larastan/extension.neon

parameters:

    paths:
        - app

    # The level 8 is the highest level
    level: 5

    ignoreErrors:
        - '#Unsafe usage of new static#'

    excludePaths:
        - ./*/*/FileToBeExcluded.php

    checkMissingIterableValueType: false
```

3: After go to **composer.json** and find section and configure code like below code.

```
"scripts": {
        "phpstan": "phpstan analyse"
    },
```

```
"extra": {
        "hooks": {
            "pre-commit": [
                "composer run-script phpstan"
            ]
        }
    },
```

```
composer run-script phpstan
```

https://github.com/nunomaduro/larastan

