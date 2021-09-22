---
title: npm库
date: 2021-08-04 11:32:47
tags:
    - npm
    - 库
---

# ora

**Elegant terminal spinner**

```http
github.com/sindresorhus/ora
```



# chalk

**Terminal string styling done right**

```http
https://github.com/chalk/chalk
```



# octokit

**The all-batteries-included GitHub SDK for Browsers, Node.js, and Deno.**

```http
https://github.com/octokit/octokit.js
```



# @octokit/core

**Extendable client for GitHub's REST & GraphQL APIs**

```http
https://github.com/octokit/core.js
```

## options

| name                   | type                 | description                                                  |
| ---------------------- | -------------------- | ------------------------------------------------------------ |
| `options.authStrategy` | `Function`           | Defaults to [`@octokit/auth-token`](https://github.com/octokit/auth-token.js#readme). See [Authentication](https://github.com/octokit/core.js#authentication) below for examples. |
| `options.auth`         | `String` or `Object` | See [Authentication](https://github.com/octokit/core.js#authentication) below for examples. |
| `options.baseUrl`      | `String`             | When using with GitHub Enterprise Server, set `options.baseUrl` to the root URL of the API. For example, if your GitHub Enterprise Server's hostname is `github.acme-inc.com`, then set `options.baseUrl` to `https://github.acme-inc.com/api/v3`. Example`const octokit = new Octokit({  baseUrl: "https://github.acme-inc.com/api/v3", });` |
| `options.previews`     | `Array of Strings`   | Some REST API endpoints require preview headers to be set, or enable additional features. Preview headers can be set on a per-request basis, e.g.`octokit.request("POST /repos/{owner}/{repo}/pulls", {  mediaType: {    previews: ["shadow-cat"],  },  owner,  repo,  title: "My pull request",  base: "master",  head: "my-feature",  draft: true, });`You can also set previews globally, by setting the `options.previews` option on the constructor. Example:`const octokit = new Octokit({  previews: ["shadow-cat"], });` |
| `options.request`      | `Object`             | Set a default request timeout (`options.request.timeout`) or an [`http(s).Agent`](https://nodejs.org/api/http.html#http_class_http_agent) e.g. for proxy usage (Node only, `options.request.agent`).There are more `options.request.*` options, see [`@octokit/request` options](https://github.com/octokit/request.js#request). `options.request` can also be set on a per-request basis. |
| `options.timeZone`     | `String`             | Sets the `Time-Zone` header which defines a timezone according to the [list of names from the Olson database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).`const octokit = new Octokit({  timeZone: "America/Los_Angeles", });`The time zone header will determine the timezone used for generating the timestamp when creating commits. See [GitHub's Timezones documentation](https://developer.github.com/v3/#timezones). |
| `options.userAgent`    | `String`             | A custom user agent string for your app or library. Example`const octokit = new Octokit({  userAgent: "my-app/v1.2.3", });` |

# octokat

```http
https://github.com/philschatz/octokat.js
```



