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
certbot_package: certbot                    # Package name to be installed, default is certbot for debian
certbot_package_state: latest               # State of the package, 'default' is latest in order to keep certbot updated, use 'present' in order to install it once and keep the current version

# Common
certbot_executable: certbot                 # Executable name to be used in script generation and cron

# Certificates generation
certbot_default_generate_mode: standalone   # Required mode generation for this certificates
certbot_default_email: mail@example.com     # Email for letsencrypt notifications
certbot_domains: []                         # List of dictionnary containing domains & options for certificates generation (See README.md)

certbot_default_http01port: 80
certbot_default_preferredchallenges: http-01

certbot_standalone_stop_services: []        # List of services required to stop during standalone certificates generation (ex: [ 'nginx', 'apache', 'haproxy', 'varnish' ])

# Command used for certificate generation in standalone mode
certbot_standalone_gen_command: >-
    {{ certbot_executable }} certonly --standalone --noninteractive
    --agree-tos --email {{ cert_item.email | default(certbot_email) }}
    --http-01-port={{ cert_item.http01port | default(certbot_default_http01port) }}
    --preferred-challenges={{ cert_item.preferredchallenges | default(certbot_default_preferredchallenges) }}
    --cert-name {{ cert_item.domains | first | replace('*.', '') }}
    -d {{ cert_item.domains | join(',') }}
    {{ certbot_standalone_gen_command_custom_preposthook | default('') }}
    {{ certbot_standalone_gen_command_custom_renewhook | default('') }}

# Command used for certificate generation in webroot mode
certbot_webroot_gen_command: >-
    {{ certbot_executable }} certonly --webroot --noninteractive
    --agree-tos --email {{ cert_item.email | default(certbot_email) }}
    -w {{ cert_item.webroot }}
    --cert-name {{ cert_item.domains | first | replace('*.', '') }}
    -d {{ cert_item.domains | join(',') }}

# Auto-renew cron
certbot_cron_autorenew: true                # Set false in order to disable certbot autorenew cron
certbot_cron_autorenew_options: "--quiet --no-self-upgrade --noninteractive" # Options for autorenew command in cron
certbot_cron_user: root                     # User for the cron job
certbot_cron_minute: 30                     # minute for the execution cron job (Use cron syntax)
certbot_cron_hour: 5                        # hour for the execution cron job (Use cron syntax)
certbot_cron_dayofmonth: '*'                # day of month for the execution cron job (Use cron syntax)
certbot_cron_month: '*'                     # month for the execution cron job (Use cron syntax)
certbot_cron_dayofweek: '*'                 # day of week for the execution cron job (Use cron syntax)

# Domains to remove
cerbot_remove_domains: []                   # A list of domains to remove, ex [ 'example.com', 'example.net' ]

# Cron
certbot_cron_random_sleep_cmd: '/bin/sleep $(/usr/bin/shuf -i 1-3600 -n 1) && '      # Default sleep command, in order to distribute task in one hour, change to '' in order to disable it
```

## Dependencies

none


## Installation

By default the installation of certbot use the official repository. In order to keep the package up to date the package state in ansible is set to latest in the default value.

If you don't want an automatic update of the certbot package please set in a variable file the following configuration in order to override the default variables.

```
certbot_package_state: false
```


## Certificates generation syntax

*Note :* For now, the current configuration of a certificate renewal is not updated, per example if you want to add a new domain in the same certificate you should remove the previous certificate with `cerbot_remove_domains` and update the `certbot_domains` configuration.

Here is differents ways to generate certificates :

```
certbot_domains:
  # Single domain in standalone mode with custom email
  - domains: [ 'example.com' ]
    mode: standalone
    email: letsencrypt@example.com

  # Single domain in standalone mode with a custom http-01 port (Use full for certbot standalone mode behind a reverse proxy). Preferred challenges can also be customized, if missing the default value is http-01
  - domains: [ 'example.com' ]
    mode: standalone
    http01port: 8888
    preferredchallenges: http-01

  # Multiples domains in standalone mode (Note : The first domain, example.com, is the "primary" domain, it is used to define a letsencrypt configuration file name, you should use the same for remove domains)
  - domains: [ 'example.com', 'sub.example.com', 'example.fr' ]
    mode: standalone

  # Single domain in webroot mode with a concatenation for the certificate in a specific file
  - domains: [ 'example.com' ]
    mode: webroot
    webroot: '/var/www/html/'
    concatenation: '/etc/ssl/example_com_combined.pem'

    ...
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
