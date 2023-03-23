# <a name="introduction"></a> Introduction

**Carthus** is a set of tools you can use to build scalable react applications. It provides certain realisation
and relationship between different react elements (such as components, data layers, routes etc.) as well as code
generation cli.

## <a name="introduction-getting_started"></a> Getting Started

Install **@carthus/cli** as global dependency

```bash
npm i @carthus/cli -g
```

Initialize Carthus inside your **React** project

```bash
cts init
```

It will install necessary dependencies for react app: **@carthus/core** and **@carthus/react**
and additionally create config file **carthus.config.json** in your project root folder.


## <a name="introduction-check_available_templates"></a> Check available templates

Now check is everything setup. Run **list** command to output all available schemas.

```bash
cts list
```

## <a name="introduction-genrate_component"></a> Generate your first component

To generate code from schema you have to execute:

```bash
cts schema-alias ComponentName ./path/to/result/folder [flags]
```

Example:

```bash
cts cm App ./src/components -f
```

**Hint:** flag **"-f"** means you want to put all generated files in folder with provided component name
so in this case will be written in **./src/components/App/** instead of **./src/components/**


# <a name="config"></a> Config file

```json
{
  "extensions": [
    "external.lib",
    "./local/lib/index.js"
  ], 
  "alias": {
    "alias": "/path/"
  }, 
  "flag": {
    "templateName": {
      "flag": "value"
    }
  }
}
```

## <a name="config-extensions"></a> Extensions

Extensions field contains all links to template libraries that will be provided to cli.

Your can provide external npm packages like **@carthus/react** as well as local custom templates.

**Note:** path to local template starts with **./**, be relative to **carthus.confgi.json** location
and lead to js file that exports array of custom schemas.


## <a name="config-alias"></a> Alias

Alias allows to shortcut path to folder where generated component shold locate.

Basically, if we configure alias:

```json
{
  "alias": {
    "forms": "./src/features/forms"
  }
}
```

These two calls will be identical:

```bash
cts cm NewForm ./src/features/forms -f
cts cm NewForm forms -f
```
**Note:** Some schemas has it own dependency on other schemas, all flags (configured and manually written) will be shared across dependencies

## <a name="config-flag"></a> Flag

This property configures default flags provided to templates.

Let's say I want to run generation of all **cm** templates with flag **-f** by default.
```json
{
  "flag": {
    "cm": {
      "f": true
    }
  }
}
```

Then we got same behavior of these two lines:

```bash
cts cm App ./src -f
cts cm App ./src
```


**Note:** some templates have multiple names they can be called (**ex:** cm, component).
Auto flag will be provided only for specified name in config.


# <a name="react_elements"></a> React elements

Consider react application as a collection of some functional elements that interact with each other. We can arrange this elements in sections:

- View -- rendering jsx
- Logic -- handling any logic, manipulating states etc
- Data Layer -- providing any external data for your application form any source (fetch, localStorage etc.)
- Routing -- configuring application routing, handling access rights
- Testing & Error tolerance -- handling errors in elements itself, and it's interactions

**React elements** provides solution to 

## <a name="react_elements-component"></a> Component (cm, component)

Main propose of component is linking [View](#react_elements-view) and [Services](#react_elements-service) to one abstraction. Basically it's the most common kind of react element
that pages, ui elements etc. are built with.

Component are created by function **CreateComponent** from **@carthus/core**. This function accepts two fields: 

- View -- link to [View](#react_elements-view) element
- providers -- array of providers that **View** will be wrapped in. Nesting providers executes in order direction. 

```typescript
const MyComponent = CreateComponent({
    View: null,
    providers: []
});
```

providers array can contain as direct links to any react function component that can be used as wrapper, primarily it used for wrapping **View** with
**Context.Provider**. 

In case if you want to pass props to wrapper, it possible to pass object

```typescript
const MyComponent = CreateComponent({
    View: null,
    providers: [{
        provider: ProviderFunction, 
        props: {/* ...props here */}
    }]
});
```


Usage:
```bash
cts cm App ./src/components
```
Optional flags:

- -p -- initialize component with [Provider](#react_elements-provider) with same name as component 


## <a name="react_elements-view"></a> View (cm-view, component-view)

View takes responsibility for component display **only**. Any other logic should be encapsulated in Service.

This element should be placed inside some Component directory in folder "View"

Usage:
```bash
cts cm-view NewPage ./src/components/ComponentName/View
```
Optional flags:

- -scss -- additionally creates .scss file
- scss-module -- additionally creates .module.scss file


## <a name="react_elements-service"></a> Service (srv, service)

Services are regular react hooks that used to handle all logic for it's [Component](#react_elements-component)

This element should be placed inside some Component directory in folder "Service"

Usage:
```bash
cts srv Name ./src/components/ComponentName/Service
```

## <a name="react_elements-provider"></a> Provider (provider)

Provider creates [Service](#react_elements-service) and allows to share it's state with context [Component](#react_elements-component)

This element should be placed inside some Component directory in folder "Service"

Usage:
```bash
cts provider Name ./src/components/ComponentName/Service -f
```


## <a name="react_elements-data"></a> Data (data)

**Note:** to use this element you have to install [zod](https://www.npmjs.com/package/zod).

Data component is used to manipulate any plain data in your application: http requests to server, read/write localStorage, write raw data etc.

Data component created by function **CreateData** from **@carthus/core**

```typescript
const MyData = CreateData({
    Schema: z.object({ value: z.string() }),
    Source: (a: number) => ({ value: a**2 }),
    middleware: [] as const
})

MyData(10)
```

Here you have to provide

- Schema -- zod object that describes received data
- Source -- function that returns expected data. It can be either sync or async function
- middleware -- array is chain of function that can add and modify but **can't** remove fields from received object. Each function is provided result of previous function as first argument.

**Note:** if data doesn't match schema error will be thrown.

Usage:
```bash
cts data UserInfo ./src/data -f
```


## <a name="react_elements-router"></a> Router (router)

**Note:** to use Router you have to install [react-router-dom v6.4+](https://www.npmjs.com/package/react-router-dom).

Router component used to handle all you application routes in one gateway.
It creates extra route by itself, there you can set **Component** as a link to [Component](#react_elements-component) element or any other react component

Also, you can provide **Guard** as link to [Guard](#react_elements-guard) component.

And finally, you can insert in routes array links to other [Routers](#react_elements-router) or [Routes](#react_elements-route).

**Note:** Router doesn't nest children routes inside itself. It means Router [Guard](#react_elements-guard) works only component of it's Router 

Router component created by function **CreateRouter** from **@carthus/core**

```typescript
const MainRouter = CreateRouter({
    path: "/",
    Guard: null,
    Component: null,
    routes: []
})
```


Usage:
```bash
cts router Main ./src/routes -g
```

- -g -- additionally creates [Guard](#react_elements-guard)


## <a name="react_elements-route"></a> Route (route)

Router component created by function **CreateRoute** from **@carthus/core**

It has same interface as [Router](#react_elements-router), but Route unlike [Router](#react_elements-router) can nest routes.
It means Route Guard](#react_elements-guard) will work for all nested routes and nested paths will be concatenated.

Usage:
```bash
cts route Home ./src/routes -g -f
```

- -g -- additionally creates [Guard](#react_elements-guard)

## <a name="react_elements-guard"></a> Guard (guard)

Guard allows you to control user's flow through route.

Example:
```typescript jsx
const Guard: FC<IMainGuardProps> = ({...props}) => {
    // any logic here...
    return <Outlet />
};
```

Usage:
```bash
cts guard Main ./src/routes
```
