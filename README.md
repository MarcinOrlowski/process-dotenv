# Process DotEnv - .env building helper tool #

[![Latest Stable Version](https://poser.pugx.org/marcin-orlowski/process-dotenv/v/stable)](https://packagist.org/packages/marcin-orlowski/process-dotenv)
[![Latest Unstable Version](https://poser.pugx.org/marcin-orlowski/process-dotenv/v/unstable)](https://packagist.org/packages/marcin-orlowski/process-dotenv)
[![License](https://poser.pugx.org/marcin-orlowski/process-dotenv/license)](https://packagist.org/packages/marcin-orlowski/process-dotenv)

`.env` (AKA DotEnv) files are often used to store project configuration (i.e. for [Laravel](https://laravel.com/)
based PHP projects). As they usually contain sensitive information as API keys or DB credentials, `.env` files should
never be versioned. This also means that if you need to use/run your project in automated pipeline, i.e. with 
Continuous Integration (CI) tools like TeamCity or Travis-CI, you need to create proper `.env` file before using the
code. As `.env` file usually contains all the project configuration, there will be much more fields than said i.e.
API key. Additionally, as project develops, new entries can be added and existing entries altered or tweaked.
It's a developers' common practice to create `.env.dist` file, fill as much as possible (ommiting sensitive information)
and put it into VCS. 

So if we treat said `.env.dist` as **template file**, then with the right tool in hand we'd be able to
create corresponding `.env` file easily. And this is where `process-dotenv` steps in. The goal of this 
small tool is pretty simple - generate `.env` file based on the template `.env.dist`, filling/replacing
specified template keys with provided values (taken either from env vars, for supplied as invocation
arguments).

**NOTE:** Whenever I say `.env` or `.env.dist` I only mean **file format**, not file name. Your
file names can be anything you like as long its content follows dot-env file format!

**NOTE:** To avoid accidental overwrites `process-dotenv` outputs processed content to standard output. To to
create physical `.env` file need to redirect stdout to a file. Please see examples for more details.

## Env variable subsitution ##

**NOTE:** all samples mimics shell session, so ommit `$` line for use in scripts.:

Let's assume our `.env.dist` template file looks like this:

    KEY=val
    BAR=zen
    FOO=

Now, knowing your app requires `KEY` to be valid API key for tests to pass we can have it replaced with 
`process-dotenv`:

    $ KEY=barbar
    $ vendor/bin/process-dotenv .env.dist > .env

which shall produce `.env` file with the following content:

    KEY=barbar
    BAR=zen
    FOO=

As you noticed, original value of `KEY` is replaced with what we provided via enviromental variable,
while `BAR` and `FOO`, for which we did not provide replacements, were copied unaltered.


## Argument subsitution ##

Aside of env variables you can also pass `key=val` pairs as `process-dotenv` invocation arguments to
achieve the same results:

    $ vendor/bin/process-dotenv .env.dist KEY=barbar > .env

**IMPORTANT:** first argument always refers to source dot-env file, followed by (optional) `KEY=VAL` pairs.
You can pass as many pairs as you need and file names can be whatever you like.

## Combined substitution ##

Both substitution methods can be used together. When key is provided as argument and
also exists as enviromental variable, then command line provided value takes precedence:

    $ KEY=barbar
    $ vendor/bin/process-dotenv .env.dist KEY=value > .env

would produce:

    KEY=value
    BAR=zen
    FOO=


## Requirements ##

 * PHP 5+ (CLI)
 
## Installation ##

Use composer to install this package as your dependency:

    $ composer require marcin-orlowski/process-dotenv

It will install `process-dotenv` script in usual `vendor/bin` folder.


## Troubleshoting ##

Please remember that certain, especially generic key names can already be set up by
your shell or system. For example `USER` is usually present and holds id of currently logged
in user,`HOME` points to home directory of said user are variables already
set, etc. You can list all of them with `printenv` or `export` to ensure none of your keys
matches, but it is good habit to be more creative and avoid such short and potentially conflicting
names.

Simple test to see if your `.env.dist` uses such "risky" keys is to run `process-dotenv`
without any own substitution provided and diff result file with dist file:

    $ vendor/bin/process-dotenv .env.dist | diff .env.dist

If you got conflicts then you can either change your keys or at least substitute that key
via command line arguments to ensure system's values won't pollute your resulting `.env`:

    $ vendor/bin/process-dotenv .env.dist USER= HOME= > .env

but this is pretty error prone and is not recommended.

**NOTE:** in case you use conflicting key (i.e. `USER`) but you want it to keep the
value set in `.env.dist` you currently must pass it as command like pair. `process-dotenv` 
cannot tell which env variables is "good" and which is "bad", so as soon as it find one exists,
it will simply use its value. That's why you must override it via command line pair.

## License ##

* Copyright &copy; 2016-2018 by Marcin Orlowski
* Process Dotenv tool is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
