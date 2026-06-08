## laravel-helper

Bash script helper for common Laravel related tasks on server

## Installation

Clone this repository and make symlink:

```
$ git clone https://github.com/wmfairuz/laravel-helper.git
$ cd laravel-helper
$ ln -s ${PWD}/laravel-helper /usr/bin/laravel-helper
```

Set DEPLOY_DIR in your shell (eg. .bashrc) or directly define when invoking the script.

## Update laravel-helper

```
$ cd laravel-helper
$ git pull
```


## Usage

```
$ laravel-helper -d /opt/my-laravel -uir
```

or with a server that only has one laravel instance, we can put the target directory in DEPLOY_DIR

```
$ export DEPLOY_DIR=/opt/my-laravel
$ laravel-helper -uir
```

or you can use current directory by omitting both -d argument and DEPLOY_DIR env variable.

You can set a specific php cli version that will be used throughout the script. The php version will be reverted to the 
original version at the end of the script run.

```
$ laravel-helper -p 8.4 -d /opt/my-laravel -uir
```

By default will use current cli php version.

On a hardened server where the app is owned by a dedicated deploy user and a web group (and you can only
deploy as that user), set the ownership explicitly with `-o owner:group` so `git pull` keeps working across
deploys and the web server keeps group access:

```
$ laravel-helper -d /opt/www/app -o deploy:uapskm -uir
```

`-o` defaults to `www-data:www-data` (the legacy behavior) and can also be set via the `OWNERSHIP` env
variable (eg. in `.bashrc`). File ownership/permission changes (`-u`, `-r`) no longer call `sudo`
themselves — run the script as root (or as a user that already owns the tree and is a member of the
target group). Service-level actions still need root — `-p` (update-alternatives), `-c`/`-f` (php-fpm),
and `-s` (supervisorctl) use `sudo`.

`-r` sets the storage/cache directory mode based on the owner: the legacy `www-data` owner keeps `775`,
while any other (eg. a hardened deploy user) gets `2770` — group-writable, no access for others, and the
setgid bit so newly created files inherit the group.

### Running as root

You can run the helper as `root` and let it drop the code/build steps to the `-o` owner:

```
# logged in as root
$ laravel-helper -d /opt/www/app -o deploy:uapskm -uifr
```

When the script detects it is running as root **and** `-o` (or the `OWNERSHIP` env var) names a different
user, it runs the code/build commands (`git`, `composer`, `php artisan`) as that owner via
`sudo -u <owner>`, so files, `.git`, and storage caches stay owned by the deploy user (and git uses that
user's credentials). The privileged service steps (`systemctl`, `supervisorctl`, `update-alternatives`)
keep running as root, so the deploy user does **not** need any sudo rights. The script exits back to your
root shell when done.

Without `-o`, ownership defaults to `www-data:www-data` and the privilege drop is skipped, so a plain
`laravel-helper -d /opt/www/app -uifr` run as root behaves exactly as before — code runs as root and the
tree is chowned to `www-data:www-data`.

## Arguments

- d - Target directory (eg: -d /opt/my-laravel)
- u - Update code using git pull (defaults to master branch)
- i - Install dependencies using composer
- r - Refresh laravel instance and fix folder permission / ownership
- t - Output Laravel log file with specified line number  (eg: -t 100 to output last 100 line)
- l - Output Laravel log file and append output as log file grow
- p - Set php cli version. php version will be reverted to original before exiting (eg: -p 8.4)
- o - Set ownership as owner:group for app files, like chown (eg: -o deploy:uapskm). Defaults to current user:group

## License

This package is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
