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

### Importer

This system will import all missing object form an organization to the catalog, so it can be used for initial import or to add missing, or manually created, artifacts to the repo, with the following workflows:

* **Import Catalog:** This worflow retrieves all the artifacts at github's organization and creates the corresponding YAML files for each one. It also will import their current state, which will be stored into an AWS S3 bucket. At the end of the import process, all the catalog will be validated.

### Provisioning

This system will provision a yaml that represents a component of the catalog (new or updated) using the Backstage CDKTF App and install its features from the "features" repository. In order for this to work we have the following workflows:

* **Catalog Reconcile - Redux:** This workflow will detect the changed objects (or single object) and update other artifacts which are afected by the modified/created/deleted object, so everything is ready for the provisioner to apply the necesary changes. After this  workflow, every artifact on the catalog may be reconcilied with the changes added. The last step of this workflow will merge the change's branch with the main one, closing the PR.

* **CDKTF Reconcile:** This workflow is launched when a PR is merged. It will load all the artifacts at `PENDING_PROVISION`or `PENDING_DELITION` state and apply all the required changes so they achive the desired state (change properties, create teams, repositories, install features...). This workflow will also scaffold all the necesary structure of the repositories if necesary (sync scaffolding can be runned as a separated workflow too).

### Reconcile

This system is in charge of reading all the YAMLs in the repository, and trying to provision all of them in order to avoid changes in the components that are not expected, and having the YAMLs in the repo representing the real structure of all of them. It is composed of just the following workflow:

* **Reconcile all artifacts**: This workflow has to be manually triggered and it will be automatically triggered by a cron job every day at 00:00 UTC for all the components. It will run a script in the Backstage CDKTF App that will read of the components in the repo (with no `ERROR` state), try to provision them, and in case some of them fail mark them as failed in order to see what went wrong.

### Validator

This system is in charge of making catalog validations, checking all artifacts and their references are valid. It has the following workflows:

* **Validate catalog**: This workflow loads all the yamls of the catalog repository and checks all of them and their references are valid.

### Scaffolder

This system is in charge of keep scaffoldings up to date, using the same package which the provisioning system uses. Composed by one workflow:

* **Sync Scaffoldings:** This workflow, manually tiggered, can apply all the pending scaffoldings on the `catalog/_scaffoldings` directory. Completed scaffoldings will be deleted from the folder.
