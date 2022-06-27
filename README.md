ldap
====

Ansible role to manage entries in already existing LDAP server.

Requirements
------------

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
* [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)
* [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/)

Role Variables
--------------

* defaults

  * `main.yaml`

    ```yaml
    ldap_uri: "ldapi:///"
    ldap_port: 389

    ldap_bind:
      bind_dn: "cn=proxy,dc=example,dc=com"
      bind_pw: "proxypassword"

    ldap_root:
      bind_dn: "cn=admin,dc=example,dc=com"
      bind_pw: "adminpassword"

    ldap_base: "dc=example,dc=com"
    ldap_entries: []

    ldap_configs:
      - regexp: "^#*base "
        line: "base {{ ldap_base }}"
      - regexp: "^#*uri ldapi:///$"
        line: "uri {{ ldap_uri }}"
      - regexp: "^#*binddn "
        line: "binddn {{ ldap_bind['bind_dn'] }}"
      - regexp: "^#*bindpw "
        line: "bindpw {{ ldap_bind['bind_pw'] }}"
      - regexp: "^#*rootbinddn "
        line: "rootbinddn {{ ldap_root['bind_dn'] }}"
      - regexp: "^#*port "
        line: "port {{ ldap_port }}"
      - regexp: "^#*pam_password crypt"
        line: "#pam_password crypt"
      - regexp: "^#*pam_password md5"
        line: "#pam_password md5"
      - regexp: "^#*pam_password exop"
        line: "pam_password exop"

    ldap_secrets: "{{ ldap_root['bind_pw'] }}"

    ldap_nsswitches:
      - regexp: '^passwd:((?:(?:\s+\S+(?!\S))(?<!\sldap))+\s*?)$'
        line: "passwd:\\1 ldap"
      - regexp: '^group:((?:(?:\s+\S+(?!\S))(?<!\sldap))+\s*?)$'
        line: "group:\\1 ldap"
      - regexp: "^shadow:([^a-z]*)"
        line: "shadow:\\1compat"

    ldap_pam_config:
      - name: session
        options:
          - regexp: '^session\s+optional\s+pam_mkhomedir.so.*$'
            line: "session optional\tpam_mkhomedir.so umask=0077 skel=/etc/skel"
            insertbefore: "# end of pam-auth-update config"
      - name: password
        options:
          - regexp: '^password(.+pam_ldap.so.*?)\s+use_authtok(.*)$'
            line: "password\\1\\2"
            backrefs: true
    ```

* vars

  ```yaml
  ldap_config:
    path: /etc/ldap.conf
    owner: root
    group: root
    mode: 0o644
    setype: etc_t

  ldap_secret:
    path: /etc/ldap.secret
    owner: root
    group: root
    mode: 0o600
    setype: etc_t

  ldap_nsswitch:
    path: /etc/nsswitch.conf
    owner: root
    group: root
    mode: 0o644
    setype: etc_t

  ldap_pam_file:
    session:
      path: /etc/pam.d/common-session
      owner: root
      group: root
      mode: 0o644
      setype: etc_t
    password:
      path: /etc/pam.d/common-password
      owner: root
      group: root
      mode: 0o644
      setype: etc_t
  ```

Dependencies
------------

*No* *dependencies*

Tags
----

* ldap.entry
* ldap.attr
* ldap.search
* ldap.client
  * ldap.client.config
  * ldap.client.secret
  * ldap.client.pam
  * ldap.client.nsswitch

Examples
--------

* `requirements.yaml`

  ```yaml
  - name: ldap
    src: https://github.com/mario-slowinski/ldap
  ```

* `playbook.yaml`

  ```yaml
  - hosts: servers
    gather_facts: true
    roles:
      - role: ldap
  ```

License
-------

[GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)

Author Information
------------------

[mario.slowinski@gmail.com](mailto:mario.slowinski@gmail.com)
