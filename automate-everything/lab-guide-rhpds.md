# LAB GUIDE - Ansible on RHPDS

#### Section 1 : Standing up the lab environment

Objective: Instructions on how to stand up the lab

#### Section 2 : Exploring the lab environment

Objective: A quick introduction to Ansible command line for students with no experience with Ansible. Those with prior knowledge of Ansible might breeze through this section.

#### Section 3 : Quick intro to F5 Load Balancer

Objective: To cover the basics of a load balancer, in this case an F5 BigIP is used as an example of a load balancer and a network device that Ansible can control and automate.

#### Section 4 : Users, Inventories, Credentials, Projects

Objective: To prepare for the lab exercises in Section 4, in this section we will cover the fundamentals of Ansible Tower.

#### Section 5 : Users, Inventories, Credentials, Projects

Objective: In this section we will create a small scale orchestration where:

-   A user launches a job to add a new web server into the load balancer pool

-   The selection of the new web server is based on (Ansible Tower) survey

-   The job will require an approval from the admin

-   The admin gets the approval request notification via Slack

-   Once approved, before adding the new web server into the pool, the system will ensure httpd is installed and running

-   If all goes well, the new web server will then be added to the load balancer pool

# Section 1 : Standing up the lab environment in RHPDS

Objective: We will be using a modified version of the Ansible F5 Automation workshop in RHPDS for the exercises, let's set it up.

REQUIREMENTS

These lab exercises requires a few things:

-   You need an access to Red Hat Product Demo System (RHPDS) located at <https://rhpds.redhat.com>, please contact your Red Hat account manager to get one

-   A computer with a full direct access to the Internet, note: connecting through corporate VPN network has been known to cause issues in accessing the lab

-   SSH client such as Putty on Windows or terminal (x-term) on Linux or Mac

LAB SETUP

1.  Login to RHPDS (<https://rhpds.redhat.com>)

2.  Go to the Services -> Catalogs. Then, under Service Catalogs > All Services > Workshops choose: Ansible F5 Automation Workshop

![](https://lh3.googleusercontent.com/jduBS4LHNAyIsFoEj3iYusGuYb11cV8yHW8lgg5YNupl8L8y996IEVUwLQdBr09ev1Zz3G9w2M0AE_BUQ7gCWSzrUhYDV-COa2P4AC7G0c6kaSxjz6drHeHk5rEAS_2ySr8-36Mq)

1.  Click the Order button

2.  Fill out the Lab form 

-   Set "SFDC Opportunity, Campaign ID, or Partner Registration" = 7013a000002gvgSAAQ

-   Note: Make sure to select the appropriate region, for example North America if you are in North America

-   Leave Number of Attendees = 1

4.  Click the Submit button

Once you click the Submit button, you should receive a confirmation email within 5-10 minutes that your lab is being prepared. It would take 30-90 minutes for the lab to be ready to use. When it is ready, you will receive a final email with an instruction on how to access your lab. For example:

Thank you for ordering Ansible Linux Automation Workshop (T)

Your environment for RHPDS-RH-arwinata-redhat.com-PROD_ANSIBLE_WORKSHOPS-4fb0_COMPLETED has been provisioned.

<... snip ...>

NOTICE: Your environment will expire and be deleted in 2 day(s) at 2020-09-03 00:00:00 -0400.

The list of VMs for this workshop is available at:[  http://f544.open.redhat.com](http://4fb0.open.redhat.com/)

And when you go to the web link provided, you should see something as followed:

![](https://lh5.googleusercontent.com/c1klrdAyiYL-BYTljBQXSJw3QP9T0mh0ZlJNFl2FhHy8Pcy4xm-leFdQGFDL0168pm0B9MSKln4tX9zWMckxgaykqh59DUcBrLxt0SSMdypnqYrW24p5j6R6Hl-kyE2DDTqIEVf8)

Scroll down to the bottom and click on student1 under Workbench Information to see details on how to access your lab. Example:

![](https://lh5.googleusercontent.com/FNyXO1DesMn24kORQkBPHdRyDST-yhlUZNMpRlYRu5mcGg3JPRywnHLeU9EcztR5r7mZvPMi9JTfbQSV1SwCBhfxYiNNkMlh-OsmQB1f6f2m4dY_yaO0I2ruQMf5B9vL37fdv-If)

1.  Follow the instructions above to access your Ansible Control Node via SSH

By now your lab is ready to use for the exercises in this lab guide. Please make sure to follow only the exercise instructions in this lab guide, and *not* the one on the web link (example above [http://f544.open.redhat.com](http://4fb0.open.redhat.com/)) you receive from RHPDS. 

# Section 2 : Exploring the lab environment

Objective: A quick introduction to Ansible command line for students with no experience with Ansible. Those with prior knowledge of Ansible might breeze through this section.

![](https://lh6.googleusercontent.com/rTq7eX7I-sKhufhld9ICPjuQa2RtfMCfnEsYDXAbuIgQ2tzWb17sD_9hJCy_0va8kSLG46llSguFB74hn9POSmC56NomuIDxKCkWuZ594W_XGydxeFJ2Kv8jnEJxD0ujf_ejMHyZ)

#### Step 1

Login to your Ansible Control Node via SSH as student1 as per the instruction in Section 1. And navigate to the home directory.

```
[student1@ansible ~]$
```

#### Step 2

Run the ansible command with the --version command to look at what is configured:

```
[student1@ansible ~]$ ansible --version
ansible 2.6.2
  config file = /home/student1/.ansible.cfg
  configured module search path = [u'/home/student1/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, May  3 2017, 07:55:04) [GCC 4.8.5 20150623 (Red Hat 4.8.5-14)]
```

Note: The Ansible version you see might differ from the above output

This command gives you information about the version of Ansible, location of the executable, version of Python, search path for the modules and location of the ansible configuration file.

#### Step 3

Use the cat command to view the contents of the ansible.cfg file.

```
[student1@ansible ~]$ cat ~/.ansible.cfg
[defaults]
stdout_callback = yaml
connection = smart
timeout = 60
deprecation_warnings = False
host_key_checking = False
retry_files_enabled = False
inventory = /home/student1/lab_inventory/hosts
[persistent_connection]
connect_timeout = 200
command_timeout = 200
[student1@ansible ~]$
```

Note the following parameters within the ansible.cfg file:

-   inventory: shows the location of the ansible inventory being used

#### Step 4

The scope of a play within a playbook is limited to the groups of hosts declared within an Ansible inventory. Ansible supports multiple [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) types. An inventory could be a simple flat file with a collection of hosts defined within it or it could be a dynamic script (potentially querying a CMDB backend) that generates a list of devices to run the playbook against.

In this lab you will work with a file based inventory written in the ini format. Use the cat command to view the contents of your inventory:

```
[student1@ansible ~]$ cat ~/lab_inventory/hosts
```


The output will look as follows with student1 being the respective student workbench:

```
[all:vars]
ansible_user=student1
ansible_ssh_pass=skfE9PXk41hgMs
ansible_port=22

[lb]
f5 ansible_host=3.235.9.14 ansible_user=admin private_ip=172.16.135.2 ansible_ssh_pass=skfE9PXk41hgMs

[control]
ansible ansible_host=3.237.237.173 ansible_user=ec2-user private_ip=172.16.98.96

[web]
node1 ansible_host=100.24.107.157 ansible_user=ec2-user private_ip=172.16.36.182
node2 ansible_host=3.235.57.122 ansible_user=ec2-user private_ip=172.16.150.46
```

Note that the IP addresses will be different in your environment.

#### Step 5

In the above output every [ ] defines a group. For example [web] is a group that contains the hosts host1 and host2.

Note: A group called all always exists and contains all groups and hosts defined within an inventory.

We can associate variables to groups and hosts. Host variables are declared/defined on the same line as the host themselves. For example for the host f5:

f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin

-   f5 - The name that Ansible will use. This can but does not have to rely on DNS

-   ansible_host - The IP address that ansible will use, if not configured it will default to DNS

-   ansible_user - The user ansible will use to login to this host, if not configured it will default to the user the playbook is run from

-   private_ip - This value is not reserved by ansible so it will default to a [host variable](http://docs.ansible.com/ansible/latest/intro_inventory.html#host-variables). This variable can be used by playbooks or ignored completely.

-   ansible_ssh_pass - The password ansible will use to login to this host, if not configured it will assume the user the playbook ran from has access to this host through SSH keys.

Does the password have to be in plain text? No, Red Hat Ansible Tower can take care of credential management in an easy to use web GUI or a user may use [ansible-vault](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html#protecting-sensitive-variables-with-ansible-vault)

Go back to the home directory, all the following exercises will be performed in the home directory.

```
[student1@ansible ~]$ cd ~
```


# Section 3 : Quick intro to F5 Load Balancer

Objective: To cover the basics of a load balancer, in this case an F5 BigIP is used as an example of a load balancer and a network device that Ansible can control and automate.

#### Step 1 - Gather Facts

Using your text editor of choice create a new file called bigip-facts.yml.

```
[student1@ansible ~]$ nano bigip-facts.yml
```

vim and nano are available on the control node, as well as Visual Studio

Ansible playbooks are YAML files. YAML is a structured encoding format that is also extremely human readable (unlike it's subset - the JSON format).

Enter the following play definition into bigip-facts.yml:

```
---
- name: GRAB F5 FACTS
  hosts: f5
  connection: local
  gather_facts: no
```

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: f5, indicates the play is run only on the F5 BIG-IP device

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: no disables facts gathering. We are not using any fact variables for this playbook.

Next, add the first task. This task will use the bigip_device_info module to grab useful information from the BIG-IP device.

```
 tasks:
- name: COLLECT BIG-IP FACTS
      bigip_device_info:
        gather_subset:
- system-info
        provider:
          server: "{{private_ip}}"
          user: "{{ansible_user}}"
          password: "{{ansible_ssh_pass}}"
          server_port: 8443
          validate_certs: no
      register: device_facts
```

A play is a list of tasks. Tasks and modules have a 1:1 correlation. Ansible modules are reusable, standalone scripts that can be used by the Ansible API, or by the ansible or ansible-playbook programs. They return information to ansible by printing a JSON string to stdout before exiting.

-   name: COLLECT BIG-IP FACTS is a user defined description that will display in the terminal output.
-   bigip_device_info: tells the task which module to use. Everything except register is a module parameter defined on the module documentation page.
-   The gather_subset: system_info parameter tells the module only to grab system level information.
-   The provider: parameter is a group of connection details for the BIG-IP.
-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory
-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with
-   Thepassword: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with
-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with. 8443 is what's being used in this lab, but could be different depending on the deployment.
-   register: device_facts tells the task to save the output to a variable bigip_device_info

Next, append the second task to above . This task will use the debug module to print the output from device_facts variable we registered the facts to.

```
- name: DISPLAY COMPLETE BIG-IP SYSTEM INFORMATION
      debug:
        var: device_facts
```

-   The name: COMPLETE BIG-IP SYSTEM INFORMATION is a user defined description that will display in the terminal output.
-   debug: tells the task to use the debug module.
-   The var: device_facts parameter tells the module to display the variable bigip_device_info.

Finally let's append two more tasks to get more specific info from facts gathered, to the above playbook.

```
- name: DISPLAY ONLY THE MAC ADDRESS
      debug:
        var: device_facts['system_info']['base_mac_address']
- name: DISPLAY ONLY THE VERSION
      debug:
        var: device_facts['system_info']['product_version']
```

-   var: device_facts['system_info']['base_mac_address'] displays the MAC address for the Management IP on the BIG-IP device
-   device_facts['system_info']['product_version'] displays the product version BIG-IP device

Because the bigip_device_info module returns useful information in structured data, it is really easy to grab specific information without using regex or filters. Fact modules are very powerful tools to grab specific device information that can be used in subsequent tasks, or even used to create dynamic documentation (reports, csv files, markdown).

Run the playbook - exit back into the command line of the control host and execute the following:

[student1@ansible ~]$ ansible-playbook bigip-facts.yml

The output will look as follows.

[student1@ansible ~]$ ansible-playbook bigip-facts.yml

PLAY [GRAB F5 FACTS] ****************************************************************************************************************************************

TASK [COLLECT BIG-IP FACTS] *********************************************************************************************************************************

changed: [f5]

TASK [DISPLAY COMPLETE BIG-IP SYSTEM INFORMATION] ***********************************************************************************************************

ok: [f5] => {

    "device_facts": {

        "changed": true,

        "failed": false,

        "system_info": {

            "base_mac_address": "0a:54:53:51:86:fc",

            "chassis_serial": "685023ec-071e-3fa0-3849dcf70dff",

            "hardware_information": [

                {

                    "model": "Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz",

                    "name": "cpus",

                    "type": "base-board",

                    "versions": [

                        {

                            "name": "cpu stepping",

                            "version": "2"

                        },

                        {

                            "name": "cpu sockets",

                            "version": "1"

                        },

                        {

                            "name": "cpu MHz",

                            "version": "2399.981"

                        },

                        {

                            "name": "cores",

                            "version": "2  (physical:2)"

                        },

                        {

                            "name": "cache size",

                            "version": "30720 KB"

                        }

                    ]

                }

            ],

            "marketing_name": "BIG-IP Virtual Edition",

            "package_edition": "Point Release 7",

            "package_version": "Build 0.0.1 - Tue May 15 15:26:30 PDT 2018",

            "platform": "Z100",

            "product_build": "0.0.1",

            "product_build_date": "Tue May 15 15:26:30 PDT 2018",

            "product_built": 180515152630,

            "product_changelist": 2557198,

            "product_code": "BIG-IP",

            "product_jobid": 1012030,

            "product_version": "13.1.0.7",

            "time": {

                "day": 15,

                "hour": 23,

                "minute": 46,

                "month": 4,

                "second": 25,

                "year": 2019

            },

            "uptime": 1738.0

        }

    }

}

TASK [DISPLAY ONLY THE MAC ADDRESS] *************************************************************************************************************************

ok: [f5] => {

    "device_facts['system_info']['base_mac_address']": "0a:54:53:51:86:fc"

}

TASK [DISPLAY ONLY THE VERSION] *****************************************************************************************************************************

ok: [f5] => {

    "device_facts['system_info']['product_version']": "13.1.0.7"

}

PLAY RECAP **************************************************************************************************************************************************

f5 : ok=4    changed=1    unreachable=0    failed=0

And finally, please confirm that you have web access to your F5 load balancer. You can find its details in your Ansible inventory file.

For example, for the entry below, you would go to [https://34.199.128.69:8443](http://34.199.128.69:8443) (the web interface listens to port 8443), username is admin, password is skfE9PXk41hgMs

[lb]

f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=skfE9PXk41hgMs

#### Step 2 - Add nodes

![](https://lh5.googleusercontent.com/wHMFdI0ck1jokElILTUE9HgARY1Yorn38vN74SjPPxME52ZPOmqUbsZX8BtFr1xD6YbSIprCR2FaI7GEIdV5dt_C-X-ijUvsRfoX6yb9CQz3aDH4v9IqOJacaqLCsKeKnsHrGiEy)

Using your text editor of choice create a new file called bigip-node.yml.

[student1@ansible ~]$ nano bigip-node.yml

vim and nano are available on the control node, as well as Visual Studio and Atom via RDP

Enter the following play definition into bigip-node.yml:

---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: CREATE NODES

    bigip_node:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      host: "{{hostvars[item].ansible_host}}"

      name: "{{hostvars[item].inventory_hostname}}"

    loop: "{{ groups['web'] }}"

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: CREATE NODES is a user defined description that will display in the terminal output.

-   bigip_node: tells the task which module to use. Everything except loop is a module parameter defined on the module documentation page.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The host: "{{hostvars[item].ansible_host}}" parameter tells the module to add a web server IP address already defined in our inventory.

-   The name: "{{hostvars[item].inventory_hostname}}" parameter tells the module to use the inventory_hostname as the name (which will be node1 and node2).

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

-   loop: tells the task to loop over the provided list. The list in this case is the group web which includes two RHEL hosts.

Run the playbook - exit back into the command line of the control host and execute the following:

[student1@ansible ~]$ ansible-playbook bigip-node.yml

The output will look as follows.

[student1@ansible]$ ansible-playbook bigip-node.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [CREATE NODES] ************************************************************

changed: [f5] => (item=node1)

changed: [f5] => (item=node2)

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0

Confirm that you can see the nodes added in F5 by logging in to F5 via the web interface, and going to Local Traffic → Nodes

![](https://lh3.googleusercontent.com/0ov5DUZQYZA4Nr8zje_JQ30-30aBhMBvJEZbGGdRv6-V7vYllVbL5FeufNI2gm9-w0d1RKJYkIeQhEe587HTKf1PBI3OYNNLenTvB3bNl-b8DByBM7Dz1NWjxebJgPULAo1pw4yg)

#### Step 3 - Create a pool and add members

Enter the following play definition into bigip-pool.yml:

---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: CREATE POOL

    bigip_pool:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      name: "http_pool"

      lb_method: "round-robin"

      monitors: "/Common/http"

      monitor_type: "and_list"

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: CREATE POOL is a user defined description that will display in the terminal output.

-   bigip_pool: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The name: "http_pool" parameter tells the module to create a pool named http_pool

-   The lb_method: "round-robin" parameter tells the module the load balancing method will be round-robin. A full list of methods can be found on the documentation page for bigip_pool.

-   The monitors: "/Common/http" parameter tells the module that the http_pool will only look at http traffic.

-   The monitor_type: "and_list" ensures that all monitors are checked.

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

Run the playbook - exit back into the command line of the control host and execute the following:

[student1@ansible ~]$ ansible-playbook bigip-pool.yml

The output will look as follows.

[student1@ansible ~]$ ansible-playbook bigip-pool.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [CREATE POOL] *************************************************************

changed: [f5]

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0

Now that the pool has been created, let's add node1 and node2 into the pool.

Enter the following play definition into bigip-pool-members.yml:

---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: ADD POOL MEMBERS

    bigip_pool_member:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      state: "present"

      name: "{{hostvars[item].inventory_hostname}}"

      host: "{{hostvars[item].ansible_host}}"

      port: "80"

      pool: "http_pool"

    loop: "{{ groups['web'] }}"

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: ADD POOL MEMBERS is a user defined description that will display in the terminal output.

-   bigip_pool_member: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The state: "present" parameter tells the module we want this to be added rather than deleted.

-   The name: "{{hostvars[item].inventory_hostname}}" parameter tells the module to use the inventory_hostname as the name (which will be host1 and host2).

-   The host: "{{hostvars[item].ansible_host}}" parameter tells the module to add a web server IP address already defined in our inventory.

-   The port: parameter tells the pool member port.

-   The pool: "http_pool" parameter tells the module to put this node into a pool named http_pool

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab. Finally there is a loop parameter which is at the task level (it is not a module parameter but a task level parameter:

-   loop: tells the task to loop over the provided list. The list in this case is the group web which includes two RHEL hosts.

Run the playbook - exit back into the command line of the control host and execute the following:

[student1@ansible ~]$ ansible-playbook bigip-pool-members.yml

The output will look as follows.

[student1@ansible ~]$ ansible-playbook bigip-pool-members.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [ADD POOL MEMBERS] ********************************************************

changed: [f5] => (item=host1)

changed: [f5] => (item=host2)

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0

Confirm that you see the new pool created and its members by logging in to your F5 load balancer web interface.

![](https://lh6.googleusercontent.com/CUS6O9IILzxLVVp8NCT6m2i8gzfH3neZoFjdDAnNisdRnT0psqCuK8KDiY1xPeZF-OY_gpPrE1d0BLKTt2O5hEMmb9fvN_ek6nT5lU7PXU6TVfvLqw7KNCOSA3R2uQ-XetCWw3fa)

![](https://lh3.googleusercontent.com/DBpyEF0xV_Z6qsnM6GGO3rDKIn72O0uRNKFGx9Rn1KWO_XMUR44Pat0VwXPJA7yc8KafwLo_aL1VxVR3YpNlDnmI7rw--OQCmagytiJe_c5SVSBklpnYNs_4B0NeWwkoqOfiodKl)

#### Step 4 - Create a virtual server

A virtual server in F5 load balancer is a combination of IP and port number. When an incoming web request comes to this virtual server, the request will then be passed to the pool of nodes that is assigned to that virtual server. Who gets to answer the requests? That depends on the load balancing policy chosen, for example if "round robin" is used, each member node of the pool will take turn to answer requests. (Remember, we did set our pool to use round robin)

Now let's create a virtual server.

Enter the following play definition into bigip-virtual-server.yml:

---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: ADD VIRTUAL SERVER

    bigip_virtual_server:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      name: "vip"

      destination: "{{private_ip}}"

      port: "443"

      enabled_vlans: "all"

      all_profiles: ['http','clientssl','oneconnect']

      pool: "http_pool"

      snat: "Automap"

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: f5, indicates the play is run only on the F5 BIG-IP device

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: no disables facts gathering. We are not using any fact variables for this playbook.

-   name: ADD VIRTUAL SERVER is a user defined description that will display in the terminal output.

-   bigip_virtual_server: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   Thepassword: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The name: "vip" parameter tells the module to create a virtual server named vip

-   The destination" parameter tells the module which IP address to assign for the virtual server

-   The port paramter tells the module which Port the virtual server will be listening on

-   The enabled_vlans parameter tells the module which all vlans the virtual server is enbaled for

-   The all_profiles paramter tells the module which all profiles are assigned to the virtuals server

-   The pool parameter tells the module which pool is assigned to the virtual server

-   The snat paramter tells the module what the Source network address address should be. In this module we are assigning it to be Automap which means the source address on the request that goes to the backend server will be the self-ip address of the BIG-IP

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

Run the playbook - exit back into the command line of the control host and execute the following:

[student1@ansible ~]$ ansible-playbook bigip-virtual-server.yml

[student1@ansible]$ ansible-playbook bigip-virtual-server.yml

PLAY [BIG-IP SETUP]*************************************************************

TASK [ADD VIRTUAL SERVER] ******************************************************

changed: [f5]

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0

Confirm that you see the new virtual server in F5:

![](https://lh3.googleusercontent.com/4wVTJOohN3TxDqBGBYkknVqial1Tb06AGt9pGIBrFJuKXa6TZ1BYfdsFXtSn-ntlTyTZyM7XRkdtxdQ-NQ95mDy8z6JrFl7KKrojJwLL_ScE_0HtphwPECYS7G8A2qlRwZWoVnb6)

Verify that the load balancing does really work:

Each RHEL web server actually already has apache running. The above exercises have successfully set up the load balancer for the pool of web servers. Open up the public IP of the F5 load balancer in your web browser:

This time use port 443 instead of 8443, e.g. [https://X.X.X.X:443/](https://x.x.x.x/)

Each time you refresh the host will change between node1 and node2. Here is animation of the host field changing: ![animation](https://lh5.googleusercontent.com/JGdU0EnwzgPCWiBo4KMnDuRTxJBvRrkWnqErzeScPGvf3BX4T6Kzx_w3IRiKR088jSUnMP_TqWp3B8H78hR2fPmFdQ5LdyRhqEC1kig-NwMg9nnXnjg7SZNLWNVXvVWloBD0wjum)

Instead of using a browser window it is also possible to use the command line on the Ansible control node. Use the curl command on the ansible_host to access public IP or private IP address of F5 load balancer in combination with the --insecure and --silent command line arguments. Since the entire website is loaded on the command line it is recommended to | grep for the student number assigned to the respective workbench. (e.g. student5 would | grep student5)

[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node1</p>

[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node2</p>

[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node1</p>


As the final step in this section, let’s remove node2 from F5 so that we can use it for the exercises in Section 5. To do this, please use the F5 web interface:
1. Go to Local Traffic → Pools → Members
2. Click on node2, then [Remove]
3. Local Traffic → Nodes 
4. Click on node2, then [Delete]. Confirm Delete



# Section 4 : Users, Inventories, Credentials, Projects

Objective: To prepare for the lab exercises in Section 4, in this section we will cover the fundamentals of Ansible Tower.

Ansible Tower Users

In Section 1 we learned a little bit about Ansible Engine, which is the actual command line (executables) of Ansible. As you saw in the exercises, you needed to login to the actual control node via ssh. This means if you need to give another user permission to use the control node, you need to give that person an ssh (shell) access. This is not preferred since it's less secure, tedious to manage, and not scalable.

Ansible Tower has a built-in Role Based Access Control (RBAC). For the purpose of this lab, we will use the built in authentication system.

There are three types of Tower Users:

Normal User: Have read and write access limited to the inventory and projects for which that user has been granted the appropriate roles and privileges.

System Auditor: Auditors implicitly inherit the read-only capability for all objects within the Tower environment.

System Administrator: Has admin, read, and write privileges over the entire Tower installation.

Let's create a user:

In the Tower menu under ACCESS click Users

Click the green plus button

Fill in the values for the new user:

Parameter  Value

FIRST NAME  Peter

LAST NAME  Parker

Organization  Default

EMAIL  wweb@example.com

USERNAME  webdev1

PASSWORD  ansible123

CONFIRM PASSWORD  ansible123

USER TYPE  Normal User

Click SAVE

![](https://lh4.googleusercontent.com/Wy2t7vQf8sUquxu4omf9z8Df7TsXfVKyrALPzYf2eTNqgnQyQq79CON4I96GrZoB4OIb3OtZOh96OYg6LjeQ5xRt-p48aakh5ANllSmWKU57Y7wEKkr_VreMA9gyzb92TZZ4A3-f)

Inventories

This is the equivalent of an inventory file in Ansible Engine. There is a lot more to it (like dynamic inventories) but let's start with the basics. There will be one inventory, the Workshop Inventory. Click the Workshop Inventory then click the Hosts button.

The inventory information at ~/lab_inventory/hosts was pre-loaded into the Ansible Tower Inventory as part of the provisioning process.

Example:

![](https://lh3.googleusercontent.com/YdT0ONWJ56KfW9BvOVOcJGIHf_DzqIN0DJFvdFImKUuzB7qkh8boRySGBRnVZtr5qklbhsOeC_Q_PcLkf6P0hXkKEgNA4RuIWFaCXMQN9QZoWiPjBUE9y5jokCApRRWf2x0VMMhA)

Credentials

Now we will examine the credentials to access our managed hosts from Tower. As part of the provisioning process for this Ansible Workshop the Workshop Credential has already been setup.

In the RESOURCES menu choose Credentials. Now click on the Workshop Credential.

Note the following information:

|

Parameter

 |

Value

 |
|

Credential Type

 |

Machine- Machine credentials define ssh and user-level privilege escalation access for playbooks. They are used when submitting jobs to run playbooks on a remote host.

 |
|

username

 |

ec2-user which matches our command-line Ansible inventory username for the other linux nodes

 |
|

SSH PRIVATE KEY

 |

ENCRYPTED - take note that you can't actually examine the SSH private key once someone hands it over to Ansible Tower

 |

Projects

An Ansible Tower Project is a logical collection of Ansible Playbooks. You can manage your playbooks by placing them into a source code management (SCM) system supported by Tower, including Git, Subversion, and Mercurial.

For this demonstration we will use playbooks stored in a Git repository:

https://github.com/redhat-partner-tech/partner-tech-days-sept2020

To configure and use this repository as a Source Control Management (SCM) system in Tower you have to create a Project that uses the repository

Go to RESOURCES → Projects in the side menu view click the green + button. Fill in the form:

|

Parameter

 |

Value

 |
|

NAME

 |

Tech Day Project

 |
|

ORGANIZATION

 |

Default

 |
|

SCM TYPE

 |

Git

 |

Now you need the URL to access the repo. Go to the Github repository mentioned above, choose the green Clone or download button on the right, click on Use https and copy the HTTPS URL. Enter the URL into the Project configuration:

|

Parameter

 |

Value

 |
|

SCM URL

 |

https://github.com/redhat-partner-tech/partner-tech-days-sept2020

 |
|

SCM UPDATE OPTIONS

 |

Tick the first three boxes to always get a fresh copy of the repository and to update the repository when launching a job

 |

Click SAVE

The new Project will be synced automatically after creation. But you can also do this manually: Sync the Project again with the Git repository by going to the Projects view and clicking the circular arrow Get latest SCM revision icon to the right of the Project.

After starting the sync job, go to the Jobs view: there is a new job for the update of the Git repository.

# Section 5 : Users, Inventories, Credentials, Projects

Objective: In this section we will create a small scale orchestration where:

-   A user launches a job to add a new web server into the load balancer pool

-   The selection of the new web server is based on (Ansible Tower) survey

-   The job will require an approval from the admin

-   The admin gets the approval request notification via Slack

-   Once approved, before adding the new web serve into the pool, the system will ensure httpd is installed and running

-   If all goes well, the new web server will then be added to the load balancer pool

JOB TEMPLATE

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. So before running an Ansible Job from Tower you must create a Job Template that pulls together:

-   Inventory: On what hosts should the job run?

-   Credentials What credentials are needed to log into the hosts?

-   Project: Where is the Playbook?

-   What Playbook to use?

Go to the Templates view, click the ![plus](https://lh5.googleusercontent.com/gTJqFBwbrTEitHRQe8XJqFyb9CVMGPYJ_9m9B9DPCflU9xaT-6tY6WwdT0cjI-SYvI6fTF7cO80eVs5kNYwHiWVn0wNcEPmcSIL_p_eWIvHiTunfGUhhQlM99l4mubgVVlMcZbkg) button and choose Job Template.

Tip

Remember that you can often click on magnfying glasses to get an overview of options to pick to fill in fields.

|

Parameter

 |

Value

 |
|

NAME

 |

Ping a node

 |
|

JOB TYPE

 |

Run

 |
|

INVENTORY

 |

Workshop Inventory

 |
|

PROJECT

 |

Tech Day Project

 |
|

PLAYBOOK

 |

automate-everything/lab-examples/ping-node1.yml

 |
|

CREDENTIAL

 |

Workshop Credentials

 |

-   Click SAVE

Launch the job and observe the output. This is just an example of a simple job to ping (Ansible ping, not network ping) node1.

Now let's create another Job Template.

|

Parameter

 |

Value

 |
|

NAME

 |

Install Web Server

 |
|

JOB TYPE

 |

Run

 |
|

INVENTORY

 |

Workshop Inventory

 |
|

PROJECT

 |

Tech Day Project

 |
|

PLAYBOOK

 |

automate-everything/lab-examples/apache.yml

 |
|

CREDENTIAL

 |

Workshop Credentials

 |
|

OPTIONS

 |

Tick the checkbox next to "Enable Privilege Escalation"

 |

-   Click SAVE

You will not be able to launch this Job Template successfully until we create a survey that asks users to supply the value of the variable "my_hostname" defined in the playbook.

<https://github.com/redhat-partner-tech/partner-tech-days-sept2020/blob/master/automate-everything/lab-examples/apache.yml>

We will configure the survey in a bit, but for now let's look at workflow.

WORKFLOW

The basic idea of a workflow is to link multiple Job Templates together. They may or may not share inventory, Playbooks or even permissions. The links can be conditional:

-   if job template A succeeds, job template B is automatically executed afterwards

-   but in case of failure, job template C will be run.

And the workflows are not even limited to Job Templates, but can also include project or inventory updates.

This enables new applications for Ansible Tower: different Job Templates can build upon each other. E.g. the networking team creates playbooks with their own content, in their own Git repository and even targeting their own inventory, while the operations team also has their own repos, playbooks and inventory.

Let's try creating a workflow which combines the 2 job templates we have created.

Set up the workflow. Workflows are configured in the Templates view, you might have noticed you can choose between Job Template and Workflow Template when adding a template.

![workflow add](https://lh3.googleusercontent.com/F2ppdy4CtaLLelF1izrUW3fh7JWH6k6BhsE1x7IySwyhLsVQhn9DVig3Xgsdsfks6S8Zgj-DHqagVURjCok1MnpOXEEXCHEmyIgRnUGofAx0Q8cUkX7366rMrZWC0ohMvdSETeQo)

-   Go to the Templates view and click the the green plus button. This time choose Workflow Template

|

NAME

 |

Deploy Web Server to Prod

 |
|

ORGANIZATION

 |

Default

 |
|

INVENTORY

 |

Workshop Inventory

 |

-   Click SAVE

After saving the template the Workflow Visualizer opens to allow you to build a workflow. You can later open the Workflow Visualizer again by using the button on the template details page.

-   Click on the START button, a new node opens. To the right you can assign an action to the node, you can choose between JOBS, PROJECT SYNC, INVENTORY SYNC and APPROVAL.

-   In this lab we'll link our two jobs together, so select the "Ping a node" job and click SELECT.

-   The node gets annotated with the name of the job. Hover the mouse pointer over the node, you'll see a red x, a green + and a blue chain-symbol appear.

-   Click the green + sign

-   Choose "Install Web Server" as the next Job

-   Leave Type set to On Success

-   Click SELECT

-   Click SAVE in the WORKFLOW VISUALIZER view

-   Click SAVE in the Workflow Template view

SURVEYS

The survey feature only provides a simple query for data - it does not support four-eye principles, queries based on dynamic data or nested menus.

On the "Deploy Web Server to Prod" workflow template, click the "Add Survey" button.

|

PROMPT

 |

Please specify a hostname

 |
|

DESCRIPTION

 |

Make sure the hostname is correct!

 |
|

ANSWER VAR NAME

 |

my_hostname

 |
|

ANSWER TYPE

 |

Text

 |
|

MIN LENGTH

 |

1

 |
|

MAX LENGTH

 |

100

 |

-   Click +ADD, then click SAVE

Then SAVE the workflow.

Launch the workflow and when the survey prompts you for a hostname, use node1 as the answer. Observe the workflow, click on DETAILS to see outputs.

NOTIFICATIONS

Now let's look at notifications. Ansible Tower supports quite a few ways of notifying users, e.g. email, Slack, etc. For the purpose of this exercise, let's use Slac (note: if you don't have a Slack account, don't worry about it, set this up anyway and save it. We need this for our exercise on Approval later.)

On the Ansible Tower screen, go to ADMINISTRATION -> Notifications.

Click the green plus sign to create a new notification template:

|

NAME

 |

My Slack Notification

 |
|

ORGANIZATION

 |

Default

 |
|

TYPE

 |

Slack

 |
|

DEST CHANNEL

 |

#general

 |
|

TOKEN

 |

<insert your Slack token here if you have one, else insert "12345"

 |

-   SAVE

Now let's go back to our workflow from the previous exercise (Deploy Web Server to Prod).

Click on NOTIFICATIONS button at the top area. And enable the approval button.

![](https://lh5.googleusercontent.com/Kg6K8NQSV3fRW2wtzpY1CFrMhGHGyboJJsDJhjB2KCf38fnanBPjOYf8-P_rOZEOfk8owsZbDdCjWsLKh3rc-tK6-zhAb5UexP3ofc9E53NvOob9Sis2__37HJpnA7DKhU3O4K8e)

Then click on PERMISSIONS button, click on the green plus sign button to add a permission. Add our webdev1 user to be able to execute the workflow:

![](https://lh4.googleusercontent.com/nVo5AILoLGw51S6qJn7vY5sZw3fBmacSfvb_Hho8A4jv73Tl7HbKq3umVIw5Oetllpo1LbTzDDuPBF6fvILBT9CnnuYy9VuUlxjue6rAx7m6s12dslVZ3z5QIicEd0MPeRM3jUa3)

Don't forget to click on SAVE.

APPROVAL

Now click on the DETAILS button to go back to your workflow main screen, then click on WORKFLOW VISUALIZER button.

Click on the START button to create a new "node". At the top pull down menu, choose "Approval".

|

NAME

 |

Approval

 |
|

DESC

 |

My approval step

 |
|

RUN

 |

Always

 |
|

CONVERGENCE

 |

ANY

 |

-   SAVE

You will have this as the result:

Now click on the Approval node, then click on the BLUE CHAIN button:

![](https://lh3.googleusercontent.com/85jJOF1jKVrKpmHRliXBQmyOeAmW977kAEFLH5JD9zDEOW2-JwstnaHae-k835zgm2nM1H5Oen4QDoRHFY7oJISrIVQMYuykyTJcfSqHZ8KloLhK3PD9qQ3XQhaWnihKOW9d3BAp)

Then click on "Ping a node":

![](https://lh3.googleusercontent.com/IjwLMWlO9tbF0rczBW1p7yWC9Xi1zWEY-PpZPagC0n6gLtOJDgCgIRk87AE9A__3Bn6EzYiKYHBPD70lG539De4YwnOrATeVIVMxxVhBuhAtf4tHdmQVKlyVS68wGIjvY91JvTfI)

Then SAVE.

![](https://lh4.googleusercontent.com/5SRbyDKVC1aA1G6UJ0cGpDOXnhRAqYJELAkbPLcUQGqZnYDw_qyXtNLDAc-VUYasZxZdjluLZpDeEbkiVkSvxBwwO8Hak3XUdyLunjGRvBmUsDbf_uE-xjArAuAf_rTFcpjLcQvK)

Then SAVE again.

After you save the workflow, open a different web browser if you have one on your computer, or simply open a new browser window in incognito mode, and login to Ansible Tower as user webdev1. And launch the workflow, again use node1 as an input to the survey. The workflow would ask the admin user for an approval.

Switch back to the browser where admin is logged in, notice the BELL notification at the top of the screen, open it:

![](https://lh6.googleusercontent.com/ofzw0W0R8UriVO27uf0gS_rx1QAaQr8UowzXfR8tIz3z1uqeFH_Q3MjIQYpGh_uBZIfhtJ3N01nonhsIZObqSIwdWPc8e_sBslID5zK18H1bjV1Aab9y0E9VFsu633-6_an-OVb9)

Go ahead and approve it.

FINAL STEPS

Let's create one more Job Template that let users add a new web server into the load balancer pool.

|

Parameter

 |

Value

 |
|

NAME

 |

Add node to bigIP Pool

 |
|

JOB TYPE

 |

Run

 |
|

INVENTORY

 |

Workshop Inventory

 |
|

PROJECT

 |

Tech Day Project

 |
|

PLAYBOOK

 |

automate-everything/lab-examples/bigip-add-node.yml

 |
|

CREDENTIAL

 |

Workshop Credentials

 |
|

OPTIONS

 |\
 |

-   Click SAVE

Then go back to our workflow and add the above new job template as followed:

![](https://lh5.googleusercontent.com/NIrhL7X7xfkwQqJ2V-TTBrNwdawI8dMLtXqu9JNg4WNMHLN2YrNPiTnSYV0zNsnatVPTPVfq4CPYJ_iLqV5r3ZZpdC0hisd9hP-oSjvtYsrPMoTRFqxvjkbxnbn_qECdnnVSCl0k)

Next, edit the survey we created earlier for our workflow and add one more survey question:

|

PROMPT

 |

Please specify an IP address

 |
|

DESCRIPTION

 |

Make sure the IP is correct!

 |
|

ANSWER VAR NAME

 |

my_ipaddy

 |
|

ANSWER TYPE

 |

Text

 |
|

MIN LENGTH

 |

1

 |
|

MAX LENGTH

 |

16

 |

-   Click +ADD, then click SAVE

Then SAVE the workflow.

As user webdev1, launch the workflow to add node2 to the F5 load balancer pool. Make sure you supply the correct IP address for your node2. You can find this in your lab_invetory file via CLI, or in Ansible Tower. Example:

node2 ansible_host=3.95.15.12 ansible_user=ec2-user private_ip=172.16.14.216

Then verify if node2 has been added to your load balancer via the F5 web interface.

End of the lab..
