# Apollo - GraphQl
> In this repo I experiment with apollo graphql. The setup and queries can be found in the master branch, the mutations in the 'Mutation branch'

To start run this app
* clone the repo
* run `yarn`
* run `yarn start`

## What I learned

### The steps to set up an app
  1) **_Set up a client_**

```js
const client = new ApolloClient({
  uri: "https://nx9zvp49q7.lp.gql.zone/graphql"
});
```
2) **_Set up the connection between React and Apollo_**

```html
<ApolloProvider client={client}>
  <div>
    <h2>My first Apollo app 🚀</h2>
  </div>
</ApolloProvider>
```

3) **_Set up a Query Component_**
```js
const GET_DOGS = gql`
  {
    dogs {
      id
      breed
    }
  }
`;

const Dogs = ({ onDogSelected }) => (

  <Query query={GET_DOGS}>

    {({ loading, error, data }) => {
      if (loading) return "Loading...";
      if (error) return `Error! ${error.message}`;

      return (
        //Whatever you want to return
      );
    }}

  </Query>
);
```
4) **_Set Up your Mutation Component_**

```js
const ADD_TODO = gql`
  mutation addTodo($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;

const AddTodo = () => {
  let input;

  return (
    <Mutation 
    mutation={ADD_TODO}
    update={(cache, { data: { addTodo } }) => {
      const { todos } = cache.readQuery({ query: GET_TODOS });
      cache.writeQuery({
        query: GET_TODOS,
        data: { todos: todos.concat([addTodo]) }
      });
    }}  
    >
      {(addTodo, { data }) => (
        <div>
          <form
            onSubmit={e => {
              e.preventDefault();
              addTodo({ variables: { type: input.value } });
              input.value = "";
            }}
          >
            <input
              ref={node => {
                input = node;
              }}
            />
            <button type="submit">Add Todo</button>
          </form>
        </div>
      )}
    </Mutation>
  );
};
```


### Refetching vs Polling
* Use `refetch` to fetch new data after a user action. Use `polling` to refetch data on a specific time interval

### Apollo Consumer
When React mounts a Query component, the query is automatically fired off. If you want to delay firing the query up to a certain moment, you can use apollo consumer:

```js
import React, { Component } from 'react';
import { ApolloConsumer } from 'react-apollo';

class DelayedQuery extends Component {
  state = { dog: null };

  onDogFetched = dog => this.setState(() => ({ dog }));

  render() {
    return (
      <ApolloConsumer>
        {client => (
          <div>
            {this.state.dog && <img src={this.state.dog.displayImage} />}
            <button
              onClick={async () => {
                const { data } = await client.query({
                  query: GET_DOG_PHOTO,
                  variables: { breed: "bulldog" }
                });
                this.onDogFetched(data.dog);
              }}
            >
              Click me!
            </button>
          </div>
        )}
      </ApolloConsumer>
    );
  }
}
```