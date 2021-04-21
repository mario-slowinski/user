user
====

Ansible role to manage OS user's account and sudoers files, including:

* create/remove user
* create/remove user's home directory
* create user's ssh keys
* set user's password
  * assign already encrypted password

    Anything matching regex `regex('^\$[0-9]\$.{60}'` is considered to be an encrypted password (starting with `$anydigit$` followed by at least 60 characters)

  * encrypt given password with `user_password_hash`

  * generate, and store locally on ansible controller, password with given:

    * encryption algorithm
    * length
    * [allowed characters](https://docs.python.org/3.8/library/string.html)

    Generated passwords are stored as plaintext in a file on ansible controller. File location is set by `user_password_file` but it **always** ends with user's name.

Requirements
------------

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)

Role Variables
--------------

* defaults

  ```yaml
  user_password_generate:      # if password set to this => generate password
  user_password_file: ""       # generated password file location
  user_password_seed: ""       # static seed to stay idempotent
  user_password_hash: ""       # password encryption algorithm
  user_password_length:        # generated password length
  user_password_chars: []      # list of allowed characters

  user_accounts: []            # list of users to manage
  ```

Dependencies
------------

This role has no dependencies.

Tags
----

* user.account
* user.sshkeys
* user.sudoers

Example Playbook
----------------

* `requirements.yml`

  ```yaml
  - name: user
    src: https://github.com/mario-slowinski/user
  ```

* playbook usage

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
