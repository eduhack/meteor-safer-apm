# SAM - Safer APM for Meteor (in development, no release yet)

A safer way to APM by providing an API instead of hooking into everything. Built for your custom deployment, no registration to a cloud service required.

## Goals

##### Standards

Use and support standards where possible. Internally this means to use code quality standards or consider standards of security.
Externally this means to support devs in making their APM GDPR (or other regulations) compliant by defining your rules of tracing. Define the level of anonymization and pseudonymisation.

##### Advanced control

Give users more control for their custom hosted APM. While devs will have more work by actively including the API, they will also have the freedom to configure and stay in control.

##### Freedom and felxibility

It should be easy to integrate, change and scale this service. However, devs should have the freedom to choose how to deploy and run this service.

##### Economical and ecological

The project and the package should aim to use as few code, space, memory and cpu as possible.


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
  isClient: Meteor.isClient,
  isServer: Meteor.isServer,
  appId: provess.env.SAM_APPID,
  token: process.env.SAM_TOKEN,
  // OPTIONAL
  name: 'awesome-app', // set the app name if you want
  version: '1.0', // set an app version if you want
  heartBeat: 5000,  // set a heartbeat in ms if you want, default is 5000
})
```

#### 6. Trace what you want

##### Trace a method

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

#### 7. Configure

Your SAM client will subscribe to a config collection, where you can manage settings from your application dashboard for each registered artifact.
Requirement is though, that the artifact (method, publication etc. has to send a metric to the SAM server at least once).

You can for example mute a certain trace without further changes in the code. 
Note, that SAM won't track changes in method- or publication names. If you rename a method the old name's records will stll exist until you delete them.
