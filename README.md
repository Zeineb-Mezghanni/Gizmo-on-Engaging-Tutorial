# EngagingSetup
Follow the instructions given on 
https://orcd-docs.mit.edu/getting-started/

# SettingUpGizmoOnEngaging


# Random (By CianMRoche)
### Copying ssh key to host when ssh-copy-id doesnt work: 
The following only works when your cluster uses an authorized_keys file for ssh accesses.  

Run in a local terminal `$env:USERPROFILE\.ssh\id_rsa.pub | ssh {...} "cat >> .ssh/authorized_keys"` where you replace `{...}` with `username@host` for appropriate platform/cluster:

### Using Jupyter notebooks (general cluster instructions)
See [this github gist](https://gist.github.com/CianMRoche/ce176a089c06fd81d26f339f99f5af87)
