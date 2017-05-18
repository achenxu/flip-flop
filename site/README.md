# Mongo-dart - MongoDB driver for Dart programming language.

[![Build Status](https://travis-ci.org/mongo-dart/mongo_dart.svg?branch=master)](https://travis-ci.org/mongo-dart/mongo_dart)

[![Coverage Status](https://coveralls.io/repos/github/mongo-dart/mongo_dart/badge.svg?branch=master)](https://coveralls.io/github/mongo-dart/mongo_dart?branch=master)

Server-side driver library for MongoDb implemented in pure Dart.

## Basic usage

### Obtaining connection

```dart

  Db db = new Db("mongodb://localhost:27017/mongo_dart-blog");
  await db.open();
```

### Querying


Method `find` returns stream of maps and accept query parameters, usually build by fluent API query builder 
provided by [mongo_dart_query](https://github.com/vadimtsushko/mongo_dart_query) as top level getter `where`

```dart

  var coll = db.collection('user');
  await coll.find(where.lt("age", 18)).toList();
  
  //....
  
  await coll
      .find(where.gt("my_field", 995).sortBy('my_field'))
      .forEach((v) => print(v));
      
  //....
  
  await coll.find(where.sortBy('itemId').skip(300).limit(25)).toList();
  
```

Method `findOne` take the same parameter and returns `Future` of just one map (mongo document)

```dart

  val = await coll.findOne(where.eq("my_field", 17).fields(['str_field','my_field']));
```


Take notice in these samples that unlike mongo shell such parameters as projection (`fields`), `limit` and `skip`
are passed as part of regular query through query builder

### Inserting documents

```dart

  await usersCollection.insertAll([
    {'login': 'jdoe', 'name': 'John Doe', 'email': 'john@doe.com'},
    {'login': 'lsmith', 'name': 'Lucy Smith', 'email': 'lucy@smith.com'}
  ]);
```

### Updating documents

You can update whole document with method `save`

```dart

  var v1 = await coll.findOne({"name": "c"});
  v1["value"] = 31;
  await coll.save(v1);
```

or you can perform field level updates with method `update` and top level getter `modify` for ModifierBuilder fluent API   

```dart

  coll.update(where.eq('name', 'Daniel Robinson'), modify.set('age', 31));

```

### Removing documents

```dart

  students.remove(where.id(id));
  /// or, to remove all documents from collection
  students.remove();    
    
```


Simple app on base of [JSON ZIPS dataset] (http://media.mongodb.org/zips.json)


```dart
import 'package:mongo_dart/mongo_dart.dart';

main() async {
  void displayZip(Map zip) {
    print(
        'state: ${zip["state"]}, city: ${zip["city"]}, zip: ${zip["id"]}, population: ${zip["pop"]}');
  }
  Db db =
      new Db("mongodb://reader:vHm459fU@ds037468.mongolab.com:37468/samlple");
  var zips = db.collection('zip');
  await db.open();
  print('''
******************** Zips for state NY, with population between 14000 and 16000,
******************** reverse ordered by population''');
  await zips
      .find(where
          .eq('state', 'NY')
          .inRange('pop', 14000, 16000)
          .sortBy('pop', descending: true))
      .forEach(displayZip);
  print('\n******************** Find ZIP for code 78829 (BATESVILLE)');
  var batesville = await zips.findOne(where.eq('id', '78829'));
  displayZip(batesville);
  print('******************** Find 10 ZIP closest to BATESVILLE');
  await zips
      .find(where.near('loc', batesville["loc"]).limit(10))
      .forEach(displayZip);
  print('closing db');
  await db.close();
}
```

### See also

- [API Doc](http://www.dartdocs.org/documentation/mongo_dart/0.1.39)

- [Feature check list](https://github.com/vadimtsushko/mongo_dart/blob/master/doc/feature_checklist.md)

- [Recent change notes](https://github.com/vadimtsushko/mongo_dart/blob/master/changelog.md)

- Additional [examples](https://github.com/vadimtsushko/mongo_dart/tree/master/example) and [tests](https://github.com/vadimtsushko/mongo_dart/tree/master/test)

- For more structured approach to communication with MongoDB: [Objectory](https://github.com/vadimtsushko/objectory)

- Checkout [redstone_mapper_mongo](https://github.com/redstone-dart/redstone/wiki/redstone_mapper_mongo) for easy transformations between Mongo Dart Maps <==> Objects <==> JSON Strings, and integrations with the [redstone](https://github.com/luizmineo/redstone.dart) server framework.
