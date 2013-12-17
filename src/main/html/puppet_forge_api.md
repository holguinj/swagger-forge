# Puppet Forge API

* [Current Version](#current-version)
    * [Planned Improvements](#planned-improvements)
 * [Features](#features)
     * [Schema](#schema)
     * <!--[Authentication](#authentication)-->
     * [Pagination](#pagination)
     * [Errors](#errors)
     * [User Agent Required](#user-agent-required)
     * [Conditional Requests](#conditional-requests)
 * [Endpoints](#endpoints)
     * [Users](#users)
     * [Modules](#modules)
     * [Releases](#releases)

The Puppet Forge API (hereafter referred to as the Forge API) enables you to write scripts and tools that interact with the Puppet Forge website, from customized module publishing to custom aggregations against module metadata. The Forge API provides you quick access to all the data on the Puppet Forge via a RESTful interface.

Tools such as the puppet module tool and librarian-puppet use the Forge API to query and install modules, and tools like Geppetto or Pulp leverage the API to solve development workflow challenges.

## Current Version

The Forge API's current version is `v3`. It’s considered regression-stable, meaning that the returned data is guaranteed to match the schema described here, however, additional data may be added in the future and existing clients may ignore any properties they do not recognize.

The API currently exposes three resource types: [Users](#users), [Modules](#modules) and [Releases](#releases).

### Planned Improvements

These features are not implemented, but should be coming soon.

* …

## Features

### Schema

* The API is accessed over HTTPS via the `forgeapi.puppetlabs.com` domain. All data is sent and received as JSON.

* Blank fields are included as `null`.

* Nested resources will use an abbreviated representation. A link to the full representation for the resource will always be included.

* All timestamps are returned in ISO 8601 format

```
YYYY-MM-DD HH:MM:SS ±HHMM
```

* The HTTP response headers include:

	* Caching hints for [conditional requests](#conditional-requests)

<!— #### Authentication

Authentication is not currently a part of the `v3` API. —>

### Pagination

List endpoints return paginated responses, such as:

```
{
  "pagination": {
    // Integer - the total number of results available
    "total": 14,

    // Integer - the maximum size of the current page
    "limit": 10,

    // Integer - the index of the first result; zero-based
    "offset": 0,

    // URI - a link to the current page of results
    "current": "/v3/users?limit=10&offset=0",

    // URI - a link to the next page of results; may be null
    "next": "/v3/users?limit=10&offset=10",

    // URI - a link to the previous page of results; may be null
    "previous": null
  },

  // Array<Hash> - the list of results
  "results": [ /* ... */ ]
}
```

### Errors

The Puppet Forge API follows RFC 2616 and RFC 6585.

Error responses will be served with a `4xx` or `5xx` status code, and will be sent as a JSON hash with a `message` key, describing the cause of the failure.

```
{ "message": "Cannot parse request body as JSON" }
```

If the error has more specific causes (e.g. trying to update a User account with invalid data), an `errors` key will be provided containing a list of specific problems with the request.

```
{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "User",
      "field": "username",
      "code": "not_unique"
    },
    {
      "resource": "User",
      "field": "display_name",
      "code": "missing"
    }
  ]
}
```

### User-Agent Required

All API requests must include a valid User-Agent header. Requests with no User-Agent header will be rejected. The User-Agent header helps identify your application or library, so we can communicate with you if necessary. If your use of the API is informal or personal, we recommend using your username as the header.

User-Agent headers are a list of one or more product descriptions, generally taking the form:

```
<name-without-spaces>/<version> (comments)
```

As an example, the following are all useful User-Agent values:

```
MyApplication/0.0.0 Her/0.6.8 Faraday/0.8.8 Ruby/1.9.3-p194 (i486-linux)

My-Library-Name/1.2.4

myusername
```

### Conditional requests

Most responses return a Last-Modified header, though some responses will also return an ETag header. You can use the values of these headers to make subsequent requests to those resources using the If-Modified-Since and If-None-Match headers. If the resource has not changed, the server will return a `304 Not Modified`.

```
$ curl -I https://forgeapi.puppetlabs.com/v3/users/puppetlabs
HTTP/1.1 200 OK
Server: nginx/1.2.1
Date: Fri, 01 Nov 2013 00:50:13 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 252
Connection: keep-alive
Status: 200 OK
X-Frame-Options: sameorigin
X-XSS-Protection: 1; mode=block
Cache-Control: public, must-revalidate
Last-Modified: Mon, 12 Nov 2012 14:51:00 GMT
X-Node: forgeapi02
X-Revision: b81ba451f67a95cfd38312093c158a6fdc919b46
```

```
$ curl -I https://forgeapi.puppetlabs.com/v3/users/puppetlabs -H "If-Modified-Since: Mon, 12 Nov 2012 14:51:00 GMT"
HTTP/1.1 304 Not Modified
Server: nginx/1.2.1
Date: Fri, 01 Nov 2013 00:51:28 GMT
Connection: keep-alive
Status: 304 Not Modified
X-Frame-Options: sameorigin
X-XSS-Protection: 1; mode=block
Cache-Control: public, must-revalidate
Last-Modified: Mon, 12 Nov 2012 14:51:00 GMT
X-Node: forgeapi02
X-Revision: b81ba451f67a95cfd38312093c158a6fdc919b46
```

## Endpoints

### Users

Provides information about Puppet Forge user accounts.

**Schema**

```
{
  // URI - a link to full details for this User
  // Included in abbreviated representations
  "uri": "/v3/users/puppetlabs",

  // String - this User's account name
  // Included in abbreviated representations
  "username": "puppetlabs",

  // String - this User's configured display name
  // Included in abbreviated representations
  "display_name": "Puppet Labs",

  // String - the unique hash for this User's gravatar
  // Included in abbreviated representations
  "gravatar_id": "fdd009b7c1ec96e088b389f773e87aec",

  // Integer - the number of Modules published by this User
  "module_count": 69,

  // Integer - the number of Releases published by this User
  "release_count": 263,

  // Date - the date this User account was created
  "created_at": "2010-05-19 05:46:26 -0700",

  // Date - the date this User account was last updated
  "updated_at": "2012-11-12 06:51:00 -0800"
}
```

**Request Parameters**

Requests for individual Users do not accept parameters.

Requests for lists of Users accept the following parameters:

* sort_by: changes the order of returned results; valid values are
  * username       - sorted ASCIIbetically by username (default)
  * modules        - sorted by the number of published Modules
  * releases       - sorted by the number of published Releases
  * downloads      - sorted by the aggregated download counts for the User's Modules
  * latest_release - sorted by most recent release date
* limit: restricts the number of results returned (defaults to 20)
* offset: omits results from the beginning of the set, used for pagination (defaults to 0)

**Examples**

*Fetch the list of Users*

    GET https://forgeapi.puppetlabs.com/v3/users

*Fetch information about a particular User*

    GET https://forgeapi.puppetlabs.com/v3/users/[username]

### Modules

Provides information about Puppet Forge modules.

**Schema**

```
{
  // URI - a link to full details for this Module
  // Included in abbreviated representations
  "uri": "/v3/modules/puppetlabs-apache",

  // String - the Module's name
  // Included in abbreviated representations
  "name": "puppetlabs",

  // User - the User who created this Module
  // Included in abbreviated representations
  "owner": { /* ... */ },

  // Release - the current Release of this Module
  // This always returns the full representation of the Release
  "current_release": { /* ... */ },

  // Array<Release> - a list of all the Releases for this Module
  "releases": [ { /* ... */ } ],

  // Integer - the number of times any version of this Module has been downloaded
  "downloads": 53023,

  // Date - the date this Module was created
  "created_at": "2010-05-19 05:46:26 -0700",

  // Date - the date this Module was last updated
  "updated_at": "2012-11-12 06:51:00 -0800"
}
```

**Request Parameters**

Requests for individual Modules do not accept parameters.

Requests for lists of Modules accept the following parameters:

* query: limits the results to Modules matching the given string
* owner: limits the results to a particular User's Modules
* tag: limits the results to Modules that have the given tag
* show_deleted: includes Modules with no Releases in the results
* sort_by: changes the order of returned results; valid values are
  * rank      - sorted by best match for the given query (default)
  * downloads - sorted by the aggregated download counts for the Module
  * latest_release - sorted by most recent release date
* limit: restricts the number of results returned (defaults to 20)
* offset: omits results from the beginning of the set, used for pagination (defaults to 0)

**Examples**

*Fetch the list of Modules*

    GET https://forgeapi.puppetlabs.com/v3/modules

*Fetch information about a particular Module*

    GET https://forgeapi.puppetlabs.com/v3/modules/[module-slug]

#### Releases

Provides information about Puppet Forge module releases.

**Schema**

```
{
  // URI - a link to full details for this Release
  // Included in abbreviated representations
  "uri": "/v3/releases/puppetlabs-apache-0.0.4",

  // String - the Release's version
  // Included in abbreviated representations
  "version": "0.0.4",

  // Module - the Module this Release belongs to
  "module": { /* ... */ },

  // Hash - the contents of the Release's metadata.json file
  "metadata": { /* ... */ },

  // URI - a link to where the tarball can be downloaded
  "file_uri": "/v3/files/puppetlabs-apache-0.0.4.tar.gz",

  // Integer - the size of the Release's tarball, in bytes
  "file_size": 9707,

  // String - the MD5 checksum of the Release's tarball
  "file_md5": "5d1d4ec6ce20986d4be3a4bd0ecba07a",

  // String - the README file for the Release, rendered as HTML
  "readme": "...",

  // String - the Changelog file for the Release, rendered as HTML
  "changelog": "...",

  // String - the LICENSE file for the Release, rendered as HTML
  "license": "...",

  // Array<String> - a list of tags applicable to this release
  "tags": [ /* ... */ ],

  // Integer - the number of times this Release has been downloaded
  "downloads": 1482,

  // Date - the date this Release was created
  "created_at": "2010-05-19 05:46:26 -0700",

  // Date - the date this Release was last updated
  "updated_at": "2012-11-12 06:51:00 -0800",

  // Date - the date this Release was deleted
  "deleted_at": null
}
```

**Request Parameters**

Requests for individual Releases do not accept parameters.

Requests for lists of Releases accept the following parameters:

* module: limits the results to Releases of a particular Module
* owner: limits the results to a particular User's Releases
* version: limits the results to Releases matching a version range
* show_deleted: include deleted Releases in the results
* sort_by: changes the order of returned results; valid values are
  * downloads    - sorted by download count (default)
  * release_date - sorted by release date
  * module       - sorted by module name
* limit: restricts the number of results returned (defaults to 20)
* offset: omits results from the beginning of the set, used for pagination (defaults to 0)

**Examples**

*Fetch the list of Releases*

    GET https://forgeapi.puppetlabs.com/v3/releases

*Fetch information about a particular Release*

    GET https://forgeapi.puppetlabs.com/v3/releases/[release-slug]