# Google African Scholarship Challenge (Google + Udacity + Andela)

## register service worker

```js
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js')
    .then(function(reg) {
      console.log(' Serviceworker Registered. Scope is ' + reg );
    }).catch(function(e) {
      console.log('Registration failed with ' + e);
    });
}
```

Or

```js

if (!navigator.serviceWorker) return;

navigator.serviceWorker.register('/sw.js').then(function(){
    console.log('Registered!');
}).catch(function(){
    console.log('Not registered...');
});

```

## Chrome canary

&rarr; https://download-chromium.appspot.com/
&rarr; Extract and then & `$ cd chrome-linux/`
&rarr; ./chrome --user-data-dir=~/.config/chromium-canary

## Hijacking Recquests

```js
self.addEventListener('fetch', function(event) {
    // TODO: respond to all requests with an html response
    // containing an element with class="a-winner-is-me".
    // Ensure the Content-Type of the response is "text/html"
    // console.log(event.request);
    event.respondWith(
        new Response('<div class="a-winner-is-me">Hi there!</div>', {headers: {"Content-Type": "text/html"}
        })
    );
});
```

```js
self.addEventListener('fetch', function(event) {
  // TODO: only respond to requests with a
  // url ending in ".jpg"
  if (event.request.url.endsWith('.jpg'))
  event.respondWith(
    fetch('/imgs/dr-evil.gif')
  );
});
```

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
      fetch(event.request).then(function(response){
      if (response.status == 404) {
        return new Response("Oops, not found");
      }
      return response;
    }).catch(function(){
      return new Response('Uh oh, that failed');
    })
  );
});
```

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).then(function(response) {
      if (response.status === 404) {
        // TODO: instead, respond with the gif at
        // /imgs/dr-evil.gif
        // using a network request
        return (fetch('/imgs/dr-evil.gif'));
      }
      return response;
    }).catch(function() {
      return new Response("Uh oh, that totally failed!");
    })
  );
});
```

## Install cache

```js
  event.waitUntil(
    // TODO: open a cache named 'wittr-static-v1'
    // Add cache the urls from urlsToCache
    caches.open('wittr-static-v1').then(function(cache){
      return cache.addAll(urlsToCache);
    })
  );
```

## Cache Response

```js
self.addEventListener('fetch', function(event) {
  // TODO: respond with an entry from the cache if there is one.
  // If there isn't, fetch from the network.
  event.respondWith(
    caches.match(event.request).then(function(response){
      if (response) return response;
        return fetch(event.request);
    })
  );
});

```

## Update CSS

```js
  `'wittr-static-v1'` # change in first To-Do to 
  caches.delete('wittr-static-v1')
```

## Adding UX

&rarr; remember to add something on index.js

```js
IndexController.prototype._registerServiceWorker = function() {
  if (!navigator.serviceWorker) return;

  var indexController = this;

  navigator.serviceWorker.register('/sw.js').then(function(reg) {
    // TODO: if there's no controller, this page wasn't loaded
    // via a service worker, so they're looking at the latest version.
    // In that case, exit early
    if (!navigator.serviceWorker.controller) return;

    // TODO: if there's an updated worker already waiting, call
    // indexController._updateReady()
    if (reg.waiting) {
      indexController._updateReady();
      return;
    }

    // TODO: if there's an updated worker installing, track its
    // progress. If it becomes "installed", call
    // indexController._updateReady()
    if (reg.installing) {
      indexController._trackInstalling(reg.installing);
      return;
    }

    // TODO: otherwise, listen for new installing workers arriving.
    // If one arrives, track its progress.
    // If it becomes "installed", call
    // indexController._updateReady()
    reg.addEventListener('updatefound', () => {
      indexController._trackInstalling(reg.installing);
    });
  });
};
```

# Indexed DB

## Getting Started with indexedDB

```js
dbPromise.then(function(db) {
  // TODO: in the keyval store, set
  // "favoriteAnimal" to your favourite animal
  // eg "cat" or "dog"
  var tx = db.transaction('keyval', 'readwrite');
  var keyValStore = tx.objectStore('keyval');
  keyValStore.put('cheetah', 'favoriteAnimal');
  return tx.complete;
}).then(function(){
  console.log('Added favoriteAnimal:cheetah to keyval');

});
```

```js
  var dbPromise = idb.open('test-db', 4, function(upgradeDb) {
  switch(upgradeDb.oldVersion) {
    case 0:
      var keyValStore = upgradeDb.createObjectStore('keyval');
      keyValStore.put("world", "hello");
    case 1:
      upgradeDb.createObjectStore('people', { keyPath: 'name' });
    case 2:
      var peopleStore = upgradeDb.transaction.objectStore('people');
      peopleStore.createIndex('animal', 'favoriteAnimal');
  // TODO: create an index on 'people' named 'age', ordered by 'age'
    case 3:
      var peopleStore = upgradeDb.transaction.objectStore('people');
      peopleStore.createIndex('age', 'age');
  }
});

// TODO: console.log all people ordered by age
dbPromise.then(function(db) {
  var tx = db.transaction('people');
  var peopleStore = tx.objectStore('people');
  var ageIndex = peopleStore.index('age');

  return ageIndex.getAll();
}).then(function(people) {
  console.log('People by age:', people);
});

// If the above doesn't work on first try, clear serviceworker and refresh browser
```

## Using the IDB Cache

```js
function openDatabase() {
  // If the browser doesn't support service worker,
  // we don't care about having a database
  if (!navigator.serviceWorker) {
    return Promise.resolve();
  }

  // TODO: return a promise for a database called 'wittr'
  // that contains one objectStore: 'wittrs'
  // that uses 'id' as its key
  // and has an index called 'by-date', which is sorted
  // by the 'time' property
  return idb.open('wittr', 1, function(upgradeDb) {
    var store = upgradeDb.createObjectStore('wittrs', {
      keyPath: 'id'
    });
    store.createIndex('by-date', 'time');
  });
}
// called when the web socket sends message data
IndexController.prototype._onSocketMessage = function(data) {
  var messages = JSON.parse(data);

  this._dbPromise.then(function(db) {
    if (!db) return;

    // TODO: put each message into the 'wittrs'
    // object store.
    var tx = db.transaction('wittrs', 'readwrite');
    var store =tx.objectStore('wittrs');
    messages.forEach((message) => {
      store.put(message);
    });
  });

  this._postsView.addPosts(messages);
};
```

## Using IDB 2

```js
  var index = db.transaction('wittrs')
    .objectStore('wittrs').index('by-date');

  return index.getAll().then(function(messages) {
    indexController._postsView.addPosts(messages.reverse());
  });
```

## Cleaning IDB

```js
  store.index('by-date').openCursor(null, "prev").then(function(cursor) {
    return cursor.advance(30);
  }).then(function deleteRest(cursor) {
    if (!cursor) return;
    cursor.delete();
    return cursor.continue().then(deleteRest);
  });
```

## Cache Photos

```js
return caches.open(contentImgsCache).then(function(cache) {
    return cache.match(storageUrl).then(function(response) {
      if (response) return response;

      return fetch(request).then(function(networkResponse) {
        cache.put(storageUrl, networkResponse.clone());
        return networkResponse;
      });
    });
  });
```

## Cleaning Photo Cache

```js
  var images = [];

  var tx = db.transaction('wittrs');
  return tx.objectStore('wittrs').getAll().then(function(messages) {
    messages.forEach(function(message) {
      if (message.photo) {
        images.push(message.photo);
      }
    });

    return caches.open('wittr-content-imgs');
  }).then(function(cache) {
    return cache.keys().then(function(requests) {
      requests.forEach(function(request) {
        var url = new URL(request.url);
        if (!images.includes(url.pathname)) cache.delete(request);
      });
    });
  });
```

## 

```js
if (requestUrl.pathname.startsWith('/avatars/')) {
  event.respondWith(serveAvatar(event.request));
  return;
}

return caches.open(contentImgsCache).then(function(cache) {
  return cache.match(storageUrl).then(function(response) {
    var networkFetch = fetch(request).then(function(networkResponse) {
      cache.put(storageUrl, networkResponse.clone());
      return networkResponse;
    });

    return response || networkFetch;
  });
});
```

# LESSON 6

```js
/*
 * Programming Quiz: Using Let and Const (1-1)
 */

const CHARACTER_LIMIT = 255;
const posts = [
	"#DeepLearning transforms everything from self-driving cars to language translations. AND it's our new Nanodegree!",
	"Within your first week of the VR Developer Nanodegree Program, you'll make your own virtual reality app",
	"I just finished @udacity's Front-End Web Developer Nanodegree. Check it out!"
];

// prints posts to the console
function displayPosts() {
	for (let i = 0; i < posts.length; i++) {
		console.log(posts[i].slice(0, CHARACTER_LIMIT));
	}
}

displayPosts();
```

## Template Literal

```js
/*
 * Instructions: Change the `greeting` string to use a template literal.
 */

const myName = '[NAME]';
const greeting = `Hello, my name is ${myName}`;
console.log(greeting);
```

```js
/*
 * Programming Quiz: Using Let and Const (1-1)
 */

const CHARACTER_LIMIT = 255;
const posts = [
	"#DeepLearning transforms everything from self-driving cars to language translations. AND it's our new Nanodegree!",
	"Within your first week of the VR Developer Nanodegree Program, you'll make your own virtual reality app",
	"I just finished @udacity's Front-End Web Developer Nanodegree. Check it out!"
];

// prints posts to the console
/* function displayPosts() {
	for (let i = 0; i < posts.length; i++) {
		console.log(posts[i].slice(0, CHARACTER_LIMIT));
	}
}

displayPosts(); */

/* const myName = '[NAME]';
const greeting = `Hello, my name is ${myName}`;
console.log(greeting); */


/*
 * Programming Quiz: Build an HTML Fragment (1-2)
 */

const cheetah = {
    name: 'Cheetah',
    scientificName: 'Acinonyx jubatus',
    lifespan: '10-12 years',
    speed: '68-75 mph',
    diet: 'carnivore',
    summary: 'Fastest mammal on land, the cheetah can reach speeds of 60 or perhaps even 70 miles (97 or 113 kilometers) an hour over short distances. It usually chases its prey at only about half that speed, however. After a chase, a cheetah needs half an hour to catch its breath before it can eat.',
    fact: 'Cheetahs have “tear marks” that run from the inside corners of their eyes down to the outside edges of their mouth.'
};

// creates an animal trading card
function createAnimalTradingCardHTML(animal) {
    const cardHTML = `<div class="card">
        <h3 class="name"> ${ animal.name } </h3>
        <img src=" ${ animal.name }.jpg" alt=" ${ animal.name }" class="picture"> +
        <div class="description">
            <p class="fact"> ${ animal.fact } </p>
            <ul class="details">
                <li><span class="bold">Scientific Name</span>:  ${ animal.scientificName } </li>
                <li><span class="bold">Average Lifespan</span>:  ${ animal.lifespan } </li>
                <li><span class="bold">Average Speed</span>:  ${ animal.speed } </li> 
                <li><span class="bold">Diet</span>:  ${ animal.diet } </li>
            </ul>
            <p class="brief"> ${ animal.summary } </p> 
        </div>
    </div>`;

    return cardHTML;
}

console.log(createAnimalTradingCardHTML(cheetah));

```

## Destructuring Arrays

```js
const things = ['red', 'basketball', 'paperclip', 'green', 'computer', 'earth', 'udacity', 'blue', 'dogs'];

const [one,,, two,,,, three] = things;

const colors = `List of Colors
1. ${one}
2. ${two}
3. ${three}`;

console.log(colors);
```

## Object Literal Shorthand

No task

## Writing a For...of Loop (1-4)

```js
const days = ['sunday', 'monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday'];

// your code goes here
for(let day of days) {
  console.log(day.charAt(0).toUpperCase()+ day.slice(1));
}
```

## SPREAD... OPERATOR

```js
const fruits = ["apples", "bananas", "pears"];
const vegetables = ["corn", "potatoes", "carrots"];

const produce = [...vegetables, ...fruits];

console.log(produce);
```

## ...REST PARAMETER

```js
function average(...nums) {
  if (nums == '') return 0;
  else {
  let total = 0;
    for (const num of nums) {
      total += num;
    }
    return total/(nums.length);
  }
}

console.log(average(2, 6));
console.log(average(2, 3, 3, 5, 7, 10));
console.log(average(7, 1432, 12, 13, 100));
console.log(average());
```

# FUNCTIONS

Regular functions can be either function declarations or function expressions, however arrow functions are always expressions.

## Convert Function into an Arrow Function (2-1)

```js
// convert to an arrow function
const squares = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].map(square => {
	return square * square;
});

console.log(...squares);

```

## Defaults and destructuring arrays

```js
function createGrid([width = 5, height = 5]) {
  return `Generates a ${width} x ${height} grid`;
}

createGrid([]); // Generates a 5 x 5 grid
createGrid([2]); // Generates a 2 x 5 grid
createGrid([2, 3]); // Generates a 2 x 3 grid
createGrid([undefined, 3]); // Generates a 5 x 3 grid
```

## Defaults and destructuring objects

```js
function createSundae({scoops = 1, toppings = ['Hot Fudge']}) {
  const scoopText = scoops === 1 ? 'scoop' : 'scoops';
  return `Your sundae has ${scoops} ${scoopText} with ${toppings.join(' and ')} toppings.`;
}

createSundae({}); // Your sundae has 1 scoop with Hot Fudge toppings.
createSundae({scoops: 2}); // Your sundae has 2 scoops with Hot Fudge toppings.
createSundae({scoops: 2, toppings: ['Sprinkles']}); // Your sundae has 2 scoops with Sprinkles toppings.
createSundae({toppings: ['Cookie Dough']}); // Your sundae has 1 scoop with Cookie Dough toppings
```

## Using Default Function Parameters (2-2)

```js
// your code goes here
function buildHouse({floors = 1, color = 'red', walls = 'brick'} = {}) {
  return `Your house has ${floors} floor(s) with ${color} ${walls} walls.`;
}


/* tests
console.log(buildHouse()); // Your house has 1 floor(s) with red brick walls.
console.log(buildHouse({})); // Your house has 1 floor(s) with red brick walls.
console.log(buildHouse({floors: 3, color: 'yellow'})); // Your house has 3 floor(s) with yellow brick walls.
*/
```

## Quiz: Building Classes and Subclasses (2-3)

```js
class Vehicle {
	constructor(color = 'blue', wheels = 4, horn = 'beep beep') {
		this.color = color;
		this.wheels = wheels;
		this.horn = horn;
	}

	honkHorn() {
		console.log(this.horn);
	}
}

// your code goes here
class Bicycle extends Vehicle {
  constructor(color, wheels = 2, horn = 'honk honk') {
    super(color, wheels, horn);
  }
}

/* tests
const myVehicle = new Vehicle();
myVehicle.honkHorn(); // beep beep
const myBike = new Bicycle();
myBike.honkHorn(); // honk honk
*/
```

# Built-ins

## Symbols

```js
const bowl = {
  [Symbol('apple')]: { color: 'red', weight: 136.078 },
  [Symbol('banana')]: { color: 'yellow', weight: 183.15 },
  [Symbol('orange')]: { color: 'orange', weight: 170.097 },
  [Symbol('banana')]: { color: 'yellow', weight: 176.845 }
};
console.log(bowl);
```

## Sets

```js
/*
 * Programming Quiz: Using Sets (3-1)
 *
 * Create a Set object and store it in a variable named `myFavoriteFlavors`. Add the following strings to the set:
 *     - chocolate chip
 *     - cookies and cream
 *     - strawberry
 *     - vanilla
 *
 * Then use the `.delete()` method to remove "strawberry" from the set.
 */


const myFavoriteFlavors = new Set();

myFavoriteFlavors.add('chocolate chip');
myFavoriteFlavors.add('cookies and cream');
myFavoriteFlavors.add('strawberry');
myFavoriteFlavors.add('vanilla');
myFavoriteFlavors.delete('strawberry');
```

## WeakSets

&rarr; a WeakSet can only contain objects

&rarr;a WeakSet is not iterable which means it can’t be looped over

&rarr;a WeakSet does not have a .clear() method

&quot; This process of freeing up memory after it is no longer needed is what is known as garbage collection. &quot;

```js
const uniqueFlavors = new WeakSet();

let flavor1 = { flavor: 'chocolate' };
let flavor2 = { flavor: 'strawberry' };

uniqueFlavors.add(flavor1);
uniqueFlavors.add(flavor2);
uniqueFlavors.add(flavor1);
```

## Maps

"A Map is an object that lets you store key-value pairs where both the keys and the values can be objects, primitive values, or a combination of the two"

## Looping through maps

```js
const members = new Map();

members.set('Evelyn', 75.68);
members.set('Liam', 20.16);
members.set('Sophia', 0);
members.set('Marcus', 10.25);

for (const member of members) {
    let [key, value] = member;
    console.log(key, value);
}
// .forEach() approach
members.forEach((value, key) => console.log(key, value));
```

## WeakMaps
&rarr; a WeakMap can only contain objects as keys,

&rarr; a WeakMap is not iterable which means it can’t be looped and

&rarr; a WeakMap does not have a .clear() method.

## Promises

```js
mySundae.then(function(sundae) {
    console.log(`Time to eat my delicious ${sundae}`);
}, function(msg) {
    console.log(msg);
    self.goCry(); // not a real method
});
```

## Proxies

```js
const richard = {status: 'looking for work'};
const handler = {
    set(target, propName, value) {
        if (propName === 'payRate') { // if the pay is being set, take 15% as commission
            value = value * 0.85;
        }
        if (propName === 'tax') {
          value = value * 0.25;
        }
        target[propName] = value;
    }
};
const agent = new Proxy(richard, handler);
agent.tax = 1000; // set the actor's pay to $1,000
agent.tax; // $850 the actor's actual pay
```

## Getter and Setters

```js
var obj = {
    _age: 5,
    _height: 4,
    get age() {
        console.log(`getting the "age" property`);
        console.log(this._age);
    },
    get height() {
        console.log(`getting the "height" property`);
        console.log(this._height);
    }
};

obj._height;
```

```js
const proxyObj = new Proxy({age: 5, height: 4}, {
    get(targetObj, property) {
        console.log(`getting the ${property} property`);
        console.log(targetObj[property]);
    }
});

proxyObj.age; // logs 'getting the age property' & 5
proxyObj.height; // logs 'getting the height property' & 4

```

## Generators

```js
function* getEmployee() {
    console.log('the function has started');

    const names = ['Amanda', 'Diego', 'Farrin', 'James', 'Kagure', 'Kavita', 'Orit', 'Richard'];

    for (const name of names) {
        yield name;
    }

    console.log('the function has ended');
}

getEmployee();

const generatorIterator = getEmployee();
generatorIterator.next();
```

```js
function* getEmployee() {
    const names = ['Amanda', 'Diego', 'Farrin', 'James', 'Kagure', 'Kavita', 'Orit', 'Richard'];
    const facts = [];

    for (const name of names) {
        // yield *out* each name AND store the returned data into the facts array
        facts.push(yield name); 
    }

    return facts;
}

const generatorIterator = getEmployee();

// get the first name out of the generator
let name = generatorIterator.next().value;

// pass data in *and* get the next name
name = generatorIterator.next(`${name} is cool!`).value; 

// pass data in *and* get the next name
name = generatorIterator.next(`${name} is awesome!`).value; 

// pass data in *and* get the next name
name = generatorIterator.next(`${name} is stupendous!`).value; 

// you get the idea
name = generatorIterator.next(`${name} is rad!`).value; 
name = generatorIterator.next(`${name} is impressive!`).value;
name = generatorIterator.next(`${name} is stunning!`).value;
name = generatorIterator.next(`${name} is awe-inspiring!`).value;

// pass the last data in, generator ends and returns the array
const positions = generatorIterator.next(`${name} is magnificent!`).value; 

// displays each name with description on its own line
positions.join('\n');
```

## Polyfill

```sh
A polyfill, or polyfiller, is a piece of code (or plugin) that provides the technology that you, the developer, expect the browser to provide natively. 
```