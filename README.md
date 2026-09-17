crowd-shibboleth-module
=======================

Shibboleth authentication module and accompanying plugin for Atlassian Crowd.

## Modules

* [shibboleth-filter](shibboleth-filter/README.md) &mdash; the Spring Security filter that authenticates users against Crowd using HTTP headers set by Shibboleth. This is the module in active use.
* `shibboleth-filter-config` &mdash; shared library that loads `ShibbolethSSOFilter.properties`, used by `shibboleth-filter`.
* `nordunet-sso` &mdash; legacy multi-domain SSO-cookie plugin, superseded in most deployments by a custom SSO-cookie plugin.

## Documentation

* [INSTALL.md](INSTALL.md) &mdash; building and installing the modules, and wiring up Apache/Shibboleth.
* [CONFIGURATION.md](CONFIGURATION.md) &mdash; full `ShibbolethSSOFilter.properties` reference.

## License

BSD 3-Clause, see [LICENSE](LICENSE).

By downloading any of the crowd-shibboleth-module files you acknowledge that you have Read and Accepted the NORDUnet IPR Policy:
https://wiki.nordu.net/display/NORDUwiki/NORDUnet+IPR+Policy
