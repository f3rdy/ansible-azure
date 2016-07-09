# ansible-azure
## install azure cli client
https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/

## Configure azure creditals 
under the file  ~/.azure/credentials, insert the following (or create a principle)

* [default]
* ad_user=ansible@xxxxxxx.onmicrosoft.com
* password=XXXXXXX
* subscription_id=XXXXXXXX

For the above, i recommend created a "ansible" user instead of using your existing adming user. The following steps can be followed to perform this 
https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/
 - After creating a user as outlined in the above instructions. in the azure portal. go "settings"->"administrators", and then add user

## Update ansible, 
This ansible script uses features only found in the latest version on ansible (notable the azure plugin and the gatewayed feature )
 upgrade to ansible 2.1.0.0
http://docs.ansible.com/ansible/intro_installation.html

## run script 
Update the group_vars/all variable. The following params exist. The script will support more masters  in the future. For now the script installs 1 master, 1 infra, x number of nodes. the nodes and infra are get labels which corrospond the tags in azure (consistent)
 - ansible-playbook -i inventory playbooks/setup.yml (warning, this may have been broken since the multi.master setup)
 - ansible-playbook  --forks=50 -i playbooks/setup_multimaster.yml

## configuration of nodes 
Under the groups/all there is a  list of vms that will get created. This list also has a attribute called tag which. The values get set to azure tags and also openshift node labels  
  - jump node : this is required. all actions are performed via this node. This node needs to come up first before any of the other nodes can be created. this is because we generate a key on the jump node which is distributed to all subsequent nodes. For convience sake the LB infront of the master is also placed on this node.(opens port 8443 and 22). In the future this will be a azure loadbalancer
  - master: we create 3 of these. no ports facing the outside. Loadbalanced by the jump node
  - infra node: currently hosts the registry and the router. not tested with multiple infra nodes yet. should work 
  - node: bunch of nodes to host applications 

 ## Post install 
  The installation of openshift is performed on the jump node. The local ansible script (setup_multimaster) execute another ansible script on the jump host(advanced install). The host file has already been placed under /etc/ansible/host by this stage and is dynamically build based on the template sitting under
   -  playbooks/roles/prepare_multi/templates/hosts.j2
 The execution of the advanced install is executed in async manner (to avoid timeouts as the advanced install can take time ). The local ansible script(setup_multimaster) polls the jump host for completion. 

Once installation is complete the following is configured 
 - router ( is not deployed by default because the region=infra is not used)
 - Register: (is  not deployed by default because the region=infra is not used) currently a bug with regard to local permisions on dir /mnt/registry to fix jump on to infranode1 and perform  ```sudo chown 1001:root /mnt/registry/ ```(is are not deployed by default because the region=infra is not used)
 - metrics, 
 - logging : waits untils everything is ready before scaling out the fluentd nodes 
 ### example node setup under groupvars/all
```
#### jump host
jumphost:
  jumphost1:
    name: jumphost1
    tags:
      region: northeurope
      zone: jumphost
      stage: jumphost
### List of masters including tags/labels to be applied. Azure gets tags while ose gets labels
masters:
  master1:
    name: master1
    tags:
      region: northeurope
      zone: infra
      stage: none
  master2:
    name: master2
    tags:
      region: northeurope
      zone: infra
      stage: none
  master3:
    name: master3
    tags:
      region: northeurope
      zone: infra
      stage: none
### infra node, user for exposing router ###
### have not tested with multiple infra nodes yet
infranodes:
  infranode1:
    name: infranode1
    tags:
      region: northeurope
      zone: infra
      stage: dev
### add as many nodes here as you like
## does not split across region yet. thats a todo
nodes:
  node1:
    name: node1
    tags:
      region: northeurope
      zone: frontend
      stage: dev

#  node2:
#    name: node2
#    tags:
#      region: northeurope
#      zone: backend
#      stage: dev
#i#nodes:
#  name: node1
#  name: node2

```

### Required params 

To setup the following params are required 
 - resource_group_name: ose86
 - subscriptionID: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
 - adminUsername: userforjumphost
 - adminPassword: passwordforjumphost
 - sshkey : (public sshkey)
 - rh_subcription_user
 - rh_subcription_pass
 - openshift_pool_id











