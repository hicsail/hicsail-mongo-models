# hicsail-mongo-models

Map JavaScript classes to MongoDB collections. 
This project was forked from [mongo-models](https://github.com/jedireza/mongo-models). We also added additional function and features that is need for out team. Check them out in the [release notes](https://github.com/hicsail/hicsail-mongo-models/blob/master/release-notes.md).

[![Build Status](https://img.shields.io/travis/hicsail/hicsail-mongo-models.svg)](https://travis-ci.org/hicsail/hicsail-mongo-models)
[![Dependency Status](https://img.shields.io/david/hicsail/hicsail-mongo-models.svg)](https://david-dm.org/hicsail/hicsail-mongo-models)
[![devDependency Status](https://img.shields.io/david/dev/hicsail/hicsail-mongo-models.svg)](https://david-dm.org/hicsail/hicsail-mongo-models?type=dev)
[![peerDependency Status](https://img.shields.io/david/peer/hicsail/hicsail-mongo-models.svg)](https://david-dm.org/hicsail/hicsail-mongo-models?type=peer)

[MongoDB](https://github.com/mongodb/node-mongodb-native)'s native driver for
Node.js is pretty good. We just want a little sugar on top.

[Mongoose](http://mongoosejs.com/) is awesome, and big. It's built on top of
MongoDB's native Node.js driver. It's a real deal ODM with tons of features.
You should check it out.

We wanted something in between the MongoDB driver and Mongoose. A light weight
abstraction where we can interact with collections via JavaScript classes and
get document results as instances of those classes.

We're also big fans of the object schema validation library
[joi](https://github.com/hapijs/joi). Joi works well for defining a model's
data schema.


## Install

```bash
$ npm install hicsail-mongo-models
```


## Usage

### Creating models

You extend the `MongoModels` class to create new model classes that map to
MongoDB collections. The base class also acts as a singleton so models share
one connection per process.

Let's create a `Customer` model.

```js
const Joi = require('joi');
const MongoModels = require('mongo-models');

class Customer extends MongoModels {
    static create(name, email, phone, callback) {

      const document = {
          name,
          email,
          phone
      };

      this.insertOne(document, callback);
    }

    speak() {

        console.log(`${this.name}: call me at ${this.phone}.`);
    }
}

Customer.collection = 'customers'; // the mongodb collection name

Customer.schema = Joi.object().keys({
    name: Joi.string().required(),
    email: Joi.string().email(),
    phone: Joi.string()
});

module.exports = Customer;
```

### Example

```js
const Customer = require('./customer');
const Express = require('express');
const MongoModels = require('mongo-models');

const app = Express();

MongoModels.connect(process.env.MONGODB_URI, {}, (err, db) => {

    if (err) {
        // TODO: throw error or try reconnecting
        return;
    }

    // optionally, we can keep a reference to db if we want
    // access to the db connection outside of our models
    app.db = db;

    console.log('Models are now connected to mongodb.');
});

app.post('/customers', (req, res) => {

    const name = req.body.name;
    const email = req.body.email;
    const phone = req.body.phone;

    Customer.create(name, email, phone, (err, customers) => {

        if (err) {
            res.status(500).json({ error: 'something blew up' });
            return;
        }

        res.json(customers[0]);
    });
});

app.get('/customers', (req, res) => {

    const filter = {
        name: req.query.name
    };

    Customer.find(filter, (err, customers) => {

        if (err) {
            res.status(500).json({ error: 'something blew up' });
            return;
        }

        res.json(customers);
    });
});

app.server.listen(process.env.PORT, () => {

    console.log(`Server is running on port ${process.env.PORT}`);
});
```


## API

See the [API reference](https://github.com/jedireza/mongo-models/blob/master/API.md).


## Have a question?

Any issues or questions (no matter how basic), open an issue. Please take the
initiative to read relevant documentation and be pro-active with debugging.


## Want to contribute?

Contributions are welcome. If you're changing something non-trivial, you may
want to submit an issue before creating a large pull request. 
Also Look at the original repo [mongo-models](https://github.com/jedireza/mongo-models) and also contribute back to the original repo as well!


## License

MIT
