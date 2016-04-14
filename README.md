# DotEnv building Helper #

DotEnv file (`.env`) are often used as runtime configuration files (i.e. Laravel based PHP projects)
and are not stored in your repository, so if you use Continuous Integration (CI) tools like Team City,
you need to create that `.env` file before tests can be started. DotEnv Helper is created to help
you with step.

The main assumption is that you usually have file like `.env.dist` in your repository (so people
know how to create production `.env`). DotEnv helper reads that file and tries to get values
for keys present in the file by looking into environmental variables or command line arguments
to produce populated, ready to use DotEnv file. Having `.env` created that way lets you keep all
needed runtime configuration (i.e. API keys etc) directly in CI configuration, in run step in Team City.

**NOTE:** To avoid accidental overwrites this tool only echoes content of echoed file, so to create
physical `.env` file for your code you need to redirect stdout with regular ` ... > .env`.

## Env variable subsitution ##

Let's assume our `.env.dist` looks like this:

    KEY=val
    BAR=zen
    FOO=

then knowing you want to have own value set for `KEY` you set your build
step in CI as shell script:

    export KEY=bar
    vendor/bin/process-env .env.dist > .env

which shall produce `.env` file with content as follow:

    KEY=bar
    BAR=zen
    FOO=

As you noticed, original value of `KEY` is replaced with what we provided, while `BAR`
and `FOO`, for which we did not provide replacements, were copied unaltered.


## Argument subsitution ##

Aside of env variables you can also pass `key=val` arguments to `process-env` achieve the same
goal:

    vendor/bin/process-env .env.dist KEY=bar > .env

**IMPORTANT:** first argument always refers to source env file.


## Combined substitution ##

Both substitution methods can be used together. When key is provided as argument and
also exists as Env variable, then command line provided value will be used:

    export KEY=bar
    vendor/bin/process-env .env.dist KEY=foo > .env

will produce:

    KEY=foo
    BAR=zen
    FOO=


## Installation ##

Use composer to install this package as your development dependency:

    composer require --dev marcin-orlowski/dotenv-helper


It will install `process-env` script in usual `vendor/bin` folder.


## License ##

* Written and copyrighted by Marcin Orlowski
* Response Builder is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)

