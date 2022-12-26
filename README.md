# Catalog 
  
This is the catalog repository for the Firestarter Backstage Application. This catalog representes all the information of the components seen in the organization, as well as all the workflows in order for the Firestarter system to work.

The systems that are represented here are the followings:

* Provision
* Reconcile
* PR-Verify
* Component Update

## Structure

This repository has all the componentes stored in the " catalog/* " path, where the `catalog-info.yaml` files are saved. Under the .github folder, all the workflows are stored. 

The definition of all the system is the following.

## Workflows

### Provisioning

This system will provision a yaml that represents a component of the catalog using the Backstage CDKTF App and install its features from the "features" repository. In order for this to work we have the following workflows:

* repo-merge: Thanks to backstage we can create a yaml that represents exactly how we want our component to behave and create a pull-request to this repository, including scaffolding if needed, with the automerge label. This label will trigger this workflow that will check the changes in the repo (normally, a change in a yaml) a will call the cdktf provision wokflow that is in charge o provisioning the yaml. After that workflow finishes, will try to automerge the branch and delete it.

* cdktf-provision: This workflow will be triggered by the repo-merge. In here, we have a required input that is the yaml that has changed in the PR and that has been calculated in previous workflow. With this information, the wokflow will try to install the features of the yalm (if they exist) and try to provision the structure of the YAML using the Backstage CDKTF App. After that, it will try to push the scaffolded files in case they exist and modify the state of the component from PROVISION to PROVISIONED or ERROR; modifying the PR and finishing the workflow.

### Reconcile

This system is in charge of reading all the YAMLs in the repository, and trying to provision all of them in order to avoid changes in the components that are not expected, and having the YAMLs in the repo representing the real structure of all of them. It is composed of just the following workflow:

* reconcile: This workflow has to be manually triggered specifying the type of the component and/or the name of it. Or it will be automatically triggered by a cron job every day at 00:00 for all the components. It will run a script in the Backstage CDKTF App that will read of the components in the repo, try to provision them, and in case some of them fail mark them as failed in order to see what went wrong.


### PR-Verify

This workflow will avoid changes in fields of the YAML that can not be changed. Every component in the catalog is represented by a json-schema, and with that there are some fields that are expected to not be able to be modified (like the field "kind" in the component). This workflow will check wheter or not a read only field has been modified depending on the type of component passed. And, in that case, will fail the PR-Verify, making it impossible to provision that change. The workflow is the following:

* pr-verify: This workflow will checkout the repo, and check the changes in the PR. In case there are changes, it will run a script in the Backstage CDKTF App that will check what fields has changed since the previous version of the file, and in case they are read only, the workflow will fail.

## Update

This simple workflow will try to provision a change in a YAML after it has been modified from the repo without the use of the Automerge label. This is specially usefull when we are modifying an existing YAML and not creating a new one. How it works is very similar to the Provision system. The workflow file is:

* merge-pr: This workflow will be triggered only when a Pull Request is closed AND the label of automerge is NOT present. So, in this case we know that we are modifying an existing YAML and not creating a new one, and we can assume that this PR has been closed after a succesful PR-Verify run. So the workflow will try to provision again the existing YAML and saving the changes to the Terraform state.
