uk-towns-mongodb
========

## Say what?

In the spirit of open source, I have made a dump of a MongoDB collection that contains data on over 43,000 towns across the United Kingdom.

Each document contains the following properties:

* **town**: the actual name of the town
* **reference**: the town name converted into an easier form to search.
* **county**: the county to which the town belongs
* **country**: the country (within the UK) to which the town belongs, i.e. England, Nothern Ireland, Scotland or Wales
* **type**: the type of town, i.e. City, Town or Other
* **latitude**: the latitude of the town
* **longitude**: the longitude of the town
* **loc**: the town in the form of MongoDB's [legacy coordinate pair](https://docs.mongodb.org/manual/reference/glossary/#term-legacy-coordinate-pairs) [longitude, latitude]

## Import

To import simply use ```mongoimport```:

```bash
mongoimport -d <database> -c towns --file ./db-data/towns.json 
```

Don't forget to update the indexes so you can use MongoDB's GeoSpatial features. Access the ```mongo``` shell and update the indexes:

```bash
db.towns.createIndex( { "loc": "2d" } )
```

That should be it.

## Usage

### MongoDB

To query using MongoDB's geospatial features, you can use the following query:

```bash
db.towns.find( {  loc : { $geoWithin: { $centerSphere: [ [<longitude>, <latitude>], <distance in radians> ] } } } )
```

### Mongoose

You can use the following schema in Mongoose:

```javascript
var mongoose = require('mongoose');

var Schema = mongoose.Schema;

var townSchema = new Schema({
    town: { type: String },
    reference: { type: String },
    county: { type: String },
    country: { type: String },
    type: { type: String },
    latitude: { type: Number},
    longitude: { type: Number },
    loc: {type: [Number], index: '2d'}
});

module.exports = mongoose.model('Town', townSchema);
```

In order to perform queries an example in NodeJS:

```javascript
var Town = require('./path/to/mongoose/model/town');

module.exports = {
    search: function(longitude, latutude, kilometres) {
        return new Promise(function(resolve, reject) {
            var radians = kilometres / 6371;
      
            Town.find({
                loc: {
                    $geoWithin: {
                        $centerSphere: [
                            [longitude, latitude], radians
                        ]
                    }
                }
            },
            function(error, doc) {
                if(error) {
                    return reject(error);
                }
        
                resolve(doc);
            });
        });
    }
};
```

## License

Copyright (c) 2016 Kieran O'Neill

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

- If we meet some day, and you think this stuff is worth it, you can buy me a beer in return.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
