#Object Document Modeling for Cassandra
###Lucene + Cassandra = Lussandra; This also works with Cassandra ;)
_____
___Take note this module is in Beta. Use at your own risk___

This module was made to work with : https://github.com/nebulr/lussandra

Install with `npm install --save lussandra-odm`

Instantiate the cassandra client by calling :

`require ('lussandra-odm').init(config).then(function () { ... });`

Where your config follows the following format :

```
var config = {
    contactPoints : [ '192.168.99.100:9042' ],
    keyspace : 'somekeyspace',
    username : 'someusername',
    password : 'somepassword',
    replication : {
        strategy: 'SimpleStrategy', //Default is 'SimpleStrategy', NOTE: Use 'NetworkTopologyStrategy' for production
        replication_factor: 1, //Default is 1 (only used with SimpleStrategy). Not used for 'NetworkTopologyStrategy'
        strategy_options: { //Strategy options is only used for NetworkTopologyStrategy, not for SimpleStrategy
            '0': 3
            // '10':3,
            // '20':3
        }
    }
};


require ('lussandra-odm').init (config).then(function () {
  // Syncing the models with cassandra will fail if NOT done in this block
  require ('./routes')(api);
});

//or

require ('lussandra-odm').init (config).then(function () {
  // Syncing the models with cassandra will fail if NOT done in this block
  var User = require ('user.model'); // Like shown below
});

```

An example of the usage :

```
// Filename - user.model.js
var cassandra   = require ('lussandra-odm').client;
var uuid        = require ('node-uuid');

var UserModel = cassandra.model ({
  email : {
    type : cassandra.types.TEXT,
    key_type : cassandra.types.PRIMARY_KEY,
    key_order : 1
  },
  id : {
    type : cassandra.types.UUID,
    key_type : cassandra.types.INDEX,
    key_order : 2
  },
  active : { type : cassandra.types.BOOLEAN, default : true },
  password : { type : cassandra.types.TEXT },
  account : { type : cassandra.types.TEXT },
  created : { type : cassandra.types.TIMESTAMP },
  latitude : { type : cassandra.types.FLOAT },
  longitude : { type : cassandra.types.FLOAT },
  accessed : { type : cassandra.types.TIMESTAMP },
  settings :  { type : cassandra.types.JSONTOTEXT },
  role : { type : cassandra.types.TEXT },
  subrole : { type : cassandra.types.TEXT },
  setup : { type : cassandra.types.BOOLEAN }
}, {
  tableName: 'user',
  pre: function(obj) {
    if(obj.isNew) {
      obj.id = uuid.v4(); // Not required. Will create a default id uuid if undefined or not added
      obj.created = new Date();
    }
    obj.accessed = new Date(); // Will update this timestamp every time the object is modified
  },
  post:function(obj) {

  }
});

cassandra.sync(UserModel);

module.exports = UserModel;
```

and can be used like this :

```
var User = require ('user.model');

var user = new User();
user.email = req.bdy.email;
user.setup = false;
user.insert({})
.then (function (newUser) {
  res.json(newUser);
  next();
});

var user = new User();
user.find ({ email : req.body.email })
.then (function (users) {
    var foundUser = users[0];
    foundUser.email = newEmail;
    foundUser.update({ id : users[0].id })
    .then(function(updatedUser) {
      res.json(updatedUser);
      next();
    });
});

```
