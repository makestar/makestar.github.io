---
title: "React.js V16.3.2 적용"
date: 2018-08-13 18:50:28 -0400
categories: react
author: Shane
---
곧 있으면 React v17도 나오고 우리가 쓰고 있는 v15.x에서 v16.x로도 많은 변화가 있었기 때문에 그에 대비해서 v16.3.x로 업데이트를 진행하였다. 업데이트를 진행하면서 React내에 변화가 좀 있었기에 React기반 라이브러리들도 모두 업데이트가 필요했다. React를 업데이트 하면서 변화된 내용들과 그 기반 라이브러리들을 업데이트 하는데 필요한 작업들을 작성해보았다.

Upgrade React v15.x → v16.3.2
------------------------------
### install react with react-dom

react에 포함되어 있던 dom관련 라이브러리들을 **react-dom** 으로 따로 분류되었다. 그래서 **react-dom** 을 같이 설치해주어야 한다.


### Change LifeCycle

v17에서 **componentWillMount**, **componentWillReceiveProps**, **ComponentWillUpdate** method들이 deprecation될 예정이다. v16까지는 warning으로 알려주기만 한다. 이 method들을 바꿔줘야 한다. 또한 이것들 대용으로 **getDerivedStateFromProps** 와 **getSnapshowBeforUpdate** 가 추가 되었다.

* getDerivedStateFromProps - componentWillReceiveProps의 대안으로 추가되었다.

* getSnapshowBeforUpdate - 안전하게(예를들면 update로 인해 만들어지기 DOM으로 부터) property들을 읽을 수 있도록 제공한다.


### Error Handling

이번에 업데이트가 되면서 에러 핸들링을 할 수 있는 함수가 추가 되었다. **ComponentDidCatch** Method를 사용하여 로그를 추적하거나 Fallback UI를 보여줄 수 있게 되었다. 아래는 예시 코드이다.

```
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false };
    }

    componentDidCatch(error, info) {
        // Display fallback UI
        this.setState({ hasError: true });
        // You can also log the error to an error reporting service
        logErrorToMyService(error, info);
    }

    render() {
        if (this.state.hasError) {
            // You can render any custom fallback UI
            return <h1>Something went wrong.</h1>;
        }
        return this.props.children;
    }
}
```


### Migrating from React.PropTypes

React에 포함되어 있던 **PropTypes** 가 따로 나오게 되었다. 그래서 **prop-types** 이라는 라이브러리 다운받고 사용해야 했다.

```
// Before (15.4 and below)
import React from 'react';

class Component extends React.Component {
  render() {
    return <div>{this.props.text}</div>;
  }
}

Component.propTypes = {
  text: React.PropTypes.string.isRequired,
}

// After (15.5)
import React from 'react';
import PropTypes from 'prop-types';

class Component extends React.Component {
  render() {
    return <div>{this.props.text}</div>;
  }
}

Component.propTypes = {
  text: PropTypes.string.isRequired,
};
```


### Migrating from React.createClass

React에 포함되어 있던 createClass가 따로 나오게 되었다. 그래서 create-react-class 라이브러리를 다운받고 **createReactClass** 을 대신 사용하면 된다.

```
// Before (15.4 and below)
var React = require('react');

var Component = React.createClass({
  mixins: [MixinA],
  render() {
    return <Child />;
  }
});

// After (15.5)
var React = require('react');
var createReactClass = require('create-react-class');

var Component = createReactClass({
  mixins: [MixinA],
  render() {
    return <Child />;
  }
});
```

### Discontinuing support for React Addons

앞으로 addons와 관련된 라이브러리들을 지원하지 않는다.  우리도 사용하던 라이브러리 **react-addons-css-transition-group** 을 **react-transition-group** 로 교체하였다. 관계된 라이브러리들은 다음과 같다.

* react-addons-create-fragment – React 16 will have first-class support for fragments, at which point this package won’t be necessary. We recommend using arrays of keyed elements instead.

* react-addons-css-transition-group - Use [react-transition-group/CSSTransitionGroup][css-trans] instead. Version 1.1.1 provides a drop-in replacement.

* react-addons-linked-state-mixin - Explicitly set the `value` and `onChange` handler instead.

* react-addons-pure-render-mixin - Use `React.PureComponent` instead.

* react-addons-shallow-compare - Use `React.PureComponent` instead.

* react-addons-transition-group - Use [react-transition-group/TransitionGroupinstead][trans-group]. Version 1.1.1 provides a drop-in replacement.

* react-addons-update - Use [immutability-helper][imm-helper] instead, a drop-in replacement.

* react-linked-input - Explicitly set the `value` and `onChange` handler instead.



### Ref 사용방법 변경

기존에는 ref=this.myRef로 사용했으면 됬지만 이제는 constructor에 미리 명시를 해주고 사용할때도 ref={this.myRef}로 사용하여야 한다. 자세한 설명은 다음 링크 참조

##### [Refs and the DOM - React][ref-dom]

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

### Accessing Refs

ref에 접근해서 사용하기 위한 방법도 바꼇다. current를 추가해주어야 한다.
`const node = this.myRef.current;`

Upgrade Redux-auth-wrapper v1 → v2
----------------------------------

업그레이드가 되면서 함수에 인자들이 바뀌게 되었다. 자세한건 아래 링크 참조
##### [Migrating from V1 · GitBook][redux-auth]


```
// Before (1.x and below)
import { UserAuthWrapper } from 'redux-auth-wrapper'
import { getSession } from '../../selectors/session'

const AuthWrapper = UserAuthWrapper({
  authSelector: getSession,
  predicate: (session) => session.isLoggedIn,
  redirectQueryParamName: 'pathname',
  wrapperDisplayName: 'AuthWrapper'
})

export default AuthWrapper

// After (2.x)
import { connectedRouterRedirect } from 'redux-auth-wrapper/history3/redirect'

const AuthWrapper = connectedRouterRedirect({
  redirectPath: '/login',
  authenticatedSelector: state => state.session.session !== null && state.session.session.isLoggedIn,
  wrapperDisplayName: 'AuthWrapper'
})

export default AuthWrapper
```

Switch  from react-addons-css-transition-group to react-transition-group
-----------------------------------------------------------------

더 이상 addons관련 라이브러리를 지원하지 않음에 따라 기존 오피셜 라이브러리로 변경하였다. 기존에 addons라이브러리에서 사용하던 ReactCSSTransitionGroup에서 새로운 라이브러리에서 CSSTransition로 바꾸었다. 자세한 내용은 아래 링크 참조
##### [React Transition Group][trans-group2]


```
// Before
import LazyLoad from 'react-lazy-load'
import React, {Component} from 'react'
import ReactCSSTransitionGroup from 'react-addons-css-transition-group';

export default class TransitionLazyLoad extends Component {
  render() {
    let { children, ...props } = this.props


    return (
      <LazyLoad { ...props }>
        <ReactCSSTransitionGroup key="1"
                                 transitionName="fade"
                                 transitionAppear={true}
                                 transitionAppearTimeout={2000}
                                 transitionEnter={false}
                                 transitionLeave={false}>
          { children }
        </ReactCSSTransitionGroup>
      </LazyLoad>
    )
  }
}


// After
import LazyLoad from 'react-lazy-load'
import React, {Component} from 'react'
import {CSSTransition} from 'react-transition-group'

export default class TransitionLazyLoad extends Component {
  state = {
    show: false
  }

  render() {
    let {children, ...props} = this.props

    return (
      <LazyLoad { ...props } onContentVisible={() => {
        this.setState(state => ({
          show: true
        }));
      }}>
        <CSSTransition
          in={this.state.show}
          timeout={500}
          classNames="fade"
        >
          {children}
        </CSSTransition>
      </LazyLoad>
    )
  }
}
```

Upgrade react-scrollspy
--------------------------

react-scrollspy모듈을 업그레이드 하면서 모듈을 불러오는 방식이 바뀌었다.

```
// Before
import {Scrollspy} from 'react-scrollspy'


// After
import Scrollspy from 'react-scrollspy'
```


후기
---------
이번 업데이트를 하면서 2년전에 셋팅했던 것들이라서 많이 바뀌었다. 서버사이드 렌더나 클라이언트 사이드 렌더에서 성능향상이 있었다. 그리고 react-router도 v3에서 v4로 업그레이드를 해야하지만 손이 많이 가는 작업이기에 이번 업데이트를 하고 다음 단계로 진행하기로 하였다. 또한 서버사이드 렌더성능 향상을 위해서 노드 버젼을 올려야 할 것 같다.

또한 React버젼이 업됨에 따라서 라이브러리들도 업데이트를 해줘야 하는데 안된 라이브러리들이 몇개 있었다. 살아있는 라이브러리를 사용하는 중요성에 대해 다시 한번 깨닫게 되었다.


[css-trans]: https://github.com/reactjs/react-transition-group
[trans-group] : https://github.com/reactjs/react-transition-group
[imm-helper] : https://github.com/kolodny/immutability-helper
[ref-dom] : https://reactjs.org/docs/refs-and-the-dom.html
[redux-auth] : https://mjrussell.github.io/redux-auth-wrapper/docs/Migrating.html
[trans-group2]: http://reactcommunity.org/react-transition-group/
