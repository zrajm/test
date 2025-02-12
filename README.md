# 1.1 test
# 1.1 test
# 1.1 test-abc-dee-eff

# abc_def-ghi


# abouc - sochou


# räksmörgås .

# ÅÄÖ


<!--*- markdown -*-->

# <a id=e8c60>Installation and Setup</a>

A description of how to download and setup a signbank, starting from scratch.


## <a id=f8a>1. Download the Source</a>

First, find the signbank you want to install and download it. There are a lot
of them on Github, and in this guide I'll assume you'll download it from there
(though most of the stuff below should apply to most signbanks, regardless of
where you downloaded it from).

Here I'll fetch the Isof Signbank (*Isof* is an abbreviation for “Institutet
för språk och folkminnen” – a Swedish government agency):

    git clone https://github.com/ISOF-ITD/FinSL-signbank.git

There exists many Signbanks to choose from:

  * [Isof Signbank](https://github.com/ISOF-ITD/FinSL-signbank)
    ([website](https://frigg.isof.se/teckenlistan/dictionary/public/translation/))
  * [Stockholm University Signbank](https://github.com/zrajm/FinSL-signbank)
    ([website](https://signbank.ling.su.se/))
  * [Finnish Signbank](https://github.com/Signbank/FinSL-signbank.git)
    ([website](https://signbank.csc.fi/))
  * [The Github Signbank
    Organization](https://github.com/orgs/Signbank/repositories) (has a list of
    additional Signbanks)

**Note:** [Building a non-default branch](#c0a)

## <a id=281>2. Set up Python *Virtual Environment*</a>

Before installing the necessary dependencies and running Django, you'll
probably want to create a Python *virtual environment* for your Signbank. This
isolates the current project from the rest of your system and installs the
dependencies in the current directory (instead of system-wide). This if you
have multiple Django projects (such as different Signbanks) installed they will
be independent from each other and might (for example) use different
dependencies, or different versions of the same dependencies.

Once a virtual environment is initiated, all subsequent Signbank commands need
to be run in the same virtual environment. This means that when you come back
to continue working with a previously installed Signbank, you'll need to
reinitialize the virtual environment, using `source venv/bin/activate`. (A
clear sign that you're inside a virtual environment is the text '(venv)' in
your shell prompt.)

As a reminder all below code snippets will start with the activation command
`source venv/bin/activate` even though you'll only need to execute if you're
not in a virtual environment (if you accidentally re-execute it, don't worry,
the command does not have any effect if you're already in a virtual
environment).

To create the virtual environment, go to the signbank directory (e.g.
`finsl-signbank`) and from there run:

    virtualenv -ppython3 venv      # alternative: python3 -m venv venv
    source venv/bin/activate

Either the `virtualenv` or `venv` techniques should work interchangeably
(AFAIK). (The `virtualenv` command also makes sure the binary `python` is
linked to `python3` within the virtual environment -- not sure if that's the
case with `venv` though.)

I got into a situation where I wasn't installed venv, and on a machine where I
wasn't admin, in which case I managed to get by just fine using `virtualenv`
(which *was* installed).


## <a id=c40>3. Install Required Python Modules</a>

    source venv/bin/activate
    #pip install wheel                        # if install fails
    pip install -r requirements.txt
    pip install django-debug-toolbar          # need for db later

NOTE: If you get the error "error: invalid command 'bdist_wheel'" when
installing the requirements, this might be solved by first installing the
Python module `wheel`. (This was not needed on Lubuntu 22.04.)

The above commands will install all the Python modules specified in
`requirements.txt` in the virtual environment directory
`venv/lib/python3.VERSION/site-packages` and a few commands in `venv/bin`. Most
notably the `django-admin` command, but also commands called
`confusable_homoglyphs`, and `sqlformat`.

The `django-debug-toolbar` will be needed later for the `createsuperuser` and
`makemigrations` commands, so you might as well install it now.


## <a id=d42>4. Configure Signbank</a>

The following files need to be edited:

    signbank/settings/base.py
    signbank/settings/production.py
    signbank/settings/settings_secret.py


### 3.1. `signbank/settings/base.py`

Here you'll need to configure the timezone and default (UI?) language.

Modified settings in `signbank/settings/base.py`:

     #: A string representing the time zone for this installation.
    -TIME_ZONE = 'Europe/Helsinki'
    +TIME_ZONE = 'Europe/Stockholm'

     #: A string representing the language code for this installation. This should be in standard language ID format.
     #: For example, U.S. English is "en-us".
    -LANGUAGE_CODE = 'fi'
    +LANGUAGE_CODE = 'en-us'

    +# FIXME: change this? Or WSGI_FILE? (see https://finsl-signbank.readthedocs.io/en/latest/installation.html)
     # The full Python path of the WSGI application object that Django's built-in servers (e.g. runserver) will use.
     WSGI_APPLICATION = 'signbank.wsgi.application'


### 3.2. `signbank/settings/production.py`

Settings changed in `signbank/settings/production.py`:

     #: IMPORTANT: The hostname that this signbank runs on, this prevents HTTP Host header attacks
    -ALLOWED_HOSTS = ['signbank.csc.fi']
    +ALLOWED_HOSTS = ['signbank.ling.su.se']

**Note:** [`ALLOWED_HOSTS` *must not contain a port number*.](#b27)

     # A list of directories where Django looks for translation files.
     LOCALE_PATHS = (
    -    '/home/signbank/signbank-fi/locale',
    +    '/home/stsbank/finsl-signbank/locale',
     )

     #: The absolute path to the directory where collectstatic will collect static files for deployment.
     #: Example: "/var/www/example.com/static/"
    -STATIC_ROOT = '/var/www/signbank/static/'
    +STATIC_ROOT = '/home/stsbank/finsl-signbank/signbank/static/'

     #: Absolute filesystem path to the directory that will hold user-uploaded files.
    -MEDIA_ROOT = '/var/www/signbank/media/'
    +MEDIA_ROOT = '/home/stsbank/finsl-signbank/media/'

`STATIC_ROOT` is a directory containing files checked out from signbank git
repo, while `MEDIA_ROOT` contain media files belonging to your own database
content (i.e. stuff you fill in yourself).


### 3.3. `signbank/settings/settings_secret.py`

Copy the `secret_settings.py` template:

    cd signbank/settings
    cp -a settings_secret.py.template settings_secret.py

And modify the settings. To generate a new secret key use
https://miniwebtool.com/django-secret-key-generator/.

     #: Make this unique, and don't share it with anybody. This is used to provide cryptographic signing.
    -SECRET_KEY = 'INSERT_$$$$_YOUR_%%%%_SECRET_@@@@_KEY_%%%%%%_HERE'
    +SECRET_KEY = '<SOMETHING ELSE>'

     #: A list of all the people who get code error notifications. When DEBUG=False and a view raises an exception,
     #: Django will email these people with the full exception information.
     ADMINS = (
    -    ('Firstname Lastname', 'your.email@address.com'),
    +    ('zrajm', 'sts-signbank@example.com')
     )

     DATABASES = {
         'default': {
             'ENGINE': 'django.db.backends.sqlite3',
    -        'NAME': '/path/to/signbank.db',
    +        'NAME': '/home/stsbank/finsl-signbank/signbank/dictionary/migrations/signbank.db',
         }
     }


## <a id=7e9>5. Setting Up the Database</a>

From what I understand this is used to populate, or at least build up the
structure of the database. I'll just quote the docs on [Database migration] in
this section:

“First create migrations for django flatpages app to add translation fields
with django-modeltranslation:”

    source venv/bin/activate
    python bin/develop.py makemigrations

After running `makemigrations` the database is created, but zero bytes in size
(for a Sqlite database). -- Possibly this command populates some 'migrations'
directories with stuff?

“Then migrate and load fixture for flatpages:”

    python bin/develop.py migrate
    python bin/develop.py loaddata flatpages_initial_data

Running the 'migrate' command above adds data to the database (growing a file
with sqlite database to about 700K in size).


[Database migration]: https://finsl-signbank.readthedocs.io/en/latest/installation.html#database-migration


### 5.1. Set Home Page Name in Database

The database contains the name of site.

If this is not set properly then links to comments will not work, but you'll
instead get redirected to whatever site is set in the database. (There might be
other effects as well, but this is the only thing I've seen so far.) In my case
this meant that when I clicked on a comment (top right, on the "Home" page) I
was redirected to:

    https://example.com/comments/cr/9/6/#c1

Instead of the expected:

    https://signbank.ling.su.se/comments/cr/9/6/#c1

Which was pretty useless.

One way to fix this problem is to dump the database to a text file, then change
the value in the dump and restore the database from the dumped file.

    cd signbank
    sqlite3 signbank.db .dump >signbank.db.dump.txt

Thereafter, open the file `signbank.db.dump.txt` in your favorite editor and
find the following line (where <domain> is whatever your current configuration
is, by default it is `example.com`).

    INSERT INTO django_site VALUES(1,'<domain>','<domain>');

Replace <domain> with your site's domain name (that could be `localhost:8000`
if you're running it locally, or `signbank.ling.su.se` if your running a server
installation of your own, or anywhere in between).

Restoring the database from the dump is achieved using:

     sqlite3 signbank.db <signbank.db.dump.txt


## <a id=89a>6. Create Admin User</a>

    source venv/bin/activate
    python bin/develop.py createsuperuser

When running `createsuperuser` you will be prompted for a username and
password. If there's already a configured superuser the username prompt will
include “(leave blank to use 'USERNAME')”.

At this point you should be able to start the Signbank:

    python bin/develop.py runserver

...Go to the Django admin interface in your browser:

    http://localhost:8000/admin/

...And login using the username and password you specified with the
`createsuperuser` command.

See also:

  * [List Django users](#ebc64e8d2e)
  * [List Django users](#list-django-users)
  + [[#ebc64e8d2e][List Django users]]


## <a id=2c0>Django Admin Pages (Static Files)</a>

Upon installing and loading the Django admin interface i noticed that
stylesheets etc weren't loaded. In the server config we had set up the
directory to be served statically as `finsl-signbank/signbank/static/` and the
CSS file the frontend was trying to load could be found in one of the
site-package directories created during the pip installation. In the end I did
the following to resolve the situation:

    cd finsl-signbank/signbank/static/
    ln -s ../../venv/lib/python3.5/site-packages/django/contrib/*/static/* .

This symlinks from the static directory for the relevant packages (which were
only two, namely `gis` and `admin`). After this the admin pages loaded just
fine. [https://signbank.ling.su.se/admin/]


## <a id=f68>Django Admin Doc Pages (`docutils`)</a>

Clicking the 'DOCUMENTATION' link of the admin page gave me an error message
saying that the documentation requires the `docutils` package. So I installed
that using the following command:

    pip install docutils

And then killed and restarted Django with:

    python bin/production.py runserver 0.0.0.0:8080


# <a id=2cc>Additional Details</a>

These are some footnotes and additional details that would've mostly been a
distraction should they've occurred in the main body of the text.

## <a id=c0a>Checking Out a Different Branch in Git</a>

**Note:** Some of the repositories may contain more than one branch. In this
case you'll get the `master` (or `main`) branch unless you manually specify
which branch you want to check out. After cloning the repository you can list
the branches using:

    git pull --all
    git branch -a

And if you want to switch branch, use:

    git checkout <branch-name>


## <a id=e9c>List Django Subcommands</a>

To list the subcommands available with `bin/develop.py` and `bin/production.py`
use the `help` subcommand, like so:

    python bin/develop.py help

Additional information for each subcommand can be obtained using:

    python bin/develop.py help SUBCOMMAND


## <a id=b27>Port number in `ALLOWED_HOSTS`</a>

Regardless of whether or not you're using a port number in the URL to access
your Signbank instance (e.g. `http://stssignbank.webtrick.se:8080/`) no port
number may be present in the `ALLOWED_HOSTS` setting (in the config file
`signbank/settings/production.py`).

If you mistakenly do add a port number to this setting, then Django will still
happily start for you, but when you try to loading the corresponding web page
you will a `ERROR 400: Bad Request.` error:


## <a id=e02>`makemigrations` fail (‘cannot import name 'ugettext_lazy'’)</a>

When I ran `python bin/develop.py makemigrations` I got a stack dump ending in
the following error message.

    ImportError: cannot import name 'ugettext_lazy' from 'django.utils.translation'

It turns out that me installing `django-debug-toolbar` also pulled in a newer
version of Django, so that any subsequent commands will use that rather than
the truly ancient version 1.11 listed in `requirements.txt`. -- Now this did
not make my Django happy!

Wiping out my virtual environment and re-installing everything except
`django-debug-toolbar`, resul

    rm -r venv/
    virtualenv -ppython3 venv
    source venv/bin/activate
    pip install -r requirements.txt
    python bin/develop.py makemigrations

Also failed, but resulted in a new error:

    ImportError: cannot import name 'Iterator' from 'collections

Googling error message revealed that “The Iterable abstract class was removed
from collections in Python 3.10.” (https://stackoverflow.com/a/72032097/351162)
And so installed `python2` on my system and I re-installed using Python using
that. (This took a couple of tries, during which I error messages indicated
that I'd also need the `python2-pip-whl` and `python2-setuptools-whl` packages,
but for simplicity's sake I'll just skip describing that here.)

    rm -r venv/
    sudo apt install python2 python2-pip-whl python2-setuptools-whl
    virtualenv -ppython2 venv
    source venv/bin/activate
    pip install wheel                        # if install fails
    pip install -r requirements.txt
    python bin/develop.py showmigrations



# [eof]
<!--
Local Variables:
markdown-hide-markup: nil
End:
-->
