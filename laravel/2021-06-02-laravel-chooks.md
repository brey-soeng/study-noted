Manage git hooks easily in your composer configuration. This command line tool makes it easy to implement a consistent project-wide usage of git hooks. Specifying hooks in the composer file makes them available for every member of the project team. This provides a consistent environment and behavior for everyone which is great. It is also possible to use to manage git hooks globally for every repository on your computer. That way you have a reliable set of hooks crafted by yourself for every project you choose to work on.

```
composer require --dev brainmaestro/composer-git-hooks
```

Add a **cghooks** script to the scripts section of your **composer.json** file. That way, commands can be run with composer **cghooks** ${command}. This is ideal if you would rather not edit your system path.

Add the following events to your **composer.json** file. The **cghooks** commands will be run every time the events occur. Go to Composer Command Events for more details about composer's event system.

```
{
    "scripts": {
        "cghooks": "vendor/bin/cghooks",
        "post-install-cmd": "cghooks add --ignore-lock",
        "post-update-cmd": "cghooks update",
    }
}
```

https://github.com/BrainMaestro/composer-git-hooks

