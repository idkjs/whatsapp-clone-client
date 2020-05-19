# Adding Router and ChatScreenRoom Component(https://www.tortilla.academy/Urigo/WhatsApp-Clone-Tutorial/master/next/step/6)

In this chapter we will learn how to build a chat room screen. In order to navigate between different screens, we will setup a router.

Since we're gonna have two screens in our app now - ChatsListScreen and ChatRoomScreen, we will need a router that will be able to alternate between them. We will be using the react-router-dom package to manage the routes of the application:

```sh
yarn add react-router-dom @types/react-router-dom
```

```ts
// ./App.tsx
import React from 'react';
import { BrowserRouter, Route, Redirect, Switch } from 'react-router-dom';
import ChatRoomScreen from './components/ChatRoomScreen';
import ChatListScreen from './components/ChatsListScreen/';

const App: React.FC = () => (
  <BrowserRouter>
    <Switch>
      {' '}
      <Route exact path="/chats" component={ChatsListScreen} />
      <Route exact path="/chats/:chatId" component={ChatRoomScreen} />
    </Switch>
    <Route exact path="/" render={redirectToChats} />
  </BrowserRouter>
);
const redirectToChats = () => <Redirect to="/chats" />;

export default App;
```
The purpose of a router is to make route managing easy and declarative. It will take care of managing the history within our app and parameterize certain screens according to our need. Essentially it's a wrap around the window.history object which is also compatible with React. I recommend you to go through the official MDN docs if you're not yet familiar with the concept.

## Route component

The <Route /> component represents a path for a route in our application. Using the colon syntax (:chatId) we basically tell the router that the /chat route should be followed by a string whose value can later on be addressed via a parameter called chatId when navigating to the route. So here's a sum-up of the routes manifest:

- `/chats` - will navigate to the ChatsListScreen.
- `/chat/:chatId` - e.g. /chat/1, will navigate to the ChatRoomScreen and will parameterize it to show data which is related to chat ID 1.
- Any other route will fallback to the `/chats` route which will redirect us to the ChatsListScreen.

Now we will implement the `ChatRoomScreen` so the router can function properly. For now we will make it a plain screen which simply prints out the information of the chat that was clicked so we can have a complete flow, and then we will take care of the rest.

To do so, we will first implement the **chat query in our backend**. This would be a **[`parameterized query`](https://www.techopedia.com/definition/24414/parameterized-query)** that will provide us with a specific `chat` according to the received ID, and it will be used by the new screen as soon as it is initialized. First we would update the Chat type to contain a messages field so that we can access all the messages in a particular chat:

```tsx
import { DateTimeResolver, URLResolver } from 'graphql-scalars';
import { chats, messages } from '../db';

const resolvers = {
  Date: DateTimeResolver,
  URL: URLResolver,

  Chat: {
    messages(chat: any) {
      return messages.filter(m => chat.messages.includes(m.id));
    },

    lastMessage(chat: any) {
      return messages.find((m) => m.id === chat.lastMessage);
    },
  },

  Query: {
    chats() {
      return chats;
    },
  },
};

export default resolvers;
```

## Add a Basic ChatRoomScreen

Update `src/App.tsx1

```ts
...
import {
  BrowserRouter,
  Route,
  Redirect,
  Switch,
  RouteComponentProps,
} from 'react-router-dom';
...

const App: React.FC = () => (
...
      <Route exact path="/chats" component={ChatsListScreen} />
//       <Route exact path="/chats/:chatId" component={ChatRoomScreen} />
      <Route
        exact
        path="/chats/:chatId"
        component={({ match }: RouteComponentProps<{ chatId: string }>) => (
          <ChatRoomScreen chatId={match.params.chatId} />
        )}
      />
...
);

const redirectToChats = () => <Redirect to="/chats" />;

export default App;
```

Open `http://localhost:3001/chats/1` to see it in action. `browser history` isnt set up yet but going to the url shows the routing is working.