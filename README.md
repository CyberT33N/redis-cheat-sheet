# Redis Cheat Sheet
Redis Cheat Sheet with the most needed stuff..


<br><br>

# Guides
- https://university.redislabs.com/courses/course-v1:redislabs+RU102JS+2021_01/course/






















<br><br>
_________________________________________
_________________________________________
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


## Node.js
- ioredis (https://github.com/luin/ioredis)
- node_redis (https://github.com/NodeRedis/node-redis)



























<br><br>
__________________________________________________
__________________________________________________
<br><br>




# Type Mappings

|Redis Type|Javascript Type|
|---|---|
|string|String|
|list|Array of String|
|set|Array of String|
|hash|Object(keys have String values)|
|float|String|
|integer|number|
























<br><br>
__________________________________________________
__________________________________________________
<br><br>




# Connect
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



## Connect await
```javascript
const runApplication = async () => {
  // Connect to Redis.
  const client = redis.createClient({
    host: 'localhost',
    port: 6379,
    // password: 'password',
  });

  // Clean up and allow the script to exit.
  client.quit();
};

try {
  runApplication();
} catch (e) {
  console.log(e);
}
```








## Event Listener
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

# Write Data

<br><br>

## SET (https://redis.io/commands/set)
```javascript
// callback
client.set('hello', 'world', (e, res) => {
  console.log(res); // OK
  client.quit();
});


// promise
const { promisify } = require('util');
const setAsync = promisify(client.set).bind(client);

setAsync('hello', 'world')
  .then(res => console.log(res)) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const reply = await client.setAsync('hello', 'world');
console.log(reply); // OK
client.quit();
```



<br><br>



## HMSET (https://redis.io/commands/hmset)
- Sets the specified fields to their respective values in the hash stored at key. This command overwrites any specified fields already existing in the hash. If key does not exist, a new key holding a hash is created.
```javascript
// callback
client.hmset(testKeyNaME, {
    name: 'John Doe',
    age: 42,
  }, (e, res) => {
  console.log(res); // OK
  client.quit();
});


// promise
const { promisify } = require('util');
const hmsetAsync = promisify(client.hmset).bind(client);

hmsetAsync(testKeyNaME, {
    name: 'John Doe',
    age: 42,
  })
  .then(res => console.log(res)) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hmsetAsync(testKeyNaME, {
  name: 'John Doe',
  age: 42,
});
console.log(res); // OK
client.quit();
```











<br><br>



## RPUSH (https://redis.io/commands/rpush)
- Insert all the specified values at the tail of the list stored at key. If key does not exist, it is created as empty list before performing the push operation. When key holds a value that is not a list, an error is returned.
```javascript
var planets = ['Mars', 'Pluto', 'Sun', 'Earth', 'Earth']

// callback
client.rpush('planets', planets, (e, res) => {
  console.log(res); // 5 <-- Will return the length of the list
  client.quit();
});


// promise
const { promisify } = require('util');
const rpushAsync = promisify(client.rpush).bind(client);

rpushAsync('planets', planets)
  .then(res => {
    console.log(res); // 5 <-- Will return the length of the list
  }) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.rpushAsync('planets', planets);
console.log(res); // 5 <-- Will return the length of the list
client.quit();
```






































<br><br>
__________________________________________________
__________________________________________________
<br><br>

# Read Data


## GET (https://redis.io/commands/get)
```javascript
// callback
client.get('hello', (e, res) => {
  console.log(res); // world
  client.quit();
});


// promises
const { promisify } = require('util');
const getAsync = promisify(client.get).bind(client);

getAsync('hello')
  .then(res => console.log(res)) // world
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.getAsync('hello');
console.log(res); // world
client.quit();
```


<br><br>


## HGETALL (https://redis.io/commands/hgetall)
```javascript
/*
testKeyNaME = {
    name: 'John Doe',
    age: 42,
  };
*/

// callback
client.hgetall(testKeyName, (e, res) => {
  console.log(typeof res); // object
  console.log(typeof res.age); // string
  client.quit();
});


// promises
const { promisify } = require('util');
const hgetallAsync = promisify(client.hgetall).bind(client);

hgetallAsync(testKeyName)
  .then(res => {
     console.log(typeof res); // object
     console.log(typeof res.age); // string
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hgetallAsync(testKeyName);
console.log(typeof res); // object
console.log(typeof res.age); // string
client.quit();
```






<br><br>


## LLEN (https://redis.io/commands/llen)
- Returns the length of the list stored at key. If key does not exist, it is interpreted as an empty list and 0 is returned. An error is returned when the value stored at key is not a list.
```javascript
/*
testKeyName = ['Mars', 'Pluto', 'Sun', 'Earth', 'Earth']
*/

// callback
client.llen(testKeyName, (e, res) => {
  console.log(res); // 5 <-- return the length of the list
  client.quit();
});


// promises
const { promisify } = require('util');
const llenAsync = promisify(client.llen).bind(client);

llenAsync(testKeyName)
  .then(res => {
     console.log(res); // 5 <-- return the length of the list
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.llenAsync(testKeyName);
console.log(res); // 5 <-- return the length of the list
client.quit();
```























































<br><br>
__________________________________________________
__________________________________________________
<br><br>


# INCRBYFLOAT (https://redis.io/commands/incrbyfloat)
- Increment the string representing a floating point number stored at key by the specified increment. By using a negative increment value, the result is that the value stored at the key is decremented (by the obvious properties of addition). If the key does not exist, it is set to 0 before performing the operation.
```javascript
// testKeyValue = 22.5

// callback
client.incrbyfloat(testKeyValue, 1, (e, res) => {
   console.log(typeof res); // string
   console.log(res); // 23.5
   client.quit();
});


// promises
const { promisify } = require('util');
const getIncrbyfloat = promisify(client.incrbyfloat).bind(client);

getIncrbyfloat(testKeyValue, 1)
  .then(res => {
    console.log(typeof res); // string
    console.log(res); // 23.5
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.incrbyfloatAsync(testKeyValue, 1);
console.log(typeof res); // string
console.log(res); // 23.5
client.quit();
```

