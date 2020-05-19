# WhatsAppClient/Server Notes

## [CORS](https://www.tortilla.academy/Urigo/WhatsApp-Clone-Tutorial/master/next/step/3)

> Unlike the previous route, we used the .json() method this time around to send a response. This will simply stringify the given JSON and set the right headers. Similarly to the client, we've defined the db mock in a dedicated file, as this is easier to maintain and look at.

> It's also recommended to connect a middleware called cors which will enable cross-origin requests. Without it we will only be able to make requests in localhost, something which is likely to limit us in the future because we would probably host our server somewhere separate than the client application. Without it it will also be impossible to call the server from our client app. Let's install the cors library and load it with the Express middleware() function:

```sh
$ yarn add cors

# and its Typescript types:

$ yarn add --dev @types/cors
```

## Use CORS

```ts
import cors from 'cors';
import express from 'express';
import { chats } from './db';

const app = express();

app.use(cors());

app.get('/_ping', (req, res) => {
  res.send('pong');
});

app.get('/chats', (req, res) => {
  res.json(chats);
});

const port = process.env.PORT || 4000;

app.listen(port, () => {
  console.log(`Server is listening on port ${port}`);
});
```

## Client Step 3.1: Define server URL

Add `.env` in root with `REACT_APP_SERVER_URL=http://localhost:4000` in your client app.

## How `REACT_APP_SERVER_URL` Works

> This will make our server's URL available under the process.env.REACT_APP_SERVER_URL member expression and it will be replaced with a fixed value at build time, just like macros. The .env file is a file which will automatically be loaded to process.env by the dotenv NPM package. react-scripts then filters environment variables which have a REACT_APP_ prefix and provides the created JSON to a Webpack plugin called DefinePlugin, which will result in the macro effect.

## Fetching

> Now let's move back into our React app folder. We will now replace the local data-mock usage with a fetch from the server. For that we can use the native fetch API, however, it needs to be used in the right life-cycle hook of the React.Component.

There are 2 naive approaches for that:

Calling fetch() outside the component, but this way that chats will be fetched even if we're not even intending to create an instance of the component.

```js
fetch().then(() => /* ... */)
const MyComponent = () => {}
```

Calling fetch() inside the component, but then it will be invoked whenever the component is re-rendered.

```ts
const MyComponent = () => {
  fetch().then(() => /* ... */)
}
```
These 2 approaches indeed work, but they both fail to deliver what's necessary on the right time. In addition, there's no way to properly coordinate async function calls with the render method of the component.

## Two React Hooks

With React hooks we can invoke the desired logic in the right life-cycle stage of the target component. This way we can avoid potential memory leaks or extra calculations. To implement a proper `fetch()`, we will be using 2 React hooks:


React.useState() - which is used to get and set a state of the component - will be used to store the chats fetched from the server.

```ts
const [value, setValue] = useState(initialValue);
```

React.useMemo() - which is used to run a computation only once certain conditions were met - will be used to run the fetch() function only once the component has mounted.

```ts
const memoizedValue = useMemo(calcFn, [cond1, cond2, ...conds]);
```

## Testing with `data-testid`

`yarn add jest-fetch-mock @testing-library/jest-dom @testing-library/react`


Now we're gonna write a basic test for the <ChatsList /> component. In the test, we'll mock a fake response from the server, and examine the contents of rendered HTML. Since the HTML of the component is a dynamic thing and is constantly subject to changes, it would be a good idea to annotate it with data-testid attributes so it can be tested regardless of its structure:

# [Testing](https://www.tortilla.academy/Urigo/WhatsApp-Clone-Tutorial/master/next/step/5)

## [jest-fetch-mock](https://www.npmjs.com/package/jest-fetch-mock)


- The jest-fetch-mock package can mock responses emitted by the Fetch API.
- The @testing-library/jest-dom package will add custom matchers that will help us examine HTML contents of DOM elements.
- The @testing-library/react package contains utility methods that will help us test React.Components with Jest.

Next, we will create a file under the src folder called setupTests.ts. This file is configured automatically by create-react-app and loaded by Jest, and we can use it to set up our testing environment according to our needs (like said earlier, Jest can be configured, so this file path can be changed). We will use that file to define a fake Fetch API using the jest-fetch-mock library:

```ts
// src/setupTests.ts
// for embedme
```

We will create another file called ChatsList.test.tsx, right next to the <ChatsList /> component under the ChatsListScreen directory, and inside we will implement our test. The test should follow these steps:

- Mock the response to contain a fake chat, so we won't need to make an actual call to our GraphQL API.

- We will create a new instance of <ChatsList /> and render it in a container element.

- We will wait for changes in the DOM caused by setState().

- We will test the contents of the container.

And this is how the implementation should look like:

```ts
//src/components/ChatsListScreen/ChatsList.test.tsx
```

Jest API is vast but pretty intuitive for the most part. It mostly consists of test descriptors and matchers. [Here's a full list of all matchers which are built into Jest's API](https://jestjs.io/docs/en/expect). Always make sure to work against it when writing tests, for optimal results.