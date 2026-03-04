**Privilege Escalation Surface**

After mounting the filesystem read-only, structured enumeration began.

Focus areas:
- /etc/passwd
- /etc/shadow
- /etc/sudoers*
- /home/*
- /root/*
- /var/lib/machines
- /lib/systemd/system

**vpn User**

A non-standard login user was identified:
- vpn:x:1000:1000:/home/vpn:/bin/bash

Inspection of /etc/sudoers.d/vpn revealed:

- vpn ALL=(root) NOPASSWD:
    - /usr/bin/systemctl restart systemd-nspawn@wat.service
    - /usr/bin/systemctl restart systemd-nspawn@noted.service
    - /usr/bin/systemctl restart systemd-nspawn@saas.service
    - /usr/local/bin/vpn.py

Key observations:
- Commands executable without password
- All commands execute as root
- Ability to restart root-owned containers
- Ability to execute custom Python code as root

This represents a direct privilege escalation vector.

**vpn.py Execution Context**

/usr/local/bin/vpn.py executes as root.

Observed behavior:
- Reads SSH_CONNECTION environment variable
- Parses IP addresses
- Generates session identifiers
- Establishes and tears down privileged sessions

Implications:
- Root processes attacker-influenced environment data
- Logic abuse or session manipulation may be feasible
- Expands attack surface beyond container restart abuse
