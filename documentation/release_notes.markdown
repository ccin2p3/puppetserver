---
layout: default
title: "Puppet Server: Release Notes"
canonical: "/puppetserver/latest/release_notes.html"
---

[Trapperkeeper]: https://github.com/puppetlabs/trapperkeeper
[service bootstrapping]: ./configuration.markdown#service-bootstrapping
[auth.conf]: ./config_file_auth.markdown

For release notes on the previous version of Puppet Server, see [docs.puppet.com](https://docs.puppet.com/puppetserver/2.4/release_notes.html).

## Puppet Server 2.5

Released August 11, 2016.

This is a feature and bug-fix release of Puppet Server.

> **Note:** If you disabled the certificate authority (CA) on Puppet Server by editing the [`bootstrap.cfg`][service bootstrapping] file on older versions of Puppet Server, be aware that Puppet Server 2.5.0 no longer uses the `bootstrap.cfg` file.
>
> To ensure that CA services remain disabled after upgrading to Server 2.5.0, create `/etc/puppetlabs/puppetserver/services.d/ca.cfg` with the CA disabled *before* upgrading. The `puppetserver` service restarts on upgrade if the service is running when you upgrade, which reloads the `ca.cfg` file.
>
> For more details, including an sample `ca.cfg` file that disables CA services, see the [bootstrap upgrade notes](./bootstrap_upgrade_notes.markdown).

### New feature: Flexible service bootstrapping/CA configuration file

To disable the Puppet CA service in previous versions of Puppet Server 2.x, users edited the [`bootstrap.cfg`][service bootstrapping] file, usually located at `/etc/puppetlabs/puppetserver/bootstrap.cfg`.

This workflow could cause problems for users performing package upgrades of Puppet Server where `bootstrap.cfg` was modified, because the package might overwrite the modified `bootstrap.cfg` and undo their changes.

To improve the upgrade experience for these users, Puppet Server 2.5.0 can load the service bootstrapping settings from multiple files. This in turn allows us to provide user-modifiable settings in a separate file and avoid overwriting any changes during an upgrade.

-   [SERVER-1470](https://tickets.puppetlabs.com/browse/SERVER-1470)
-   [SERVER-1247](https://tickets.puppetlabs.com/browse/SERVER-1247)
-   [SERVER-1213](https://tickets.puppetlabs.com/browse/SERVER-1213)

### New feature: Signing CSRs with OIDs from a new arc

Puppet Server 2.5.0 can sign certificate signing requests (CSRs) from Puppet 4.6 agents that contain a new custom object identifier (OID) arc to represent secured extensions for use with [`trapperkeeper-authorization`][Trapperkeeper].

> **Aside:** Trapperkeeper powers the [HOCON `auth.conf` and authorization methods][auth.conf] introduced in Puppet Server 2.2.0. This new CSR-signing functionality in Server 2.5.0 builds on features added to Puppet 4.6 and the addition of X.509 extension-based authorization rules added to Trapperkeeper alongside Puppet Server 2.4.

To sign CSRs wth the new OID arc via the Puppet 4.6 command-line tools, use the `puppet cert sign --allow-authorization-extensions` command. See the [`puppet cert` man page](https://docs.puppet.com/puppet/4.6/reference/man/cert.html) for details. This workflow is similar to signing DNS alt names.

The new OID arc is "puppetlabs.1.3", with a long name of "Puppet Authorization Certificate Extension" and short name of `ppAuthCertExt` (where "puppetlabs" is our registered OID arc 1.3.6.1.4.1.34380). Set the extension "puppetlabs.1.3.1" (`pp_authorization`) on CSRs that need to be authenticated via the new workflow. We've also included an default alias of `pp_auth_role` at extension "puppetlabs.1.3.13" for common workflows. See [the Puppet CSR attributes and certificate extensions documentation](https://docs.puppet.com/puppet/4.6/reference/ssl_attributes_extensions.html) for more information.

We've also improved the CLI output of `puppet cert list` and `puppet cert sign` to work better with the `--human-readable` and `--machine-readable` flags, and we allow administrators to force a prompt when signing certificates with the `--interactive` flag.

This allows for easier automated failover to authorized nodes within a Puppet infrastructure and provides tools for creating new, securely automated workflows, such as automated component promotions within Puppet-managed infrastructure.

-   [SERVER-1305](https://tickets.puppetlabs.com/browse/SERVER-1305)

### Bug fix: Unrecognized parse-opts

Puppet Server 2.4.x used a deprecated API for a Clojure CLI option-parsing library. As a result, calls to `puppetserver gem` (either directly, or indirectly by using a `puppetserver_gem` package resource) generated unexpected warning messages:

    Warning: Could not match Warning: The following options to parse-opts are unrecognized: :flag

Puppet Server 2.5.0 updates this library, which prevents this error message from appearing.

-   [SERVER-1378](https://tickets.puppetlabs.com/browse/SERVER-1378)

### Bug fix: Puppet Server no longer ships with an empty PID file

When installed on CentOS 6, Puppet Server 2.4.x included an empty PID file. When running `service puppetserver status`, Puppet Server returned an unexpected error message: `puppetserver dead but pid file exists`.

When performing a clean installation of Puppet Server 2.5.0, no PID file is created, and `service puppetserver status` should return the expected `not running` message.

-   [SERVER-1455](https://tickets.puppetlabs.com/browse/SERVER-1455)
-   [EZ-84](https://tickets.puppetlabs.com/browse/EZ-84)

### Other changes

-   [SERVER-1310](https://tickets.puppetlabs.com/browse/SERVER-1310): Error messages in Puppet Server 2.5.0 use the same standard types as other Puppet projects.
-   [SERVER-1121](https://tickets.puppetlabs.com/browse/SERVER-1121): JRuby pool management code for the [Trapperkeeper Webserver Service][Trapperkeeper] is now its own open-source project, [puppetlabs/jrubyutils](https://github.com/puppetlabs/jruby-utils).