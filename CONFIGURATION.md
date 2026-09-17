# Shibboleth SSO Filter Configuration Guide

This document explains the configuration options for `ShibbolethSSOFilter`
(module `shibboleth-filter`, config parsed by `shibboleth-filter-config`).
Options are read from a Java `.properties` file; see
[shibboleth-filter/src/main/resources/ShibbolethSSOFilter.example.properties](shibboleth-filter/src/main/resources/ShibbolethSSOFilter.example.properties)
for a runnable example, and [shibboleth-filter/README.md](shibboleth-filter/README.md)
for how to wire the filter itself into Crowd.

## Locating the config file

By default the file must be named `ShibbolethSSOFilter.properties` and be on
the classpath (typically `%crowd-webapp%/WEB-INF/classes`). You can override
the location with the `SHIBBOLETH_FILTER_CONFIG` environment variable, set to
an absolute filesystem path.

## Core Settings

| Property | Default | Description |
|---|---|---|
| `directory.name` | *(required)* | Name of the Crowd directory used to create/update users. Startup fails if unset. |
| `create.user` | `true` | Whether to create a new Crowd user when a Shibboleth-authenticated user doesn't exist yet. If `false`, unknown users are rejected. |
| `create.users.disabled` | `false` | When a new user is created, create the account **inactive** instead of active. This does *not* stop user creation — set `create.user=false` for that. |
| `enable.user.accounts` | `false` | If a matching Crowd user already exists but is disabled, treat them as authenticated anyway (does not re-enable the account in Crowd). |
| `reload.config` | `false` | Periodically re-read the properties file if it changes on disk. |
| `reload.config.interval` | `3600` (1 hour) | Seconds between reload checks, used while `reload.config=true`. Applies only when the property is present but not parseable as a number; if the property is omitted entirely, reload checks happen on every request. Always set this explicitly when `reload.config=true`. |

## Header Mapping

| Property | Default | Description |
|---|---|---|
| `headers.firstname` | `givenName` | HTTP header containing first name. |
| `headers.lastname` | `sn` | HTTP header containing last name. |
| `headers.mail` | `mail` | HTTP header containing email address. |
| `headers.to.attributes` | *(none)* | Comma-separated list of header names to copy onto the user as Crowd attributes on login. |
| `headers.urldecode` | `false` | URL-decode `REMOTE_USER` and all mapped header values (first/last name, email, dynamic group header) before use. Takes precedence over `headers.latin1toutf8` when both are set. |
| `headers.latin1toutf8` | `true` | Re-encode first/last name from Latin-1 to UTF-8. Useful when the SP/container mangles non-ASCII names. Ignored when `headers.urldecode=true`. |

## Group Mapping

Static group mappings assign a user to a group based on regex matches against
request headers:

```
group.[GROUP_NAME].sensitive=[true|false]
group.[GROUP_NAME].exclusive=[true|false]
group.[GROUP_NAME].match.[HEADER_NAME]=[REGEX]
```

- `sensitive` (default `true`) &mdash; case-sensitive matching.
- `exclusive` (default `true`) &mdash; when `true`, *all* `match.*` conditions must match (AND); when `false`, *any* one matching is enough (OR).
- `match.[HEADER_NAME]` &mdash; a header to test and a regex pattern to test it against. The pattern is matched with `find()`, i.e. it matches anywhere in the header value, not just the whole value — anchor with `^`/`$` if you need an exact match.
- If a user has previously been added to a mapped group but no longer matches, they are removed from it on next login.

```
# User gets group "Administrators" only if eduPersonAffiliation contains admin@university.edu
group.Administrators.sensitive=true
group.Administrators.exclusive=true
group.Administrators.match.eduPersonAffiliation=admin@university.edu

# User gets group "Students" if eduPersonAffiliation contains student@university.edu (case insensitive)
group.Students.sensitive=false
group.Students.exclusive=false
group.Students.match.eduPersonAffiliation=student@university.edu
```

## Dynamic Group Mapping

Dynamic groups add a user to whatever groups are listed in a single header,
splitting on a delimiter:

| Property | Default | Description |
|---|---|---|
| `dynamic.group.header` | *(none)* | Header containing a delimited list of group names. Dynamic group mapping is disabled unless this is set. |
| `dynamic.group.delimiter` | `;` | Delimiter used to split the header value. |
| `dynamic.group.purge.prefix` | *(none)* | If set, any of the user's existing groups starting with this prefix are removed unless still present in the header value. Leave unset to disable purging. |

```
# If eduPersonEntitlement contains "group1;group2;group3"
# the user is added to groups "group1", "group2", "group3"
dynamic.group.header=eduPersonEntitlement
dynamic.group.delimiter=;
dynamic.group.purge.prefix=shib_group_
```

## Sync Configuration

| Property | Default | Description |
|---|---|---|
| `sync.[appname]` | *(none)* | URL to call to sync the user's generated password into another application named `appname` in Crowd. |
| `sync.every.login` | `false` | If `true`, sync on every login when group membership changed; if `false`, sync only happens when the user is first created. |

## Home Organizations

```
home.organizations=foo,bar
```

Comma-separated list of domain suffixes. If `REMOTE_USER` is `alice@foo`, the
`foo` suffix is stripped and the user is treated as local user `alice`, who is
**not** created/updated/group-synced by this filter (their identity is assumed
to be managed by a separate directory instead).
