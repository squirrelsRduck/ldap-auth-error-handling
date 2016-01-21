
# passport-ldapauth

[Passport](http://passportjs.org/) authentication strategy against LDAP server. This module is a Passport strategy wrapper for [ldapauth-fork](https://github.com/vesse/node-ldapauth-fork)

## Node v0.12

### `dtrace-provider` issue

Fork of [node-ldapauth](https://github.com/trentm/node-ldapauth) - A simple node.js lib to authenticate against an LDAP server.

## Node v0.12 / `dtrace-provider` issue


Currently the latest released version of [ldapjs](https://github.com/mcavage/node-ldapjs) which this module depends on does not install succesfully on Node v0.12 on Mac (see [issue #258](https://github.com/mcavage/node-ldapjs/issues/258)) due to old `dtrace-provider` dependency. To work around the issue, add dependency to `ldapjs` master to your `package.json`:

```json
{
  "dependencies": {
    "ldapjs": "mcavage/node-ldapjs",

    "passport-ldapauth": "0.3.0"

    "ldapauth-fork": "2.3.1"
  }
}
```

`dtrace-provider` is an optional dependency, ie. if you don't need it there's no need to do anything.

### SSL issue

This also comes form `ldapjs` (see [issue #258](https://github.com/mcavage/node-ldapjs/issues/258)), and the same workaround solves it.

### Microsoft AD LDAP protocol

Error 49 handles much more than just Invalid credentials, partial support for MS AD LDAP has been implemented (see [issue #35](https://github.com/vesse/passport-ldapauth/issues/35)).  Any additional supported data/comments could be added in the future.

## Install

```
npm install passport-ldapauth
```

## Status

[![Build Status](https://travis-ci.org/vesse/passport-ldapauth.png)](https://travis-ci.org/vesse/passport-ldapauth)
[![Dependency Status](https://gemnasium.com/vesse/passport-ldapauth.png)](https://gemnasium.com/vesse/passport-ldapauth)

## Usage

### Configure strategy

```javascript
var LdapStrategy = require('passport-ldapauth');

passport.use(new LdapStrategy({
    server: {
      url: 'ldap://localhost:389',
      ...
    }
  }));
```

* `server`: LDAP settings. These are passed directly to [ldapauth-fork](https://github.com/vesse/node-ldapauth-fork). See its documentation for all available options.
    * `url`: e.g. `ldap://localhost:389`
    * `bindDn`: e.g. `cn='root'`
    * `bindCredentials`: Password for bindDn
    * `searchBase`: e.g. `o=users,o=example.com`
    * `searchFilter`:  LDAP search filter, e.g. `(uid={{username}})`. Use literal `{{username}}` to have the given username used in the search.
    * `searchAttributes`: Optional array of attributes to fetch from LDAP server, e.g. `['displayName', 'mail']`. Defaults to `undefined`, i.e. fetch all attributes
    * `tlsOptions`: Optional object with options accepted by Node.js [tls](http://nodejs.org/api/tls.html#tls_tls_connect_options_callback) module.
* `usernameField`: Field name where the username is found, defaults to _username_
* `passwordField`: Field name where the password is found, defaults to _password_
* `passReqToCallback`: When `true`, `req` is the first argument to the verify callback (default: `false`):

        passport.use(new LdapStrategy(..., function(req, user, done) {
            ...
            done(null, user);
          }
        ));

Note: you can pass a function instead of an object as `options`, see the [example below](#options-as-function)

### Authenticate requests

Use `passport.authenticate()`, specifying the `'ldapauth'` strategy, to authenticate requests.

#### `authenticate()` options

In addition to [default authentication options](http://passportjs.org/guide/authenticate/) the following flash message options are available for `passport.authenticate()`:

 * `badRequestMessage`: missing username/password (default: 'Missing credentials')
 * `invalidCredentials`: `InvalidCredentialsError`, `NoSuchObjectError`, and `/no such user/i` LDAP errors (default: 'Invalid username/password')
 * `userNotFound`: LDAP returns no error but also no user (default: 'Invalid username/password')
 * `constraintViolation`: user account is locked (default: 'Exceeded password retry limit, account locked')

And for [Microsoft AD messages](http://www-01.ibm.com/support/docview.wss?uid=swg21290631), these flash message options can also be used (used instead of `invalidCredentials` if matching error code is found):

 * `invalidLogonHours`: not being allowed to login at this current time (default: 'Not Permitted to login at this time')
 * `invalidWorkstation`: not being allowed to login from this current location (default: 'Not permited to logon at this workstation')
 * `passwordExpired`: expired password (default: 'Password expired')
 * `accountDisabled`: disabled account (default: 'Account disabled')
 * `accountExpired`: expired account (default: 'Account expired')
 * `passwordMustChange`: password change (default: 'User must reset password')
 * `accountLockedOut`: locked out account (default: 'User account locked')

## Express example

```javascript
var express      = require('express'),
    passport     = require('passport'),
    bodyParser   = require('body-parser'),
    LdapStrategy = require('passport-ldapauth');

var OPTS = {
  server: {
    url: 'ldap://localhost:389',
    bindDn: 'cn=root',
    bindCredentials: 'secret',
    searchBase: 'ou=passport-ldapauth',
    searchFilter: '(uid={{username}})'
  }
};

var app = express();

passport.use(new LdapStrategy(OPTS));

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));
app.use(passport.initialize());

app.post('/login', passport.authenticate('ldapauth', {session: false}), function(req, res) {
  res.send({status: 'ok'});
});

app.listen(8080);
```

### Active Directory over SSL example

Simple example config for connecting over `ldaps://` to a server requiring some internal CA certificate (often the case in corporations using Windows AD).

```javascript
var fs = require('fs');

var opts = {
  server: {
    url: 'ldaps://ad.corporate.com:636',
    bindDn: 'cn=non-person,ou=system,dc=corp,dc=corporate,dc=com',
    bindCredentials: 'secret',
    searchBase: 'dc=corp,dc=corporate,dc=com',
    searchFilter: '(&(objectcategory=person)(objectclass=user)(|(samaccountname={{username}})(mail={{username}})))',
    searchAttributes: ['displayName', 'mail'],
    tlsOptions: {
      ca: [
        fs.readFileSync('/path/to/root_ca_cert.crt')
      ]
    }
  }
};
...
```

<a name="options-as-function"></a>
## Asynchronous configuration retrieval

Instead of providing a static configuration object, you can pass a function as `options` that will take care of fetching the configuration. It will be called with the `req` object and a callback function having the standard `(err, result)` signature. Notice that the provided function will be called on every authenticate request.

```javascript
var getLDAPConfiguration = function(req, callback) {
  // Fetching things from database or whatever
  process.nextTick(function() {
    var opts = {
      server: {
        url: 'ldap://localhost:389',
        bindDn: 'cn=root',
        bindCredentials: 'secret',
        searchBase: 'ou=passport-ldapauth',
        searchFilter: '(uid={{username}})'
      }
    };

    callback(null, opts);
  });
};

var LdapStrategy = require('passport-ldapauth');

passport.use(new LdapStrategy(getLDAPConfiguration,
  function(user, done) {
    ...
    return done(null, user);
  }
));
```

## License

MIT

## About the fork

This fork was originally created and published because of an urgent need to get newer version of [ldapjs](http://ldapjs.org/) in use to [passport-ldapauth](https://github.com/vesse/passport-ldapauth) since the newer version supported passing `tlsOptions` to the TLS module. Since then a lot of issues from the original module ([#2](https://github.com/trentm/node-ldapauth/issues/2), [#3](https://github.com/trentm/node-ldapauth/issues/3), [#8](https://github.com/trentm/node-ldapauth/issues/8), [#10](https://github.com/trentm/node-ldapauth/issues/10), [#11](https://github.com/trentm/node-ldapauth/issues/11), [#12](https://github.com/trentm/node-ldapauth/issues/12), [#13](https://github.com/trentm/node-ldapauth/pull/13)) have been fixed, and new features have been added as well.

Multiple [ldapjs](http://ldapjs.org/) client options have been made available.

## Usage

```javascript
var LdapAuth = require('ldapauth-fork');
var options = {
    url: 'ldaps://ldap.example.com:636',
    ...
};
var auth = new LdapAuth(options);
...
auth.authenticate(username, password, function(err, user) { ... });
...
auth.close(function(err) { ... })
```

## Install

    npm install ldapauth-fork


## License

MIT. See "LICENSE" file.


## `LdapAuth` Config Options

[Use the source Luke](https://github.com/vesse/node-ldapauth-fork/blob/master/lib/ldapauth.js#L25-93)


## express/connect basicAuth example

```javascript
var connect = require('connect');
var LdapAuth = require('ldapauth-fork');

// Config from a .json or .ini file or whatever.
var config = {
  ldap: {
    url: "ldaps://ldap.example.com:636",
    bindDn: "uid=myadminusername,ou=users,o=example.com",
    bindCredentials: "mypassword",
    searchBase: "ou=users,o=example.com",
    searchFilter: "(uid={{username}})"
  }
};

var ldap = new LdapAuth({
  url: config.ldap.url,
  bindDn: config.ldap.bindDn,
  bindCredentials: config.ldap.bindCredentials,
  searchBase: config.ldap.searchBase,
  searchFilter: config.ldap.searchFilter,
  //log4js: require('log4js'),
  cache: true
});

var basicAuthMiddleware = connect.basicAuth(function (username, password, callback) {
  ldap.authenticate(username, password, function (err, user) {
    if (err) {
      console.log("LDAP auth error: %s", err);
    }
    callback(err, user)
  });
});
```
