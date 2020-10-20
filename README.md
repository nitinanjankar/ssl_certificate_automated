Role Name
=========

This role will ask the user whether to create a SSL certificates for OCP resources or not. And based on input will proceed further and create or skip the creation and implementation of SSL signed certificates.

Requirements
------------

## OS specific:

1. Bastion/Controller/Image Signing host must be created beforehand.
2. OCP 4.x cluster must be deployed.
3. Bastion/Controller/Image Signing host must must have below tools installed on it.
   - Git
   - Ansible 2.9 or above
   - docker

## Tools Required:
1.  AWS Account
2.  RHEL machine(controller node) with Redhat Subscription (7 or higher)
3.  Copy the key which is used to ssh controller node in .ssh folder in controller node.
3.  Git
4.  Ansible 2.9 or above
5.  boto
6.  boto3
7.  python >= 2.7

## Pull the code from GitLab

```
cd /home/ec2-user/
mkdir /home/ec2-user/ssl_certificates 
cd /home/ec2-user/ssl_certificates
git clone <path of git repo>
```

## Update host file

Edit inventory file:
```
cd openshift4/utils/ssl_cert
vi inventories/ansible_hosts

```

| Variable  | Description  | Where Set  |
|---|---|---|
|**master_ip**| master node ip from which you want to take backup | inventories/ansible_hosts |
|**worker_ip**| worker node ip from which you want to take backup | inventories/ansible_hosts |
|**bastion_ip**| master node ip from which you want to take backup | inventories/ansible_hosts |
|**ansible_ssh_user**| ssh user to connect respective hosts | inventories/ansible_hosts |
|**ansible_ssh_private_key_file**| ssh key path which is use to ssh master node | inventories/ansible_hosts |


Group Variables
--------------


Edit variable file

```
vi inventories/group_vars/all

```

| Variable  | Description  | Sample Values (False values are default. Need to change only when required.)  |
|---|---|---|
|**ocp_login_token**| Token value of OCP cluster user's login | "xfagshssdhg3weg7whwh" |
|**ocp_server_url**| OCP cluster URL for CLI login | "http://xxxxx.io:6443" |
|**certificate_path**| Certificate path to copy on remote host |  /var/home/core/ssl_certs |
|**acme_sh_become_user**| The user on the system that acme.sh will run as | "root" |
|**acme_sh_renew_time_in_days**| Certificates that need to be renewed will be consider by acme.sh | 90 |
|**acme_sh_copy_certs_to_path**| The final destination for your certificates | "/etc/ssl/ansible" |
|**acme_sh_default_force_issue**| Only consider using this to update your DNS provider | False |
|**acme_sh_default_force_renew**| This could be useful to use if your certificates expired | False |
|**acme_sh_default_dns_provider**| Which DNS provider should you use | "dns_aws" |
|**acme_sh_default_dns_provider_api_keys**| AWS credentials for instance | "aws_access_key_id", "aws_secret_access_key" |
|**acme_sh_default_dns_sleep**| acme.sh sleep after attempting to set the TXT record to your DNS records | 60 |
|**acme_sh_domains**| This list contains a list of domains | "<< api_domain1 >>", "*.<< apps_domain >>" |

# Where to get acme_sh_domains details?

API to the fully qualified domain name: << api_domain1 >>

```
oc whoami --show-server | cut -f 2 -d ':' | cut -f 3 -d '/' | sed 's/-api././'
```

WILDCARD to your Wildcard Domain: << apps_domain >>

```
oc get ingresscontroller default -n openshift-ingress-operator -o jsonpath='{.status.domain}'
```

# Which DNS provider should you use?

 A list of supported providers can be found at: https://github.com/Neilpang/acme.sh#7-automatic-dns-api-integration
 As for getting the name to use, you can find that at: https://github.com/Neilpang/acme.sh/tree/master/dnsapi

## Set the environment variable for AWS

```
export AWS_ACCESS_KEY_ID="xxxxxxxxxxxxxxxx"
export AWS_SECRET_ACCESS_KEY="yyyyyyyyyyyyyyyyyyyyyyyyyy"
```

## Running ansible code:

Before running the ansible code you must be at <b> openshift4/utils/ssl_cert </b>

```
cd openshift4/utils/ssl_cert

ansible-playbook -i inventories/ansible_hosts deploy-ssl-cert.yml
```


## Enable certificates to apply on *.app.<<domain_name>>

1. Login to AWS web console
2. Go to <b>Certificate Manager</b> service 
3. For your domain, Click on Reimport Certificates and add respective certs generated on << certificate_path >> variable's location.
4. Validate your certificates got reflected in OCP Web Console.