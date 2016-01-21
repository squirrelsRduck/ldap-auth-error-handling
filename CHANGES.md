
## Changes

* v0.3.1
    * [#35](https://github.com/vesse/passport-ldapauth/issues/35) - Show more specific error messages from Microsoft AD login errors if identified.
* v0.3.0
    * [#10](https://github.com/vesse/passport-ldapauth/issues/10) - Add support for fetching groups. While this is really coming from [ldapauth-fork](https://github.com/vesse/node-ldapauth-fork), updated the minor version of this library as well to draw attention to new features.
* v0.2.6
    * [#24](https://github.com/vesse/passport-ldapauth/pull/24) - Pass `req` to options function, enables request specific LDAP configuration.
* v0.2.5
    * [#21](https://github.com/vesse/passport-ldapauth/issues/21) - Handle `constraintViolationError` as a login failure instead of an error.
* v0.2.4
    * Inherit from [passport-strategy](https://github.com/jaredhanson/passport-strategy) like `passport-local` and others do.
* v0.2.3
    * Documentation using the same keys as ldapjs (bindDn and bindCredentials)
* v0.2.2
    * Allow configuring flash messages when calling `passport.authenticate()`
    * Return HTTP 400 when username or password is missing
* v0.2.1
    * Passport as peerDependency, prevents version incompatibility
* v0.2.0
    * [#8](https://github.com/vesse/passport-ldapauth/issues/8) - Possibility to provide a callback function instead of options object to constructor (contributed by Linagora)
    * Update Passport dependency to 0.2.0
    * Get rid of `var self = this;`
* v0.1.2
    * [#6](https://github.com/vesse/passport-ldapauth/issues/6) - Handle NoSuchObjectError as login failure.
* v0.1.1
    * Documentation changes due to renaming git repository of `ldapauth-fork`
* v0.1.0
    * Use [ldapauth-fork](https://github.com/vesse/node-ldapauth-fork) instead of
      [ldapauth](https://github.com/trentm/node-ldapauth)
        * ldapjs upgraded to 0.6.3
        * New options including `tlsOptions`
    * Refactored tests
* v0.0.6 (14 July 2013)
    * Fixes [#1](https://github.com/vesse/passport-ldapauth/issues/1)
    * Updated devDependencies
* v0.0.5 (16 April 2013)
    * Create LDAP client on every request to prevent socket being closed due
      to inactivity.
* v0.0.4 (14 April 2013)
    * Fixed passport-ldapauth version range.
* v0.0.3 (14 April 2013)
    * Initial release.




# node-ldapauth-fork Changelog

## 2.3.3

- [issue #20] Sanitize user input

## 2.3.2

- [issue #19] Added messages to options asserts

## 2.3.1

- [issue #14] Use bcryptjs instead of the C++ version.

## 2.3.0

- [passport-ldapauth issue #10] Added support for fetching user groups. If `groupSearchBase` and `groupSearchFilter` are defined, a group search is conducted after the user has succesfully authenticated. The found groups are stored to `user._groups`:

```javascript
new LdapAuth({
  "url": "ldaps://ldap.example.com:636",
  "adminDn": "cn=LdapAdmin,dc=local",
  "adminPassword": "LdapAdminPassword",
  "searchBase": "dc=users,dc=local",
  "searchFilter": "(&(objectClass=person)(sAMAccountName={{username}}))",
  "searchAttributes": [
    "dn", "cn", "givenName", "name", "memberOf", "sAMAccountName"
  ],
  "groupSearchBase": "dc=groups,dc=local",
  "groupSearchFilter": "(member={{dn}})",
  "groupSearchAttributes": ["dn", "cn", "sAMAccountName"]
});
```

## 2.2.19

- [issue #9] Configurable bind parameter. Thanks to @oanuna

## 2.2.18

- [issue #8] Fix options to actually work as documented

## 2.2.17

- Added `bindCredentials` option. Now defaulting to same names as ldapjs.

## 2.2.16

- Added option `includeRaw` for including `entry.raw` to the returned object (relates to ldapjs issue #238)

## 2.2.15

- [issue #5] Handle missing bcrypt and throw a explanatory exception instead

## 2.2.14

- [issue #4] Log error properties code, name, and message instead of the object

## 2.2.12

- [issue #1] Add more ldapjs options

## 2.2.11

- [passport-ldapauth issue #3] Update to ldapjs 0.7.0 fixes unhandled errors when using anonymous binding

## 2.2.10

- Try to bind with empty `adminDn` string (undefined/null equals no admin bind)

## 2.2.9

- [ldapauth issue #13] bcrypt as an optional dependency

## 2.2.8

- [ldapauth issue #2] support anonymous binding
- [ldapauth issue #3] unbind clients in `close()`
- Added option `searchScope`, default to `sub`

## 2.2.7

- Renamed to node-ldapauth-fork

## 2.2.6

- Another readme fix

## 2.2.5

- Readme updated

## 2.2.4

- [ldapauth issues #11, #12] update to ldapjs 0.6.3
- [ldapauth issue #10] use global search/replace for {{username}}
- [ldapauth issue #8] enable defining attributes to fetch from LDAP server

# node-ldapauth Changelog

## 2.2.3 (not yet released)

(nothing yet)


## 2.2.2

- [issue #5] update to bcrypt 0.7.5 (0.7.3 fixes potential mem issues)


## 2.2.1

- Fix a bug where ldapauth `authenticate()` would raise an example on an empty
  username.


## 2.2.0

- Update to latest ldapjs (0.5.6) and other deps.
  Note: This makes ldapauth only work with node >=0.8 (because of internal dep
  in ldapjs 0.5).


## 2.1.0

- Update to ldapjs 0.4 (from 0.3). Crossing fingers that this doesn't cause breakage.


## 2.0.0

- Add `make check` for checking jsstyle.
- [issue #1] Update to bcrypt 0.5. This means increasing the base node from 0.4
  to 0.6, hence the major version bump.


## 1.0.2

First working version.

