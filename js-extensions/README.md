# Jenkins JavaScript Extensions

Jenkins JavaScript Extensions are extensions which contribute to the UI in Jenkins BlueOcean.
This module is used to define extension points - locations where the application accepts plugin-provided implementations.
This module is also used to process the plugin extension point implementations to provide to BlueOcean.

This module is published via npm as `@jenkins-cd/js-extensions` so that other plugins, external to the blueocean project, can make use of it.
Plugins can themselves make use of extension points.

Jenkins JavaScript Extensions are based on the extensibility model already established by Jenkins, based on data and views, with the ability to inherit views based on parent data types.

Jenkins JavaScript Extensions: `@jenkins-cd/js-extensions` module exports:
- `Renderer` - a React component to conveniently render extensions
- `store` - the `ExtensionStore` instance (which must be initialized before it can be used)
- `classMetadataStore` - class/capability metadata store
- `dataType()` - function for filtering extensions based on the data type
- `componentType()` - function for filtering extensions based on the required component type (e.g. React class)

### ExtensionStore API

The `ExtensionStore` API is very simple, all public methods are asynchronous:

- `getExtensions(extensionPointName, [filter,] onload)`
    This method will async load data, filter the extensions based on the provided `filter`s, and call the onload handler with a list of extension exports, e.g. the React classes or otherwise exported references.
    `filter` - a filter function currently the module exports the following functions - see the exported functions for the commonly used filters

### ClassMetadataStore API

- `getClassMetadata(dataType, onload)`
    This will return a list of type information, from the [classes API](../blueocean-rest/README.md#classes_API), this method also handles caching results locally.

### Rendering extension points

The most common usage pattern is to use the exported `Renderer`, specifying the extension point name, any necessary contextual data, and optionally specifying a data type.

    import Extensions from '@jenkins-cd/js-extensions';
    ...
    <Extensions.Renderer extensionPoint="jenkins.navigation.top.menu" />

For example, rendering the test results for a build may be scoped to the specific type of test results in this manner:

    <Extensions.Renderer extensionPoint="test-results-view" filter={dataType(data)} testResults={data} />

The `ExtensionRenderer` component optionally uses the [classes API](../blueocean-rest/README.md#classes_API) to look up an appropriate, specific set of views for the data being displayed.
This should works seamlessly with other [capabilities](../blueocean-rest/README.md#capabilities).


### Defining extension point implementations

Extensions are defined in a `jenkins-js-extensions.yaml` file in the javascript source directory of a plugin by defining a list of extensions similar to this:

    # Extensions in this plugin
    extensions:
      - component: AboutNavLink
        extensionPoint: jenkins.topNavigation.menu
      - component: components/tests/AbstractTestResult
        extensionPoint: jenkins.test.result
        dataType: hudson.tasks.test.AbstractTestResultAction

Properties are:
- `component`: a module from which the default export will be used
- `extensionPoint`: the extension point name
- `dataType`: an optional Java data type this extension handles

For example, the `AboutNavLink` might be defined as a default export:

    export default class NavLink extends React.Component {
        ...
    }

#### Enforcing specific component types

In order to ensure a specific component is returned, an extension point may also use the `componentType` filter - it accepts an object prototype (e.g. an ES6 class), e.g.:

    import TestResults from './base-components/TestResults';
    ...
    <Extensions.Renderer extensionPoint="test-view" filter={componentType(TestResults)} ... />

Extensions are not limited to React components.
The `componentType` filter will attempt to match returned components by a series of prototype and typeof checks to appropriately filter returned types including ES6 classes.
