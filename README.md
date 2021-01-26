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

## GUI
- https://redislabs.com/redis-enterprise/redis-insight/
- https://github.com/uglide/RedisDesktopManager
- https://github.com/qishibo/AnotherRedisDesktopManager
















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

# Structure
![Test Image 1](https://i.imgur.com/BPjQY9v.png)
- You can use any structure you want. Most common it is to use : for seperating. However you can also only use your key + data types.
```javascript
keyPrefix(optional) - collectionName(optional) - keyName:string/list/set/hash
```

<br><br>


# Data Types (https://redis.io/topics/data-types)

<br><br>

## Type Mappings

|Redis Type|Javascript Type|Example|Description|
|---|---|---|---|
|string|String|foo|Strings are the most basic kind of Redis value. Redis Strings are binary safe, this means that a Redis string can contain any kind of data, for instance a JPEG image or a serialized Ruby object.|
|list|Array of String|apple banana orange|Redis Lists are simply lists of strings, sorted by insertion order. It is possible to add elements to a Redis List pushing new elements on the head (on the left) or on the tail (on the right) of the list. **Compared to sets a list can contain duplicated members**|
|set|Array of String|apple banana orange|Redis Sets are an unordered collection of Strings. It is possible to add, remove, and test for existence of members in O(1) (constant time regardless of the number of elements contained inside the Set).**Sets are containers for unique members and this means you can not insert duplicated members**|
|hash|Object(keys have String values)|username hans password test123|Redis Hashes are maps between string fields and string values, so they are the perfect data type to represent objects|
|float|String|
|integer|number|

<br><br>


## Hash
- All values are string









<br><br>








## Sorted Sets (ZSET)
- https://www.youtube.com/watch?v=pk2TAc89bwE
- First we make sure to choose an unique prefix and metric and maybe aswell a timestamp for our syntax:
```javascript
prefix:metric:date:id
```

<br><br>

Because sets can not contain duplicated data we will give each member in our set an unique ID like in this example:
```javascript
planets: Earth:1, Pluto:2, Earth:3
```

<br><br>
What are the benefits of this way of using sorted sets?
- It is always sorted
- Efficiently fetch a small range: O((log n) + m)
- Efficient inserts: O(log n)
- Store multiple data inside of a single key saves memory




























































<br><br>
__________________________________________________
__________________________________________________
<br><br>




# Key Generator Script
```javascript
const config = require('better-config');
const shortId = require('shortid');
const timeUtils = require('../../../utils/time_utils');

// Prefix that all keys will start with, taken from config.json
let prefix = config.get('dataStores.redis.keyPrefix');

/**
 * Takes a string containing a Redis key name and returns a
 * string containing that key with the application's configurable
 * prefix added to the front.  Prefix is configured in config.json.
 *
 * @param {string} key - a Redis key
 * @returns {string} - a Redis key with the application prefix prepended to
 *  the value of 'key'
 * @private
 */
const getKey = key => `${prefix}:${key}`;

/**
 * Generates a temporary unique key name using a the short string
 * generator module shortid.
 *
 * Used in week 3 geo for temporary set key names.
 *
 * @returns - a temporary key of the form tmp:PPBqWA9
 */
const getTemporaryKey = () => getKey(`tmp:${shortId.generate()}`);

/**
 * Takes a numeric site ID and returns the site information key
 * value for that ID.
 *
 * Key name: prefix:sites:info:[siteId]
 * Redis type stored at this key: hash
 *
 * @param {number} siteId - the numeric ID of a site.
 * @returns - the site information key for the provided site ID.
 */
const getSiteHashKey = siteId => getKey(`sites:info:${siteId}`);

/**
 * Returns the Redis key name used for the set storing all site IDs.
 *
 * Key name: prefix:sites:ids
 * Redis type stored at this key: set
 *
 * @returns - the Redis key name used for the set storing all site IDs.
 */
const getSiteIDsKey = () => getKey('sites:ids');

/**
 * Takes a numeric site ID and a UNIX timestamp, returns the Redis
 * key used to store site stats for that site for the day represented
 * by the timestamp.
 *
 * Key name: prefix:sites:stats:[year-month-day]:[siteId]
 * Redis type stored at this key: sorted set
 *
 * @param {number} siteId - the numeric ID of a site.
 * @param {number} timestamp - UNIX timestamp for the desired day.
 * @returns {string} - the Redis key used to store site stats for that site on the
 *  day represented by the timestamp.
 */
const getSiteStatsKey = (siteId, timestamp) => getKey(`sites:stats:${timeUtils.getDateString(timestamp)}:${siteId}`);

/**
 * Takes a name, interval and maximum number of hits allowed in that interval,
 * returns the Redis key used to store the rate limiter data for those parameters.
 *
 * Key name: prefix:limiter:[name]:[interval]:[maxHits]
 * Redis type stored at this key: string (containing a number)
 *
 * @param {string} name - the unique name of the resource.
 * @param {number} interval - the time period that the rate limiter applies for (mins).
 * @param {number} maxHits - the maximum number of hits on the resource
 *  allowed in the interval.
 * @returns {string} - the Redis key used to store the rate limiter data for the
 *  given parameters.
 */
const getRateLimiterKey = (name, interval, maxHits) => {
  const minuteOfDay = timeUtils.getMinuteOfDay();
  return getKey(`limiter:${name}:${Math.floor(minuteOfDay / interval)}:${maxHits}`);
};

/**
 * Returns the Redis key used to store geo information for sites.
 *
 * Key name: prefix:sites:geo
 * Redis type stored at this key: geo
 *
 * @returns {string} - the Redis key used to store site geo information.
 */
const getSiteGeoKey = () => getKey('sites:geo');

/**
 * Returns the Redis key used for storing site capacity ranking data.
 *
 * Key name: prefix:sites:capacity:ranking
 * Redis type stored at this key: sorted set
 *
 * @returns {string} - the Redis key used for storing site capacity ranking data.
 */
const getCapacityRankingKey = () => getKey('sites:capacity:ranking');

/**
 * Returns the Redis key used for storing RedisTimeSeries metrics
 * for the supplied site ID.
 *
 * Key name: prefix:sites:ts:[siteId]:[unit]
 * Redis type stored at this key: RedisTimeSeries
 *
 * @param {number} siteId - the numeric ID of a solar site
 * @param {string} unit - the metric unit name
 * @returns {string} - the Redis key used for storing RedisTimeSeries metrics
 *  for the supplied site ID.
 */
const getTSKey = (siteId, unit) => getKey(`sites:ts:${siteId}:${unit}`);

/**
 * Returns the Redis key name used to store metrics for the site represented
 * by 'siteId', the metric type represented by 'unit' and the date represented
 * by 'timestamp'.
 *
 * Key name: prefix:metric:[unit]:[year-month-day]:[siteId]
 * Redis type stored at this key: sorted set
 *
 * @param {number} siteId - the numeric site ID of the site to get the key for.
 * @param {*} unit - the name of the measurement unit to get the key for.
 * @param {*} timestamp - UNIX timestamp representing the date to get the key for.
 * @returns {string} - the Redis key used to store metrics for the specified metric
 *  on the specified day for the specified site ID.
 */
const getDayMetricKey = (siteId, unit, timestamp) => getKey(
  `metric:${unit}:${timeUtils.getDateString(timestamp)}:${siteId}`,
);

/**
 * Returns the name of the Redis key used to store the global sites data feed.
 *
 * Key name: prefix:sites:feed
 * Redis type stored at this key: stream
 *
 * @returns {string} - the Redis key used to store the global site data feed.
 */
const getGlobalFeedKey = () => getKey('sites:feed');

/**
 * Returns the name of the Redis key used to store the data feed for the site
 * represented by 'siteId'.
 *
 * Key name: prefix:sites:feed:[siteId]
 * Redis type stored at this key: stream
 *
 * @param {number} siteId - numeric ID of a specific site.
 * @returns {string} - the Redis key used to store the data feed for the
 *  site represented by 'siteId'.
 */
const getFeedKey = siteId => getKey(`sites:feed:${siteId}`);

/**
 * Set the global key prefix, overriding the one set in config.json.
 *
 * This is used by the test suites so that test keys do not overlap
 * with real application keys and can be safely deleted afterwards.
 *
 * @param {*} newPrefix - the new key prefix to use.
 */
const setPrefix = (newPrefix) => {
  prefix = newPrefix;
};

module.exports = {
  getTemporaryKey,
  getSiteHashKey,
  getSiteIDsKey,
  getSiteStatsKey,
  getRateLimiterKey,
  getSiteGeoKey,
  getCapacityRankingKey,
  getTSKey,
  getDayMetricKey,
  getGlobalFeedKey,
  getFeedKey,
  setPrefix,
  getKey,
};

```






































































<br><br>
__________________________________________________
__________________________________________________
<br><br>




# CLI (https://redis.io/topics/rediscli)
```javascript
redis-cli
```






























































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


## SDIFFSTORE (https://redis.io/commands/sdiffstore)
- This command is equal to SDIFF (https://redis.io/commands/sdiff), but instead of returning the resulting set, it is stored in destination. If destination already exists, it is overwritten.

<br><br>

Syntax:
```javascript
SDIFFSTORE keyName setName1 setName2
```

```javascript
/*
Our set A like this:
planetsRed: Sun, Earth, Pluto, Moon

Our set B like this:
planetsBlue: Sun, Earth, Pluto, Neptun

It will create a set of the difference at set difference and look like this:
difference: Moon, Neptun
*/

const query = [
  'difference',
  'planetsRed',
  'planetsBlue',
];

// callback
client.sdiff(...query, (e, res) => {
  console.log(res); // 2 - the number of elements in the resulting set.
  client.quit();
});


// promises
const { promisify } = require('util');
const sdiffAsync = promisify(client.sdiff).bind(client);

sdiffAsync(...query)
  .then(res => {
     console.log(res); // 2 - the number of elements in the resulting set.
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.sdiffAsync(...query);
console.log(res); // 2 - the number of elements in the resulting set.
client.quit();
```

















<br><br>



## HSET (https://redis.io/commands/hset)
- Sets field in the hash stored at key to value. If key does not exist, a new key holding a hash is created. If field already exists in the hash, it is overwritten. As of Redis 4.0.0, HSET is variadic and allows for multiple field/value pairs.

<br><br>

Syntax:
```javascript
hashName fieldName fieldValue
```

```javascript
/* ----
EXAMPLE #1 - Single Value - Will produce:
github: url https://github.com
----- */

const query = [
  'github',
  'url',
  'https://github.com',
];

// callback
client.hset(...query, (e, res) => {
  console.log(res); // OK
  client.quit();
});


// promise
const { promisify } = require('util');
const hsetAsync = promisify(client.hset).bind(client);

hsetAsync(...query)
  .then(res => console.log(res)) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hsetAsync(...query);
console.log(res); // OK
client.quit();











/* ----
EXAMPLE #2 - Multiple Values - Will produce:
earth: diameterKM 12756 dayLengthHours 24 meanTempC 15 moonCount 1
----- */
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);
const earthProps = {
    diameterKM: 12756,
    dayLengthHours: 24,
    meanTempC: 15,
    moonCount: 1,
};

// Set the fields one at a time...
await Promise.all(
  Object.keys(earthProps).map(
    key => client.hsetAsync('earth', key, earthProps[key]),
  ),
);
```



<br><br>



## HMSET (https://redis.io/commands/hmset)
- Sets the specified fields to their respective values in the hash stored at key. This command overwrites any specified fields already existing in the hash. If key does not exist, a new key holding a hash is created.
- Instead to HSET you can directly insert an Object. This is way more usefully than manually iterate over each object key at HSET.

<br><br>

Syntax:
```javascript
HMSET hashName field1 "Hello" field2 "World"
```

<br><br>

```javascript
const query = [
  'github',
  {
    url: 'https://github.com',
    domain: 'com',
  }
];

// callback
client.hmset(...query, (e, res) => {
  console.log(res); // OK
  client.quit();
});


// promise
const { promisify } = require('util');
const hmsetAsync = promisify(client.hmset).bind(client);

hmsetAsync(...query)
  .then(res => console.log(res)) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hmsetAsync(...query);
console.log(res); // OK
client.quit();
```
















































<br><br>



## ZADD (https://redis.io/commands/zadd)
- Adds all the specified members with the specified scores to the sorted set stored at key. It is possible to specify multiple score / member pairs. If a specified member is already a member of the sorted set, the score is updated and the element reinserted at the right position to ensure the correct ordering.

<br><br>

Syntax:
```javascript
ZADD sortedSetname 2 "value" 3 "value"
```

<br><br>

```javascript
const query = [
  'planets',
  1,
  'Earth',
  2,
  'sun',
];

// callback
client.zadd(...query, (e, res) => {
  console.log(res); // OK
  client.quit();
});


// promise
const { promisify } = require('util');
const zaddAsync = promisify(client.zadd).bind(client);

zaddAsync(...query)
  .then(res => console.log(res)) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.zaddAsync(...query);
console.log(res); // OK
client.quit();
```




















<br><br>



## RPUSH (https://redis.io/commands/rpush)
- Insert all the specified values at the tail of the list stored at key. If key does not exist, it is created as empty list before performing the push operation. When key holds a value that is not a list, an error is returned.


<br><br>

Syntax:
```javascript
RPUSH listName value
```

```javascript
/*
Will produce:
planets: Mars, Pluto, Sun, Earth, Earth
*/


const planets = ['Mars', 'Pluto', 'Sun', 'Earth', 'Earth'];
const query = ['planets', planets];

// callback
client.rpush(...query, (e, res) => {
  console.log(res); // 5 <-- Will return the length of the list
  client.quit();
});


// promise
const { promisify } = require('util');
const rpushAsync = promisify(client.rpush).bind(client);

rpushAsync(...query)
  .then(res => {
    console.log(res); // 5 <-- Will return the length of the list
  }) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.rpushAsync(...query);
console.log(res); // 5 <-- Will return the length of the list
client.quit();
```





<br><br>




# INCRBYFLOAT (https://redis.io/commands/incrbyfloat)
- Increment the string representing a floating point number stored at key by the specified increment. By using a negative increment value, the result is that the value stored at the key is decremented (by the obvious properties of addition). If the key does not exist, it is set to 0 before performing the operation.
```javascript
/*
Our string looks like this:
temperature: 22.5
*/


const query = [
  'temperature',
  1,
];

// callback
client.incrbyfloat(...query, (e, res) => {
   console.log(typeof res); // string
   console.log(res); // 23.5
   client.quit();
});


// promises
const { promisify } = require('util');
const getIncrbyfloat = promisify(client.incrbyfloat).bind(client);

getIncrbyfloat(...query)
  .then(res => {
    console.log(typeof res); // string
    console.log(res); // 23.5
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.incrbyfloatAsync(...query);
console.log(typeof res); // string
console.log(res); // 23.5
client.quit();
```





<br><br>



## SADD (https://redis.io/commands/sadd)
- Add the specified members to the set stored at key. Specified members that are already a member of this set are ignored. If key does not exist, a new set is created before adding the specified members. An error is returned when the value stored at key is not a set.
- **Duplicated data will not be included**
```javascript
/* If set would exist and look like this:
planets: Saturn, Neptun, Earth

Then after sadd it would look like this:
planets: Saturn, Neptun, Mars, Sun
*/

const planets = ['Mars', 'Pluto', 'Sun', 'Earth'];
const query = ['planets', planets];

// callback
client.sadd(...query, (e, res) => {
  console.log(res);
  client.quit();
});


// promise
const { promisify } = require('util');
const saddAsync = promisify(client.sadd).bind(client);

saddAsync(...query)
  .then(res => {
    console.log(res);
  }) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.saddAsync(...query);
console.log(res);
client.quit();
```






























































## SMOVE (https://redis.io/commands/smove)
- Move member from the set at source to the set at destination. This operation is atomic. In every given moment the element will appear to be a member of source or destination for other clients. If the source set does not exist or does not contain the specified element, no operation is performed and 0 is returned. Otherwise, the element is removed from the source set and added to the destination set. When the specified element already exists in the destination set, it is only removed from the source set. An error is returned if source or destination does not hold a set value.

Syntax:
```javascript
SMOVE setNameA setNameB value
```

```javascript
/* If set A look like this:
planetsRed: Saturn, Neptun, Earth

If set B look like this:
planetsBlue: Sun, Moon

Then after query we would have:
planetsRed: Saturn, Neptun
planetsBlue: Sun, Moon, Earth
*/


const query = ['planetsRed', 'planetsBlue', 'Earth'];

// callback
client.smove(...query, (e, res) => {
  console.log(res);
  client.quit();
});


// promise
const { promisify } = require('util');
const smoveAsync = promisify(client.smove).bind(client);

smoveAsync(...query)
  .then(res => {
    console.log(res);
  }) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.smoveAsync(...query);
console.log(res);
client.quit();
```

















## HINCRBY (https://redis.io/commands/hincrby)
- Increments the number stored at field in the hash stored at key by increment. If key does not exist, a new key holding a hash is created. If field does not exist the value is set to 0 before the operation is performed. The range of values supported by HINCRBY is limited to 64 bit signed integers.

Syntax:
```javascript
HINCRBY hashName field number
```

```javascript
/*
hash looks like this:
birth: Peter 1993 Lena 1994
*/


const query = ['birth', 'Peter', 2];

// callback
client.hincrby(...query, (e, res) => {
  console.log(res); // 1995
  client.quit();
});


// promise
const { promisify } = require('util');
const hincrbyAsync = promisify(client.hincrby).bind(client);

hincrbyAsync(...query)
  .then(res => {
    console.log(res); // 1995
  }) // OK
  .then(() => client.quit());
  
  
// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hincrbyAsync(...query);
console.log(res); // 1995
client.quit();
```


























































































<br><br>
__________________________________________________
__________________________________________________
<br><br>

# Read Data

<br><br>

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


## HGET (https://redis.io/commands/hget)
```javascript
/*
Our hash look like this:
github: url https://github.com domain com
*/

const query = [
  'github',
  'url'
];

// callback
client.hget(...query, (e, res) => {
  console.log(res); // https://github.com
  client.quit();
});


// promises
const { promisify } = require('util');
const hgetAsync = promisify(client.hget).bind(client);

hgetAsync(...query)
  .then(res => {
     console.log(res); // https://github.com
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hgetAsync(...query);
console.log(res); // https://github.com
client.quit();
```


<br><br>


## HGETALL (https://redis.io/commands/hgetall)
- Returns all fields and values of the hash stored at key. In the returned value, every field name is followed by its value, so the length of the reply is twice the size of the hash.
```javascript
/*
Our hash look like this:
github: url https://github.com domain com year 1993
*/

// callback
client.hgetall(github, (e, res) => {
  console.log(res); // {url: 'https://github.com', domain: 'com', year: '1993'}
  console.log(typeof res.year); // string
  client.quit();
});


// promises
const { promisify } = require('util');
const hgetallAsync = promisify(client.hgetall).bind(client);

hgetallAsync(github)
  .then(res => {
    console.log(res); // {url: 'https://github.com', domain: 'com', year: '1993'}
    console.log(typeof res.year); // string
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.hgetallAsync(github);
console.log(res); // {url: 'https://github.com', domain: 'com', year: '1993'}
console.log(typeof res.year); // string
client.quit();
```
























<br><br>


## LLEN (https://redis.io/commands/llen)
- Returns the length of the list stored at key. If key does not exist, it is interpreted as an empty list and 0 is returned. An error is returned when the value stored at key is not a list.
```javascript
/*
Our list looks like this:
planets: Earth, Sun, Moon, Neptun, Saturn
*/

// callback
client.llen('planets', (e, res) => {
  console.log(res); // 5 <-- return the length of the list
  client.quit();
});


// promises
const { promisify } = require('util');
const llenAsync = promisify(client.llen).bind(client);

llenAsync('planets')
  .then(res => {
     console.log(res); // 5 <-- return the length of the list
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.llenAsync('planets');
console.log(res); // 5 <-- return the length of the list
client.quit();
```













<br><br>


## LRANGE (https://redis.io/commands/lrange)
- Returns the specified elements of the list stored at key. The offsets start and stop are zero-based indexes, with 0 being the first element of the list (the head of the list), 1 being the next element and so on. These offsets can also be negative numbers indicating offsets starting at the end of the list. For example, -1 is the last element of the list, -2 the penultimate, and so on.
- **Be carefully with large datasets cause of perfomance**
```javascript
/*
Our list looks like this:
planets: Earth, Sun, Moon, Neptun, Saturn
*/


// callback
client.lrange(testKeyName, 0, -1, (e, res) => {
  console.log(res); // ['Earth', 'Sun', 'Moon', 'Neptun', 'Saturn']
  client.quit();
});


// promises
const { promisify } = require('util');
const lrangeAsync = promisify(client.lrange).bind(client);

lrangeAsync(testKeyName, 0, -1)
  .then(res => {
     console.log(res); // ['Earth', 'Sun', 'Moon', 'Neptun', 'Saturn']
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.lrangeAsync(testKeyName, 0, -1);
console.log(res); // ['Earth', 'Sun', 'Moon', 'Neptun', 'Saturn']
client.quit();
```









<br><br>


## SCARD (https://redis.io/commands/scard)
- Returns the set cardinality (number of elements) of the set stored at key.
```javascript
/*
Our list looks like this:
planets: Mars, Pluto, Sun, Earth
*/


// callback
client.scard('planets', (e, res) => {
  console.log(res); // 4
  client.quit();
});


// promises
const { promisify } = require('util');
const scardAsync = promisify(client.scard).bind(client);

scardAsync('planets')
  .then(res => {
     console.log(res); // 4
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.scardAsync('planets');
console.log(res); // 4
client.quit();
```










<br><br>


## SMEMBERS (https://redis.io/commands/smembers)
- Returns all the members of the set value stored at key. This has the same effect as running SINTER with one argument key.
- **Be carefully with large datasets cause of perfomance. Use SSCAN instead:**

<br><br>

Syntax:
```javascript
SMEMBERS setname
```

```javascript
/*
Our set looks like this:
planets: Mars, Pluto, Sun, Earth
*/


// callback
client.smembers('planets', (e, res) => {
  console.log(res); // ['Mars', 'Pluto', 'Sun', 'Earth']
  client.quit();
});


// promises
const { promisify } = require('util');
const smembersAsync = promisify(client.smembers).bind(client);

smembersAsync('planets')
  .then(res => {
     console.log(res); // ['Mars', 'Pluto', 'Sun', 'Earth']
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.smembersAsync('planets');
console.log(res); // ['Mars', 'Pluto', 'Sun', 'Earth']
client.quit();
```














<br><br>


## SISMEMBER (https://redis.io/commands/sismember)
- Returns if member is a member of the set stored at key.
```javascript
/*
Our set looks like this:
planets: Sun, Earth, Pluto
*/

const query = [
  'planets',
  'Earth',
];

// callback
client.sismember(...query, (e, res) => {
  console.log(res); // true
  client.quit();
});


// promises
const { promisify } = require('util');
const sismemberAsync = promisify(client.sismember).bind(client);

sismemberAsync(...query)
  .then(res => {
     console.log(res); // true
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.sismemberAsync(...query);
console.log(res); // true
client.quit();
```













<br><br>


## SDIFF (https://redis.io/commands/sdiff)
- Returns the members of the set resulting from the difference between the first set and all the successive sets.

<br><br>

Syntax:
```javascript
key1 = {a,b,c,d}
key2 = {c}
key3 = {a,c,e}
SDIFF key1 key2 key3 = {b,d}
```

```javascript
/*
Our set A like this:
planetsRed: Sun, Earth, Pluto, Moon

Our set B like this:
planetsBlue: Sun, Earth, Pluto, Neptun
*/

const query = [
  'planetsRed',
  'planetsBlue',
];

// callback
client.sdiff(...query, (e, res) => {
  console.log(res); // ['Moon', 'Neptun']
  client.quit();
});


// promises
const { promisify } = require('util');
const sdiffAsync = promisify(client.sdiff).bind(client);

sdiffAsync(...query)
  .then(res => {
     console.log(res); // ['Moon', 'Neptun']
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.sdiffAsync(...query);
console.log(res); // ['Moon', 'Neptun']
client.quit();
```
























<br><br>


## ZREVRANGE (https://redis.io/commands/zrevrange)
- Returns the specified range of elements in the sorted set stored at key. The elements are considered to be ordered from the highest to the lowest score. Descending lexicographical order is used for elements with equal score.

<br><br>

Syntax:
```javascript
ZREVRANGE sortedSetName 0 -1
```

```javascript
/*
Our sorted set looks like this:
planets: Sun:1, Earth:2, Pluto:3, Moon:4
*/

const query = [
  'planets',
  0,
  -1,
];

// callback
client.zrevrange(...query, (e, res) => {
  console.log(res); // ['Sun', 'Earth', 'Pluto', 'Moon']
  client.quit();
});


// promises
const { promisify } = require('util');
const zrevrangeAsync = promisify(client.zrevrange).bind(client);

zrevrangeAsync(...query)
  .then(res => {
     console.log(res); // ['Sun', 'Earth', 'Pluto', 'Moon']
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.zrevrangeAsync(...query);
console.log(res); // ['Sun', 'Earth', 'Pluto', 'Moon']
client.quit();
```















<br><br>


## TTL (https://redis.io/commands/ttl)
- Returns the remaining time to live of a key that has a timeout. This introspection capability allows a Redis client to check how many seconds a given key will continue to be part of the dataset.


<br><br>


Syntax:
```javascript
TTL keyName
```

```javascript
/*
Our key looks like this:
username: Peter123

And we would set an expire date before:
EXPIRE username 10
*/

// callback
client.ttl('username', (e, res) => {
  console.log(res); // 10
  client.quit();
});


// promises
const { promisify } = require('util');
const ttl = promisify(client.ttl).bind(client);

ttlAsync('username')
  .then(res => {
     console.log(res); // 10
   })
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.ttlAsync('username');
console.log(res); // 10
client.quit();
```




























































































































































<br><br>
__________________________________________________
__________________________________________________
<br><br>

# Delete Data

<br><br>


## LREM (https://redis.io/commands/lrem)
- Removes the first count occurrences of elements equal to element from the list stored at key. The count argument influences the operation in the following ways:
<br> count > 0: Remove elements equal to element moving from head to tail.
<br> count < 0: Remove elements equal to element moving from tail to head.
<br> count = 0: Remove all elements equal to element.
- **Be carefully with large datasets cause of perfomance**

<br><br>

Syntax:
```javascript
LREM listName count value
```

```javascript
/*
Our list looks like this:
planets: Sun, Earth, Pluto
*/

const query = [
  testKeyName,
  1,
  'Earth',
];


// callback
client.lrem(...query, (e, res) => {
  console.log(res)  // ['Sun', 'Pluto']
  client.quit();
});


// promises
const { promisify } = require('util');
const lremAsync = promisify(client.lrem).bind(client);

getAsync(...query)
  .then(res => console.log(res)) // ['Sun', 'Pluto']
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.lremAsync(...query);
console.log(res);  // ['Sun', 'Pluto']
client.quit();
```


<br><br>




## RPOP (https://redis.io/commands/rpop)
- Removes and returns the last elements of the list stored at key. By default, the command pops a single element from the end of the list. When provided with the optional count argument, the reply will consist of up to count elements, depending on the list's length.
```javascript
/*
Our list looks like this:
planets: Mars, Pluto, Sun, Earth
*/

// callback
client.rpop('planets', (e, res) => {
  console.log(res)  // Earth <-- ['Mars', 'Pluto', 'Sun']
  client.quit();
});


// promises
const { promisify } = require('util');
const rpopAsync = promisify(client.rpop).bind(client);

rpopAsync('planets')
  .then(res => console.log(res)) // Earth <-- ['Mars', 'Pluto', 'Sun']
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.rpopAsync('planets');
console.log(res);  // Earth <-- ['Mars', 'Pluto', 'Sun']
client.quit();
```


<br><br>









## SREM (https://redis.io/commands/srem)
- Remove the specified members from the set stored at key. Specified members that are not a member of this set are ignored. If key does not exist, it is treated as an empty set and this command returns 0. An error is returned when the value stored at key is not a set.
```javascript
/*
Our set looks like this:
planets: Mars, Pluto, Sun, Earth
*/

const query = [
  'planets',
  'Pluto',
];

// callback
client.srem(...query, (e, res) => {
  console.log(res)  // 1 <-- shows how many data got removed
  client.quit();
});


// promises
const { promisify } = require('util');
const sremAsync = promisify(client.srem).bind(client);

sremAsync(...query)
  .then(res => console.log(res)) // 1 <-- shows how many data got removed
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.sremAsync(...query);
console.log(res);  // 1 <-- shows how many data got removed
client.quit();
```




























## EXPIRE (https://redis.io/commands/expire)
- Set a timeout on key. After the timeout has expired, the key will automatically be deleted. A key with an associated timeout is often said to be volatile in Redis terminology.

<br><br>

Syntax:
```javascript
EXPIRE keyName timeoutValue
```

```javascript
/*
Our string looks like this:
username: Peter123
*/

const query = [
  'EXPIRE mykey 10',
];

// callback
client.expire(...query, (e, res) => {
  console.log(res)  // 1 <-- timeout was set
  client.quit();
});


// promises
const { promisify } = require('util');
const expireAsync = promisify(client.expire).bind(client);

expireAsync(...query)
  .then(res => console.log(res)) // 1 <-- timeout was set
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.expireAsync(...query);
console.log(res);  // 1 <-- timeout was set
client.quit();
```



















































<br><br>
__________________________________________________
__________________________________________________
<br><br>





# Scripts 
- https://www.youtube.com/watch?v=jEMK9vBUWr0


<br><br>

## LOAD (https://redis.io/commands/script-load)
- Load a script into the scripts cache, without executing it. After the specified command is loaded into the script cache it will be callable using EVALSHA with the correct SHA1 digest of the script, exactly like after the first successful invocation of EVAL.
```javascript
const query = [
  'load',
  sourceHere(),
];

// callback
client.script(...query, (e, res) => {
  console.log(res) // returns SHA
  client.quit();
});


// promises
const { promisify } = require('util');
const scriptAsync = promisify(client.script).bind(client);

scriptAsync(...query)
  .then(res => console.log(res)) // returns SHA
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.scriptAsync(...query);
console.log(res); // returns SHA
client.quit();
```













<br><br>

## EVALSHA (https://redis.io/commands/evalsha)
- Evaluates a script cached on the server side by its SHA1 digest. Scripts are cached on the server side using the SCRIPT LOAD command. The command is otherwise identical to EVAL.
```javascript
const query = [
  sha,
  1, // number of redis keys
  keyName, // where the script will oeprate on
  value, // value which will be set if script successfully
];

// callback
client.evalsha(...query, (e, res) => {
  console.log(res) // 1
  client.quit();
});


// promises
const { promisify } = require('util');
const evalshaAsync = promisify(client.evalsha).bind(client);

evalshaAsync(...query)
  .then(res => console.log(res)) // 1
  .then(() => client.quit());
  

// await
const bluebird = require('bluebird');
bluebird.promisifyAll(redis);

const res = await client.evalshaAsync(...query);
console.log(res); // 1
client.quit();
```


































































<br><br>
__________________________________________________
__________________________________________________
<br><br>


# Pipeline vs Transactions
- In easy words a transactions will block other clients requests until the transaction was successfully
<br><br>

## Use a Pipeline when:
- You have two or more commands to execute
- Can wait for the responses of all commands at one

<br><br>

## Use transaction if, in addition:
- You require atomic execution of a set of commands
- You can afford to block other clients while these command execute



<br><br>
<br><br>


# Pipeline
- https://www.youtube.com/watch?v=sXCov8qlRv8
- Instead of sending single requests each time we can use a pipeline to stack our operations inside of one single request!
<br><br>

## EXEC (https://redis.io/commands/exec)
```javascript
const pipeline = client.batch();
const testKey = `${testKeyPrefix}:example_pipeline`;
const testKey2 = `${testKeyPrefix}:example_pipeline_2`;

// all of the pipeline operations ARE NOT executed before we use exec later!
pipeline.hset(testKey, 'available', 'true');
pipeline.expire(testKey, 1000);
pipeline.sadd(testKey2, 1);

// we recieve array with all request results
const responses = await pipeline.execAsync();

expect(responses).toHaveLength(3);
expect(responses[0]).toBe(1);
expect(responses[1]).toBe(1);
expect(responses[2]).toBe(1);
```


<br><br>
<br><br>

# Transaction (https://www.youtube.com/watch?v=_XMYREa5vjg)

<br><br>


## MULTI (https://redis.io/commands/multi)
- Marks the start of a transaction block. Subsequent commands will be queued for atomic execution using EXEC.
```javascript
const transaction = client.multi();
const testKey = `${testKeyPrefix}:example_transaction`;
const testKey2 = `${testKeyPrefix}:example_transaction_2`;

transaction.hset(testKey, 'available', 'true');
transaction.expire(testKey, 1000);
transaction.sadd(testKey2, 1);

const responses = await transaction.execAsync();

expect(responses).toHaveLength(3);
expect(responses[0]).toBe(1);
expect(responses[1]).toBe(1);
expect(responses[2]).toBe(1);
```










