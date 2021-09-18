

* copy the example.inventory to a file called inventory. Configure the
list of IP's for the PI's. This should be the current DHCP addresses
that are reachable.

* copy the example.vars.yml to a file called vars.yml and edit to match your
netowrk needs. The IP's listed here will be the permanent/static IP's to configure
on the PI's.

* Run the ansible play to update the PI's. NOTE this assumed you have already
setup ssh pre-shared keys so that ansible can login

```
ansible-playbook -i inventory main.yml

```

* Reboot the PI's so the new configs will take affect

```
ansible-playbook -i inventory reboot.yml

```
