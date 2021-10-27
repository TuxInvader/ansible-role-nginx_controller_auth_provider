NGINX Controller Auth Provider
==============================

Manage NGINX Controller Authentication Providers.
The Auth provider resource enables an administrator to link authentication services with an external provider such as Active Directory. 
One or more group mappings can be created to link external and internal groups for role assignments.

The user role is how permissions are assigned to users.  The user role defines the objects / endpoints within NGINX Controller that the user can interact with and what they can do with those objects.
NGINX Controller includes three built-in Roles that cannot be modified: admin, user, and guest.

Requirements
------------

[NGINX Controller](https://www.nginx.com/products/nginx-controller/)

Role Variables
--------------

### Required Variables

`controller.fqdn` - FQDN of the NGINX Controller instance

`controller.auth_token` - Authentication token for NGINX Controller

`nginx_controller_auth_provider.metadata.name` -  name of the Authentication Provider

### Template Variables

This role has multiple template related variables. The descriptions and defaults for all these variables can be found in **[vars/main.yml](./vars/main.yml)**

Dependencies
------------

Example Playbook
----------------

To use this role you can create a playbook such as the following (let's name it `nginx_controller_user_role.yaml` for the purposes of this example).

```yaml
- hosts: localhost
  gather_facts: no

  vars:
    nginx_controller_user_email: "user@example.com" # Required by nginx_controller_generate_token role
    nginx_controller_user_password: "mySecurePassword" # Required by nginx_controller_generate_token role
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false

  tasks:
    - name: Retrieve the NGINX Controller auth token
      include_role:
        name: nginxinc.nginx_controller_generate_token

    - name: Configure the Auth Provider
      include_role:
        name: nginxinc.nginx_controller_auth_provider
      vars:
        nginx_controller_auth_provider:
          metadata:
            name:  # the name of the user role
            displayName:   # a friendly display name for the user role (spaces and special characters allowed)
            description:  # a description of the user role
          desiredState:
            provider:
              type:             # The type of provider, ACTIVE_DIRECTORY
              connection:
                - uri:          # The ldap(s) url of the provider
                  sslMode:      # SSL Mode (PLAIN_TEXT, REQUIRE, VERIFY_CA)
                  rawCa: |
                    -----BEGIN CERTIFICATE-----
                    MIIFyjCCA7KgAwIBAgIJAP9cqU0MKsAtMA0GCSqGSIb3DQEBCwUAMHoxCzAJBgNV
                    <-- snip -->
                    GOy2S3QRVAIMJw8axbh3G5gzdj54/dUyX39ITTwHRQCLlKb2dTZII553ONj+ndqI
                    fkifDFvo8zPc/vIxji4y3s2WFW4I5Fw6TprI1/R/8GWgN2bvQNVVWyLY/UauJg==
                    -----END CERTIFICATE-----
              bindUser:
                type:         # The bind method, PASSWORD
                username:     # The bind user
                password:     # The bind password
              userFormat:           # The login format (USER_DOMAIN, UPN)
              defaultLoginDomain:   # The default domain for login
              domain:               # The domain DN
              pollIntervalSec:      # Poll interval in seconds (300)
              groupCacheTimeSec:    # Cache time in seconds (600)
              honorStaleGroups:     # Use stale groups if AD unavailable (true,false)
              groupSearchFilter:    # Filter to apply to group search ( (objectClass=group) )
              groupMemberAttribute: # Group attribute of user (memberOf)
              groupMappings:
                - external:         # External group name
                  internal:
                    ref:            # Internal group ref (/platform/auth/groups/admin_group)
                  caseSensitive:    # Case sesntive (true,false)
```

You can then run `ansible-playbook nginx_controller_auth_provider.yaml` to execute the playbook.

Alternatively, you can also pass/override any variables at run time using the `--extra-vars` or `-e` flag like so `ansible-playbook nginx_controller_auth_provider.yaml -e "nginx_controller_user_email=user@company.com nginx_controller_user_password=notsecure nginx_controller_fqdn=controller.example.local nginx_controller_validate_certs=false"`

You can also pass/override any variables by passing a `yaml` file containing any number of variables like so `ansible-playbook nginx_controller_auth_provider.yaml -e "@nginx_controller_user_role_vars.yaml"`

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

[Mark Boddington](https://github.com/TuxInvader)

&copy; [NGINX, Inc.](https://www.nginx.com/) 2020
