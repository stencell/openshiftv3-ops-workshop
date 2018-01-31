# OpenShift V3 Workshop Labs

These labs are specific for the Operations team or any OpenShift Admin.

## Lab Exercises 

Setting up Cluster (don't do these)

* [Stand Up EC2 Instances](standing_up_hosts_on_ec2.md)
* [Setup a Non-HA OCP Cluster](setting_up_nonha_ocp_cluster.md)
* [Post Installation](using_ootb_cockpit.md)

Standalone Labs

* [Deploying Metrics](deploying_metrics.md)
* [Deploying Logging](aggr_logging.md)
* [Managing Users Overview](managing_users_overview.md)
* [Assigning Users to Project Roles](assigning_users_to_project_roles.md)
* [Creating Custom Roles](creating_custom_roles.md)
* [Assigning Limit Ranges and Quotas](assigning_limit_ranges_and_quotas.md)
* [Setting up Default Limit Ranges and Quotas for Projects](setting_up_default_limit_ranges_and_quotas_for_projects.md)
* [Limiting Number of Self-Provisioned Projects](limiting_number_of_self-provisioned_projects.md)

## App-Dev Lab Exercises

* [BlueGreen Deployment](https://github.com/RedHatWorkshops/openshiftv3-workshop/blob/master/9_Blue_Green_Deployments.adoc)

## Extended Lab Exercises 

Labs that require additional resources (e.g. storage, servers, services)

* [Adding an LDAP Provider](adding_an_ldap_provider.md)

## CloudForms 4.6

* git clone https://github.com/stencell/openshift-ansible.git
* cd openshift-ansible
* git checkout release-3.7
* git branch
* cd
* ansible-playbook -i <your-inventory-file> /root/openshift-ansible/playbooks/byo/openshift-management/config.yml -e openshift_management_install_management=true -e openshift_management_install_beta=true -e openshift_management_app_template=cfme-template -e openshift_management_storage_class=cloudprovider
