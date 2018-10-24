# Ansible Role: Certbot

An Ansible Role that install and configure Certbot.

We want this role as simple, configurable and interoperable as possible.


## Role behavior

This ansible role install and configure Certbot.


## Requirements

Compliant with :
- Debian 9 (Stretch)

Other:
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant


## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

Modifiables variables and possible values are listed below :

```
# Installation
certbot_package: certbot            # Package name to be installed, default is certbot for debian
certbot_package_state: latest       # State of the package, 'default' is latest in order to keep certbot updated, use 'present' in order to install it once and keep the current version

# Auto-renew cron
certbot_cron_autorenew: true        # Set false in order to disable certbot autorenew cron
certbot_cron_executable: certbot    # Executable name to be used in cron
certbot_cron_autorenew_options: "--quiet --no-self-upgrade --noninteractive" # Options for autorenew command in cron
certbot_cron_user: root             # User for the cron job
certbot_cron_minute: 30             # minute for the execution cron job (Use cron syntax)
certbot_cron_hour: 5                # hour for the execution cron job (Use cron syntax)
certbot_cron_dayofmonth: '*'        # day of month for the execution cron job (Use cron syntax)
certbot_cron_month: '*'             # month for the execution cron job (Use cron syntax)
certbot_cron_dayofweek: '*'         # day of week for the execution cron job (Use cron syntax)

# Domains to remove
cerbot_remove_domains: []           # A list of domains to remove, ex [ 'example.com', 'example.fr' ]
```

## Dependencies

none


## Installation

By default the installation of certbot use the official repository. In order to keep the package up to date the package state in ansible is set to latest in the default value.

If you don't want an automatic update of the certbot package please set in a variable file the following configuration in order to override the default variables.

```
certbot_package_state: false
```


## Auto Renewal of letsencrypt certificates

In order to renew automaticaly the certificates a cron is set to execute a quiet renewal based on renewal configuration. This run by default every day at 5h30.

You can disable the autorenew or override this value with the classic cron syntax and the following variables :

```
certbot_cron_autorenew: true        # Set false in order to disable certbot autorenew cron
certbot_cron_executable: certbot    # Executable name to be used in cron
certbot_cron_autorenew_options: "--quiet --no-self-upgrade --noninteractive" # Options for autorenew command in cron
certbot_cron_user: root             # User for the cron job
certbot_cron_minute: 30             # minute for the execution cron job (Use cron syntax)
certbot_cron_hour: 5                # hour for the execution cron job (Use cron syntax)
certbot_cron_dayofmonth: '*'        # day of month for the execution cron job (Use cron syntax)
certbot_cron_month: '*'             # month for the execution cron job (Use cron syntax)
certbot_cron_dayofweek: '*'         # day of week for the execution cron job (Use cron syntax)
```


## Remove previously configured domains

In order to remove old letsencrypt configuration and certificates you can use the `certbot_remove_domains` variable.

Use case : We need to delete old example.com and example.net configurations and certificates. Set in a host/group/task/... variable file the following configuration :
```
cerbot_remove_domains: [ 'example.com', 'example.net' ]
```


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
