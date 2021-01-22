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


## Connect
```javascript
// ---- Method #1 - Callback ----
const redis = require('redis');

// Create a client and connect to Redis.
const client = redis.createClient({
  host: 'localhost',
  port: 6379,
  // password: 'password',
});

// Run a Redis command, receive response in callback.
client.set('hello', 'world', (err, reply) => {
  console.log(reply); // OK

  // Run a second Redis command now we know that the
  // first one completed.  Again, response in callback.
  client.get('hello', (getErr, getReply) => {
    console.log(getReply); // world

    // Quit client and free up resources.
    client.quit();
  });
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

