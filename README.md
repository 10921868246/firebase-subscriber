# Firebase Subscriber

FirebaseSubscriber is an abstract layer on top of [Firebase official SDK](https://www.firebase.com/docs/web/api/).
The main purpose of FirebaseSubscriber is to:

- Abstract logic token expiring/re-auth from application logic
- Abstract event unsubscribing by an additional `Channel` layer


## Usage:

```javascript
const FirebaseSubscriber = require('firebase-subscriber');

const getAuthToken = function() {
  // request application api here to get fresh firebase auth token
  // return a promise
  // this function would be invoked whenever the firebase auth token is expired
}

const subscribe = FirebaseSubscriber.subscriber(endPoint, { getAuthToken });
const channel = subscribe('/my-test-path');

channel.on('child_added', function(val) {
  // `val` here is the result of snapshot.val()
  console.log('on child added', val);
})
channel.on('value', function(val) {
  console.log('on value', val);
})


channel.off(); //=> unsubscribe ALL event handlers bound on the channel
```

## API:

### `.subscriber()`

The `.subscriber()` method takes two arguments, `endPoint` and `options` for `Connection`, and returns a function for subscribing certain path of a database.

Please refer to the [Connection](#connection) section for the details of `options`.

### Channel

A `Channel` instance is returned by `subscribe` function.

#### `channel.on(eventName, handler)`:

Wrap [Firebase.on()](https://www.firebase.com/docs/web/api/query/on.html),
invoke handler with `snapshot.val()`

#### `channel.off()`:

Unregister *ALL* event handlers on the channel

#### `channel.onDisconnect(callback)`:

Invoke callback with disconnected ref, for example:

```javascript
channel.onDisconnect(function(presenceRef) {
  presenceRef.set('offline')
})
```

#### Setter Methods

`Channel` instances are equipped with some setter methods simply delegate to its underlying firebase `ref`:

- `channel.set()`
- `channel.push()`
- `channel.remove()`
- `channel.update()`

### Connection

`Connection` is a configurable factory, which

  - takes two arguments: `firebaseEndPoint` and `options`
  - returns singleton connection, which would auto re-auth when expired

#### `options`:

| Option | Description |
| --- | --- |
| `getAuthToken` | A function which fetches firebase auth token from your application server and returns a promise |
| `isAnonymous` | A flag to determine if auth anonymously, default: `false` |

#### Usage

```javascript
import { Connection } from 'firebase-subscriber';

const getConnection = Connection(firebaseEndpoint, { getAuthToken });
const connection1 = getConnection();
const connection2 = getConnection();

expect(connection1).to.equal(connection2);
```

##### Auth Anonymously

```javascript
// specify `isAnonymous: true` in the options to create an anonymous connection
// returns singleton connection with auto re-auth as well
const getConnection = Connection(firebaseEndpoint, { isAnonymous: true });
const connection = getConnection()
```

## Testing

`$ npm test`

## LICENCE

MIT
