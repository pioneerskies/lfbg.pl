# Looking For Bad Guys

## The CLI version

Warning! This is beta software!
We use it in production every day, but keep calm and test it on your
system with caution.

## Scope

This little script's scope is to act as regex collection in order to do
code scanning about maliciuos code and files.
The are 3 search models:

* filenames
* malicious-snippets
* executables

The former will search for filenames based on regexes listed in _include.list_,
excluding those matching regexes in _exclude.list_. The latter will search for
files matching the _include.list_ regexes, than scan the code for patterns
matching the _regex.list_.

## Why?

All began thanks to the idea born from [25yearsofprogramming](http://25yearsofprogramming.com/php/findmaliciouscode.htm).
In @welaika we have a lot of hosted sites, hosted on massive hosting services,
with intrinsic security holes. We had some attacks, the most were web-shells
injections or defacements; so we was in need of a passive security scan to monitor
our site's code, just like lfbg from 25yearsofprogramming.

But that is a PHP script, with all the side effects (some are subjective):

* slow
* need a webserver
* so fu***ng slow scanning hundreds of thousends of files
* almost usable only interactively
* a browser killer
* a resources killer
* too hardcoded

So we decided to write down a few lines of Perl code in order to make:

* fast as it can
* the best regex engine built in
* easily configurable regex collections (with comments too)
* ultra portable (just few installs from cpan)
* non-interactive

## Search for what?

I'm working on making _lfbg.pl_ a web-shell and htaccess hack detector. But it
is a little search and scan engine based on regular expressions, so it could
be extended in any direction.
But I'm assuming that the most you write regexes to scan for generic dangerous
code, the most you'll get false positives; and the most you'll get false positives
the most you'll ignore the report in your inbox! :)
So I'm focusing on reading webshells' code and htaccess attack technics, cathcing
significant and representative snippets and writing down regular expression
matching them.

## Dissection

Files, folders and what they do...

    ├── lfbg.conf.sample        # your config sample cp it to lfbg.conf
    ├── lfbg.pl                 # main script. Invoke it!
    ├── lib                     # all the PM will go here. Leave it alone
    │   ├── Lfbg.pm
    ├── models                  # each subfolder is a search&scan method
    |   |                         this is where you have to write your own regexes!
    │   ├── filenames           # this one search for suspicious filenames
    │   │   ├── exclude.list        # excluding regex collection
    │   │   ├── include.list        # including regex collection
    │   │   └── regex.list          # not used here...see below
    │   ├── malicious-snippets  # this one search inside code for dangerous snippets
    │   │   ├── exclude.list
    │   │   ├── include.list
    │   │   └── regex.list          # code pattern matching regexes
    │   ├── template
    │   │   ├── exclude.list
    │   │   ├── include.list
    │   │   └── regex.list
    │   ├── executables         # list executable files
    │   │   ├── exclude.list
    │   │   ├── include.list
    │   │   └── regex.list
    └── README.md               # your looking at this now, actually :)

### Perl CPAN dependencies

lfbg.pl relies on some perl modules; the better way to install them is using `cpanm`

    $ curl -L https://cpanmin.us | perl - --sudo App::cpanminus

so you can easily install all requirements

    $ cpanm Moose FindBin File::Find Data::Dumper POSIX Net::SMTP_auth HTML::Entities HTML::Strip File::stat

Shout if something missing.

### Usage

Create the config file

    cp lfbg.conf.sample lfbg.conf

and fill at least the `scanpath` option with the path of your interest

    $scanpath='/Users/fuzzy/dev/thesitetobescanned';

Supported flags are

    -v, -verbose    print results on STDOUT
    -m, -mail       send mail report (if configured in lfbg.conf)
    -list           list available methods

So take a look to installed scan methods

    ./lfbg.pl -list

and you can run the command invoking at least one method, e.g.:

    ./lfgb.pl -v malicious-snippets

### The "executables" exeption

The model named "executables" isn't based on a regex library. You can see that `regex.list`
is actually empty. The model will list all the files (directories escluded _a priori_)
with executable bit set. You can filter files to check using `include.list` and `exclude.list`.

This is useful due to many hack patterns leaving php scripts or images executable, when they
normaly aren't.

## Status and features (sort of changelog)

At the moment we are working (slowly) on the script in this way:

- [x] we run it on our backup server: backups in the night, sec scanning during the
day on the backed up files
- [ ] we are refactoring the shipped regex collection: too many false positives in
the original lfbg; and we are specializing the scan (I'll write more about)
- [x] we are implementing the _exclude_ functions (not a hard work, so it will be
shipped soon)
- [x] multiple scan paths
- [x] scan paths with globbing star (*)
- [x] custom user methods with no ```git pull``` pain in the ass: if you'll add a search
model you should update the core code easilly
- [ ] we are planning (it is just on my local dev branch ;) ) to implement a sec
check specifically for wordpress sites based on the md5 hashes of the core's files
- [ ] what's you're request? Leave an issue here on github!

## Just a piece of security

We are moving towards to use three passive security check tools:

* NeoPI
* Maldet
* Lfbg.pl

So we are not interested in writing a full featured search&scan script or an
IDS or whatsoever similar.

## Sharing

We will be happy, one day, if anyone using _lfbg.pl_ will share usefull regexes
adding them to the shipped collection :)
I'll write documentation about best contribuition method and best private regex
collection extension method. At the moment you have to know that you can add a
model like this...

### Adding a model

Well, if you add a folder inside _models_, named e.g. _my_scan_, this will be a
new valid search&scan method. For doing this:

    $ cp -r models/template models/my_scan

then test with

    $ ./lfbg.pl -list

and it should be listed. So you can use it

    $ ./lfbg.pl -v my_scan

Configure at your wish filling ```models/my_scan/*``` with your regular expressions

### Regular expressions properties

We support all the pcre with the following assumptions:

* you have not to escape trailing slash "/"
* all is considered case-insensitive
* you must write *one regex per line*
* you can comment each regex following a " #" at the end:


```

    AddHandler\s.+[^\.php]$  #in htaccess
    AddType application/x-httpd-php \.jpeg #in htaccess
    AddType application/x-httpd-perl \.png #in htaccess
    edoced_46esab #generic dangerous functions

```
