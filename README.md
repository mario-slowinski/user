user
====

Ansible role to manage user's account, groups and sudoers files, including:

* create/remove group
* create/remove user locally or in LDAP
* create/remove user's home directory
* create/copy user's ssh keys
* add an user's existing ssh public key from `authorized_keys[]`:
  * direct value, optionally with [!unsafe](https://docs.ansible.com/ansible/latest/user_guide/playbooks_advanced_syntax.html#unsafe-or-raw-strings) prefix
  * relative or absolute path to a key file
* set user's password
  * assign already encrypted password

    Anything matching below is considered to be already set or encrypted password:
    * `!` password for locked account
    * `regex('^\$[0-9]\$.{60}'` starting with `$anydigit$` followed by at least 60 characters
    * `regex('^\{[A-Z0-9]+\}.+')` starting with `{SHA}` followed by length in bytes

  * encrypt given password with `user_password_hash`

    Please use [ansible-vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to encrypt variable with password.

    ```bash
    echo -n "secret" | ansible-vault encrypt_string --vault-id @prompt --stdin-name password
    ```

  * generate, and store locally on ansible controller, password with given:

    * encryption algorithm
    * length
    * [allowed characters](https://docs.python.org/3.8/library/string.html)

    Generated passwords are stored as plaintext in a file on ansible controller. File location is set by `user_password_file` but it **always** ends with user's name.

* manage sudoers entries
  * in `/etc/sudoers` if `user_sudoers: [{file: "sudoers", ...}]`
  * in `/etc/sudoers.d/name` if `user_sudoers: [{file: "name", ...}]`

Requirements
------------

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
* [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)

Role Variables
--------------

* `defaults`
  * `password`

    ```yaml
    user_password_generate:   # generate password if set to this
    user_password_file: ""    # generated password file location
    user_password_seed: ""    # static seed to stay idempotent
    user_password_hash: ""    # password encryption algorithm
    user_password_length:     # generated password length
    user_password_chars: []   # list of allowed characters
    ```

  * `groups`

    ```yaml
    user_groups: []           # list of OS groups to add/remove
    ```

  * `accounts`

    ```yaml
    user_accounts: []         # list of users to manage
    - name: username
      dn: ""                  # DN of account entry in LDAP
      objectClass: []         # objectClasses of account entry in LDAP
      attributes: {}          # attributes of account entry in LDAP
        uid: ""               # i.e.
        cn: ""
        homeDirectory: ""
        loginShell: ""
        userPassword: ""      # follows the same rules as local account
      authorized_keys: []     # list of keys to add/remove
        - key: "{{ playbook_dir }}/files/id_rsa.pub"
          state: absent       # remove key matching the one in file
        - key: "~/.ssh/id_rsa.pub"
    ```

  * `sudoers`

    ```yaml
    user_sudoers:             # list of sudoers entries
      - file: "ansible"       # store entries in /etc/sudoers.d/ansible
        user: "ansible"       # user allowed to sudo
        host: "ALL"           # on hosts
        runas:               
          - user: "ALL"       # as this user
            group: "ALL"      # as this grup
            cmd: "ALL"        # run this command
    ```

* `vars`

  ```yaml
  user_sudoers_config: {}     # sudoers file attributes
  user_homedir: {}            # home directory attributes
  user_skel_path: /etc/skel   # location of skel for home dir
  user_query_homedir: ""      # JMESPath query to filter user that should have home dir created
  ```

Dependencies
------------

* [ldap](https://github.com/mario-slowinski/ldap)

Tags
----

* **user.group** - Manage groups
* **user.account** - Manage user's account
* **user.ldap** - Manage user's in LDAP
* **user.homedir** - Manage user's home directory
* **user.sshkeys** - Manage user's sshkeys
  * **user.sshkeys.authorized** - Manage ssh public key in authorized keys
  * **user.sshkeys.private** - Copy ssh keys
* **user.sudoers** - Manage sudoers

Examples
--------

* `requirements.yml`

  ```yaml
  - name: user
    src: https://github.com/mario-slowinski/user
  ```

* `playbook.yaml`

  ```yaml
  - hosts: servers
    gather_facts: no
    roles:
      - role: user
  ```

License
-------

[GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)

Author Information
------------------

[mario.slowinski@gmail.com](mailto:mario.slowinski@gmail.com)
