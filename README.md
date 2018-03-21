# VM Creation and Provisioning Framework

## Introduction
The ansible scripts can be used to create virtual machine with specific configurations on VMWare Hyperviser (ESX/ESXi). The scripts configure and secure the machines with the company policy standards, and then install necessary software packages. 

There are following scripts:
  - `create_vm.yml`: Creates, Configures, Secures Virtual Machines and installs base packages packages 
  - `decommission_vm.yml`: Deconfigures, Uninstalls packages and Deletes Virtual machines
  - `update_windows.yml`: Updates all windows machines for security update
  - `inventory.yml`: [NOT A SCRIPT] A list of machines in form of array.

## Quick Usage
- Creating Virtual Machines, Configure, Secure and Install base packages
`ansible-playbook create_vm.yml`
- Only Create Virtual Machines (Skip Configure, Secure and Installation of Base packages)
`ansible-playbook create_vm.yml --tags=vm_provision`
- Configure, Secure and Installation of Base packages, Assuming Machines are already created and running (any or combination of `configuration`, `security`, `base_packages)`
`ansible-playbook create_vm.yml --tags=configuration,security,base_packages`
- Deconfigure and Delete Virtual Machine (Decommissioning Process)
`ansible-playbook decommission_vm.yml`
- Update Windows Machines
`ansible-playbook update_windows.yml`

## Configuration

### Inventory - List of Servers to be managed
>The inventory is being managed in a different way here, compared to the standard way of managing hosts. To easily manage the inventory and specify configurations

Sample Inventory
```
---
irving_database_servers:
  - vm_ip: "10.106.176.46"
    vm_name: "JAYDEEP009"
    vm_template: "2012_R2_STD Fusion"
    os: "win"
    packages: "sql_client"    # Comma separated special indentification of base packages
    data_partitions: []
      
  - vm_ip: "10.106.176.45"
    vm_name: "JAYDEEP007"
    vm_template: "Fusion_RHEL6_Template"
    os: "linux"

irving_jboss_servers:
  - vm_ip: "10.106.176.45"
    vm_name: "JAYDEEP007"
    vm_template: "Fusion_RHEL6_Template"
    os: "linux"

irving_apache_servers:
  - vm_ip: "10.106.176.46"
    vm_name: "JAYDEEP009"
    vm_template: "2012_R2_STD Fusion"
    packages: "sql_client"
    os: "win"
    data_partitions: []

# The inventory variable below should be assigned with the target list variable
inventory: "{{ irving_database_servers }} + {{ irving_database_servers }}" 
```
> Tip: You can also assign the inventory variable from the command line
`ansible-playbook create_vm.yml --extra-vars "inventory={{irving_jboss_servers}}"`

### VM Configurations
As shown in the inventory.yml file, some parameters have to be specified per VM. Following parameters are mendatory to specify in the invetory file per host.

    vm_ip: "<target_ip_address_of_the_machine>"
    vm_name: "<unique_host_name>"
    vm_template: "<the_template_name_to_be_used_to_clone_the_machine_from>"
    os: "win | linux"

There is a list of common variables in config/common.yml file. The common variables are the default value to be used if explicit value is not specified in the inventory file.
The variable can always be overridden in the inventory file (make sure to remove the "default_" from the variable name). 

For example:

    irving_jboss_servers:
      - vm_ip: "10.106.176.45"
        vm_name: "JAYDEEP007"
        vm_template: "Fusion_RHEL6_Template"
        os: "linux"
        vm_network_port_group: NETWORK_24_123_BLA_blA
        
   