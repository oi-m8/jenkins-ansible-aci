# Jenkins-ansible-aci
This is an ongoing project to provide another way to interact with Cisco's ACI.

The idea is to provide a pre-configured jenkins server to which API calls can be made to push policies to ACI, or jenkins build jobs initiated from the Jenkins GUI.

In production, it is good practice to have jenkins executing build jobs on slave machines rather than on the same host that Jenkins is running on. If the volume of requests is expected to be high, this setup may struggle so it would be advisable to expand into dedicated ansible slaves. 

This project is based off the [jenkins/jenkins image on dockerhub](https://hub.docker.com/r/jenkins/jenkins/).

These build jobs and playbooks were tested on the [Cisco ACI sandbox](https://sandboxapicdc.cisco.com).

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

```
Docker
```

### Installing

1. Ensure docker is running*
```
docker --version
```
*Note: if running docker on Windows; "Linux containers" will have to be enabled

2. Clone this repo
```
git clone https://github.com/ben-clifton/jenkins-ansible-aci.git
```

3. Run the dockerfile to build the image that the container will be run on
```
docker build -t <image name:tag> -f ./jenkins-ansible .
```

4. List all docker images and copy the image id
```
docker image ls
```

5. Start a container based on the newly created image
```
docker run --rm -p <host machine port number>:8080 <image id>
```

6. Start sending API requests to the server.

The following credentials are for the account used in the initial setup of this container, to login to Jenkins and make any required changes please login with the below account:
 * Username: apiuser
 * Password: password

## API endpoints
Below is a list of the API endpoints that represent Jenkins build jobs that have been configured so far.

A user account "apiuser" and api-token have been created and are required for Jenkins to authenticate the request.

Each endpoint will have different parameters to provide with the API call depending on the policy being pushed.

The common parameters that can/should be included with every API call are:
* description (optional) - Policy description
* token - All Jenkins build jobs have been configured with "aci_helper" as the token
* username (required) - Username of ACI account 
* password (required) - Password of ACI account
* state (required) - "present" for creation, "absent" for removal
* apic (required) - hostname or IP of APIC
* output_level (optional) - debug, info or normal

### /Tenant
Query params:
* tenant - Name of the tenant
```
GET
http://apiuser:11af030ccf245b3c36721825ba4ae49365@localhost:<port number>:8080/job/schedule_tenant/buildWithParameters?token=aci_helper&tenant=odysseus&description=test&username=<username>&password=<password>&state=present&apic=<apic IP/hostname>
```
### /VRF
Query params:
* vrf - Name of the VRF to be configured
* tenant - Name of the tenant that the VRF will be configured under
* policy_control_direction - Ingress or egress
* policy_control_preference - enforced unenforced
```
GET
http://apiuser:11af030ccf245b3c36721825ba4ae49365@localhost:<port number>:8080/job/vrf/buildWithParameters?token=aci_helper&tenant=<tenant name>&description=test&username=<username>&password=<password>&state=present&apic=<apic IP/hostname>&policy_control_direction=ingress&vrf=<vrf name>&output_level=info&policy_control_preference=enforced
```
### /Application Profile
Query params:
* ap - Name of the application profile to be configured
* tenant - Name of the tenant in which the application profile will be created
```
GET
http://apiuser:11af030ccf245b3c36721825ba4ae49365@localhost:<port number>:8080/job/application_profile/buildWithParameters?token=aci_helper&description=test&username=<username>&password=<password>&state=present&apic=sandboxapicdc.cisco.com&ap=<ap name>&tenant=<tenant name>&host=<apic IP/hostname>
```

## Built With

* [Jenkins](https://jenkins.io/doc/)
* [Ansible](https://docs.ansible.com/ansible/latest/index.html) 
* [Docker](https://docs.docker.com/)

## Acknowledgments

* To the authors of all the out-of-the-box ansible aci modules
  * Jacob McGill (@jmcgill298) - [ACI tenant](https://docs.ansible.com/ansible/latest/modules/aci_tenant_module.html#aci-tenant-module), [ACI VRF](https://docs.ansible.com/ansible/latest/modules/aci_vrf_module.html#aci-vrf-module)
  * Swetha Chunduri (@schunduri) - [ACI application profile](https://docs.ansible.com/ansible/latest/modules/aci_ap_module.html#aci-ap-module)
* To the authors of https://hub.docker.com/r/jenkins/jenkins/
* To everyone else involved with writing the tools that this project is using

