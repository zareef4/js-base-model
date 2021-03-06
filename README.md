[![alt text][1.1]][1]
[![alt text][2.1]][2]

[1.1]: http://i.imgur.com/tXSoThF.png (Twitter)
[2.1]: http://i.imgur.com/0o48UoR.png (GitHub)

[1]: http://www.twitter.com/innovaeinc
[2]: http://www.github.com/ericching

# js-base-model

JavaScript supports the JSON format natively, which makes it the preferred way to define a domain model with it. Since JavaScript is a loosely typed language, defining your domain model this way comes with several shortcomings:

 - No type-checking:
    - You cannot define a property to be of a specific type, e.g. boolean, string, a prototype object, etc.
    - A boolean property, for example, can accept any value without throwing an error, e.g. "true", {}, ["true"], true, etc.
 - No constraint validation:
    - You cannot guarantee that certain properties in your domain model are required fields.
    - You cannot guarantee that certain properties can only accept non-blank values.
    - You cannot guarantee that a property only accept values that are defined in a list (e.g. ['M', 'F']).
    - You cannot guarantee that a property is of a specific type, e.g. boolean, string, a prototype object, etc.
    - You cannot guarantee that only properties that are defined by you are populated.

js-base-model solves the aforementioned issues.

## What's New
### 0.2.5
 - Fixed a validation bug related to objects with children.

### 0.2.4
 - Added the validateModel flag to the constructor. Set it to false to not validate the model upon instantiation. The default is true.

### 0.1.4
 - Fixed a bug in toJSON() that omits the _id property that Meteor uses.

### 0.1.0
 - First release.

## Features
 - type-checking for properties
 - constraint validation (required, blank, and choice for now)

## Requirements
 - [Underscore.js](http://underscorejs.org/)

## Usage
### Supported Constraints
 - type: "boolean", "number", "string", or a base model instance, e.g. AddressModel.
 - required: true or false
 - blank: true or false (for "string" type only)
 - minLength: the minimum number of characters for type "string"
 - maxLength: the maximum number of characters for type "string"

### Defining the Model(s)
```javascript
// AddressModel Definition
AddressModel = function (document, transformFromDB, validateModel) {
    BaseModel.call(this, 'AddressModel', document, transformFromDB, validateModel);
};

BaseModel.extendedBy(
    AddressModel,
    {
        street1: {
            type: "string",
            required: true,
            blank: false,
            minLength: 5,
            maxLength: 80
        },
        street2: {
            type: "string",
            required: false
        },
        city: {
            type: "string",
            required: true,
            blank: false
        },
        stateOrProvince: {
             type: "string",
             required: true,
             blank: false
        },
        zipOrPostalCode: {
            type: "string",
            required: true,
            blank: false
        },
        country: {
            type: "string",
            required: true,
            blank: false
        }
     }
);

// UserModel Definition
UserModel = function (document, transformFromDB, validateModel) {
    BaseModel.call(this, 'UserModel', document, transformFromDB, validateModel);
};

BaseModel.extendedBy(
    UserModel,
    {
        name: {
            type: "string",
            required: true,
            blank: false,
            maxLength: 80
        },
        age: {
            type: "number"
        },
        gender: {
            type: "string",
            required: true,
            choice: ['M', 'F']
        },
        address: {
            type: AddressModel,
            required: false
        }
    }
);
```

### Instantiating a Model
```javascript
var user = new UserModel({
    name: "John Doe",
    gender: "M",
    age: 21,
    address: new AddressModel({
        street1: "1 Test Drive",
        city: "Test City",
        stateOrProvince: "Test State",
        zipOrPostalCode: "12345",
        country: "United States"
    });
});
```

### Converting the Model to JSON
```javascript
var json = user.toJSON();
```

## Meteor Support
[Meteor](http://www.meteor.com) is an open-source platform based on Node.js. It handles domain model in the JSON format.

js-base-model works well with Meteor. With this class you can define strongly-typed domain models while enjoying the benefits of this incredible platform.

To transform collection documents (JSON format) to a predefined domain model:
```javascript
UserCollection = new Meteor.Collection('User', {
    // Transform MongoDB documents before they're returned in a fetch, findOne or find call,
    // and before they are passed to observer callbacks.
    transform: function (document) {
        return new UserModel(document, true, true);
    }
});
```

To save a domain model to a collection:
```javascript
UserCollection.insert(user.toJSON());
```

### Meteor Collection API Support
[Collection API](https://github.com/crazytoad/meteor-collectionapi) provides a RESTful service for a collection.

To support js-base-model, add src/meteor/collection-api/collectionApiOverride.js to your library.

### Installation
1. Install Meteorite so that you can install third-party packages.
```
sudo -H npm install -g meteorite
```

2. Add the following line to <PROJECT>/smart.json inside "packages", e.g.:
```
"packages": {
    ...
    "js-base-model": {}
}
```

3. Add the following line to the end of <PROJECT>/.meteor/packages:
```
 js-base-model
```

4. Install the js-base-model smart package:
```
mrt install
```

## Unit Tests
Unit tests are available in test/baseModel.js and relies on [CasperJS](http://casperjs.org).

### Test script: test/test.sh
