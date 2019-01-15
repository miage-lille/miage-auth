# MIAGE Authentication

The objective of this workshop are :

- Learn to write professional javascript
- Learn to secure an API
- Learn to secure connexion to API with JWT

## Javascript Tooling

We already discover [Prettier](https://prettier.io), an opinionated code formatter.

```zsh
# npm i -D is a shortcut for npm install --save-dev
npm i -D prettier
```

> 📌 Prettier enforces a consistent code style (i.e. code formatting that won't affect the AST\*) across your entire codebase because it disregards the original styling by parsing it away and re-printing the parsed AST with its own rules that take the maximum line length into account, wrapping code when necessary.

### ESLint

[ESLint](https://eslint.org/) is a tool for identifying and reporting on patterns found in ECMAScript/JavaScript code, with the goal of making code more consistent and avoiding bugs. ESlint have two kind of rules : **Formatting rules** and **Code quality rules**

We delegated formatting rules to prettier which alleviates the need for this whole category of rules! Prettier is going to reprint the entire program from scratch in a consistent way, so it's not possible for the programmer to make a mistake there anymore :)

Using ESLint for code quality [rules](https://eslint.org/docs/rules/) avoid a lot of dummy bugs that can be statically detected and encourage best practices in code.

1. Install ESLint

```zsh
npm i -D eslint
```

2. Setup your rules

You can setup each rules in ESLint. For the purpose of this course, I decide to use [Airbnb guidelines](https://github.com/airbnb/javascript) which are fines ! Many of those practices can be checked with ESLint.

```zsh
#npx is a command from npm cli to execute binaries from ./node_modules/bin
npx eslint --init
#select 'Use a popular style guide'
#select 'Airbnb'
#answer N to the question
#select 'JSON'
#answer Y to download dependencies

touch .eslintignore
```

You have two file : `.eslintrc.json` define the configuration and `.eslintignore` tell eslint to ignore some files.

3. Work with Prettier

```zsh
npm i -D eslint-config-prettier
```

This plugin may turn off ESLint's formatting rules.

4. Edit .eslintrc.json

```json
{
  "extends": ["airbnb-base", "prettier"],
  "env": {
    "node": true
  }
}
```

This defines global variables that are predefined.

5. If you use VSCode, you can integrate eslint :

- install [eslint plugin ](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- Alter code user parameters (menu : Code > Préférences > Paramètres ) to add :

```json
    "eslint.autoFixOnSave": true
```

This will auto fix your code on save when using VSCode.

6. Add a new script in your `package.json`

```json
{
...
  "scripts": {
  "test": "jest",
  "prettier": "prettier --single-quote --trailing-comma es5 --write \"./src/**/*.js\"  ",
  "eslint": "eslint . --fix"
  },
...
}
```

## Application Programming Interface

Modern applications use web API to communicate together. Web APIs point out a way to communicate through a protocol over a socket (http, websocket, protobuf) synchronously or asynchronously.

IT industry uses many standards or protocols : SOAP/XML, XML-RPC, RESTfull/JSON, REST-RPC/JSON, ...
The lasts are the most used nowadays but there also emerging tehcnologies : Facebook GraphQL, Netflix Falcor or Google grpc.io are gaining adepts since few years.

During this training we will learn to create a REST/Json API with Node.js, serialiaze data in a database, secure your API and deploy it on a PaaS provider.

### Specification :

Our project is to build an authentication system.

### User stories :

1. _As_ an anonymous user, _I want_ to create an account _so that_ I could login.
2. _As_ an anonymous user, _I want_ to login _so that_ I could access authenticate actions.
3. _As_ an authenticate user, _I want_ to create a group _so that_ I could be a group admin.
4. _As_ a group admin, _I want_ to add or delete user _so that_ I could make my group grow.
5. _As_ a group member, _I want_ to get a list of all _so that_ I could list most recent posts.
6. _As_ an authenticated user, _I want_ to get a list of all groups _so that_ I could ask to join one of them.
7. _As_ an API operator, _I want_ secure my apis _so that_ I could deliver a good service to end-users.

### Organize your application

Your code will be store in the following organization.

    |- src
        |- api : routing inteface files
        |- controller : business code files
        |_ model : database model files

### Create the signup / login API

_We will use REST-RPC approach and JSON for response and request format. In this approach will will use HTTP verbs to describe the action on ressources defined in URL. But counter Restfull we can also define procedure in url and will not make all responses cacheable._

0. Logging

Before we start we will use a professional logger : `console.log` or `console.error` are synchronous when the destination is a terminal or a file, so they are not suitable for production. To log our application we will use [pino](https://www.npmjs.com/package/pino) lib.

Install pino

```
npm i -S pino pino-pretty
```

Then we will define our logger in `src/logger.js` with a console transport :

```js
const pino = require('pino');

const logger = pino({
  prettyPrint: { colorize: true },
  level: process.env.LEVEL || 'info',
});

module.exports = logger;
```

As you can see, we use an environnement variable `LEVEL` to define the logging level. For development this variable is define in a `.env`.
We use the lib `env-cmd` to load this file for development.

```sh
echo "LEVEL=debug\n" >> .env
npm i -D env-cmd
```

See [npm dev script](package.json#9) to watch how to use it.

1. Model

Your model will be store in PostgreSQL. To manage your development database :

- If you are using University's computer you can request for a postgreSQL database ressource.
- If you are using OSX, install [PostgresAPP](https://postgresapp.com/)
- If you are accurate with Docker, you can use [postgres container](https://hub.docker.com/_/postgres/)
- If you're using Ubuntu on your personnal computer, you can also [install postgreSQL](https://doc.ubuntu-fr.org/postgresql)
- If you're not behind university proxy, you can also create a [free PaaS database](https://www.elephantsql.com/plans.html)

Then install [sequelize](http://docs.sequelizejs.com) the ORM we will use :

```
npm i -S pg pg-hstore sequelize bcrypt
```

Read the user's model which is defined in [src/model/users.js](./src/model/users.js)

Setup the path to database in the environnement variable `DATABASE_URL`in the file `.env`.

First time you will do `npm run dev` the database will be create (see [src/model/index.js](./src/model/index.js))

To manage your database, you should install [PgAdmin](https://www.pgadmin.org/download/)

2. Routing

We could define those routes :

Public

    POST   /api/v1/users/login "log a user"
    POST   /api/v1/users/  "create a user"

Authenticated

    GET    /api/v1/users/  "get info of logged user"
    PUT    /api/v1/users/  "modify info of logged user"
    DELETE /api/v1/users/  "delete the logged user"

To manage api, we will introduce [Express](http://expressjs.com/en/4x/api.html) which is a minimalist web framework.

```
npm i -S express
```

The main thing to understand with Express is the [middelware](http://expressjs.com/en/guide/using-middleware.html) composition.

To test your api, you should install [Insomnia](https://insomnia.rest/download/) or [Postman](https://www.getpostman.com/)

#### Setup your api

Edit [src/api/index.js](src/api/index.js) :

```js
const express = require('express');
// create an express Application for our api
const api = express();
// apply a middelware to parse application/json body
api.use(express.json({ limit: '1mb' }));
// create an express router that will be mount at the root of the api
const apiRoutes = express.Router();
apiRoutes.get('/', (req, res) =>
  res.status(200).send({ message: 'hello from my api' })
);

// root of our API will be http://localhost:5000/api/v1
api.use('/api/v1', apiRoutes);
module.exports = api;
```

Edit [src/index.js](src/api/index.js) :

```js
const api = require('./api');
const db = require('./model');
const logger = require('./logger');

const port = process.env.PORT || 8080;
const ip = process.env.IP || '0.0.0.0';
// connect to database
db.sequelize
  .sync()
  .then(() =>
    // start the api
    api.listen(port, ip, err =>
      err
        ? logger.error(`🔥  Failed to start API : ${err.stack}`)
        : logger.info(`🌎  API is listening on port ${port}`)
    )
  )
  .catch(err => logger.error(`🔥  Failed to connect database : ${err.stack}`));
```

Now you can start your API with `npm run dev` and test with `curl http://localhost:5000/api/v1`

#### Serve the public routes

api (sometime called route) aim is to expose routing and controller (sometime called business) to manage logic of the app.

We will use a commodity with `omit` method from [lodash](https://lodash.com/docs/) lib to avoid to return private attributes from users.
Because it is the only method we need, we will not import the whole lib.

```sh
npm i --save lodash.omit
```

Take a look at [src/api/users.js](src/api/users.js) and [/src/controller/users.js](src/controller/users.js)

You probably notice we generate [JWT Token](https://jwt.io/introduction/) when we login a user. We will use it to manage Authentication, passing the token through the Authorization header when calling a protected route.

Install jwt-simple to encode/decode tokens

```sh
npm i -S jwt-simple
```

Edit [src/api/index.js](src/api/index.js) :

```js
const express = require('express');
const { apiUsers } = require('./users');
// create an express Application for our api
const api = express();
// apply a middelware to parse application/json body
api.use(express.json({ limit: '1mb' }));
// create an express router that will be mount at the root of the api
const apiRoutes = express.Router();
// connect api users router
apiRoutes.use('/users', apiUsers);
apiRoutes.get('/', (req, res) =>
  res.status(200).send({ message: 'hello from my api' })
);

// root of our API will be http://localhost:5000/api/v1
api.use('/api/v1', apiRoutes);
module.exports = api;
```

3. Manage Authentication

We will use [Passport](http://www.passportjs.org/docs/downloads/html/) an authentication middleware for Node.js. Passport can be unobtrusively dropped in to any Express-based web application.

Install :

```
echo "JWT_SECRET=IWillBeBack\n" >> .env
npm i -S passport passport-jwt
```

Edit `controller/auth.js` :

```js
const { ExtractJwt, Strategy } = require('passport-jwt');
const passport = require('passport');

const { getUser } = require('./users');

// Create our strategy
const jwtStrategy = opts =>
  new Strategy(opts, (jwtPayload, done) =>
    getUser(jwtPayload)
      .then(user => {
        if (user) {
          done(null, user);
        } else {
          done(null, false);
        }
        return null;
      })
      .catch(err => done(err, false))
  );

// Init passport with our jwt strategy
const initAuth = () => {
  const opts = {};
  opts.secretOrKey = process.env.JWT_SECRET;
  opts.jwtFromRequest = ExtractJwt.fromAuthHeaderWithScheme('jwt');
  passport.use(jwtStrategy(opts));
};

// Create a middleware to check authentication
const isAuthenticated = (req, res, next) =>
  passport.authenticate('jwt', { session: false }, (err, user) => {
    if (err) {
      return next(err);
    }
    if (!user) {
      return next(new Error('UNAUTHORIZED USER'));
    }
    req.user = user;
    return next();
  })(req, res, next);

module.exports = {
  isAuthenticated,
  initAuth,
};
```

Then edit `api/index.js`:

```js
const express = require('express');
const { apiUsers, apiUsersProtected } = require('./users');
const { isAuthenticated, initAuth } = require('../controller/auth');
// create an express Application for our api
const api = express();
initAuth();

// apply a middelware to parse application/json body
api.use(express.json({ limit: '1mb' }));
// create an express router that will be mount at the root of the api
const apiRoutes = express.Router();
apiRoutes
  // test api
  .get('/', (req, res) =>
    res.status(200).send({ message: 'hello from my api' })
  )
  // connect api users router
  .use('/users', apiUsers)
  // api bellow this middelware require Authorization
  .use(isAuthenticated)
  .use('/users', apiUsersProtected)
  .use((err, req, res, next) => {
    res.status(403).send({
      success: false,
      message: `${err.name} : ${err.message}`,
    });
    next();
  });

// root of our API will be http://localhost:5000/api/v1
api.use('/api/v1', apiRoutes);
module.exports = api;
```

## Exercices

1. Develop remaining Users API

2. Develop Group model, API & business to fulfill the user stories

   A Group have :

   - Title
   - Short Description
   - Metadatas (use a JSON Column)

   A Group has one owner (a User).
   A Group has also many members (Users) and a User belongs to many Group

3. Secure you api

Install [hpp](https://www.npmjs.com/package/hpp) and [helmet](https://www.npmjs.com/package/helmet) npm libs and setup the middlewares.

NPM also check if your dependencies have known vulnerabilities

```sh
npm audit
```

4. Deploy your application to Clever-Cloud (PaaS Provider)

- Create an account on [clever cloud](https://www.clever-cloud.com) linked with your github account to get 20\$ free credits
- Create a pico env with a free postgresql addon
- Setup your environment variables
- Add a clever remote to your Git
- Git push your application. [Read the doc](https://www.clever-cloud.com/doc/nodejs/nodejs/)
- Think to stop the app when you're not using it

5. Enforce SSL when you test API on remote server 

You should use the [express-sslify](https://www.npmjs.com/package/express-sslify) npm lib 
