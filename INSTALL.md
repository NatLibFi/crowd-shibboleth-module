# Installation

This version works only with Crowd 3.0+ due to API changes.

This repo has three independent Maven modules (no reactor `pom.xml`), built and
installed separately:

* `shibboleth-filter-config` &mdash; shared configuration-loading library, required by `shibboleth-filter`.
* `shibboleth-filter` &mdash; the Shibboleth authentication filter for Crowd; see [shibboleth-filter/README.md](shibboleth-filter/README.md) and [CONFIGURATION.md](CONFIGURATION.md) for how it works and how to configure it.
* `nordunet-sso` &mdash; legacy multi-domain SSO-cookie plugin, providing the `/ssocookie`, `/setcookie` and `/setEmail` servlets used by the "Apache Shibboleth Module" / "Enabling" steps below. Many deployments replace this with their own SSO-cookie plugin; only build it if you need those endpoints.

# Steps

## Creating JAR files

Requires the [Atlassian Plugin SDK](https://developer.atlassian.com/server/framework/atlassian-sdk/) and a matching JDK for your target Crowd version. Adjust paths below to your SDK installation.

```shell
git clone https://github.com/NatLibFi/crowd-shibboleth-module
cd crowd-shibboleth-module

# 1. Shared config library
cd shibboleth-filter-config
atlas-package
CONFIG_VERSION=$(grep -m1 '<version>' pom.xml | sed -e 's/.*<version>//' -e 's/<\/version>.*//')
cp target/*.jar /opt/atlassian/crowd/crowd-webapp/WEB-INF/lib

# 2. Shibboleth filter (depends on the jar just built above)
cd ../shibboleth-filter
mvn install:install-file -DgroupId=com.eduix.crowd -DartifactId=shibboleth-filter-config \
    -Dversion="$CONFIG_VERSION" -Dpackaging=jar \
    -Dfile=../shibboleth-filter-config/target/shibboleth-filter-config-"$CONFIG_VERSION".jar
atlas-package
cp target/*.jar /opt/atlassian/crowd/crowd-webapp/WEB-INF/lib
chown crowd: /opt/atlassian/crowd/crowd-webapp/WEB-INF/lib/*.jar

# 3. (Optional, legacy) NORDUnet multi-domain SSO plugin
cd ../nordunet-sso
atlas-package
cp target/*.jar /opt/atlassian/home/plugins/
chown crowd: /opt/atlassian/home/plugins/*
```

## Files

* Download `ShibbolethSSOFilter.properties`

```shell
cd /opt/atlassian/crowd/crowd-webapp/WEB-INF/classes
curl -o ShibbolethSSOFilter.properties https://raw.githubusercontent.com/NatLibFi/crowd-shibboleth-module/master/shibboleth-filter/src/main/resources/ShibbolethSSOFilter.example.properties
# or: wget -O ShibbolethSSOFilter.properties https://raw.githubusercontent.com/NatLibFi/crowd-shibboleth-module/master/shibboleth-filter/src/main/resources/ShibbolethSSOFilter.example.properties
```

* Edit `applicationContext-CrowdSecurity.xml` by following the instructions in [shibboleth-filter/README.md](shibboleth-filter/README.md).

## Apache Shibboleth Module

The steps below use the `nordunet-sso` plugin's `/ssocookie` servlet as the
Shibboleth-protected discovery URL. If you're not using that plugin (e.g. you
have your own SSO-cookie handling), protect the actual login URL
(`filterProcessesUrl`, e.g. `/console/j_security_check`) with Shibboleth
instead, so the filter sees `REMOTE_USER` and the configured headers directly.

* Require Shibboleth authentication on the `ssocookie` servlet:

```
<Location /crowd/plugins/servlet/ssocookie>
  AuthType shibboleth
  Require shibboleth
  ShibUseHeaders on
  ShibRequestSetting requireSession 1
</Location>
```

* Ensure the Shibboleth module is getting a `REMOTE_USER` setting.
* Configure your `attribute-map.xml` appropriately.

## Enabling

* Locate the source to an Atlassian-compatible login page. For example, the Crowd demo app login page is located at `/opt/atlassian/crowd/demo-webapp/login.jsp`.
* Add a link to login with Shibboleth. For example:

```
<p>Login with <a href="https://example.com/crowd/plugins/servlet/ssocookie?redirectTo=%2fdemo%2Fsecure%2Fconsole%2Fconsole.action">Shibboleth</a></p>
```
