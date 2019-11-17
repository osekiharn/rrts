# Section 17 React and Redux with Typescript

## 254 React and Redux Overview

#### pros

- Far, far easier to avoid extremely common typos, like incorrect action types.
- Gives dev's a far better understanding of the type of data flowing around
- Much easier to refactor just about anything

#### Cons

- Not the best type definition files (especially around redux)
- Tons of generics flying around
- Tons of imports, as just about everything (action creator, action, reducer, store, component) need to be aware of different types
- Redux inherently functional in nature, tough integration with TS classes

## 255 App Overview

## 256 Generating the App

```
$ npx create-react-app rrts --typescript
```

See: https://github.com/facebook/create-react-app/issues/6891

## 257 Simple Components

```tsx
import React from 'react'
import ReactDOM from 'react-dom'

class App extends React.Component {
  render() {
    return <div>Hi there</div>
  }
}

ReactDOM.render(<App />, document.querySelector('#root'))
```

## 258 Interfaces with Props

pass a prop down into the app component.

```tsx
import React from 'react'
import ReactDOM from 'react-dom'

class App extends React.Component {
  render() {
    return <div>{this.props.color}</div>
  }
}

ReactDOM.render(<App color="red" />, document.querySelector('#root'))
```

```tsx
interface AppProps {
  color: string
}

class App extends React.Component<AppProps> {}
```

this pattern is going to be repeated for just about every single class based react component you ever create.

You're going to define your component you're going to define an interface right above it that describes the structure of props that you expect to pass them to the component and then you're going to reference that interface right nest to react component.

```tsx
interface AppProps {
  color: string
}
```

color is a required property.
So if we want to show app we are required to pass in some value for color. if we want to mark a property as being optional.
Well just like with the normal interface right after that key it will put down a question mark like so.

```tsx
interface AppProps {
  color?: string
}
```

that means that is an optional property so we can show App either with the color prop of without it. now it does't matter.

## 259 Handling Component State

I'm going to first begin by initializing a state object inside my class.

```tsx
class App extends React.Component<AppProps> {
  state = {
    counter: 0
  }

  onIncremnet = (): void => {
    this.setState({ counter: this.state.counter + 1 })
  }

  onDecrement = (): void => {
    this.setState({ counter: this.state.counter - 1 })
  }

  render() {
    return (
      <div>
        <button onClick={this.onIncrement}>Increment</button>
        <button onClick={this.onDecrement}>Decrement</button>
        {this.state.counter}
      </div>
    )
  }
}
```

## 260 Confusing Component State!

```tsx
  constructor(props: AppProps) {
    super(props)

    this.state = {
      counter: 0
    }
  }
```

`constructor`にすると`this.state.counter`のところでエラーになる

```
Property 'counter' does not exist on type 'Readonly<{}>'
```

state is going to be of type `readonly<S>` that means that this is going to be a readonly object of type S.
And like we just said `S` is going to be some generic type do we pass in at this point we have not passed in any type for `S`. the default for `S` inside of here is an empty object with no properties whatsoever.

```tsx
interface AppState {
  counter: number
}

class App extends React.Component<AppProps, AppState> {
  ...
}
```

## 261 Functional Components

How to define a functional component.

we need to essentially put on a return annotation here that indicates that we're going to return some JSX element.

```tsx
const App = (props: AppProps): JSX.Element => {
  return <div>{props.color}</div>
}
```

## 262 Redux Setup

```sh
$ npm install redux react-redux axios redux-thunk
```

```tsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, applyMiddleware } from 'redux'
import { Provider } from 'react-redux'
import thunk from 'redux-thunk'

const store = createStore(reducers, applyMiddleware(thunk))

ReactDOM.render(
  <Provider store={store}>
    <App color="red" />
  </Provider>,
  document.querySelector('#root')
)
```

Remember some libraries out there some javascript libraries include type definition files.

So react redux is one library where we need to separately instal that type definition file.

You can always mouse over this error message and I'll tell you exactly what to do.

`npm install @types/react-redux`

```
mkdir src/components/App.tsx
```

```
mkdir src/reducers/index.ts
```

Notice how we're just using a `.ts` this time because we don't need to have any jsx inside of here.

```ts
import { combineReducers } from 'redux'

export const reducers = combineReducers({
  counter: () => 1
})
```

## 263 Action Creators with Typescript

`jsonplaceholder.typicode.com/todos`

`mkdir src/actions/index.ts`

```ts
export const fetchTodos = () => {
  return dispatch => {}
}
```

So we need to provide an annotation or dispatch. Remember dispatch as a function and off the top of my head I sure don't know what different arguments dispatch takes and I don't know what types it takes either.

So either we try to add in a manual type annotation right here for a function which would be probably pretty challenging of we have to go route through the type definition file of redux or redux thunk.

So we don't have to try to rewrite this type ourseleves.
Instead we're just going to import this interface and use it to annotate dispatch.

```ts
import axios from 'axios'
import { Dispatch } from 'redux'

const url = `https://jsonplaceholder.typicode.com/todos`;

export const fetchTodos = () => {
  return async (dispatch: Dispatch) => {
    const response = await = axios.get(url)
    dispatch({
      type: 'FETCH_TODOS',
      payload: response.data
    })
  }
}
```

First off we're getting back a response from that json API but we have no idea exactly what structure of data got returned.

## 264 Action Types Enum

I want to create an interface that has an ID, a title and completed.

```ts
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

...
  return async (dispatch: Dispatch) => {
    const response = await axios.get<Todo[]>(url);
```

I want to create an enum to represent all the different types I expect to have inside of my app to do so.

```
$ touch src/actions/types.ts
```

If you do that by default typescript is going to assign a zero to that first right

```ts
// src/actions/types.ts
export enum ActionTypes {
  fetchTodos
}
```

```ts
import { ActionTypes } from './types'
...
    dispatch({
      type: ActionTypes.fetchTodos,
      payload: response.data
    })
```

## 265 The Generic Dispatch Function

```ts
interface FetchTodosAction {
  type: ActionTypes.fetchTodos;
  payload: Todo[];
}
...
    dispatch<FetchTodosAction>({
      type: ActionTypes.fetchTodos,
      payload: response.data
    })
```

obviously you get some additional type safety

# 266 A Reducer with Enums

`touch src/reducers/todos.ts`

```ts
import { Todo, FetchTodosAction } from '../actions'
import { ActionTypes } from '../actions/types'

export const todosReducers = (state: Todo[] = [], action: FetchTodosAction) => {
  switch (action.type) {
    case ActionTypes.fetchTodos:
      return action.payload
    default:
      return state
  }
}
```

## 267 Validating Store Structure

```ts
import { combineReducers } from 'redux'
import { todosReducer } from './todos'
import { Todo } from '../actions'

export interface StoreState {
  todos: Todo[]
}

export const reducers = combineReducers<StoreState>({
  todos: todosReducer
})
```

## 268 Connecting a Component to Redux

```tsx
import React from 'react'
import { connect } from 'react-redux'
import { Todo, fetchTodos } from '../actions'
import { StoreState } from '../reducers'

interface AppProps {
  todos: Todo[]
  fetchTodos(): any
}

export class _App extends React.Component<AppProps> {
  render() {
    return <div>Hi threre!</div>
  }
}

const mapStateToProps = ({ todos }: StoreState): { todos: Todo[] } => {
  return { todos }
}

export const App = connect(mapStateToProps, { fetchTodos })(_App)
```

## 269 Rendering a List

```ts
export class _App extends React.Component<AppProps> {
  onButtonClick = (): void => {
    this.props.fetchTodos()
  }

  renderList(): JSX.Element[] {
    return this.props.todos.map((todo: Todo) => {
      return <div key={todo.id}>{todo.title}</div>
    })
  }

  render() {
    return (
      <div>
        <button onClick={this.onButtonClick}>Fetch</button>
        {this.renderList()}
      </div>
    )
  }
}
```

## 270 Adding in Delete Functionality

```ts
export enum ActionTypes {
  fetchTodos,
  deleteTodo
}
```

```ts
export interface DeleteTodoAction {
  type: ActionTypes.deleteTodo
  payload: number
}

export const deleteTodo = (id: number): DeleteTodoAction => {
  return {
    type: ActionTypes.deleteTodo,
    payload: id
  }
}
```

## 271 Breaking Out Action Creators

`touch src/actions/todos.ts`

`src/actions/index.ts`のソースを`todos.ts`に移し替える。

```ts
// src/actions/index.ts
export * from './todos'
export * from './types'
```

```ts
// src/reducers/todos.ts
import { Todo, FetchTodosAction, ActionTypes } from '../actions'
```

## 272 Expressing Actions as Typd Union

```ts
import {
  Todo,
  FetchTodosAction,
  DeleteTodoAction,
  ActionTypes
} from '../actions'

export const todosReducer = (
  state: Todo[] = [],
  action: FetchTodosAction | DeleteTodoAction
) => {
  ...
}
```

ソースが肥大化する

```ts
// actions/types.ts
import { FetchTodosAction, DeleteTodoAction } from './todos'

export type Action = FetchTodosAction | DeleteTodoAction
```

```ts
// reducers/todos.ts
import { Todo, Action, ActionTypes } from '../actions'

export const todoReducer = (state: Todo[] = [], action: Action) => {
  ...
}
```

## 273 Type Guards in Reducers

```ts
export const todosReducer = (state: Todo[] = [], action: Action) => {
  switch (action.type) {
    case ActionTypes.deleteTodo:
      return state.filter((todo: Todo) => todo.id !== action.payload)
```
