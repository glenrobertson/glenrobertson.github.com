---
layout: page
title: Django JS/CSS compression with django-pipeline
---

[`django-pipeline`](http://django-pipeline.readthedocs.org/en/latest/) is a Django application that will easily handle Javascript and CSS compression for you. 

It integrates with Django awesomely, by:
* Using `staticfiles` post-processing step to perform the compression
* Providing a template tag to point to the minified file paths
* Making nice use of the `DEBUG` setting to decide whether to load the source or compressed files

This post explain how to set up [`django-pipeline`](https://github.com/cyberdelia/django-pipeline) along with [`mincss`](https://mincss.readthedocs.org/en/latest/) and [`slimit`](http://pypi.python.org/pypi/slimit) for CSS and Javascript compression, as part of your deployment process.

It is assumed that you already have a `requirements.txt` in your Django project, and `./manage.py collectstatic` is part of your deploy script.

### Install libraries
The first thing you need to do is install `django-pipeline`, `mincss`, and `slimit`. Note that you need to use `Django>=1.4` or `django-staticfiles>=1.2.1` to be able to use `django-pipeline` version 1.3.0.

Add the following to your `requirements.txt`:

    django-pipeline==1.3.0
    cssmin==0.1.4
    slimit==0.7.4

Then re-run `pip install -r requirements.txt`


### Configure your settings.py
There are a few changes you need to make to your `settings.py` to get `django-pipeline` running. These are:

1. Add `pipeline` to the installed apps
1. Change the `staticfiles` app's file storage engine to `pipeline.storage.PipelineStorage`:
1. Tell `pipeline` to use `cssmin` and `slimit` as the minification engines:
1. Tell `pipeline` where your css/js files are located:

Here is an example configuration:

    INSTALLED_APPS = (
        ...
        'pipeline',
        ...
    )

    STATICFILES_STORAGE = 'pipeline.storage.PipelineStorage'

    PIPELINE_CSS_COMPRESSOR = 'pipeline.compressors.cssmin.CSSMinCompressor'
    PIPELINE_CSSMIN_BINARY = 'cssmin'
    PIPELINE_JS_COMPRESSOR = 'pipeline.compressors.slimit.SlimItCompressor'

    PIPELINE_CSS = {
        'my_app': {
            'source_filenames': (
                'lib/bootstrap/css/bootstrap.css',
                'css/*',
            ),
            'output_filename': 'min.css',
            'variant': 'datauri',
        },
    }
    PIPELINE_JS = {
        'my_app': {
            'source_filenames': (
                'lib/jquery-1.9.0.js',
                'lib/bootstrap/js/bootstrap.js',
                'js/app.js',
            ),
            'output_filename': 'min.js',
        }
    }

The basic `PipelineStorage` engine is being used, but pipeline also provides other file storage engines, documented [here](https://django-pipeline.readthedocs.org/en/latest/storages.html)

In the `PIPELINE_CSS` and `PIPELINE_JS` dictionaries, we have configurations labelled with `my_app`, that contain the Twitter bootstrap library, JQuery, along with custom CSS in the `css/` directory and a custom Javascript file at `js/app.js`. The order of the entries in `source_filenames` is important if one library depends on another library.

The setting `'variant':'datauri'` inside `PIPELINE_CSS['my_app']` will tell `django-pipeline` to embed fonts and images directly inside your compiled CSS. You can read more about that [here](http://django-pipeline.readthedocs.org/en/latest/configuration.html#embedding-fonts-and-images)


Change the lines in `source_filenames` of the `PIPELINE_CSS` and `PIPELINE_JS` dictionaries to point to CSS/JS files in your Django applications static directory. After you have done that, you should be able to run `./manage.py collectstatic`, which will generate `min.css` and `min.js` files in your `STATIC_ROOT` directory.

### Add template tags to your base template
Now that the compressed CSS and Javascript files have been generated, you need to use the `compressed` template tags to include the files. Add the following inside the `<head></head>` of your base Django template:

    {% raw %}
    {% load compressed %}
    {% compressed_css 'my_app' %}
    {% compressed_js 'my_app' %}
    {% endraw %}

Depending on your `DEBUG` and `PIPELINE` settings, the template tag will either load the compressed files that were generated, or will load all of the original source files. 

The source files will be loaded if `DEBUG = True`, or the compressed files will be loaded if `DEBUG = False`. However, if `DEBUG = True` and `PIPELINE = True`, then the source files will still be loaded.

### Done!
Now your project will be able to serve up compressed JS and CSS files in production, while allowing you to work on the source files in development. I would recommend downloading all your third party libraries as source files in case you need to edit them easily.

`django-pipeline` is a popular library that is being actively developed as of early 2013. It has a lot of other awesome features that I haven't explored yet, such as compilers for Coffeescript, LESS, SASS, etc, as well as compression libraries in addition to `mincss` and `slimit`. Note that I chose to go with `mincss` and `slimit` for ease of installation via `pip`.
