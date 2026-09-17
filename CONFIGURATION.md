# Shibboleth SSO Filter Configuration Guide

This document explains the various configuration options available for the Shibboleth SSO filter.

## Basic Configuration Properties

### Core Settings
- `reload.config` - Enable/disable periodic config reloading (default: false)
- `reload.config.interval` - Seconds between config reload checks (default: 1)
- `directory.name` - Name of the Crowd directory to use for user management (required)

### Header Mapping
- `headers.firstname` - HTTP header containing first name (default: givenName)
- `headers.lastname` - HTTP header containing last name (default: sn)  
- `headers.mail` - HTTP header containing email address (default: mail)
- `headers.to.attributes` - Comma-separated list of headers to store as user attributes

### User Management
- `create.user` - Create new users if they don't exist (default: true)
- `enable.user.accounts` - Enable newly created user accounts (default: false)
- `create.users.disabled` - Disable creating new users entirely (default: false)

## Group Mapping Configuration

Group mappings allow you to automatically assign users to groups based on Shibboleth headers.

### Basic Group Mapping Syntax
```
group.[GROUP_NAME].[PROPERTY]=[VALUE]
```

### Properties for Group Mappings:
- `group.[GROUP_NAME].sensitive` - Case sensitive matching (default: true)
- `group.[GROUP_NAME].exclusive` - All conditions must match (default: true)  
- `group.[GROUP_NAME].match.[HEADER]` - Header name and regex pattern to match

### Example Group Mappings:
```
# User gets group "Administrators" only if they have eduPersonAffiliation=admin@university.edu
group.Administrators.sensitive=true
group.Administrators.exclusive=true
group.Administrators.match.eduPersonAffiliation=admin@university.edu

# User gets group "Students" if they have eduPersonAffiliation=student@university.edu (case insensitive)
group.Students.sensitive=false
group.Students.exclusive=false
group.Students.match.eduPersonAffiliation=student@university.edu
```

## Dynamic Group Mapping

Dynamic groups allow users to be automatically added to groups based on a semicolon-separated list in a header.

### Dynamic Group Properties:
- `dynamic.group.header` - Header containing group names (required for dynamic groups)
- `dynamic.group.delimiter` - Delimiter for group names (default: ;)
- `dynamic.group.purge.prefix` - Prefix for groups to purge from users

### Example:
```
# If eduPersonEntitlement header contains "group1;group2;group3"
# User will be added to groups "group1", "group2", and "group3"
dynamic.group.header=eduPersonEntitlement
dynamic.group.delimiter=;
dynamic.group.purge.prefix=shib_group_
```

## Sync Configuration

### Application Sync URLs:
- `sync.[appname]` - URL to call for syncing user credentials to application

### Sync Options:
- `sync.every.login` - Sync users on every login (default: false)

## Home Organizations

Use home organizations when you have users in separate directories that should not be managed by this filter.

```
home.organizations=foo,bar
```

This configuration treats users from domains "foo" and "bar" as existing in a separate directory and will not create/update them in Crowd.