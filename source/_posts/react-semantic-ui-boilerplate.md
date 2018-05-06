---
title: React Semantic UI boilerplate
date: 2018-05-03 07:50:15
tags: [react, redux, redux-saga, semantic-ui]
---


## init app
npx comes with npm 5.2+ and higher
``` bash
# npx create-react-app react-semantic-ui-boilerplate
# cd react-semantic-ui-boilerplate
# yarn start
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
# yarn add react-redux reduxsauce seamless-immutable

```

Edit `index.js`
{% codeblock lang:javascript %}
import React from 'react';
import ReactDOM from 'react-dom';
import './styles/index.css';
import Root from './containers/Root';
import registerServiceWorker from './registerServiceWorker';
import configureStore from "./redux/configureStore";

const initialState = {};
const store = configureStore(initialState);

ReactDOM.render(<Root store={store}/>, document.getElementById('root'));
registerServiceWorker();

{% endcodeblock %}

Create `src\containers\Root.js`

{% codeblock lang:javascript %}
import React from 'react';
import {Provider} from 'react-redux';
import App from './App';

export default class Root extends React.Component {

    render() {
        return (
            <Provider store={this.props.store}>
                <App>
                </App>
            </Provider>
        );
    }
}
{% endcodeblock %}

Create `src\redux\configureStore.js`
{% codeblock lang:javascript %}
import {applyMiddleware, createStore, compose} from 'redux';

import reducers from '../reducers';

export default function configureStore(initialState) {

    const middleware = [];
    const enhancers = [];

    // add middleware here

    enhancers.push(applyMiddleware(...middleware));

    const store = createStore(reducers, initialState, compose(...enhancers));

    return store

}
{% endcodeblock %}

Create `src\reducers\index.js`
{% codeblock lang:javascript %}
import {combineReducers} from 'redux';

export default combineReducers({
    user: require('../reducers/UserRedux').reducer,
});

{% endcodeblock %}

And reducer `src\reducers\UserRedux.js`
{% codeblock lang:javascript %}
import {createReducer, createActions} from 'reduxsauce'
import Immutable from 'seamless-immutable'

const {Types, Creators} = createActions({
    getUser: null,
    getUserSuccess: ['data'],
    getUserFail: ['error']
}, {prefix: 'USER'});

export const UserTypes = Types;
export default Creators


export const INITIAL_STATE = Immutable({
    loading: false,
    users: []
});

const getUser = (state) => {
    return state.merge({loading: true})
};

const getUserSuccess = (state, {data}) => {
    return state.merge({loading: false, users: data})
};

const getUserFail = (state, {error}) => {
    return state.merge({loading: false, errors: error})
};

export const reducer = createReducer(INITIAL_STATE, {
    [Types.GET_USER]: getUser,
    [Types.GET_USER_SUCCESS]: getUserSuccess,
    [Types.GET_USER_FAIL]: getUserFail,
});
{% endcodeblock %}


Move `App.js` to `src\containers\App.js`

{% codeblock lang:javascript %}
import React, {Component} from 'react';
import '../styles/App.css';

import 'semantic-ui-css/semantic.min.css';
import 'font-awesome/css/font-awesome.css';
import {Button, Segment} from "semantic-ui-react";

import {connect} from 'react-redux';
import UserActions from '../reducers/UserRedux'

class App extends Component {
    render() {
        return (
            <Segment padded={'very'}>
                <div>Loading: {this.props.loading ? "Yes" : "No"}</div>
                <Button onClick={() => {
                    this.props.getUser()
                }} primary>Get User</Button>
            </Segment>
        );
    }
}


const mapStateToProps = (state, ownProps) => {
    return {
        loading: state.user.loading,
    };
};

const mapDispatchToProps = (dispatch) => {
    return {
        getUser: () => dispatch(UserActions.getUser())
    }
};

export default connect(mapStateToProps, mapDispatchToProps)(App);
{% endcodeblock %}

Refresh http://localhost:3000/, `loading: No` initial state works

{% asset_img 3.png %}

Click Get User, `loading: Yes`, state `user.loading` changed

## Set up redux-saga
``` bash
# yarn add redux-saga
```
Edit `index.js`

{% codeblock lang:javascript %}
...
import rootSagas from './sagas'
...
const store = configureStore(initialState, rootSagas);
...
{% endcodeblock %}

Edit `src\redux\configureStore.js`
{% codeblock lang:javascript %}
...
import createSagaMiddleware from 'redux-saga'
export default function configureStore(initialState, rootSagas) {
    ...
    // add middleware here
    const sagaMiddleware = createSagaMiddleware();
    middleware.push(sagaMiddleware);
    ...
    sagaMiddleware.run(rootSagas);
    return store
}
{% endcodeblock %}

Create `src\sagas\index.js`
{% codeblock lang:javascript %}
import {takeLatest, all} from 'redux-saga/effects'
import {UserTypes} from "../reducers/UserRedux";
import {getUser} from "./UserSaga";


export default function* root() {
    yield all([
        takeLatest(UserTypes.GET_USER, getUser),
    ])
}
{% endcodeblock %}


And `src\sagas\UserSaga.js`
{% codeblock lang:javascript %}
import {put, call} from 'redux-saga/effects'
import {delay} from 'redux-saga'
import UserActions from '../reducers/UserRedux'

export function* getUser(action) {

    yield call(delay, 1000);

    // do async task, call api,...
    const api_response = [{
        id: 1,
        name: 'Tony'
    }];

    yield put(UserActions.getUserSuccess(api_response))
}
{% endcodeblock %}

Edit `src\containers\App.js`
{% codeblock lang:javascript %}
...
render() {
    return (
        <Segment padded={'very'}>
            <div> loading: {this.props.loading ? "Yes" : "No"}</div>
            <div> users: {JSON.stringify(this.props.users)}</div>
            <Button onClick={() => {
                this.props.getUser()
            }} primary>Get User</Button>
        </Segment>
    );
}
...
const mapStateToProps = (state, ownProps) => {
    return {
        loading: state.user.loading,
        users: state.user.users,
    };
};
...   
{% endcodeblock %}

Refresh http://localhost:3000/

{% asset_img 4.png %}

state `user.users` == `[]`, initial state works

Click Get User, wait .5 second, got user object, saga works!

{% asset_img 5.png %}

## Set up react-router

``` bash
# yarn add react-router react-router-dom react-router-redux@v5.0.0-alpha.9 history
```

Edit `index.js`
{% codeblock lang:javascript %}
...
import createHistory from 'history/createBrowserHistory';
...
const history = createHistory();
const store = configureStore(initialState,rootSagas,history);

ReactDOM.render(<Root store={store} history={history}/>, document.getElementById('root'));
...
{% endcodeblock %}

Edit `src\redux\configureStore.js`
{% codeblock lang:javascript %}
...
import {routerMiddleware} from 'react-router-redux';
...
export default function configureStore(initialState, rootSagas, history) {
    ...
    const reduxRouterMiddleware = routerMiddleware(history);
    middleware.push(reduxRouterMiddleware);
    
    enhancers.push(applyMiddleware(...middleware));
    ...
}
{% endcodeblock %}

Edit `src\containers\Root.js`
{% codeblock lang:javascript %}
...
import {ConnectedRouter} from 'react-router-redux';
import routes from '../routes';
...
render() {
    return (
        <Provider store={this.props.store}>
            <App>
                <ConnectedRouter history={this.props.history}>
                    {routes}
                </ConnectedRouter>
            </App>
        </Provider>
    );
}
...
{% endcodeblock %}

Edit `src\containers\App.js`
```javascript
import React, {Component} from 'react';
import '../styles/App.css';

import 'semantic-ui-css/semantic.min.css';
import 'font-awesome/css/font-awesome.css';

import {connect} from 'react-redux';

class App extends Component {
    render() {
        return (
            <div className='App'>
                {this.props.children}
            </div>
        );
    }
}


const mapStateToProps = (state) => {
    return {};
};

const mapDispatchToProps = (dispatch) => {
    return {
    }
};

export default connect(mapStateToProps, mapDispatchToProps)(App);
```

Create `src\routes.js`
{% codeblock lang:javascript %}
import React from 'react';
import {Route, Switch} from 'react-router';
import {
    HomeView,
    NotFoundView,
    AboutView
} from "./containers";

export default (
    <Switch>
        <Route exact path="/" component={HomeView}/>
        <Route path="/about" component={AboutView}/>
        <Route path="*" component={NotFoundView}/>
    </Switch>
);
{% endcodeblock %}

Create `src\containers\index.js`
```javascript
export {default as HomeView} from './HomeView'
export {default as NotFoundView} from './NotFoundView'
export {default as AboutView} from './AboutView'

```

Create `src\containers\HomeView.js`
{% codeblock lang:javascript %}
import React from 'react';
import {connect} from "react-redux";
import MainLayout from "../components/MainLayout";
import {Segment} from "semantic-ui-react";

class HomeView extends React.Component {
    render() {
        return (
            <MainLayout>
                <Segment>Home</Segment>
            </MainLayout>
        )
    }
}

const mapStateToProps = (state) => {
    return {}
};

const mapDispatchToProps = (dispatch) => {
    return {}
};

export default connect(mapStateToProps, mapDispatchToProps)(HomeView);
{% endcodeblock %}

Create `src\containers\AboutView.js`
{% codeblock lang:javascript %}
import React from 'react';
import MainLayout from "../components/MainLayout";
import {Segment} from "semantic-ui-react";

export default class AboutView extends React.Component {
    render() {
        return (
            <MainLayout>
                <Segment>About us</Segment>
            </MainLayout>
        );
    }
}
{% endcodeblock %}

Create `src\containers\NotFoundView.js`
```javascript
import React from 'react';
import EmptyLayout from "../components/EmptyLayout";

export default class AboutView extends React.Component {
    render() {
        return (
            <EmptyLayout>
                <h1>404 Not found</h1>
            </EmptyLayout>
        );
    }
}

```

And `src\components\MainLayout.js`
``` javascript
import React, {Component} from 'react';
import {Container, Menu} from "semantic-ui-react";
import {push} from 'react-router-redux';
import {connect} from "react-redux";

class MainLayout extends Component {
    render() {
        return (
            <div style={{flex: 1}}>
                <Menu style={{flex: 1}} fixed='top'>
                    <Container>
                        <Menu.Item header>
                            Hello world
                        </Menu.Item>
                        <Menu.Item onClick={() => {
                            this.props.dispatch(push('/'))
                        }} as='a'>Home</Menu.Item>
                        <Menu.Item onClick={() => {
                            this.props.dispatch(push('/about'))
                        }} as='a'>About</Menu.Item>
                    </Container>
                </Menu>
                <Container style={{marginTop: '60px'}}>
                    {this.props.children}
                </Container>
            </div>
        )
    }
}

const mapStateToProps = (state) => {
    return {}
};

const mapDispatchToProps = (dispatch) => {
    return {
        dispatch
    }
};


export default connect(mapStateToProps, mapDispatchToProps)(MainLayout);


```


And `src\components\EmptyLayout.js`
``` javascript
import React, {Component} from 'react';
import {Container, Segment} from "semantic-ui-react";

class EmptyLayout extends Component {
    render() {
        return (
            <Container>
                <Segment style={{flex: 1}}>
                    {this.props.children}
                </Segment>
            </Container>

        )
    }
}

export default EmptyLayout;
```

It's time to test http://localhost:3000/

`HomeView`
{% asset_img 6.png%}

`AboutView`
{% asset_img 7.png%}

`fake view`
{% asset_img 8.png%}

Make sure top menu home, about and browser back button work

## Set up API client
```bash
yarn add apisauce
```
Create `src\services\Api.js`

```javascript
import apisauce from 'apisauce'

const create = (baseURL = 'https://jsonplaceholder.typicode.com') => {

    const api = apisauce.create({
        // base URL is read from the "constructor"
        baseURL: baseURL,
        // here are some default headers
        headers: {
            'Cache-Control': 'no-cache',
            'Content-Type': 'application/json',
        },
        // 10 second timeout...
        timeout: 10000
    });

    const getUser = () => api.get('/users');

    return {
        getUser
    }
};

export default {
    create
}
```
Edit `src\sagas\index.js`
```javascript
...
import API from '../services/Api'
export const api =  API.create('https://jsonplaceholder.typicode.com');
export default function* root() {
    yield all([
        takeLatest(UserTypes.GET_USER, getUser, api),
    ])
}
...
```
Edit `src\sagas\UserSaga.js`
```javascript
import {put, call} from 'redux-saga/effects'
import UserActions from '../reducers/UserRedux'
import {delay} from "redux-saga";


export function* getUser(api, action) {

    const response = yield call(api.getUser);
    
    // more delay to see loading indicator
    yield call(delay, 1000);

    if (response.ok) {
        yield put(UserActions.getUserSuccess(response.data))
    } else {
        yield put(UserActions.getUserFail(response.body))
    }

}
```
Edit `src\containers\HomeView.js`
```javascript
import React from 'react';
import {connect} from "react-redux";
import MainLayout from "../components/MainLayout";
import {Container, Header, Segment, Table} from "semantic-ui-react";
import UserActions from "../reducers/UserRedux";

class HomeView extends React.Component {
    componentDidMount() {
        this.props.getUser()
    }

    _render_row = (record) => {
        return (<Table.Row key={record.id}>
            <Table.Cell collapsing>
                {record.id}
            </Table.Cell>
            <Table.Cell>
                {record.name}
            </Table.Cell>
            <Table.Cell>{record.phone}</Table.Cell>
            <Table.Cell>{record.website}</Table.Cell>
        </Table.Row>)
    };

    render() {
        return (
            <MainLayout>
                <Container textAlign={'left'}>
                    <Header>Home</Header>
                    <Segment style={{minHeight:300}} basic loading={this.props.loading}>
                        <Table>
                            <Table.Header>
                                <Table.Row>
                                    <Table.HeaderCell>ID</Table.HeaderCell>
                                    <Table.HeaderCell>Name</Table.HeaderCell>
                                    <Table.HeaderCell>Phone</Table.HeaderCell>
                                    <Table.HeaderCell>Website</Table.HeaderCell>
                                </Table.Row>
                            </Table.Header>
                            <Table.Body>
                                {this.props.users.map((record) => {
                                    return this._render_row(record)
                                })}
                            </Table.Body>
                        </Table>
                    </Segment>
                </Container>
            </MainLayout>
        )
    }
}

const mapStateToProps = (state) => {
    return {
        users: state.user.users,
        loading: state.user.loading,
    }
};

const mapDispatchToProps = (dispatch) => {
    return {
        getUser: () => dispatch(UserActions.getUser())
    }
};

export default connect(mapStateToProps, mapDispatchToProps)(HomeView);

```

Refresh http://localhost:3000/, we got home page

{% asset_img 10.png %}

## Log and debug with Reactotron
```javascript
# yarn add --dev reactotron-apisauce reactotron-react-js reactotron-redux reactotron-redux-saga
```

Edit `src\redux\configureStore.js`
```javascript
if (process.env.NODE_ENV === 'production') {
    module.exports = require('./configureStore.prod');
} else {
    module.exports = require('./configureStore.dev');
}
```

Create `src\configureStore.dev.js`
```javascript
import {applyMiddleware, compose} from 'redux';
import {routerMiddleware} from 'react-router-redux';
import createSagaMiddleware from 'redux-saga'
import reducers from '../reducers';
import Reactotron from 'reactotron-react-js'
import {api} from '../sagas'
import '../config/Reactotron'

api.addMonitor(Reactotron.apisauce);

export default function configureStore(initialState, rootSagas, history) {

    const middleware = [];
    const enhancers = [];

    // add middleware here
    const sagaMonitor = Reactotron.createSagaMonitor();
    const sagaMiddleware = createSagaMiddleware({sagaMonitor});
    middleware.push(sagaMiddleware);

    const reduxRouterMiddleware = routerMiddleware(history);
    middleware.push(reduxRouterMiddleware);

    enhancers.push(applyMiddleware(...middleware));

    const store = Reactotron.createStore(reducers, initialState, compose(...enhancers));

    sagaMiddleware.run(rootSagas);

    return store

}
```

Create `src\configureStore.prod.js`
```javascript
import {applyMiddleware, createStore, compose} from 'redux';
import createSagaMiddleware from 'redux-saga'
import {routerMiddleware} from 'react-router-redux';

import reducers from '../reducers';


export default function configureStore(initialState, rootSagas, history) {

    const middleware = [];
    const enhancers = [];

    // add middleware here
    const sagaMiddleware = createSagaMiddleware();
    middleware.push(sagaMiddleware);

    const reduxRouterMiddleware = routerMiddleware(history);
    middleware.push(reduxRouterMiddleware);

    enhancers.push(applyMiddleware(...middleware));

    const store = createStore(reducers, initialState, compose(...enhancers));

    sagaMiddleware.run(rootSagas);

    return store

}
```

Edit `src\services\Api.js`
```javascript
...
const create = (baseURL = 'https://jsonplaceholder.typicode.com') => {
    ...
    return {
        addMonitor:api.addMonitor,
        getUser
    }
    ...
}
...
```

Create `src\config\Reactotron.js`
```javascript
import Reactotron from 'reactotron-react-js'
import {reactotronRedux} from 'reactotron-redux'
import sagaPlugin from 'reactotron-redux-saga'
import Immutable from 'seamless-immutable'
import apisaucePlugin from 'reactotron-apisauce'

import {trackGlobalErrors} from 'reactotron-react-js'

Reactotron
    .configure({name: 'React boilerplate'}) // we can use plugins here -- more on this later
    .use(apisaucePlugin())
    .use(trackGlobalErrors({offline: false}))
    .use(reactotronRedux({onRestore: Immutable}))
    .use(sagaPlugin())
    .connect(); // let's connect!

Reactotron.clear();
```

Download and open Reactotron.app (https://github.com/infinitered/reactotron)

Reload react app and see how Reactotron log action, saga, api response

{% asset_img 11.png %}

View state `user`

{% asset_img 12.png %}

Finish!