awscred
-------

A small standalone library to resolve AWS credentials and region details
using, in order: environment variables, INI files, and HTTP calls (either to
EC2 metadata or ECS endpoints, depending on environment).  Queues HTTP calls to
ensure no thundering herd effect will occur when credentials expire.

Example
-------

```js
var awscred = require('awscred')

awscred.load(function(err, data) {
  if (err) throw err

  console.log(data.credentials)
  // { accessKeyId: 'ABC',
  //   secretAccessKey: 'DEF',
  //   sessionToken: 'GHI',
  //   expiration: Sat Apr 25 2015 01:16:01 GMT+0000 (UTC) }

  console.log(data.region)
  // us-east-1
})
```

Or just load the credentials, if you know the region already:

```js
awscred.loadCredentials(function(err, data) {
  if (err) throw err

  console.log(data)
  // { accessKeyId: 'ABC',
  //   secretAccessKey: 'DEF',
  //   sessionToken: 'GHI',
  //   expiration: Sat Apr 25 2015 01:16:01 GMT+0000 (UTC) }
})
```

Or just load the region, synchronously:

```js
console.log(awscred.loadRegionSync())
// us-east-1
```



API
---

### awscred.load([options], cb)
### awscred.loadCredentialsAndRegion([options], cb)

Resolves AWS credentials and region details, and calls back with an object containing
`credentials` and `region` properties as highlighted in the example above.

`options` include:

  - `filename`: the name of the INI file to parse, defaults to `'~/.aws/credentials'` for credentials and `'~/.aws/config'` for region
  - `profile`: the name of the INI profile to use, defaults to `'default'`
  - `timeout`: the ms timeout on the http call to the EC2 or ECS metadata service, defaults to `5000`
  - `credentialsCallChain`: array of functions to resolve credentials, defaults to `awscred.credentialsCallChain` below
  - `regionCallChain`: array of functions to resolve region, defaults to `awscred.regionCallChain` below

All options are also passed to `http.request`, so any standard Node.js HTTP
options may be used as well.

The following environment variables are checked by default:

  - `AWS_ACCESS_KEY_ID`, `AMAZON_ACCESS_KEY_ID`, `AWS_ACCESS_KEY`
  - `AWS_SECRET_ACCESS_KEY`, `AMAZON_SECRET_ACCESS_KEY`, `AWS_SECRET_KEY`
  - `AWS_SESSION_TOKEN`, `AMAZON_SESSION_TOKEN`
  - `AWS_REGION`, `AMAZON_REGION`, `AWS_DEFAULT_REGION`
  - `AWS_PROFILE`, `AMAZON_PROFILE`

### awscred.loadCredentials([options], cb)

As above, but only resolves credentials, does not look up region. Calls
back with just the credentials object (containing `accessKeyId`,
`secretAccessKey`, and optionally `sessionToken` and `expiration` properties).

### awscred.loadRegion([options], cb)

As above, but only resolves region, does not look up credentials. Calls
back with just the region string.

### awscred.loadRegionSync([options])

As above, but returns the region directly from this function using synchronous calls.

### awscred.credentialsCallChain

The array of credential loading functions used to determine call order. By default:
`[loadCredentialsFromEnv, loadCredentialsFromIniFile, loadCredentialsFromHttp]`

### awscred.regionCallChain

The array of region loading functions used to determine call order. By default:
`[loadRegionFromEnv, loadRegionFromIniFile]`

### awscred.loadCredentialsFromEnv
### awscred.loadRegionFromEnv
### awscred.loadRegionFromEnvSync
### awscred.loadCredentialsFromIniFile
### awscred.loadRegionFromIniFile
### awscred.loadRegionFromIniFileSync
### awscred.loadCredentialsFromHttp
### awscred.loadCredentialsFromEc2Metadata
### awscred.loadCredentialsFromEcs
### awscred.loadProfileFromIniFile
### awscred.loadProfileFromIniFileSync

Individual methods to load credentials and region from different sources.
`loadCredentialsFromHttp` will choose between `loadCredentialsFromEc2Metadata`
and `loadCredentialsFromEcs` depending on whether the
`AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment variable is set (as it is on ECS).

### awscred.merge(obj, [options], cb)

Populates the `region` and `credentials` properties of `obj` using the
appropriate `load` method – depending on whether they're already set or not.
