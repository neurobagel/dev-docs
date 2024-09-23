---
title: Setting up SSH keys

---

To add public keys for the existing users

- switch to substitute user
    - `sudo su -`

- Navigate to home directory
    - `cd /home`
Should be able to see a home directory for each user there

- Navigate to the home directory of the user
    - `cd arman`


- Navigate to the .ssh folder of the user
    - `cd .ssh`


- Add the ssh key to the authorized key file 
    - `nano authorized_keys`