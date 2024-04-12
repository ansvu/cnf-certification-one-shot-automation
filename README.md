# CNF Oneshot Automation Flow
![Oneshot Automaiton Sequence](img/oneshot-automation-seq.png)

[Oneshot-seq-diagram-link](https://kroki.io/mermaid/svg/eNq9VU1v4zYQvedXDFAE3gCW06RIgcrAAo7sOF60iWG3u2eaHlncSKRKUsm6gf97h6Qly5bTY3NIEM2bNzNvPnh5-S6ksDG892yGBfbiXqo0GtvrQ_iSLJe9GHqDAo1hG_xdSPw5ljaLVBrZbYmfbq_gHYzV6gVjWOWMvwxhN9wcYe6uYCCVxQNylVc4TEWex8C1KIySQ0iVtJER_xDPza_lj-Fu-BPTWr1lyNZQMpvB-yEOuYP317gmJFj8YYn-lAN2vd3u8vLC4N8VSo5jwTaaFRdAPyXTVnBRMmlhBMzAXwZ1x3LvLHONaS42mYUlZ7KDSRwmyehL9IpapOIMz9hhxsksmosSc5Kxg5h4FiUlcjugsjJmB1x1c31wuEfMC-4idsxTZ34uUZpMpDbKCBh5pLk-7_Do4z49AEf6mArOrFASTCVsN8lZDU4IHD3PDXBmurAvDrbANTwyC0q6eiFhluVqc-HBmooEvVl9uvntpg-3t7_Qr7u7K2_zo6JISRj1IYlhaYnZ9SCyNJpCbsDDRtHnz_e1tWw6ZOoO3Qd7kiF_AZbn4Lx9uqaxj2IYHVmoCmNw3QSg8ItKAu82NwnWr-7b9pQ7Cdx_nuVFeeAf1wUoiSZTNuI0wYzkCkGeGikIWE8OfFcrE5-0q9Tqu9OUa2QW-1CVa_-XWct4Rjp-nXhp-tTYVSHCGIwpgwlJ5H2gCV2T1d5glfu0rvjBjZSdyVfaxsNyNKoS5WFjyDeI07Azud4nESwaS6U9sJOBp5wEyqDTcV3_4fGFOue23njmslrlwlAlWavOtgbfGKVDxw9uMt33KZKv3rYDOIp6OSyzlWliUXNocVOhi2BWqfu_69cEfDhpO2T1Pnfbyun0doeBNlx7iJuED3q-xjJX23aVIWjA-5hhsg8NDxRg0LpFOzMAXpr5wncr0Dx_e5oslo0W04PuR7Drmjvg3fXGtlPTgL19vjDOe-X6VZQ52v32jMPeiTB8Hywm6Tv2xR-E7YOuFxn2d6SBj13ObggHW1bksEHpxG0FnNbSubsbTv3eA96Eza7dL1XZPT3VXaDehPKmJ-UtMEe6CHuSIFGnyGloFw9D1Y4qTOBetwe920mH2888XWHXhf31_f-uShDMvRVHLPUauNRnMSh6qZrXJ1Jl60FxPI_NmfnghfLzSJ2V7l1wffWus6DLH0xWdPm3dGJeBb55LCspnVdsJxY6cE6ufwEhYfoV)


## cnf-certification-one-shot-automation
This repo purpose is to automate the partners CNF certification that have been done the sanity checking for following steps:  
- Using Preflight script or manual to perform sanity checking and all these test cases that preflight `must be passed!`
**Main Test Case must pass as follow:**
```shellSession
======================================================
Image Name           Test Case                 Status    
------------------------------------------------------
oneshot-demo2-nginx-ubi9   HasLicense                PASSED    
oneshot-demo2-nginx-ubi9   HasUniqueTag              PASSED    
oneshot-demo2-nginx-ubi9   LayerCountAcceptable      PASSED    
oneshot-demo2-nginx-ubi9   HasNoProhibitedPackages   PASSED    
oneshot-demo2-nginx-ubi9   HasRequiredLabel          PASSED    
oneshot-demo2-nginx-ubi9   RunAsNonRoot              PASSED    
oneshot-demo2-nginx-ubi9   HasModifiedFiles          PASSED    
oneshot-demo2-nginx-ubi9   BasedOnUbi                PASSED    
Verdict: PASSED
```
For details of sanity check script using preflight scan, it can be found [here](https://github.com/ansvu/quick_scan_container_images_online_offline)

- Second method to do pre-testing the container images  
To use dci-pipeline with including all the container images without specify or set to true `create_container_project: true`  
```yaml
    preflight_containers_to_certify:
      - container_image: "quay.io/avu0/oneshot-demo1-nginx-ubi9:1-22"
      - container_image: "quay.io/avu0/oneshot-demo2-nginx-ubi9:1-22"
      - container_image: "quay.io/avu0/oneshot-demo3-nginx-ubi9:1-22"
```
Note that when using dci-pipeline to perform the container images sanity checking, it does not enable loglevel to debug by default compare with `quick_scan_container_images_online_offline.sh` since it includes it prints out hasModifiedFile test case failure and convert to excelsheet also. 

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
    - check: v1.0/chart-testing
      type: Mandatory
      outcome: PASS
      reason: Chart tests have passed
    - check: v1.0/has-readme
      type: Mandatory
      outcome: PASS
      reason: Chart has a README
    - check: v1.0/not-contains-crds
      type: Mandatory
      outcome: PASS
      reason: Chart does not contain CRDs
    - check: v1.0/signature-is-valid
      type: Mandatory
      outcome: SKIPPED
      reason: 'Chart is not signed : Signature verification not required'
    - check: v1.0/helm-lint
      type: Mandatory
      outcome: PASS
      reason: Helm lint successful
    - check: v1.1/images-are-certified
      type: Mandatory
      outcome: PASS
      reason: 'Image is Red Hat certified : quay.io/avu0/oneshot-demo2-nginx-ubi9:1-22'
    - check: v1.0/not-contain-csi-objects
      type: Mandatory
      outcome: PASS
      reason: CSI objects do not exist
    - check: v1.0/required-annotations-present
      type: Mandatory
      outcome: PASS
      reason: All required annotations present
    - check: v1.0/contains-test
      type: Mandatory
      outcome: PASS
      reason: Chart test files exist
    - check: v1.0/contains-values-schema
      type: Mandatory
      outcome: PASS
      reason: Values schema file exist
    - check: v1.0/is-helm-v3
      type: Mandatory
      outcome: PASS
      reason: API version is V2, used in Helm 3
    - check: v1.0/contains-values
      type: Mandatory
      outcome: PASS
      reason: Values file exist
    - check: v1.1/has-kubeversion
      type: Mandatory
      outcome: PASS
      reason: Kubernetes version specified
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

## Setup DCI-Pipeline Repository with Hooks
```shellSession
$ ls -1
deprecated_ctl_settings
hooks
inventories
ocp-workload
pipelines
README.md

$ tree .
.
├── deprecated_ctl_settings
│   ├── settings-cmm23.8-tnfv4.5.5.yml
│   └── settings-recertify-cmm23.8-new.yml
├── hooks
│   └── install.yml
├── inventories
│   └── pool
│       ├── cluster1
│       └── cluster1-workload.yml
├── ocp-workload
│   └── hooks
│       ├── install.yml
│       ├── post-run.yml
│       ├── pre-run.yml
│       ├── teardown.yml
│       ├── tests.yml
│       ├── wait_for_container_published-oldway-OK.yml
│       ├── wait_for_container_published.yml
│       └── wait_for_one_container.yml
├── pipelines
│   ├── ansible.cfg
│   ├── ansible.log
│   ├── certify-oneshot-automation-pipeline.yml
│   ├── container-e2e-pipeline.yml
│   ├── container-hooks-pipeline.yml
│   ├── create-helmchart-pipeline.yml
│   ├── create-openshift-cnf-pipeline.yml
│   ├── helmchart-verifier-report-pipeline.yml
│   ├── oneshot-container-pipeline.yml
│   ├── oneshot-helmchart-pipeline.yml
│   ├── test-container-hooks-pipeline.yml
│   ├── test-helmchart-nopr-pipeline.yml
│   └── test-hooks-query-pipeline.yml
└── README.md
```

## Prepare DCI Pipeline Oneshot Container
This `DCI Pipeline` Setting will include `create container certification project`, update mandatory parameters, scan container+submit the results and attach the product-listing to newly created project. 

**pipelines/oneshot-container-pipeline.yml:**
```yaml
---
- name: oneshot-container
  stage: container
  prev_stages: [ocp-upgrade, ocp]
  topic: OCP-4.14
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: ~/avu-dci-pipeline-test/pipelines/ansible.cfg
  ansible_inventory: ~/avu-dci-pipeline-test/inventories/@QUEUE/@RESOURCE-workload.yml
  dci_credentials: ~/.config/dci-pipeline/dci_credentials.yml
  ansible_extravars:
    dci_cache_dir: ~/dci-cache-dir
    dci_config_dir: ~/avu-dci-pipeline-test/ocp-workload
    dci_gits_to_components:
      - ~/avu-dci-pipeline-test
    dci_local_log_dir: ~/upload-errors
    dci_tags: ["oneshot", "preflight", "step1"]
    dci_workarounds: []

    organization_id: 15451045
    page_size: 200
    
    # docker auth and backend access
    partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"
    
    # reduce the job duration
    do_must_gather: false
    check_workload_api: false

    # run preflight check container tests,
    # create cert project, and submit tests results
    preflight_containers_to_certify:
      - container_image: "quay.io/avu0/oneshot-demo2-nginx-ubi9:1-22"
        create_container_project: true
        short_description: "I am doing a full-automation e2e auto-publish for following image oneshot-demo2-nginx-ubi9"
        pyxis_product_lists:
          - "6397a9d2aea4e1694b0fe1c4"

    # update the cert project for container
    cert_settings:
      # container
      auto_publish: true
      build_categories: "Standalone image"
      registry_override_instruct: "These are instructions of how to override settings"
      email_address: "email@example.com"
      application_categories: "Networking"
      os_content_type: "Red Hat Universal Base Image (UBI)"
      privileged: false
      release_category: "Generally Available"
      repository_description: "This is a test repository for one-shot automaiton"

  use_previous_topic: true
  inputs:
    kubeconfig: kubeconfig_path
...
```
## Prepare DCI Pipeline Oneshot Helmchart
This `DCI Pipeline` Setting will include `create helmchart certification project`, update mandatory parameters, and attach the product-listing to newly created project. 

**pipelines/oneshot-helmchart-pipeline.yml:**
```yaml
---
- name: oneshot-helmchart
  stage: workload
  prev_stages: container
  topic: OCP-4.14
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: ~/avu-dci-pipeline-test/pipelines/ansible.cfg
  ansible_skip_tags:
    - hook-post-run
  ansible_inventory: ~/avu-dci-pipeline-test/inventories/@QUEUE/@RESOURCE-workload.yml
  dci_credentials: ~/.config/dci-pipeline/dci_credentials.yml
  ansible_extravars:
    dci_cache_dir: ~/dci-cache-dir
    dci_config_dir: ~/avu-dci-pipeline-test/ocp-workload
    dci_gits_to_components:
      - ~/avu-dci-pipeline-test
    dci_local_log_dir: ~/upload-errors
    dci_tags: ["oneshot", "helmchart", "helmchart-pr", "step2"]
    dci_workarounds: []
  
    # docker auth and backend access
    partner_creds: "/var/lib/dci-openshift-app-agent/demo-auth.json"
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"

    organization_id: 15451045
    page_size: 200
    
    # reduce the job duration
    do_must_gather: false
    check_workload_api: false

    # run helmchart tests and generate report.yaml
    do_chart_verifier: true
    partner_name: "redhat-arkady-test"
    partner_email: "redhat-arkady-test@redhat.com"
    github_token_path: "/var/lib/dci-openshift-app-agent/github-token.txt"
    #sandbox_repository: "ansvu/charts"

    # create helmchart cert project at connect
    helmchart_to_certify:
      - repository: "https://github.com/ansvu/yingoneshotchart"
        short_description: "This is a test for one shot automation together with container+chart+helmchart-pr"
        chart_name: "yingoneshotchart"
        create_helmchart_project: true
        pyxis_product_lists:
          - "6397a9d2aea4e1694b0fe1c4"

    dci_charts:
      - name: yingoneshotchart
        chart_file: https://ansvu.github.io/yingoneshotchart/yingoneshotchart-0.1.5.tgz
        deploy_chart: true
        flags: "-W --helm-install-timeout 5m0s"
        create_pr: true

    # update the cert project for helm chart
    cert_settings:
      # helmchart
      email_address: "avu@redhat.com"
      application_categories: "Networking"
      github_usernames: "ansvu"
      long_description: "This is a long 100+ characters description about this sample chart"
      distribution_method: "undistributed"
      distribution_instructions: "Instruction how to get this helm-chart"

  use_previous_topic: true
  inputs:
    kubeconfig: kubeconfig_path
...
```

## Prepare DCI Pipeline Openshift-cnf E2E Certification Setting
This `DCI Pipeline` Setting will include `create Openshift-cnf certification project` and attach the product-listing to newly created project. Once this `vendor validated` project is created, then the support-case also created automatically. To finalize the `Vendor-valiated` certification, it needs the Backend or someone who access to saleforce BE to approve it.  

**pipelines/create-openshift-cnf-pipeline.yml:**
```yaml
---
- name: create-openshift-cnf
  stage: workload
  topic: OCP-4.14
  ansible_playbook: /usr/share/dci-openshift-app-agent/dci-openshift-app-agent.yml
  ansible_cfg: ~/avu-dci-pipeline-test/pipelines/ansible.cfg
  ansible_inventory: ~/avu-dci-pipeline-test/inventories/@QUEUE/@RESOURCE-workload.yml
  dci_credentials: ~/.config/dci-pipeline/dci_credentials.yml
  ansible_extravars:
    dci_cache_dir: ~/dci-cache-dir
    dci_config_dir: ~/avu-dci-pipeline-test/ocp-workload
    dci_gits_to_components:
      - ~/avu-dci-pipeline-test
    dci_local_log_dir: ~/upload-errors
    dci_tags: ["openshift-cnf", "cnf", "debug", "create"]  
    dci_workarounds: []
  
    #custom settings
    check_for_existing_projects: true
    organization_id: 15451045
    page_size: 200

    do_must_gather: false
    check_workload_api: false
    pyxis_apikey_path: "/var/lib/dci-openshift-app-agent/demo-pyxis-apikey.txt"

    cnf_to_certify:
      - cnf_name: "YingOneShot0.1.5 on OCP4.14"
        create_cnf_project: true
        pyxis_product_lists:
          - "6397a9d2aea4e1694b0fe1c4"

    cert_settings:
      email_address: "email@example.com"          

  use_previous_topic: true
  inputs:
    kubeconfig: kubeconfig_path
...
```

## How To Run One Shot CNF Certification Automation
```shellSession
$ export KUBECONFIG=/var/lib/dci-openshift-app-agent/kubeconfig
$ oc project oneshot
$ KUBECONFIG=$KUBECONFIG dci-pipeline-schedule oneshot-container oneshot-helmchart
```

## Oneshot Automation Successful Post Status
- Container published on catalog  
![Container Published Status on Catalog](img/oneshot-automation-container-published-cat.png)

- DCI Pipeline Stages Status 
![DCI Pipeline Stages Status](img/oneshot-automation-dci-ci.png)

- Chart Released from Charts Repository 
![Chart Released from Charts Repository](img/oneshot-automation-chart-released-pr.png)

- Container and helm chart published from portal 
![Container and helm chart published from portal](img/oneshot-automation-published-portal.png)

- DCI CI Container Jobs
click [Container CI Job](https://www.distributed-ci.io/jobs/8d8fa931-e892-4f38-be73-1fad94ecb6ee)

- DCI CI Helmchart Jobs
click [Helmchart CI Job](https://www.distributed-ci.io/jobs/58b6d4a3-db60-4ad3-962e-d7b22eb5321c)
