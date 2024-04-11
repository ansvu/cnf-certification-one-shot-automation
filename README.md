# CNF Oneshot Automation Flow
![Oneshot Automaiton Sequence](img/oneshot-automation-seq.png)

## cnf-certification-one-shot-automation
This repo purpose is to automate the partners CNF certification that have been done the sanity checking for following steps:
- Using Preflight script or manual to perform sanity checking and all these test cases that preflight is testing and `it Must be passed!` and `its verdict passed` also.  
**Main Test Case must pass as follow:**
```shellSession
======================================================
Image Name           Test Case                 Status    
------------------------------------------------------
universal-smf-ava   HasLicense                PASSED    
universal-smf-ava   HasUniqueTag              PASSED    
universal-smf-ava   LayerCountAcceptable      PASSED    
universal-smf-ava   HasNoProhibitedPackages   PASSED    
universal-smf-ava   HasRequiredLabel          PASSED    
universal-smf-ava   RunAsNonRoot              PASSED    
universal-smf-ava   HasModifiedFiles          PASSED    
universal-smf-ava   BasedOnUbi                PASSED    
Verdict: PASSED
```
For details of sanity check script using preflight scan, it can be found [here](https://github.com/ansvu/quick_scan_container_images_online_offline)

<details>
<summary>Preflight All Test Cases</summary>

```shellSession
  
[Container Policy]: invoked on container images
- HasLicense
- HasUniqueTag
- LayerCountAcceptable
- HasNoProhibitedPackages
- HasRequiredLabel
- RunAsNonRoot
- HasModifiedFiles
- BasedOnUbi

[Container Root Exception Policy]: automatically applied for container images if preflight determines a root exception flag has been added to your Red Hat Connect project
- HasLicense
- HasUniqueTag
- LayerCountAcceptable
- HasNoProhibitedPackages
- HasRequiredLabel
- HasModifiedFiles
- BasedOnUbi

[Container Scratch Exception Policy]: automatically applied for container checks if preflight determines a scratch exception flag has been added to your Red Hat Connect project
- HasLicense
- HasUniqueTag
- LayerCountAcceptable
- HasRequiredLabel
- RunAsNonRoot
```

</details>

- Using chart-verifier either use dci to run or manually to generate report.yaml
<details>
<summary>Chart Verifier Test Checking</summary>

```shellSession
    - check: v1.1/images-are-certified
      type: Mandatory
      outcome: PASS
      reason: 'Image is Red Hat certified : registry.access.redhat.com/ubi8/nginx-118:1-42'
    - check: v1.0/signature-is-valid
      type: Mandatory
      outcome: SKIPPED
      reason: 'Chart is not signed : Signature verification not required'
    - check: v1.0/contains-test
      type: Mandatory
      outcome: PASS
      reason: Chart test files exist
    - check: v1.1/has-kubeversion
      type: Mandatory
      outcome: PASS
      reason: Kubernetes version specified
    - check: v1.0/chart-testing
      type: Mandatory
      outcome: PASS
      reason: Chart tests have passed
    - check: v1.0/has-readme
      type: Mandatory
      outcome: PASS
      reason: Chart has a README
    - check: v1.0/contains-values-schema
      type: Mandatory
      outcome: PASS
      reason: Values schema file exist
    - check: v1.0/is-helm-v3
      type: Mandatory
      outcome: PASS
      reason: API version is V2, used in Helm 3
    - check: v1.0/required-annotations-present
      type: Mandatory
      outcome: PASS
      reason: All required annotations present
    - check: v1.0/contains-values
      type: Mandatory
      outcome: PASS
      reason: Values file exist
    - check: v1.0/helm-lint
      type: Mandatory
      outcome: PASS
      reason: Helm lint successful
    - check: v1.0/not-contain-csi-objects
      type: Mandatory
      outcome: PASS
      reason: CSI objects do not exist
    - check: v1.0/not-contains-crds
      type: Mandatory
      outcome: PASS
      reason: Chart does not contain CRDs
```
</details>

For more details of `chart-verifier` test cases, please click [chart-verifier-test-cases-details](https://github.com/redhat-certification/chart-verifier/blob/main/docs/helm-chart-checks.md)

## Prerequisites

- [DCI](https://blog.distributed-ci.io/introduction-to-the-red-hat-distributed-ci.html) is Red Hat Distributed CI, written in Ansible.
- [Preflight](https://github.com/redhat-openshift-ecosystem/openshift-preflight) is a command-line interface for validating if OpenShift operator bundles and containers meet minimum requirements for Red Hat OpenShift Certification.
- Set up a jumphost with internet access, install the dci-openshift-appagent, detailed guide can be found in this link [dci-openshift-app-agent-install](https://doc.distributed-ci.io/dci-openshift-app-agent/)
- Install dci-pipeline  
  [DCI-PIPELINE](https://github.com/redhat-cip/dci-pipeline) 
- It is recommended consistently check latest version of the DCI app agent package, and upgrade to latest version before using.
```bash
$ sudo dnf upgrade --refresh --repo dci -y
```
- DCI Control server credential
  [create remote-ci credentials](https://www.distributed-ci.io/remotecis)
  Click on `credentials.yaml` then save contents to a path where `su - dci-openshift-app-agent` user is or you can save any path you legit. 
  This file will be used by the dci-pipeline parameter `dci_credentials: /etc/dci-pipeline/dci_credentials.yml`
- Again container images sanity check with preflight and CNF helmchart `report.yaml` must be green and passed all the tests before `one-shot-cnf-certification-automation` to work
- Create github token to be used for helmchart PR  
  `github_token_path: "/opt/cache/dcicertbot-token.txt"`  
  [how-to-create-github-token](https://github.com/redhatci/ansible-collection-redhatci-ocp/blob/main/roles/create_certification_project/README.md#github-token)
- Create Pyxis Apikey    
A token to access specific partner Pyxis catalog data using REST API. [Create Pyxis API Key](https://connect.redhat.com/account/api-keys) 
- Kubeconfig   
A kubeconfig that can access the OCP cluster 
- Product-listing ID  
Before a container or helm chart/operator can be publicly listed into RedHat catalog, a Product-Listing must be created, it only need to create once according to CNF type.
Follow this link to [Create-Product-Listing](https://connect.redhat.com/manage/products)  
- Organization ID  
Mandatory when using create_container_project. Company ID will be used for the verification of container certification project Organization-ID Company-Profile.
![Get Redhat OrgID](img/redhat-org-id.png)

## Prepare DCI Pipeline Container E2E Certification Setting
This `DCI Pipeline` Setting will include `create container certification project`, update mandatory parameters, scan container+submit the results and attach the product-listing to newly created project. 

**dci-pipeline-settings-container-e2e-certification.yaml:**
```yaml
---
- name: Certification-Container-E2E
  stage: certifycontainer
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: /usr/share/dci-openshift-app-agent/ansible.cfg
  ansible_inventory: /etc/dci-openshift-app-agent/hosts.yml
  dci_credentials: /etc/dci-pipeline/dci_credentials.yml
  ansible_extravars:
    dci_name: create,update,attach,scan container certification project using DCI Pipeline
    dci_configuration: Using DCI Pipeline to create container certification project  
    dci_tags: ["debug", "certification-container"]
    dci_cache_dir: /var/lib/dci-pipeline
    dci_config_dirs: [/etc/dci-openshift-agent]
    dci_workarounds: []
    partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
    check_for_existing_projects: true
    organization_id: 1111111
    do_must_gather: false
    preflight_run_health_check: false
    check_workload_api: false
    page_size: 400 #lab has a lot of archive projects so it needs to define page_size higher than partner account
    #preflight_container_wait: 0 # value in minute if waiting for backend to publish your images to catalog due to the delay.
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"
    preflight_containers_to_certify:
      - container_image: "quay.io/avu0/auto-publish-ubi8-nginx-demo7:v120"
        create_container_project: true
        short_description: "I am doing a full-automation e2e auto-publish for following image auto-publish-ubi8-nginx-demo7"
    
    cert_settings:
       auto_publish: false
       build_categories: "Standalone image"
       registry_override_instruct: "<p>This is an instruction how to get the image link.</p>"
       email_address: "me@redhat.com"
       application_categories: "Networking"
       os_content_type: "Red Hat Universal Base Image (UBI)"
       privileged: false
       release_category: "Generally Available"
       repository_description: "This is a test for Demo how to automate to create project,SCAN and update settings"
    
    cert_listings:
      published: false
      type: "container stack"
      pyxis_product_list_identifier: "222222222222222222222222"
      attach_product_listing: true

  components: []
  inputs:
    kubeconfig: /var/lib/dci-openshift-app-agent/kubeconfig
```
## Prepare DCI Pipeline Helmchart E2E Certification Settings
This `DCI Pipeline` Setting will include `create helmchart certification project`, update mandatory parameters, and attach the product-listing to newly created project. 

**dci-pipeline-settings-helmchart-e2e-cert-chartverifier-pr.yaml:**
```yaml
---
- name: Certification-Helmchart-PR
  stage: helmchartcreation
  prev_stages: certifycontainer
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: /usr/share/dci-openshift-app-agent/ansible.cfg
  ansible_inventory: /etc/dci-openshift-app-agent/hosts.yml
  dci_credentials: /etc/dci-pipeline/dci_credentials.yml
  topic: OCP-4.13
  ansible_extravars:
    dci_name: create,update,attach,helmchart certification project using DCI Pipeline
    dci_configuration: Using DCI Pipeline to create helmchart certification project and PR  
    dci_tags: ["debug", "certification-helmcart"]
    dci_cache_dir: /var/lib/dci-pipeline
    dci_config_dirs: [/etc/dci-openshift-agent]
    dci_workarounds: []
    partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
    check_for_existing_projects: true
    organization_id: 11111111
    do_must_gather: false
    check_workload_api: false
    page_size: 400
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"
      helmchart_to_certify:
      - repository: "https://github.com/ansvu/finalchart"
        short_description: "This is a short description finalchart"
        chart_name: "finalchart"
        create_helmchart_project: true

      cert_settings:
        email_address: "avu@redhat.com"
        distribution_method: "undistributed"
        github_usernames: "ansvu"
        application_categories: "Networking"
        long_description: "This is a long description about this final chart"
        distribution_instructions: "You must be present to get this helm-chart!"

      cert_listings:
        attach_product_listing: true
        published: false
        type: "container stack"
        pyxis_product_list_identifier: "2222222222222222222222222"

  components: []
  inputs:
    kubeconfig: "/var/lib/dci-openshift-app-agent/kubeconfig"
...

- name: Certification-Helmchart-PR
  stage: helmchartpr
  prev_stages: helmchartcreation
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: /usr/share/dci-openshift-app-agent/ansible.cfg
  ansible_inventory: /etc/dci-openshift-app-agent/hosts.yml
  dci_credentials: /etc/dci-pipeline/dci_credentials.yml
  topic: OCP-4.13
  ansible_extravars:
    dci_name: test helmchart PR on github using DCI Pipeline
    dci_configuration: Testing helmpchart PR with pipeline 
    dci_tags: ["debug", "certification-helmchart-pr"]
    dci_cache_dir: /var/lib/dci-pipeline
    dci_config_dirs: [/etc/dci-openshift-agent]
    dci_workarounds: []
    do_must_gather: false
    check_workload_api: false
    do_chart_verifier: true
    partner_name: "redhat-arkady-test"
    partner_email: "redhat-arkady-test@redhat.com"
    github_token_path: "/var/lib/dci-openshift-app-agent/github-token.txt"
    dci_charts:
      - name: finalchart
        chart_file: https://ansvu.github.io/finalchart/finalchart-0.1.4.tgz
        deploy_chart: true
        create_pr: true

  components: []
  inputs:
    kubeconfig: /var/lib/dci-openshift-app-agent/kubeconfig
...
```

## Prepare DCI Pipeline Openshift-cnf E2E Certification Setting
This `DCI Pipeline` Setting will include `create Openshift-cnf certification project` and attach the product-listing to newly created project. Once this `vendor validated` project is created, then the support-case also created automatically. To finalize the `Vendor-valiated` certification, it needs the Backend or someone who access to saleforce BE to approve it.  

**dci-pipeline-settings-openshift-cnf-e2e-cert.yaml:**
```yaml
---
- name: Openshift-CNF-Project-Creation Using Pipeline
  stage: openshiftcnf
  prev_stages: helmchartpr
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: /usr/share/dci-openshift-app-agent/ansible.cfg
  ansible_inventory: /etc/dci-openshift-app-agent/hosts.yml
  dci_credentials: /etc/dci-pipeline/dci_credentials.yml
  topic: OCP-4.13
  ansible_extravars:
    dci_name: create openshift-cnf project using DCI Pipeline
    dci_configuration: Using DCI Pipeline to create openshift-cnf project  
    dci_tags: ["debug", "openshift-cnf"]
    dci_cache_dir: /var/lib/dci-pipeline
    dci_config_dirs: [/etc/dci-openshift-agent]
    dci_workarounds: []
    partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
    check_for_existing_projects: true
    organization_id: 11111111
    do_must_gather: false
    check_workload_api: false
    page_size: 400
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"
    cnf_to_certify:
      - cnf_name: "DemoCNF23.8 on OCP4.14"
        create_cnf_project: true

    cert_listings:
      email_address: "me@redhat.com"
      published: true
      type: "container stack"
      pyxis_product_list_identifier: "22222222222222222222222"
      attach_product_listing: true

  components: []
  inputs:
    kubeconfig: /var/lib/dci-openshift-app-agent/kubeconfig
```

## How To Run One Shot CNF Certification Automation
```shellSession
$ dci-pipeline dci-pipeline-settings-helmchart-e2e-cert-chartverifier-pr.yaml dci-pipeline-settings-helmchart-e2e-cert-chartverifier-pr.yaml dci-pipeline-settings-openshift-cnf-e2e-cert.yaml
```
