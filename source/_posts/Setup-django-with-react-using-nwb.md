---
title: Setup django with react using nwb
date: 2018-05-14 08:18:05
tags: [django, react, reactjs, nwb]
---


## django project 
```bash
# django-admin startproject djangoreact
# cd djangoreact
# python manage.py migrate
# pwd
/Users/me/djangoreact
```

## react app with nwb
https://github.com/insin/nwb
*A toolkit for React, Preact, Inferno & vanilla JS apps, React libraries and other npm modules for the web, with no configuration (until you need it)*

Let's start a react app
```bash
# pwd
/Users/me/djangoreact
# nwb new react-app reactjs
# ls -l
db.sqlite3
djangoreact
manage.py
reactjs

# cd reactjs
# yarn start
```

## config django template

Create folder `templates`
```bash
# cd /Users/me/djangoreact
# mkdir templates
# touch templates/react.html
# echo "react entry point" > templates/react.html
```

Edit `settings.py`
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'templates')
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Edit `urls.py`
```python
from django.views.generic import TemplateView

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^.*$', TemplateView.as_view(template_name='react.html'))
]

```

Run django server and make sure we see `react entry point`, so our view `react.html` works.

## config nwb to link react to django

```bash
# yarn add html-webpack-harddisk-plugin
```

Edit `nwb.config.js`
```javascript
const HtmlWebpackHarddiskPlugin = require('html-webpack-harddisk-plugin');
const path = require('path');
const isPro = process.env.NODE_ENV === 'production';
module.exports = {
    type: 'react-app',
    webpack: {
        // dont forget delete src/index.html to use this config: mountid, title, favicon
        html: {
            mountId: 'app',
            title: 'Django react',
            // favicon: 'src/favicon.ico'
            //this setting is required for HtmlWebpackHarddiskPlugin to work
            alwaysWriteToDisk: true,
            filename: 'react.html'
        },
        publicPath: isPro ? "/static/" : "http://localhost:3000/",
        extra: {
            plugins: [
                // this will copy an `index.html` for django to use
                new HtmlWebpackHarddiskPlugin({
                    outputPath: path.resolve(__dirname + "/../", 'templates')
                })
            ]
        },
        config: function (config) {
            if (!isPro) {
                config.entry = [
                    'webpack-dev-server/client?http://0.0.0.0:3000',
                    'webpack/hot/only-dev-server',
                    './src/index.js'
                ];
            }

            return config
        }
    },
    devServer: {
        // allow django host, in case you use custom domain for django app
        allowedHosts: ["0.0.0.0"]
    }
};

```

Start react app again
```bash
# cd reactjs
# yarn start
```

Check `templates/react.html`, we should see something like
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <title>Django react</title>
  </head>
  <body>
    <div id="root"></div>
  <script type="text/javascript" src="http://localhost:3000/app.js"></script></body>
</html>

```

This is react entry point for development, we use `publicPath` == `http://localhost:3000/` for this purpose

For production, we need to prepare react js files somewhere for django to `collectstatic` 

Now, restart django app and check `http://127.0.0.1:8000/`

We should see our react app!

## production deployment 

```bash
# cd reactjs
# yarn build
```

All js files should be ready in `reactjs/dist`. Check `templates/react.html` to see the changes.

Now we need to tell django go to `collectstatic`

Edit `settings.py`

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'static_dist')
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'reactjs/dist'),
]
```

Restart django app and check again `http://127.0.0.1:8000/`, react js is served by django static

***commit dist or not! it doesn't matter***
We will want to fix `reactjs/.gitignore` if we want to commit `dist`, otherwise our CI server must do `yarn build` first, then `collectstatic`

Now we have an app with react at frontend, backend by django, next step is add django rest framework,...etc 

Github: https://github.com/tamhv/djangoreact