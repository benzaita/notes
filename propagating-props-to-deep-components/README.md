# How do you propagate properties to "deeply nested" components?

By "deeply nested" I mean Button in this example hierarchy:

```
Page
  Header
  ListOfItems
    Item
    Item
      Button
```

More specifically: how do you provide the `onClick` handler to that `Button`?

We've tried this at first - explicitly "threading" the property through the hierarchy:

```
const Page = ({onEdit, ...}) => (
  <Header .../>
  <ListOfItems onEdit={onEdit} .../>
)

const ListOfItems = ({onEdit, items, ...}) =>
  items.map(item => <Item onEdit={onEdit} .../>)

const Item = ({onEdit, ...}) => (
  <Label .../>
  <Button onClick={onEdit}>Edit</Button>
)
```

However this is quite painful - every time we need a new handler, we need to "thread" it through the entire hierarchy.

So we decided to encapsulate the handlers in an object:

```
const Page = ({handlers, ...}) => (
  <Header .../>
  <ListOfItems handlers={handlers} .../>
)

const ListOfItems = ({handlers, items, ...}) =>
  items.map(item => <Item handlers={handlers} .../>)

const Item = ({handlers, ...}) => (
  <Label .../>
  <Button onClick={handlers.onEdit}>Edit</Button>
)
```

While this scales more easily (adding a new handler does not require "opening" all the components in the hierarchy), and is quite trivial to understand, there is a caveat to it.

We wanted to add `defaultProps` to this hierarchy. We decided to add `defaultProps` only to the component that actually needs that specific handler:

```
Item.defaultProps = {
   ...,
   handlers: {
     onEdit: () => {}
   }
}
```

This works fine if `ListOfItems` does not provide the `handlers` property of course. But what happens if the `handlers` object _does exist_ but simply does not contain the `onEdit` handler?

React only does a shallo merge of `defaultProps` with the actual properties, and therefore in this example:

```
const App = () =>
  <Page handlers={{onDelete: doSomething}}/>
```

the `handlers` property of `Item` does not contain the `onEdit` handler, although we defined it in `Item.defaultProps`.

While there are probably ways to make this work (implementing `getDefaultProps`? ditching `defaultProps` in favor of a custom logic?), we decided to use `React.createContext()` instead:

```
const HandlersContext = React.createContext({onEdit: () => {})

const App = () =>
  <HandlersContext.Provider value={{onDelete: doSomething}}>
    <Page handlers={{onDelete: doSomething()}}/>
  </HandlersContext.Provider>

...

const Item = () => (
  <HandlersContext.Consumer>
    { handlers =>
      <Button onClick={handlers.onEdit}>Edit</Button>
    }
  </HandlersContext.Consumer>
)
```

There is no need to "thread" the handlers through the hierarchy anymore, and we are now fine with React only shallow-merging the default properties, because we "flattened" the properties by introducing a new component (the `HandlersContext`).

How are you propagating properties in your component hierarchy?
