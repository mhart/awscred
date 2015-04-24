aws-credentials
---------------

A small standalone library to resolve AWS credentials and region details
using, in order: environment variables, INI files, and EC2 metadata (for IAM
roles). Robust retries in the face of HTTP errors, and queues calls to ensure
no thundering herd effect will occur when credentials expire.

Example
-------

```js
var credentials = require('aws-credentials')

credentials.load(function(err, data) {
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

API
---

### credentials.load([options], cb)
### credentials.loadCredentialsAndRegion([options], cb)

Resolves AWS credentials and region details, and calls back with an object containing
`credentials` and `region` properties as highlighted in the example above.

`options` include:

  - `filename`: the name of the INI file to parse, defaults to `'~/.aws/config'`
  - `profile`: the name of the INI profile to use, defaults to `'default'`
  - `timeout`: the ms timeout on the http call to the EC2 metadata service, defaults to `5000`

All options are also passed to `http.request`, so any standard Node.js HTTP
options may be used as well.

The following environment variables are checked by default:

  - `AWS_ACCESS_KEY_ID`, `AMAZON_ACCESS_KEY_ID`, `AWS_ACCESS_KEY`
  - `AWS_SECRET_ACCESS_KEY`, `AMAZON_SECRET_ACCESS_KEY`, `AWS_SECRET_KEY`
  - `AWS_SESSION_TOKEN`, `AMAZON_SESSION_TOKEN`
  - `AWS_REGION`, `AMAZON_REGION`, `AWS_DEFAULT_REGION`
  - `AWS_PROFILE`, `AMAZON_PROFILE`

### credentials.loadCredentials([options], cb)

As above, but only resolves credentials, does not look up region. Calls
back with just the credentials object (containing `accessKeyId`,
`secretAccessKey`, and optionally `sessionToken` and `expiration` properties).

### credentials.loadRegion([options], cb)

As above, but only resolves region, does not look up credentials. Calls
back with just the region string.

### credentials.credentialsCallChain

The array of credential loading functions used to determine call order. By default:
`[loadCredentialsFromEnv, loadCredentialsFromIniFile, loadCredentialsFromEc2Metadata]`

### credentials.regionCallChain

The array of region loading functions used to determine call order. By default:
`[loadRegionFromEnv, loadRegionFromIniFile]`

### credentials.loadCredentialsFromEnv
### credentials.loadRegionFromEnv
### credentials.loadCredentialsFromIniFile
### credentials.loadRegionFromIniFile
### credentials.loadCredentialsFromEc2Metadata
### credentials.loadProfileFromIniFile

Individual methods to load credentials and region from different sources
