# Services

One of the main concepts within Angel, which is borrowed from FeathersJS, is a *service*. You more than likely have already dealt with another implementation of the service concept. In Angel, a *service* is a class that acts as a Web interface and exposes CRUD actions operating on a set of data. Angel services extend `Routable`, and thus can be mounted on a certain path and become REST endpoints.

The Angel core includes the `Service` base class, as well as two in-memory service classes. The `angel_mongo` package includes two service classes that let you interact with a database without writing complex code yourself.

Services can also be filtered or reacted to with hooks, which is covered in the next topic.

A service looks like this:

```dart
class MyService extends Service {
  // GET /
  // Fetch all resources
  @override Future<List> index([Map params]);

  // GET /:id
  // Fetch one resource, by its ID
  @override Future read(id, [Map params]);

  // POST /
  // Create a resource. This endpoint should return
  // the created resource.
  @override Future create(data, [Map params]);

  // PATCH /:id
  // Modifies a resource. Clients can submit only the data
  // they want to change, and the corresponding resource will
  // have only those fields changed. This endpoint should return
  // the modified resource.
  @override Future modify(id, data, [Map params]);

  // POST /:id
  // Overwrites a resource. The existing resource is completely
  // replaced by the new data. This endpoint should return the
  // new resource.
  @override Future update(id, data, [Map params]);

  // DELETE /:id
  // Deletes a resource. This endpoint should return the
  // deleted resource.
  @override Future remove(id, [Map params]);
}
```

# Service Parameters and Middleware.
You might notice that each service method accepts an optional `Map` of parameters. When accessed via HTTP (i.e., not over Websockets), `req.query` is passed here. To pass custom parameters to a service, you should create a middleware to do so. `@Middleware` annotations can be prepended to service classes or service methods. For example, the following will pass `foo='bar'` to every method in the service:

```dart
Future<bool> myMiddleware(RequestContext req, res) async {
  req.query['foo'] = 'bar';
  return true;
}

@Middleware(const [myMiddleware])
class MyService extends Service {
  // Responds with "['bar']"
  @override index([Map params]) => [params['foo']];
}
```

# Mounting Services
As mentioned above, services extend `Routable`, so you can simply `app.use()` them. You can also supplement them with additional routes or middleware, placed *before* the mounting of a service:

```dart
app.get("/user/:id/todos", (req, res) async => someAction()));

// Another way to apply a middleware to a service
app.all("/user/*", 'some middleware', middleware: ['some', 'more', 'middleware']);

app.use('/user/', new MongoTypedService<User>(db.collection("users")));
```