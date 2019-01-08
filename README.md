# SAM - SAFER APM for Meteor (in development, no release yet)

A safer way to APM by providing an API instead of hooking into everything. Built for your custom deployment, no registration to a cloud service required.

## Goals

##### Standards

Use and support standards where possible. Internally this means to use code quality standards or consider standards of security.
Externally this means to support devs in making their APM GDPR (or other regulations) compliant by defining your rules of tracing. Define the level of anonymization and pseudonymisation.

##### Advanced control

Give users more control for their custom hosted APM. While devs will have more work by actively including the API, they will also have the freedom to configure and stay in control.

##### Freedom and felxibility

It should be easy to integrate, change and scale this service.  However, devs should have the freedom to choose how to integrate, deploy and operate this service as much as possible.

##### Economically and ecologically balance

The project and the package should aim to use as few code, space, memory and cpu as possible but not in the expense of functionality.

##### Respect

The service has to respect the design and execution of the code it traces. Thus it is design to be "receiving" and won't alter any of your code. No namespace pollution, no injections, no hooks.
All API functions are provided as wrappers for your functions.

## How it works

The APM system consists of the APM Server (this project) and the API package `eduhack:sam` that is added to the client.

You can register as many new clients via the API as you want, so you can monior multiple clients via one APM system.

A registered client sends nothing by default. You have to include the metrics call on your own. You can register the following interfaces with SAM:

**Server**

* Meteor methods
* Meteor publications
* user sessions
* errors
* rate limit exceedances
* startup performance

**Client**

* errors
* Tracker autoruns
* Method calls
* Subscriptions
* Rendering performance (frontend agnostic)
* Routes
* startup performance

### Installation

#### 1. Deploy your SAM build on the server

To deploy a build you can either clone this repo and build one of the `release-*` branches or download the release binary attached to the repo.
Note, that we do not wrap the build into a MUP / Docker image. This is up to you as we don't want to force people into using a certain way of deployment.

If you are interested in creating a fork that uses MUP and aim to continuous integrate our builds / releases into the fork, feel free to open an issue.

#### 2. Follow the installation routine

You have to create an account to login into your dashboard, manage apps etc. Following the installation wizard.

#### 3. Generate a new app id and registration token

Your clients (that can also be an app's server environment) will have to authenticate to your SAM deployment using a remote connection and DDP login.
You can add a new application and view their app id and auth token 

#### 4. Include SAM API in your app

```bash
$ meteor add eduhack:sam
```

#### 5. Register your client

Include the following line at the very beginning of your server or client startup routine code (to ensure you collect metrics as early as possible):

```javascript
import { SAM } from 'eduhack:sam'
SAM.register({
  // REQUIRED
  remote: 'https://myapmserverurl.tld:1234',
  appId: provess.env.SAM_APPID,
  token: process.env.SAM_TOKEN,
  // OPTIONAL
  name: 'awesome-app', // set the app name if you want
  version: '1.0', // set an app version if you want
  heartBeat: 5000,  // set a heartbeat in ms if you want, default is 5000
  startup: true // trace startup performance
})
```

#### 6. Trace what you want

You integrate the API straight forward or abstract it away, which is up to you. The following examples use it straight forward.

##### Trace a method

Wrap a Meteor method's function using `SAM.method(fct)`.

```javascript
import { SAM } from 'eduhack:sam'

const tracedMethod = SAM.method(function (/* ... */) {
  // ...
})

// either the classic way
Meteor.methods({
  'someMethod': tracedMethod
})

// or using mdg:validated-method
const validatedTracedMethod = new ValidatedMethod({
  name: 'some-validated-method',
  validate() {
    // ...
  },
  run: tracedMethod
})
```

You could also integrate SAM as mixin in your `mdg:validated-method`.

##### Trace a publication

Wrap a publication's function using `SAM.publication(fct)` and pass to `Meteor.publish`:

```javascript
import { SAM } from 'eduhack:sam'

const tracedPublication = SAM.publication(function (/* ... */) {
  // ...
})
Meteor.publish('myPublication', tracedPublication)
```

##### Trace an Error

```javascript
try {
  // do something that causes an error here
} catch (e) {
  SAM.error(e)
}
```

### User sessions

To track user logins / logouts etc. you can place `SAM.user()` in `Accounts` related callbacks. For example:

*server*

```javascript
Accounts.onLogin((details) => {
  SAM.user(details)
  // ...
})

Accounts.onLoginFailure((details) => {
  SAM.user(details)
  // ...
})

Accounts.onLogout((details) => {
  SAM.user(details) // indicate a logout success
  // ...
})
```

The `details` object contains the following data:

**type** String

The service name, such as “password” or “twitter”.

**allowed** Boolean

Whether this login is allowed and will be successful (if not aborted by any of the validateLoginAttempt callbacks). False if the login will not succeed (for example, an invalid password or the login was aborted by a previous validateLoginAttempt callback).

**error** Exception

When allowed is false, the exception describing why the login failed. It will be a Meteor.Error for failures reported to the user (such as invalid password), and can be a another kind of exception for internal errors.

**user** Object

When it is known which user was attempting to login, the Meteor user object. This will always be present for successful logins.

**connection** Object

The connection object the request came in on. See Meteor.onConnection for details.

**methodName** String

The name of the Meteor method being used to login.

**methodArguments** Array




### Trace arbitrary intervals

To trace any performance you can place `SAM.start(id)` and `SAM.end(id)` into your code.


#### 7. Configure

Your SAM client will subscribe to a config collection, where you can manage settings from your application dashboard for each registered artifact.
Requirement is though, that the artifact (method, publication etc. has to send a metric to the SAM server at least once).

You can for example mute a certain trace without further changes in the code. 
Note, that SAM won't track changes in method- or publication names. If you rename a method the old name's records will stll exist until you delete them.
