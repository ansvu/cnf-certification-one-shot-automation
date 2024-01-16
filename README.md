# cnf-certification-one-shot-automation
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
- It is recommended consistently check latest version of the DCI app agent package, and upgrade to latest version if it not before to use.
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
