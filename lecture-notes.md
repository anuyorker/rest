## RESTISH

*(REST Industry Standard Habits)*

RESTful routing is all about standardization and norms. Being able to predict and understand what a resource is by looking at the URL we use to interact with it. Different verbs (POST, PUT, DELETE, GET) for specifying different actions on that resource.

*What is a resource?*

An abstract entity, in our experience, often corresponding to a model or instance in sequelize. But really a resource is not necessarily a model or instance. It's just a category of data.

*Why does RESTISH matter?*

It establishes universal / reusable / predictable patterns (regardless of technology) for "clients" (frontend code).

*When do we use the query string?*

For open-ended filtering by potentially many criteria.

### Problem 1

```js
// GET /api/bears/delete/1
router.get('/api/bears/delete/:id', (req, res, next) =>
  Bear.destroy(req.params.id)
  .then(() => res.sendStatus(204))
  .catch(next)
)
```

*REST Faux Pas: Mixing Verbs & Nouns*

This could / should look like:

```js
// DELETE /api/bears/1
router.delete('/api/bears/:id', (req, res, next) =>
  Bear.destroy(req.params.id)
  .then(() => res.sendStatus(204))
  .catch(next)
)
```

### Problem 2

```js
// GET /api/games/2/4
router.get('/api/games/:gameId/:moveNumber', (req, res, next) =>
  Game.findById(req.params.gameId)
  .then(game => game.moves[req.params.moveNumber])
  .then(move => res.json(move))
  .catch(next)
)
```

*REST Faux Pas: Opaque Resource*

It is not clear what resource we are referring to.

This could / should look like:

```js
// GET /api/games/2/moves/4
router.get('/api/games/:gameId/moves/:moveNumber', (req, res, next) =>
  Game.findById(req.params.gameId)
  .then(game => game.moves[req.params.moveNumber])
  .then(move => res.json(move))
  .catch(next)
)
```

### Problem 3

```js
// GET /api/reports/5
router.get('/api/reports/:userId', (req, res, next) =>
  Report.findAll({
    where: { userId: req.params.userId }
  })
  .then(reports => res.json(reports))
  .catch(next)
)
```

*REST Faux Pas: Misleading Resource*

The above URI, as a client request, makes it look like the resource we are operating on is a report resource instance NOT a set of reports by user id.

This could / should look like:

```js
// GET /api/users/5/reports
router.get('/api/users/:userId/reports', (req, res, next) =>
  Report.findAll({
    where: { userId: req.params.userId }
  })
  .then(reports => res.json(reports))
  .catch(next)
)
```

...or:

```js
// GET /api/reports?userId=5
router.get('/api/reports', (req, res, next) =>
  Report.findAll({
    where: req.query
  })
  .then(reports => res.json(reports))
  .catch(next)
)
```

### Problem 4

```js
// GET /api/review/products/3
router.get('/api/reviews/products/:productId', (req, res, next) =>
  Review.findAll({
    where: { productId: req.params.productId }
   })
  .then(review => res.json(review))
  .catch(next)
)
```

*REST Faux Pas: Unpredictable / Non-Standard URI Structure*

This could / should look like:

```js
// GET /api/reviews?productId=3
router.get('/api/reviews', (req, res, next) =>
  Review.findAll({
    where: req.query
   })
  .then(review => res.json(review))
  .catch(next)
)
```

...or:

```js
// GET /api/products/3/reviews
router.get('/api/products/:productId/reviews', (req, res, next) =>
  Review.findAll({
    where: { productId: req.params.productId }
   })
  .then(review => res.json(review))
  .catch(next)
)
```

### What about auth?

Can be tricky, here's one way:

- Login: `PUT /api/loggedInUser`
- Signup: `POST /api/loggedInUser`
- Logout: `DELETE /api/loggedInUser`

***NONE OF THIS IS REST***

...but today it sort of is.

Here's the wikipedia link on "REST": https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints.
