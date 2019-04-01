WIP WIP WIP WIP WIP WIP WIP WIP WIP WIP WIP WIP WIP WIP 

# Reusable Shared-State Logic in ReactJS

Until Hooks were introduced, React did not offer a way to isolate reusable behaviour and reuse it in different components. Hooks offer a primitive to isolate and reuse stateful behaviour between components — “stateful” being local state. But what about isolating and reusing behaviour that requires shared state?

## “Local state” vs. “Shared state”

With “local state” I refer to state which is stored and consumed by the same component, for example like in the snippet below:

```jsx
class Counter extends React.Component {
  ...
  
  render() {
    return (
      <div>
        {this.state}
        <button
          onClick={() => this.setState(this.state + 1)}
        >
          Increment
        </button>
      </div>
    )
  }
}
```

By "shared state" I mean state which is stored in one place, but consumed in another. State managed with Redux is one example - the state is stored in the Redux store, but consumed in some connected component. But any state that was [lifted up](https://reactjs.org/docs/lifting-state-up.html) becomes "shared state" - it was lifted to be shared between several components.

## Isolating local-state behavior

Hooks are a simple React primitive that allows us to isolate behaviour when it manages local state. We can create a `useIncrementor` custom hook that takes care of the stateful behaviour (incrementing by one). This hook can now be reused in different components.

```jsx
function useIncrementor() {
  const [value, setValue] = useState()
  const increment = () => setValue(value + 1)
  return [value, increment]
}

function Counter() {
  const [value, increment] = useIncrementor()
  
  return (
    <div>
      {value}
      <button
        onClick={() => increment()}
      >
        Increment
      </button>
    </div>
  )
}
```

Note, however, that the state managed by `useIncrementor` is local to the `Counter` component.

## Isolating shared-state behaviour

When the state is managed in one place, and consumed in another, we need to somehow "connect" the two.

### Redux

Take Redux for example: the state is managed in the Redux store and consumed in downstream components. The way to connect the two is to use the `connect` function provided by `react-redux`.

```jsx
const Counter = connect(
  state => ({
    value: state.value
  }),
  dispatch => ({
    increment: dispatch({type: 'increment'})
  })
)(props =>
  <div>
    {props.value}
    <button onClick={() => props.increment()}>Increment</button>
  </div>
)
```

To isolate the behavior we can separate that a bit:

```js
const withIncrementor = connect(
  state => ({
    value: state.value
  }),
  dispatch => ({
    increment: dispatch({type: 'increment'})
  })
)

export default withIncrementor
```


