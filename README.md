# Process DotEnv - .env building helper tool #

[![Latest Stable Version](https://poser.pugx.org/marcin-orlowski/process-dotenv/v/stable)](https://packagist.org/packages/marcin-orlowski/process-dotenv)
[![Latest Unstable Version](https://poser.pugx.org/marcin-orlowski/process-dotenv/v/unstable)](https://packagist.org/packages/marcin-orlowski/process-dotenv)
[![License](https://poser.pugx.org/marcin-orlowski/process-dotenv/license)](https://packagist.org/packages/marcin-orlowski/process-dotenv)

DotEnv file (`.env`) are often used as runtime configuration files (i.e. [Laravel](https://laravel.com/)
based PHP projects) and are not stored in your repository, so if you use Continuous Integration (CI) 
tools like TeamCity or Travis-CI, you need to create that `.env` file before tests can be started. 
`Process-DotEnv` is a tool wass created to help you with this task.

**NOTE:** Whenever I say `.env` or `.env.dist` I only mean **file format**, not file name. Your
file names can be anything you like as long its content follows dot-env file format!

The main assumption is that you usually have file name `.env.dist` in your repository so people
using your code can easily figure out how to create production ready `.env` file. So if we'd 
tread your `.env.dist` as **template**, then with the right tool you'd be able to easily create
production ready `.env` file. and this is where `process-dotenv` steps in. The goal (and code :)
is pretty simple - generate `.env` file based on the template `.env.dist` but with all neccessary 
changes applied. So ths tool reads your `.env.dist` file and spits it out **replacing** all values
that you wanted it to change either by setting enviromental variables or as command line arguments.

**NOTE:** To avoid accidental overwrites `process-dotenv` does not create any files but echoes the final
content, so to create physical `.env` file for your code you need to redirect output to file with regular
redirection: ` ... > .env`.

## Env variable subsitution ##

Let's assume our `.env.dist` file looks like this:

    KEY=val
    BAR=zen
    FOO=

Now, knowing your app requires `KEY` to be valid i.e. API key for tests to pass we can have it replaced with 
`process-dotenv` (sample mimics shell session, for scripts get rid of `$`):

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


## Installation ##

Use composer to install this package as your development dependency:

    $ composer require --dev marcin-orlowski/process-dotenv


It will install `process-dotenv` script in usual `vendor/bin` folder.


## Troubleshoting ##

Please remember that certain, especially generic key names can already be set up by
your shell or system. For example, on key `USER` that holds id of currently logged
in user or `HOME` that points to home directory of said user are variables already
set. You can list all of them with `printenv` or `export` to ensure none of your keys
matches. 

Simple test to see if your `.env.dist` uses such "risky" keys is to simply run `process-dotenv`
without any substitution and then diff result file with dist file:

    $ vendor/bin/process-dotenv .env.dist | diff .env

If you got "conflicts" then you can either change your keys or at least substitute that key
via command line arguments to ensure system's values won't pollute your resulting `.env`:

    $ vendor/bin/process-dotenv .env.dist USER= HOME= > .env

**NOTE:** in case you use conflicting key (i.e. `USER`) but you want it to keep the
value set in `.env.dist` you currently must pass it as command like pair. `process-dotenv` 
cannot tell which env variables is "good" and which is "bad", so as soon as it find one exists,
it will simply use its value. That's why you must override it via command line pair.

## License ##

* Written and copyrighted by Marcin Orlowski
* Process Dotenv tool is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)

