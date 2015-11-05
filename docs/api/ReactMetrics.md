## React Metrics API References

### <a id='metrics'></a>[`metrics(config, [options])(Component)`](#metrics)

Wraps your root level component and returns wrapped component which exposes `metrics` context where your child component can access APIs defined in your configuration.

#### Usage

ES6

```
class Application extends React.Component {
    ...
}

export default metrics(config, options)(Application);
```

ES7 decorator

```javascript
@metrics(config, options)
class Application extends React.Component {
    ...
}
```

#### `config`

A configuration object to create metrics instance.

Example:

```javascript
{
    enabled: true,
    vendors: [
        {
            name: "Google Analytics",
            api: new GoogleAnalytics({
                trackingId: "UA-********-*"
            })
        },
        {
            name: "Adobe Tag Manager"
            api: new AdobeTagManager({
                seedFile: "****"
            })
        }
    ],
    pageViewEvent: "pageLoad",
    pageDefaults: () => {
        return {
            siteName: "My Web Site",
            ...
        };
    },
    customParams: {
        ...
    },
    debug: true
}

```

- `vendors`(required) - An array of tracking api configuration object for each [vendor](/docs/Guides.md#vendor).
- `enabled`(optional) - A flag to enable or disable metrics functionality. Default value is `true`.
- `pageViewEvent`(optional) - A default page view event name. You can optionally override this value by sending other event name from page view call. Default value is `pageLoad`.
- `pageDefaults`(optional) - A function to return common page view tracking metrics that's sent for all page view call. This will receive [`routeState`](/docs/Guides.md#routeState) argument where you can use to send route specific information. Default value is a function which returns an empty object.
- `customParams`(optional) - An optional object which gets merged into `pageDefaults` if specified.
- `requestTimeout`(optional) - An optional time out value for the tracking request if specified. Default value is `15000` ms.
- `cancelOnNext`(optional) - An optional flag to indicate whether the pending request should be canceled when the route changes if specified. Default value is `true`.
- `getRouteState`(optional) - A function which returns the new [`routeState`](/docs/Guides.md#routeState) upon route change, returns `null` otherwise. This takes old and new props as arguments. Pass your own function to override [default logic](/src/react/getRouteState.js).
- `findRouteComponent`(optional) - A function which returns the route handler component. Pass your own function to override [default logic](/src/react/findRouteComponent.js).

#### `options`

Example:

```
{
    autoTrackPageView: false,
    useTrackBinding: true,
    attributePrefix: "custom"
}
```

- `autoTrackPageView`(optional) - A flag to indicate whether a page view is triggered automatically by a route change detection or not. Default value is `true`.
- `useTrackBinding`(optional) - A flag to indicate whether metrics should use track binding. Default value is `true`.
- `attributePrefix`(optional) - An element attribute prefix to use for tracking bining. Default value is `data-metrics`.

### <a id='exposeMetrics'></a>[`exposeMetrics`](#exposeMetrics)

Wraps your component and returns wrapped component which is aware of `willTrackPageView` static method in your component. `willTrackPageView` gets called when react-metrics [detects your component as route handler component](/docs/Guides.md#routeHandlerDetection).
`willTrackPageView` will receive [`routeState`](/docs/Guides.md#routeState).

#### Usage

ES6

```javascript
class MyComponent extends React.Component {
    ...
}

MyComponent.willTrackPageView = (routeState) => {
    return myTrackingData;
}

export default exposeMetrics(MyComponent);
```

ES7 decorator

```javascript
@exposeMetrics
class MyComponent extends React.Component {
    static willTrackPageView(routeState) {
        return myTrackingData;
    }
}
```

### <a id='PropTypes'></a>[`PropTypes`](#PropTypes)

| type | description          |
| ------------- | ----------- |
| metrics | A type which exposes all API methods|
| location | A type which contains [route information](/docs/Guides.md#location) |


### <a id='createMetrics'></a>[`createMetrics(config)`](#createMetrics)

Low level factory API which creates [metrics instance](/docs/api/Core.md#metrics-api-references). This can be used for non-React project or for decoupling metrics from React component by using it from Flux store or Redux middleware.

Example:

```javascript
// creating middleware for Redux

import {createMetrics} from "react-metrics";

const metrics = createMetrics(config);

export default function metricsMiddleware() {
    return next => action => {
        const returnValue = next(action);
        switch (action.type) {
            case ActionTypes.ROUTE_CHANGE:
                const {location} = action;
                const paths = location.pathname.substr(1).split("/");
                const routeState = location;
                metrics.setRouteState(routeState);
                metrics.api.pageView({
                    category: !paths[0] ? "landing" : paths[0]
                });
        }
        return returnValue;
    };
}
```