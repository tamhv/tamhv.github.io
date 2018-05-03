---
title: React Semantic UI boilerplate
date: 2018-05-03 07:50:15
tags: [react, redux, redux-saga, semantic-ui]
---


## init app
npx comes with npm 5.2+ and higher
``` bash
# npx create-react-app react-semantic-ui-boilerplate
npx: installed 67 in 7.066s

Creating a new React app in /Users/me/react-semantic-ui-boilerplate.

Installing packages. This might take a couple of minutes.
Installing react, react-dom, and react-scripts...
...
✨  Done in 61.86s.

Success! Created react-semantic-ui-boilerplate at /Users/me/react-semantic-ui-boilerplate
Inside that directory, you can run several commands:

  yarn start
    Starts the development server.

  yarn build
    Bundles the app into static files for production.

  yarn test
    Starts the test runner.

  yarn eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd react-semantic-ui-boilerplate
  yarn start
  
# cd react-semantic-ui-boilerplate
# yarn start
Compiled successfully!

You can now view react-semantic-ui-boilerplate in the browser.

  Local:            http://localhost:3000/
  On Your Network:  http://192.168.0.103:3000/

Note that the development build is not optimized.
To create a production build, use yarn build.
```
Open http://localhost:3000/ on browser
{% asset_img 1.png %}

## Set up Semantic UI
``` bash
# yarn add semantic-ui-react semantic-ui-css font-awesome
```
Edit `App.js`
{% codeblock lang:javascript %}
import React, {Component} from 'react';
import './App.css';

import 'semantic-ui-css/semantic.min.css';
import 'font-awesome/css/font-awesome.css';
import {Button, Segment} from "semantic-ui-react";

class App extends Component {
    render() {
        return (
            <Segment padded={'very'}>
                <Button primary>Semantic UI is ready</Button>
                <Button secondary>Semantic UI is ready</Button>
            </Segment>
        );
    }
}

export default App;

{% endcodeblock %}

Refresh http://localhost:3000/ to make sure semantic ui works

{% asset_image 2.png %}

## Set up redux
``` bash
# yarn add react-redux

```


