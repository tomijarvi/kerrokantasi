[![Build status](https://travis-ci.org/City-of-Helsinki/kerrokantasi.svg?branch=master)](https://travis-ci.org/City-of-Helsinki/kerrokantasi)
[![codecov](https://codecov.io/gh/City-of-Helsinki/kerrokantasi/branch/master/graph/badge.svg)](https://codecov.io/gh/City-of-Helsinki/kerrokantasi)
[![Requirements](https://requires.io/github/City-of-Helsinki/kerrokantasi/requirements.svg?branch=master)](https://requires.io/github/City-of-Helsinki/kerrokantasi/requirements/?branch=master)

Kerro kantasi
=============
Kerro kantasi is an implementation of the eponymous REST API for publishing and participating in public hearings.
It has been built to be used together with participatory democrary UI (https://github.com/City-of-Helsinki/kerrokantasi-ui). Nothing should prevent it from being used with other UI implementations.

Kerro kantasi supports hearings with rich metadata and complex structures. Citizens can be asked for freeform input,
multiple choice polls and map based input. See the Helsinki instance of the UI for examples: https://kerrokantasi.hel.fi/. In addition to gathering citizen input, kerrokantasi-ui is also the primary editor
interface for hearings.

If you wish to see the raw data used by hearings, you may examine the API instance serving the UI: https://api.hel.fi/kerrokantasi/v1/

Technology
----------
Kerro kantasi is implemented using the Python programming language, with Django and Django Rest Framework as the
main structural components.

The API allows for both publishing and editing hearings. Kerrokantasi-ui contains the primary hearing editor.

Kerro kantasi has been designed to allow for anonymous participation in hearings, although that is up to the hearing designer. Citizens may also login to the platform, which allows them to use same identity for multiple hearings and participate in hearings requiring login. Creating and editing hearings always requires a login.

Authentication is handled using JWT tokens. All API requests requiring a login must include `Authorization` header containing a signed JWT token. A request including a JWT token with valid signature is processed with the permissions of the user indicated in the token. If the user does not yet exist, an account is created for them. This means that Kerro kantasi is only loosely coupled to the authentication provider.

Development quickstart
----------------------
0. Check you have "Prequisites" listed below
1. Create database `kerrokantasi` with PostGIS (see "Prepare database" below)
2. copy `config_dev.toml.example` to `config_dev.toml`. Check the contents (see "Configuration" below)
3. prepare and activate virtualenv (See "Install" below)
4. `pip install -r requirements.txt`
4. `python manage.py compilemessages`
4. `python manage.py collectstatic` (See "Choose directories for transpiled files" below)
5. `python manage.py runserver` (See "Running in development" below)

Installation
------------
This applies to both development and simple production scale. Note that you won't need to follow this approach to the letter if you have your own favorite Python process.

### Prerequisites

* Python 3.6 or later
* PostgreSQL with PostGIS extension
* Application server implementing WSGI interface (fe. Gunicorn or uwsgi), not needed for development

### Prepare database

Kerro kantasi requires full access to the database, as it will need to create database structures before
first run (and if you change the structures). Easiest way to accomplish this is to create a database user and make this user the owner of the Kerro kantasi database. Example:

     createuser kerrokantasi
     createdb -l fi_FI -E utf8 -O kerrokantasi -T template0 kerrokantasi

Both of these commands must be run with database superuser permissions or similar. Note the UTF8 encoding for the database. Locale does not need to be finnish.

After creating the database, PostGIS extension must be activated:

     psql kerrokantasi
     create extension postgis;     

This too requires superuser permissions, database ownership is not enough. PostGIS extension can also require the installation of package ending in -scripts, which might not be marked as mandatory.

### Choose directories for transpiled files and user uploaded files

Kerro kantasi has several files that need to be directly available using HTTP. Mostly they are used for the
internal administration interface, which allows low level access to the data. The files include a browser based editor and map viewer among others. Django calls such artifacts "static files".  The "collectstatic" command gathers all these to a single directory, which can then be served using any HTTP server software (for development, Django runserver will work). You will need to choose this directory. Conventional choices would be:

     /home/kerrokantasi-api/static or
     /srv/kerrokantasi-api/static or even
     /var/www/html/static

Kerro kantasi also allows hearing creators to upload images and other materials relevant to the hearings. These are usually called "media" in Django applications. Access to media is controlled by Kerro kantasi code, because material belonging to unpublished hearing must by hidden. This means that the media directory must NOT be shared using an HTTP server.

Location examples:

     /home/kerrokantasi-api/media
     /srv/kerrokantasi-api/media

For development you probably want to have a common parent directory, which contains directories for static files, media files and code. Having media directory among the code might get messy.

### Choose a directory for the code

Kerro kantasi code files can reside anywhere in the file system. Some conventional places might be:

     /home/kerrokantasi/kerrokantasi-api (data directories would be in the home directory)
     /opt/kerrokantasi-api, with data in /srv/kerrokantasi-api

For development a directory among your other projects is naturally a-ok. You probably already have this code in such a directory.

### Prepare virtualenv

Note that virtualenv can be created in many ways. `virtualenv` command shown here is old-fashioned but generally works well. See here for Python 3 native instructions: https://docs.python.org/3/tutorial/venv.html

     virtualenv -p python3 venv
     source venv/bin/activate

### Install required packages

Install all required packages with pip command:

     pip install -r requirements.txt

### Compile translation .mo files

     python manage.py compilemessages

You will now need to configure Kerro kantasi. Read on.

Configuration
-------------
Kerro kantasi can be configured using either `config_dev.toml` file or environment variables. For production use we recommend using environment variables, especially if you are using containers. WSGI-servers typically allow setting
environment variables from their own configuration files.

Kerro kantasi source code contains heavily commented example configuration file `config_dev.toml.example`. It serves both as a configuration file template and the documentation for configuration options.

### Configuration using config_dev.toml directly, for development

Directly using config_dev.toml is quite nice for development. Just `cp config_dev.toml.example config_dev.toml`. If you have created a database called `kerrokantasi` you will be almost ready to start developing. (Some `config_dev.toml` editing may be needed, overindulgence may cause laxative effects).

### Configuration using environment variables, recommended for production

Read through `config_dev.toml.example` and set those environment variables your configuration needs. The environment variables are named exactly the same as the variables in `config_dev.toml.example`. In fact, you could make a copy of the file, edit the copy and source it using the shell mechanisms. This would set all uncommented variables in the file. Many application servers (and docker-compose) can also read the KEY=VALUE format used in the file and set environment variables based on that.

Do note that nothing prevents you from using config_dev.toml in production if you so wish.

### Running using development server

Just execute the normal Django development server:
`python manage.py runserver`

runserver will reload if you change any files in the source tree. No need to restart it (usually).

### Running using WSGI server ("production")

Kerro kantasi is a Django application and can thus be run using any WSGI implementing server. Examples include gunicorn and uwsgi.

WSGI requires a `callable` for running a service. For kerrokantasi this is:
`kerrokantasi.wsgi:application`
The syntax varies a bit. Some servers might want the file `kerrokantasi/wsgi.py` and `application` as callable.

In addition you will need to server out static files separately. Configure your HTTP server to serve out the files in directory specified using STATIC_ROOT setting with the URL specified in STATIC_URL setting.

Development processes
---------------------

###Updating requirements

Kerrokantasi uses two files for requirements. The workflow is as follows.

`requirements.txt` is not edited manually, but is generated
with `pip-compile`.

`requirements.txt` always contains fully tested, pinned versions
of the requirements. `requirements.in` contains the primary, unpinned
requirements of the project without their dependencies.

In production, deployments should always use `requirements.txt`
and the versions pinned therein. In development, new virtualenvs
and development environments should also be initialised using
`requirements.txt`. `pip-sync` will synchronize the active
virtualenv to match exactly the packages in `requirements.txt`.

In development and testing, to update to the latest versions
of requirements, use the command `pip-compile`. You can
use [requires.io](https://requires.io) to monitor the
pinned versions for updates.

To remove a dependency, remove it from `requirements.in`,
run `pip-compile` and then `pip-sync`. If everything works
as expected, commit the changes.

### Testing

To run all tests, execute command in project root directory.

     py.test

Run test against particular issue.

    py.test -k test_7 -v

### Internationalization

Translations are maintained on [Transifex][tx].

* To pull new translations from Transifex, run `npm run i18n:pull`
* As a translation maintainer, run `npm run i18n:extract && npm run i18n:push` to push new source files.

[tx]: https://www.transifex.com/city-of-helsinki/kerrokantasi/dashboard/
