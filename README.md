# Wordpress Ansible Role

![wordpress Logo](assets/logo.png)

[Wordpress](https://wordpress.org) is a popular open source publishing platform. This ansible role helps to install wordpress

- with everything run in [Docker](https://www.docker.com/) containers
- powered by [the official wordpress container image](https://hub.docker.com/_/wordpress/)


## Installing

To configure and install wordpress on your own server(s), you should use a playbook like [Mother of all self-hosting](https://github.com/mother-of-all-self-hosting/mash-playbook) or write your own.

# Configuring this role for your playbook

```
wordpress_enabled: true
wordpress_hostname: 'example.org'

# Must be a Mysql DB!
wordpress_db_host:

wordpress_db_name:
wordpress_db_user:
wordpress_db_password:
```

## Support

- Github issues: [moan0s/role-wordpress/issues](https://github.com//role-wordpress/issues)
