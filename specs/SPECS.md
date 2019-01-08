# SPECIFICATIONS

This document is not a full classified specification but rather a collection of the most important requirements.

## Terms

We use [RFC2119](https://www.ietf.org/rfc/rfc2119.txt) to indicate must, must not, should, should not and may.

The term remote refers to applications using this application. The term includes it's server and client side of execution.

## Public API

### Endpoint for remotes

The app must provide a single public endpoint (Meteor method), which is used to send any traced data.

There should be one endpoint for all calls.

The endpoint should use DDP over REST.

The endpoint must not be used by the remote's client execution environment (attack surface).

### Remote's library API

The remote must include the `eduhack:sam` package in order to trace data.

The remote's client must not call the application directly but the respective SAM API on it's own server. The library should facade these calls with proper API design. 

## Authentication

### Register client application

The app must provide a way to generate a new Meteor "user" whose `_id` and login token will be used for clients. The user represents the client application.

Clients must login with these token in order to a) authenticate the app and b) allow relational mapping of the data to the app.

The login should use login via DPP with app-id and token over REST and JWT.

### Define roles for each application

A client may get certain roles assigned, so the application must provide a way to define roles.

Client requests that disobey the rules must be catched in order to prevent unintended 

## Configuration

The client app should receive a configuration document

## Data handling

### Validation

All incoming data must be validated according to a minimal feasible document schema for each type of data.

All validation fails should generate a quiet error (not sent back to the client) an 