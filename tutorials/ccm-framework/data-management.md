### Table Of Contents
* [Introduction](#introduction)
* [Choosable Data Levels](#choosable-data-levels)
  * [Data Level 1: JavaScript Object](#data-level-1-javascript-object)
  * [Data Level 2: IndexedDB](#data-level-2-indexeddb)
  * [Data Level 3: Server-side Database](#data-level-3-server-side-database)
* [Dataset Properties](#dataset-properties)
* [Usage Example](#usage-example)
* [Supported Operations](#supported-operations)
* [Synchron vs. Asynchron](#synchron-vs-asynchron)
* [Data Caching](#data-caching)
* [Server Interface](#server-interface)
* [Usable Network Protocols](#usable-network-protocols)
* [Real-time Communication](#real-time-communication)

### Introduction
The [_ccm_ framework](Framework) provides a service for component developer for flexible data management.  
The service could be used with the method `ccm.store`. This method allows the usage of _ccm_ datastores. A _ccm_ datastore can manage datasets in one of three [choosable data levels](#choosable-data-levels) and could also be used autonomously of _ccm_ components for easy data management.

### Choosable Data Levels

#### Data Level 1: JavaScript Object
On the first level the data will be managed in an [local JavaScript object](http://akless.github.io/ccm-developer/api/ccm/ccm.Datastore.html#local). Than all managed datasets are fugitive data which are gone when webpage is reloaded.

#### Data Level 2: IndexedDB
On the second level the data will be managed in the clientside database [IndexedDB](https://en.wikipedia.org/wiki/Indexed_Database_API). Than all managed data is still there after page reload. This is specially interesting for offline functionality.

#### Data Level 3: Server-side Database
On the third level the data will be managed in any server-side database of choice. The server must have an [_ccm_ compatible interface](#server-interface). Than all managed datasets are stored persistent on a server and they are not bounded to a specific client. Different [network protocols](#usable-network-protocols) are possible for communication between client and server.

The following image shows an overview about _ccm_ data management:

![Overview about ccm data management.](http://akless.github.io/ccm-developer/images/DataManagement.png)

### Dataset Properties
Every managed dataset must have a property `key` which contains the unique key of the dataset. There are no conventions for other properties. They can be as they want.

The following code shows an example for a _ccm_ dataset:

```json
{
  "key": "quizz",
  "question": "What is the meaning of life?",
  "answers": [ "nothing", "don't know", "ccm programming", "other things" ],
  "type": "multiple_choice"
}
```

### Usage Example
The following code shows an first simple example for working with a _ccm_ datastore:

```js
// create a empty datastore of data level 1
var store = ccm.store();

// store new dataset with unique key 'test' in created datastore
var created_dataset = store.set( { key: 'test', value: 4711 } );

// get stored dataset
var requested_dataset = store.get( 'test' );

// update stored dataset
var updated_dataset = store.set( { key: 'test', value: 'foo' } );

// delete stored dataset
var deleted_dataset = store.del( 'test' );
```

For more detail how to use the method `ccm.store` see the [documentation of the method](http://akless.github.io/ccm-developer/api/ccm/ccm.html#.store).

### Supported Operations
A _ccm_ datastore supports the following operation:

Operation  | Description
---------- | ----------------------------------------------------
get(key)   | Get a single dataset with the given key.
get(query) | Get all datasets that matches the given query.
set(data)  | Create a new dataset that contains the given data.
set(prio)  | Updates a existing dataset with given priority data.
del(key)   | Delete a existing dataset with given key.

For more details how to use these operations see the [part for _ccm_ datastores](http://akless.github.io/ccm-developer/api/ccm/ccm.Datastore.html) in the [API](http://akless.github.io/ccm-developer/api/ccm/) of the [_ccm_ framework](Framework). Here you see which data level supports which operations:

Operation  | L1: JavaScript Object | L2: IndexedDB | L3: Redis   | L3: MongoDB
---------- |:---------------------:|:-------------:|:-----------:|:------------:
get(key)   | supported             | supported     | supported   | supported
get(query) | supported             | -             | -           | supported*
set(data)  | supported             | supported     | supported   | supported
set(prio)  | supported             | supported     | supported** | supported***
del(key)   | supported             | supported     | supported   | supported

\* all specific MongoDB queries are possible  
\** no partial update (updates always complete dataset)  
\*** support deep partial updates (with dot notation)  

Note that on data level 3 the server not only can have Redis and MongoDB as database. This are only common examples. For example the server-side database must not be an NoSQL database. Which of the operations other databases support depends mainly on the [server interface](#server-interface).

### Synchron vs. Asynchron
Each of the [datastore operations](#supported-operations) has a callback as second parameter. The result of an operation will be provided to the callback function as first parameter. If the operation don't causes any asynchron operation, the result will be also provided as return value. Operations on [data level 1](#data-level-1-javascript-object) are synchron and operations on higher [data levels](#choosable-data-levels) are asynchron. No matter of synchron or asynchron, in case of doubt use the callback function to collect the requested dataset(s). Using callbacks gives also the possibility to switch anytime to another data level without code changes.

The following example shows collecting the result of an _ccm_ datastore operation in both ways:

```js
var store = ccm.store();
var synchron_result = store.get( function ( asynchron_result ) {
  console.log( asynchron_result );
} );
console.log( synchron_result );
```

### Data Caching
On [data level 1](#data-level-1-javascript-object) the data will be managed in an [local JavaScript object](http://akless.github.io/ccm-developer/api/ccm/ccm.Datastore.html#local). On higher data levels this object acts as a local cache. Then all results of every datastore operation are local cached. When requesting datasets that are already cached, the datasets are taken from there. This results in a shorter execution time. Another effect is that requests for already cached datasets are synchron and the result can received directly as return value of the method call:

```js
// provides a datastore of data level 2
var store = ccm.store( { store: 'my_store' } );

// The first request for the dataset is asynchron and the result
store.get( 'test', function () {  // dataset is now local cached.

  // The second dataset request is synchron and the result is receivable
  var result = store.get( 'test' );  // as return value of the method call.

} );
```

### Server Interface
When a client uses a _ccm_ datastore of [data level 3](#data-level-3-server-side-database) for data management, than the client must declare an URL to an _ccm_ compatible server interface in the [datastore settings](http://akless.github.io/ccm-developer/api/ccm/ccm.types.html#.settings). Then all [datastore operations](#supported-operations) will be forwarded to this server interface. The server interface then can forward this operations to the server-side database. In this step the server interface must transform the according data in the form which the specific database can handle. Then the server interface sends the results back to the client in the form the client-side datastore can receive them.

The following example shows a very simple dummy server interface in [PHP](https://en.wikipedia.org/wiki/PHP) which receives the forwarded datastore operations and send them back to client without forwarding to a database:

```php
<?php
// receive datastore operation
$key     = $_GET[ 'key' ];      // get
$dataset = $_GET[ 'dataset' ];  // set (create or update)
$del     = $_GET[ 'del' ];      // del

// send dummy response back to client
if ( $key )     die( array( 'key' => $key, 'value' => 'foo' ) );   // get
if ( $dataset ) { $dataset[ 'value' ] = 'bar'; die( $dataset ); }  // set
if ( $del )     die( array( 'key' => $del, 'value' => 'baz' ) );   // del
?>
```

The server interface must not be written in PHP. For example it can also be written in JavaScript (with [Node.js](https://en.wikipedia.org/wiki/Node.js)). An better professional server interface should think of the following aspects:

* Cross-domain data transmissions with CORS or JSONP
* Support of different [network protocols](#usable-network-protocols)
* Security mechanisms for
  * user authentication
  * user rights for data operations
  * secure [network protocols](#usable-network-protocols)
* More than one choosable database
* Support of stable and efficient realtime communication
* Server stability, availability and scalability

### Usable Network Protocols
Different network protocols could be used for data transmission between client and server depending on the supported network protocols of the [server interface](#server-interface). For example an server interface could support some of the following protocols:

* [Hypertext Transfer Protocol (HTTP)](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
* [Hypertext Transfer Protocol Secure (HTTPS)](https://en.wikipedia.org/wiki/HTTPS)
* [WebSocket (WS)](https://en.wikipedia.org/wiki/WebSocket)
* [WebSocket Secure (WSS)](https://en.wikipedia.org/wiki/WebSocket#Proxy_traversal)

To choose on client-side which supported network protocol should be used, simple use the corresponding prefix in the URL to the server interface in the datastore settings. For example:

* **http**://path/to/server/interface.php
* **https**://path/to/server/interface.php
* **ws**://path/to/server/interface.php
* **wss**://path/to/server/interface.php

### Real-time Communication
In case of [realtime communication](https://en.wikipedia.org/wiki/WebRTC) with [Web Socket](https://en.wikipedia.org/wiki/WebSocket) as [network protocol](#usable-network-protocols) for a _ccm_ datastore with [data level 3](#data-level-3-server-side-database), the server informs every active client about changing datasets. To react to this informations on client-side there only must be declared a `onChange` callback for the datastore. The parameters of the callback contain the server informations about the changed dataset.

The following code shows an example for declaring  a `onChange` callback:

```js
ccm.store( {
  store: 'my_store',
  url: 'http://path/to/server/interface.php',
  onChange: function () {
    console.log( arguments );  // Shows the server informations about changed
  }                            // stored datasets in the developer console.
} );
```

Note that changing a dataset not triggers the own onChange event. The server informs only **other** clients about changed data. Open the same website in a second browser tab to trigger the event.