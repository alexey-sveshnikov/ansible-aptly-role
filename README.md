Aptly
========

Aptly is a brilliant software for creating both your own debian repositories and official repository mirrors with git-like snapshot management. This role helps you to install aptly and setup your own repository with nginx as web server.


Notes
=====

1. This role is tested on Ubuntu 14.04 and aptly 0.5.9 only (the latest as for 25th May 2014). Patches for other distributions and versions are welcome.
1. This role configures aptly as your own repository server only. Mirrors management wasn't tested.


Requirements
------------

You need a GPG public/private keys pair to sign your repository.

Use the following shell commands to obtain your own keys pair:

    $ sudo apt-get install gnupg
    $ gpg --gen-key

       Please select what kind of key you want:
        (1) RSA and RSA (default)
        (2) DSA and Elgamal
        (3) DSA (sign only)
        (4) RSA (sign only)
        Your selection? 3

        ... answer questions about your name, key expiration period, etc. ...

        gpg: checking the trustdb
        gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
        gpg: depth: 0  valid:   4  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 4u
        gpg: next trustdb check due at 2019-05-23

    ==> pub   2048D/61B1BA69 2014-05-24  <== 61B1BA69 is the ID of key you generated
              Key fingerprint = 6C50 B46A 3D75 0C14 041B  6C99 FBBA 0275 61B1 BA69
              uid                  Ivan Ivanov (repository key) <ivan@example.com>

Once you generated the key, you should export it to the corresponding files. Replace key ID with yours (you can find it in the end of previous command's output, see above)

    $ mkdir -p secrets/aptly  # (assuming you are in the directory where inventory.ini and playbooks are located. Also see 'Optional Variables' section)
    $ gpg --export-secret-keys --armor 61B1BA69 > secrets/aptly/private.key
    $ gpg --export --armor 61B1BA69 > secrets/aptly/public.key


Role Variables
--------------

#### Should be defined:

1. `aptly_secret_key_id`: ID of the GPG key
1. `aptly_repositories`: Your repositories specification. See an example below.

#### Optional variables:

1. `aptly_user`: user who owns repository files. Default is 'aptly'
1. `aptly_virtualhost`: name of virtualhost in nginx config file.
1. `aptly_secret_key_path`: path to GPG keys relative to playbook. Defaults is 'secrets/aptly'
1. `aptly_secret_key_file`: name of private GPG key. Default is 'private.key'


Dependencies
------------

Gnupg for key generation. None if you already have keys.


Example Playbook
-------------------------

    -
        name: Install aptly
        gather_facts: no  # optional
        hosts:
            - repositories
        vars:
            aptly_secret_key_id: <GNUPG KEY ID>
            aptly_repositories:
                - 
                    # With optional parameters
                    name: yourcompany-dev
                    comment: Developent packages
                    distribution: trusty
                    component: main
                    architectures: amd64,i386  # This is the default value. I recommend not to change it.
                -
                    # Using default settings
                    name: yourcompany-testing
                -
                    name: yourcompany-prod
        roles:
            - aptly


How to setup clients
--------------------

    apt-key add secrets/aptly/public.key
    echo 'deb http://server_name/yourcompany-dev trusty main' > /etc/apt/sources.list.d/your_company_dev.list"


How to upload new package
-------------------------
Here is an idea:

    scp mybw_0.1_amd64.deb server_name:/tmp/
    ssh server_name 'sudo -u aptly -H aptly repo add yourcompany-dev /tmp/mybw_0.1_amd64.deb'
    ssh server_name 'sudo -u aptly -H aptly publish update saucy yourcompany-dev'

I found no way to upload signed .deb packages (.changes file)


License
-------

BSD

Author Information
------------------

http://github.com/alexey-sveshnikov