# Redis Cheat Sheet
Redis Cheat Sheet with the most needed stuff..


<br><br>

# Guides
- https://university.redislabs.com/courses/course-v1:redislabs+RU102JS+2021_01/course/

<br><br>



# Installation


<br><br>


## Docker
```bash
sudo docker run --name my-first-redis -p 6379:6379 redis
```
















<br><br>
__________________________________________________
__________________________________________________
<br><br>




## Clients
<br><br>


#### Node.js
- ioredis (https://github.com/luin/ioredis)
- node_redis (https://github.com/NodeRedis/node-redis)








<br><br>
__________________________________________________
__________________________________________________
<br><br>




## Connect
```javascript
const redis = require('redis');

const connectWithPortAndHost = () => {
  // Connect with port and host.
  const client = redis.createClient({
    port: 6379,
    host: '127.0.0.1',
  });

  // Send PING command expect PONG response.
  client.ping((err, response) => console.log(response));
  client.quit();
};


const connectToDefaultPortAndHost = () => {
  // Connect to default port (6379) and host (127.0.0.1)
  const client = redis.createClient();

  // Send PING command expect PONG response.
  client.ping((err, response) => console.log(response));
  client.quit();
};


const connectUsingURL = () => {
  // Connect using a URL format.
  const client = redis.createClient({
    url: 'redis://127.0.0.1:6379',
  });

  // Send PING command expect PONG response.
  client.ping((err, response) => console.log(response));
  client.quit();
};


const connectWithPassword = () => {
  // Connect and provide a password.
  const client = redis.createClient({
    host: '127.0.0.1',
    port: 6379,
    password: 'secretPassword',
  });

  // Send PING command expect PONG response.
  client.ping((err, response) => console.log(response));
  client.quit();
};


const connectWithSSL = () => {
  const client = redis.createClient({
    host: '127.0.0.1',
    port: 6379,
    tls: {
      key: 'contents of key file',
      cert: 'contents of cert file',
      ca: ['contents of ca cert file'],
    },
  });

  // Send PING command expect PONG response.
  client.ping((err, response) => console.log(response));
  client.quit();
};
```











#### Event Listener
- ready (Connections was successfully)
```javascript
client.on('ready', () => console.log('Connection successfully'));
```
- end (Connection has been closed)
```javascript
client.on('end', () => console.log('Connection was closed'));
```
- reconnecting (Connection was lost and reconnect was tried)
```javascript
client.on('end', (o) => {
  console.log(`Connection was lost.. try reconnect..
  Attempt number: ${o.attempt}
  Time in MS since last attempt: ${o.delay}
  `);
});
```






















<br><br>
__________________________________________________
__________________________________________________
<br><br>


## .set()
```javascript
// callback
client.set('hello', 'world', (err, reply) => {
  console.log(reply); // OK
});
```






<br><br>
__________________________________________________
__________________________________________________
<br><br>


## .get()
```javascript
// callback
client.get('hello', (getErr, getReply) => {
  console.log(getReply); // world
});
```

