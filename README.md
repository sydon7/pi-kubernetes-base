# pi-cluster-base

This project an adaptation of the great work done by geerlingguy here.
 https://github.com/geerlingguy/raspberry-pi-dramble

There were a few things I wanted to change and a few updates I wanted to make. This
repo has just a sub-set of the features.

# Overview of changes
I have made a few changes

* The first is updating tasks/disable-swap.yml ... on the version of debian I have
swap wasn't not getting fully removed

* The second is removing part of the playbook items, I didn't want to build a
drupal site, I wanted to just get the basic cluster setup and ready to work.

* Some of the kubernets packages in the orignal geerlingguy repo would not install (they
pointed to amd64 instead of arm64).

# Buidling the base cluster:

## pi version
I am using this image raspion_lite_arm64-2020-08-24 (or newer)

## Basic setup
You can follow the steps outline by geerlingguy https://www.pidramble.com/wiki/setup/prepare
here are my cliff notes for each PI

* Image the SD card with 64bit raspion

* Make sure to create a file on the card called "ssh". When the PI boots, it will enable
sshd by default. (touch /Volumes/boot/ssh assuming your card is mounted at /Volumes)

* Find your PI. Assuming your home lan was 10.1.1.0/24. They should have gotten a DHCP address
we are going to find them and give each PI a static address
```
  sudo nmap -sP 10.1.1.0/24
```

* ssh into the pi (google the default user/password if you need to)
```
 ssh pi@10.1.1.99
```


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


* Login to each pi and make sure they are back up with their static IP's we configured.

* One more setup step, lets patch and reboot the PI's... before we build them they should be up to date
```
apt-get update
apt-get upgrade
reboot

```

## Kubernets cluster build

* Copy the inventory.sav to a file called inventory and configure the hosts file to match
your nodes

* Copy the example.config.yml file to a file calle config.yml and adjust the config.

* Install the needed roles from geerlingguy.

```
ansible-galaxy install -r requirements.yml --force
```

* Install openshift

```
pip3 install openshift

```

* Ok, build the nodes:

```
ansible-playbook -i inventory main.yml

```
* After a bit... you should have a working cluster. Lets check. login to the master node and
run the following commands as root (it shoud have configured it with everything we need).

Make sure you can see all the nodes and their status is Ready. There are two nodes in
this cluster.

```
root@kube10:~# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
kube10   Ready    master   5h36m   v1.19.7
kube11   Ready    <none>   5h36m   v1.19.7

```


Check that you can see some base containers running. They might take a minute but
should all end up in a Running state.


```
root@kube10:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   coredns-f9fd979d6-bgpfm               1/1     Running   1          5h36m
kube-system   coredns-f9fd979d6-jmlhj               1/1     Running   1          5h36m
kube-system   etcd-kube10                           1/1     Running   3          5h36m
kube-system   kube-apiserver-kube10                 1/1     Running   3          5h36m
kube-system   kube-controller-manager-kube10        1/1     Running   3          5h36m
kube-system   kube-flannel-ds-h6xll                 1/1     Running   1          5h36m
kube-system   kube-flannel-ds-v59hj                 1/1     Running   0          5h35m
kube-system   kube-proxy-9ktjb                      1/1     Running   0          5h35m
kube-system   kube-proxy-mbjsx                      1/1     Running   1          5h36m
kube-system   kube-scheduler-kube10                 1/1     Running   3          5h36m

```



