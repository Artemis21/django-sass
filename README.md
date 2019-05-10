django-sass
===========

The absolute simplest way to use [Sass]() with Django. Pure Python,
minimal dependencies, and no special configuration required.

[Source code on GitHub](https://github.com/coderedcorp/wagtail-cache)


Installation
------------

1. Install from pip.

```
pip install django-sass
```

2. Add to your `INSTALLED_APPS` (you only need to do this in a dev environment,
you would not want this in your production settings file, although it adds zero
overhead):

```python
INSTALLED_APPS = [
    ...,
    'django_sass',
]
```

3. Congratulations, you're done 😀


Usage
-----

In your app's static files, use Sass as normal. The only difference is that
you should not traverse upwards using `../` in `@import` statements. For example:

```
app1/
|- static/
   |- app1/
      |- scss/
         |- _colors.scss
         |- app1.scss
app2/
|- static/
   |- app2/
      |- scss/
         |- _colors.scss
         |- app2.scss
```

In `app2.scss` you could reference app1's and app2's `_colors.scss` import as so:

```scss
@import 'app1/scss/colors';
@import 'app2/scss/colors';
// Or since you are in app2, you can reference its colors with a relative path.
@import 'colors';
```

Then to compile `app2.scss` and put it in the `css` directory,
run the following management command:

```
python manage.py sass app2/static/app2/scss/app2.scss app2/static/app2/css/app2.css
```

Or, you can compile the entire `scss` directory into
a corresponding `css` directory. This will traverse all subdirectories as well:

```
python manage.py sass app2/static/app2/scss/ app2/static/app2/css/
```

In your Django HTML template, reference the CSS file as normal:

```html
{% load static %}
<link href="{% static 'app2/css/app2.css' %}" rel="stylesheet">
```

✨✨ **Congratulations, you are now a Django + Sass developer!** ✨✨

Now you can commit those CSS files to version control, or run `collectstatic` and deploy them as normal.

For an example project layout, see `testproject/` in this repository.


Watch Mode
----------

To have `django-sass` watch files and recompile them as they change (useful in development),
add the ``--watch`` flag.

```
python manage.py sass app2/static/app2/scss/ app2/static/app2/css/ --watch
```


Example: deploying compressed CSS to production
-----------------------------------------------

To compile minified CSS, use the `-t` flag to specify compression level (one of:
"expanded", "nested", "compact", "compressed"). The default is "expanded" which
is human-readable.

```
python manage.py sass app2/static/app2/scss/ app2/static/app2/css/ -t compressed
```

You may now optionally commit the CSS files to version control if so desired,
or omit them, whatever fits your needs better. Then run `collectsatic` as normal.

```
python manage.py collectstatic
```

And now proceed with deploying your files as normal.


Limitations
-----------

* `@import` statements must reference a path relative to a path in `STATICFILES_FINDERS`
  (which will usually be an app's `static/` directory or some other directory specified
  in `STATICFILES_DIRS`). Or they can reference a relative path equal to or below the
  current file. It does not support traversing up the filesystem (i.e. `../`).

  Legal imports:
  ```scss
  @import 'file-from-currdir';
  @import 'subdir/file';
  @import 'another-app/file';
  ```
  Illegal imports:
  ```scss
  @import '../file';
  ```

* Only files ending in `.scss` are supported for now.

* Source map generation is not supported yet.

* Only supports `-t` and `-p` options similar to `pysassc`. Ideally `django-sass` will
  be as similar as possible to the `pysassc` command line interface.
  **Note:** if using with Bootstrap, specify `-p 8` as Bootstrap requires higher floating
  point precision to work correctly.

Feel free to file an issue or make a pull request to improve any of these limitations. 🐱‍💻


Why django-sass?
----------------

Other packages such as [django-libsass](https://github.com/torchbox/django-libsass)
and [django-sass-processor](https://github.com/jrief/django-sass-processor),
while nice packages, require `django-compressor` which itself depends on several
other packages that require compilation to install.

* If you simply want to use Sass in development without installing a web of unwanted
  dependencies, then `django-sass` is for you.
* If you don't want to deploy any processors or compressors to your production server,
  then `django-sass` is for you.
* If you don't want to change the way you reference and serve static files,
  then `django-sass` is for you.
* And if you want the absolute simplest installation and setup possible for doing Sass,
  `django-sass` is for you too.

django-sass only depends on libsass (which provides pre-built wheels for Windows, Mac,
and Linux), and of course Django (any version).