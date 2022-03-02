# Ansible role to install and maintain Nextcloud setups

[![Build Status](https://github.com/systemli/ansible-role-nextcloud/workflows/Integration/badge.svg?branch=main)](https://github.com/systemli/ansible-role-nextcloud/actions?query=workflow%3AIntegration)
[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-nextcloud-blue.svg)](https://galaxy.ansible.com/systemli/nextcloud/)

This role is meant to deploy and upgrade Nextcloud instances to Debian
systems.

# Using the Ansible module `nextcloud_app`

This role uses the Ansible module `nextcloud_app` for managing
Nextcloud apps. There is a [pull request](https://github.com/ansible/ansible/pull/36744)
to include the module into Ansible.

# How it works

* The requested Nextcloud version is installed into
  `{{ nextcloud_work_dir }}/nextcloud-{{ nextcloud_version }}`.
* The symlink `{{ nextcloud_work_dir }}/nextcloud-current` points to the
  active Nextcloud installation.

# Preliminaries

The role takes care of installing and upgrading Nextcloud and its apps. It
doesn't install or configure MariaDB/MySQL, mail or web service. The latter
has to be done separately.

PHP dependencies required for Nextcloud are installed by that role.

Some modules used in this role are new to Ansible 2.2 and it's tested with
Ansible 2.2, 2.3 and 2.4.

The following preliminaries need to be met:

* Ansible 2.2 or newer
* Debian target system with the following requirements:
  * MariaDB/MySQL server with admin permissions (may be remote)
  * Webserver (e.g. Apache2/Nginx) with basic PHP support
  * Mail server if Nextcloud instance shall send out mails (may be
    remote)
* The Nextcloud `data` directory needs to be located outside
  `nextcloud_work_dir`

# Usage example

* Integrate the role in your playbook: 
    
```
- hosts: cloud.example.org
  roles:
    - nextcloud
  tags:
    - nextcloud
```

* Configure role variables in `host_vars` for the target system (see
  `defaults/main.yml` for a list of supported config variables):  
    
```
# Nextcloud settings

nextcloud_workdir: "/var/www/cloud.example.org/nextcloud"
nextcloud_data_dir: "/srv/nextcloud/data"
nextcloud_config:
  system:
    memcache.local: '\OC\Memcache\APCu'
    memcache.distributed: '\OC\Memcache\Redis'
    memcache.locking: '\OC\Memcache\Redis'
    redis:
      host: localhost
      port: 6379
    trusted_domains:
      - cloud.example.org
  apps:
    notify_push:
      base_endpoint: "https://cloud.example.org/push"
nextcloud_mysql_password: "******"
nextcloud_admin_password: "******"
nextcloud_notify_push: True

nextcloud_apps:
  - admin_audit
  - calendar
  - contacts
```

* Deploy Nextcloud to the target system:  
    
  `ansible-playbook site.yml -t nextcloud -l cloud.example.org --diff`

* Configure the VirtualHost in your webserver. The following is an example
  snippet for a jinja2 Apache2 vhost template:  
    
```
{% set nextcloud_webroot = nextcloud_workdir|d() + '/nextcloud-current' %}
	DocumentRoot {{ nextcloud_webroot }}
	<Directory {{ nextcloud_webroot }}>
		Options FollowSymlinks
		AllowOverride All

		<IfModule mod_dav.c>
			Dav off
		</IfModule>

		SetEnv HOME {{ nextcloud_webroot }}
		SetEnv HTTP_HOME {{ nextcloud_webroot }}
	</Directory>

    # Proxy rules for the Nextcloud notify_push daemon
    ProxyPass /push/ws ws://127.0.0.1:7867/ws
    ProxyPass /push/ http://127.0.0.1:7867/
    ProxyPassReverse /push/ http://127.0.0.1:7867/
```

# Upgrading Nextcloud

To upgrade a Nextcloud instance, it's sufficient to bump the version
in Role variable `nextcloud_version` and run the role again.

## Manually upgrading Nextcloud

If you prefer to upgrade Nextcloud manually, you can configure the role to not
perform any uprades by setting `nextcloud_upgrade` to `False`. Beware that you
also have to set `nextcloud_instance` to something that doesn't contain
`nextcloud_version` in this case:

```
nextcloud_upgrade: False
nextcloud_instance: "{{ nextcloud_workdir }}/nextcloud"
```

# Notification push daemon support

When `nextcloud_notify_push` is enabled, the [Nextcloud notify_push
daemon](https://github.com/nextcloud/notify_push) will be installed, configured
and enabled.

Please beware that further configuration might be needed:
* Redis needs to be configured for caching.
* Configure your webserver as reverse proxy. See the [upstream
  documentation](https://github.com/nextcloud/notify_push#reverse-proxy)
  for details.
* Set `base_endpoint` for the `notify_push` app accordingly:
  ```
  nextcloud_config:
  [...]
    apps:
      notify_push:
        base_endpoint: "https://cloud.example.org/push"
  ```

# Testing & Development

## Tests

For developing and testing the role we use Github Actions, Molecule, and Vagrant. On the local environment you can easily test the role with

Run local tests with:

```
molecule test 
```

Requires Molecule, Vagrant and `python-vagrant` to be installed.

# License

This Ansible role is licensed under the GNU GPLv3 or later.

# Author

[https://www.systemli.org](https://www.systemli.org)
