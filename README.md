# Hoverboard

A very lightweight (anti-gravity?) data model and [Flux](https://facebook.github.io/flux/) store with actions and a state change listener.

[![NPM version][npm-image]][npm-url] [![Downloads][downloads-image]][npm-url] [![Bower version][bower-image]][bower-url]

[![Build Status][travis-image]][travis-url] [![Coverage Status][coveralls-image]][coveralls-url] [![Dependency status][david-dm-image]][david-dm-url] [![Dev Dependency status][david-dm-dev-image]][david-dm-dev-url]


## Installation

You can use either npm or bower to install Hoverboard, or [download the standalone files here](https://github.com/jesseskinner/hoverboard/tree/master/dist).

For more information, check out the [Concept](#concept), [Usage](#usage), [Documentation](#documentation) and [FAQ](#faq) below.


```
npm install hoverboard
```

```
bower install hoverboard-flux
```

## Concept

Hoverboard greatly simplifies [Flux](https://facebook.github.io/flux/) while staying true to the concept.

Hoverboard() takes a store definition, and returns actions automatically. Hoverboard will create actions for all the action handlers you define.

Hoverboard also makes it incredibly easy to watch the state of a store. You can add a listener by calling the model as a function. Your callback will be called immediately at first, and again whenever the state changes. This works great with React.js, because you can easily re-render your component whenever the store changes its state. Now you can finally ditch those "Controller-Views".

```javascript
UserProfileStore(function (props) {
	React.render(
		<UserProfile ...props />,
		document.getElementById('user-profile')
	);
});
```

Worried about your state being mutated? You can easily provide a custom `getState` function that will help prevent mutation or enforce immutability, or even return an API of getters for your state.

Hoverboard was inspired by other Flux implementations, like [Alt](https://github.com/goatslacker/alt) and [Reflux](https://github.com/spoike/refluxjs). Those versions are very lightweight, but Hoverboard is practically weightless.

## Usage

Here's how you might use Hoverboard to keep track of clicks with a ClickCounter.

```javascript
var ClickCounter = Hoverboard({
	click(state, text) => {
		value: state.value + 1,
		log: [...state.log, text]
	},
	reset() {
		// go back to defaults
		value: 0,
		log: []
	}
}, {
	// initial state
	value: 0,
	log: []
});

// listen to changes to the state
var unsubscribe = ClickCounter(clickState =>
	document.write(JSON.stringify(clickState) + "<br>");
);

ClickCounter.click('first');
ClickCounter.click('second');

// re-initialize back to empty state
ClickCounter.init();

unsubscribe();

ClickCounter.click("This won't show up");
```

If you run this example, you'll see this:

```javascript
{"value":0,"log":[]}
{"value":1,"log":["first"]}
{"value":2,"log":["first","second"]}
{"value":0,"log":[]}
```

To see how Hoverboard fits into a larger app, with ReactJS and a router, check out the [Hoverboard TodoMVC](http://github.com/jesseskinner/hoverboard-todomvc/).


## Documentation

Hoverboard is a function that takes a store as a single parameter, either an object or a function, and returns an object containing actions.

### Syntax

```javascript
actions = Hoverboard(store);
```

#### `store` parameter

`store` can either be an object or a function.

- If passed a function, Hoverboard will create the store by instantiating the function with `new store()`.

- If passed an object, Hoverboard will set the object as the `prototype` of an empty function, and instantiate that.

- Either way, your store will end up as an instance of a function.

	```javascript
	actions = Hoverboard({
		someAction: function(){
			return { year: 1955 };
		}
	});
	```

	```javascript
	actions = Hoverboard(function(){
		return { year: 1985 };
	});
	```

	```javascript
	var StoreClass = function(){};

	StoreClass.prototype.someAction = function(){
		return { year: 2015 };
	};

	actions = Hoverboard(StoreClass);
	```

##### `store` - user defined properties

- `store.action` (optional)

	- Any methods will be exposed as actions in the returned `actions` object.

		```javascript
		actions = Hoverboard({
			hideItem: function(state, id) {
				var items = state.items;

				items[id].hidden = true;
			
				// return the new state
				return { items: items };
			},

			loadItem: function(state, id) {			
				// can also return a promise or callback function, for async
				return api.getItem(id).then(function (result) {
					var items = state.items;
					items[id] = result;

					return { items: items };

				}, function (error) {
					// put the error in the state
					return { error: error };
				});
			},

			init: function() {
				return {
					// return a promise
					items: api.getItems().catch(function (error) {
						return { error: error };
					}),

					// can do nested objects and arrays
					clock: {
						// can use callback for anything
						time: function (callback) {
							setInterval(function () {
								callback(new Date);
							}, 1000);
						}
					}
				};
			}
		});

		actions.init();

		actions.loadItem("abc").then(function () {
			actions.hideItem("abc");
		});
		
		```

#### Return value

`Hoverboard(store)` will return an `actions` object.

##### `actions` object methods

- `actions()`

	- Returns the store's current state.

- `unsubscribe = actions(function(state) { /* do stuff */ })`

	- Adds a listener to the state of a store.
	
	- The listener callback will be called immediately, and again whenever the state changed.

    - Returns an unsubscribe function. Call it to stop listening to the state.

		```javascript
		unsubscribe = actions(function(state) {
			alert(state.value);
		});
        
        // stop listening
        unsubscribe();
		```

- `actions.handleSomeAction(arg1, arg2, ..., argN)`

	- Calls an action handler on the store, passing through any arguments.

	- Only created for action handlers with a name like onAction (matching `/^on[A-Z]/`).
	
		```javascript
		actions = Hoverboard({
			notification: function(state, message) {
				alert(message); // 123
			}
		});

		actions.notification(123);
		```


## FAQ

*Q: Is this really Flux?*

Yes. Flux requires that data flows in one direction, and Hoverboard enforces that.

When you call an action on a store, you can't get back a return value. The only way to get
data out of a store is by calling `getState`. So this ensures that data flows following
the `Action -> Dispatcher -> Store -> View` flow that is central to Flux.

---

*Q: Does Hoverboard depend on React.js? Or can I use ____ instead?*

You can use Hoverboard with any framework or library. It works really well with
React, just because it's simple to pass an entire state object as props to a
component, and have React figure out how to update the DOM efficiently.

That said, Hoverboard can still work with any other method of programming, but you
might have to do more work to decide how to update your views when the state changes.

As other frameworks start adopting React's strategy of updating the DOM, Hoverboard
will be a good fit to use with those frameworks as well.

Check out [virtual-dom](https://github.com/Matt-Esch/virtual-dom) as an alternative
to using React in your projects.

---

*Q: Is Hoverboard isomorphic? Can I use it on a node.js server?*

Yes, it can work on the server. You can add listeners to stores, and render the
page once you have everything you need from the stores. You should probably unsubscribe
from the store listeners once you've rendered the page, so you don't render it twice.

---

*Q: How does Hoverboard handle asynchronous loading of data from an API?*

There are two ways to achieve this. One way is to load the API outside of the store, and call
actions to pass in the loading state, data and/or error as it arrives:

```javascript
var Store = Hoverboard({
	onLoading: function(isLoading) {
		this.setState({ isLoading: isLoading });
	},
	onData: function(data) {
		this.setState({ data: data });
	},
	onError: function(error) {
		this.setState({ error: error });
	}
});

Store.loading(true);

getDataFromAPI(function(error, data){
	Store.loading(false);

	if (error) {
		Store.error(error);
	}

	if (data) {
		Store.data(data);
	}
});
```

Another way is to call the API from within your store itself. If you want to
defer API calls until the last minute, a nice place to do this is from your
`getInitialState` function, because it will only be called when the first action
or `getState` call is made.

```javascript
var Store = Hoverboard({
	getInitialState: function() {
		var self = this;

		getDataFromAPI(function(error, data){
			self.setState({ isLoading: false, data: data, error: error });
		});

		return { isLoading: true, data: null, error: null };
	}
});
```

---

*Q: If Hoverboard stores only have a single getter, how can I have both getAll and getById?*

You can use actions and state. If you have a list of items, and want to view a single item,
then you might want to have an items property that contains the list, and an item property
that contains the item you need. Something like this:

```javascript
var ItemStore = Hoverboard({
	onItems: function (items) {
		this.setState({ items: items });

		// update item whenever list of items changes
		this.updateItem();
	},
	onViewById: function (id) {
		this.setState({ id: id });

		// update item whenever ID changes
		this.updateItem();
	},
	updateItem: function() {
		var item = null;

		if (this.state.id && this.state.items) {
			// using underscore for this example for simplicity
			item = _.find(this.state.items, { id: this.state.id });
		}

		this.setState({ item: item });
	}
});

ItemStore.items([{ id: 123 /* ... */ }]);

// getAll
var items = ItemStore.getState().items;

// getById
ItemStore.viewById(123);
var item = ItemStore.getState().item;
```

---

*Q: Hold on. There's no global dispatcher and there's no `waitFor`, so are you sure it's really Flux?*

Yes. Ultimately Hoverboard acts as the dispatcher. Every action calls one specific action
handler in one store, so the dispatching is simple. Like Facebook's Dispatcher, Hoverboard
ensures that only one action is handled at a time, and won't allow action handlers to
pass data back to the action caller.

`waitFor` is a mechanism that Facebook's Dispatcher provides to help multiple stores
coordinate their response to a particular action. In Hoverboard, a store doesn't need
to know about which actions were called, or if some asynchronous response triggered
a change. Whenever one store changes, the other can update itself immediately.

How would you avoid using `waitFor` in Hoverboard? Let's compare using an example from the
[Facebook Flux Dispatcher tutorial](http://facebook.github.io/flux/docs/dispatcher.html):

```javascript
var flightDispatcher = new Dispatcher();

// Keeps track of which country is selected
var CountryStore = {country: null};

// Keeps track of which city is selected
var CityStore = {city: null};

// Keeps track of the base flight price of the selected city
var FlightPriceStore = {price: null};

// When a user changes the selected city, we dispatch the payload:
flightDispatcher.dispatch({
  actionType: 'city-update',
  selectedCity: 'paris'
});

// This payload is digested by CityStore:
flightDispatcher.register(function(payload) {
  if (payload.actionType === 'city-update') {
    CityStore.city = payload.selectedCity;
  }
});

// When the user selects a country, we dispatch the payload:
flightDispatcher.dispatch({
  actionType: 'country-update',
  selectedCountry: 'australia'
});

// This payload is digested by both stores:
CountryStore.dispatchToken = flightDispatcher.register(function(payload) {
  if (payload.actionType === 'country-update') {
    CountryStore.country = payload.selectedCountry;
  }
});

// When the callback to update CountryStore is registered, we save a
// reference to the returned token. Using this token with waitFor(),
// we can guarantee that CountryStore is updated before the callback
// that updates CityStore needs to query its data.
CityStore.dispatchToken = flightDispatcher.register(function(payload) {
  if (payload.actionType === 'country-update') {
    // `CountryStore.country` may not be updated.
    flightDispatcher.waitFor([CountryStore.dispatchToken]);
    // `CountryStore.country` is now guaranteed to be updated.

    // Select the default city for the new country
    CityStore.city = getDefaultCityForCountry(CountryStore.country);
  }
});

// The usage of waitFor() can be chained, for example:
FlightPriceStore.dispatchToken =
  flightDispatcher.register(function(payload) {
    switch (payload.actionType) {
      case 'country-update':
      case 'city-update':
        flightDispatcher.waitFor([CityStore.dispatchToken]);
        FlightPriceStore.price =
          getFlightPriceStore(CountryStore.country, CityStore.city);
        break;
  }
});
```

Here's how the same example would work with Hoverboard:

```javascript
// Keeps track of which country is selected
var CountryStore = Hoverboard({
	update: function (state, selectedCountry) {
		return { country: selectedCountry };
	}
}, { country: null });

// Keeps track of which city is selected
var CityStore = Hoverboard({
	init: function (state) {
		return 
	},

	update: function (state, selectedCity) {
		return { city: selectedCity };
	}
}, function (callback) {
	// listen to the CountryStore
	CountryStore(function (countryState) {
	    // Select the default city for the new country
	    if (countryState.country && state.city === null) {
			callback({
				city: getDefaultCityForCountry(countryState.country)
			});
		}
	});

	callback({ city: null });
});

CityStore.init();

// Keeps track of the base flight price of the selected city
var FlightPriceStore = Hoverboard(function (callback) {
	// called when either country or city change
	function updatePrice() {
		var country = CountryStore().country;
		var city = CityStore().city;

		if (country && city) {
			callback({
				price: getFlightPriceStore(country, city)
			});
		}
	}

	// listen to changes from both the country and city stores
	CountryStore(updatePrice);
	CityStore(updatePrice);

	return { price: null };
});

// When a user changes the selected city, we call an action:
CityStore.update('paris');

// When the user selects a country, we call an action:
CountryStore.update('australia');
```

It's pretty much the same code, just written a different way. In both examples,
the FlightPriceStore waits for the CountryStore and CityStore to change, and the flow
of data moves through the same logic and processes.


## Versioning

Hoverboard follows [semver versioning](http://semver.org/). So you can be sure that the API won't change until the next major version.


## Testing

Clone the GitHub repository, run `npm install`, and then run `mocha` to run the tests. Hoverboard has 100% test coverage.


## Contributing

Feel free to [fork this repository on GitHub](https://github.com/jesseskinner/hoverboard/fork), make some changes, and make a [Pull Request](https://github.com/jesseskinner/hoverboard/pulls).

You can also [create an issue](https://github.com/jesseskinner/hoverboard/issues) if you find a bug or want to request a feature.

Any comments and questions are very much welcome as well.


## Author

Jesse Skinner [@JesseSkinner](http://twitter.com/JesseSkinner)


## License

MIT

[coveralls-image]: https://coveralls.io/repos/jesseskinner/hoverboard/badge.png
[coveralls-url]: https://coveralls.io/r/jesseskinner/hoverboard

[npm-url]: https://npmjs.org/package/hoverboard
[downloads-image]: http://img.shields.io/npm/dm/hoverboard.svg
[npm-image]: http://img.shields.io/npm/v/hoverboard.svg
[travis-url]: https://travis-ci.org/jesseskinner/hoverboard
[travis-image]: http://img.shields.io/travis/jesseskinner/hoverboard.svg
[david-dm-url]:https://david-dm.org/jesseskinner/hoverboard
[david-dm-image]:https://david-dm.org/jesseskinner/hoverboard.svg
[david-dm-dev-url]:https://david-dm.org/jesseskinner/hoverboard#info=devDependencies
[david-dm-dev-image]:https://david-dm.org/jesseskinner/hoverboard/dev-status.svg
[bower-url]:http://badge.fury.io/bo/hoverboard-flux
[bower-image]: https://badge.fury.io/bo/hoverboard-flux.svg
