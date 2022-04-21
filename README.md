# Sample sock-shop migration using ansible role
Sock-shop application migration from 3.11 to 4.10 using ansible

## Pre-requisites

* OCP3 v3.11 and OCP4 clusters, these would be the _source_ and _destination_ clusters used to migrate workloads
* MTC 1.5.4 installed on source, MTC 1.7.0 installed on destination
* Velero must be deployed on both clusters
* S3 temporary migration storage must be provisioned on AWS for mig-controller to consume

**Note**: This sample includes a migration using aws-s3 bucket, and performs direct, cutover migration.

## Mig-controller config variables

A number of parameters need to be set with the correct environment in order to create the CRs for migration purposes. A [sample](https://github.com/konveyor/mig-e2e/config/mig_controller.yml.example) file is supplied, make a copy of this file `config/mig_controller.yml`, **please make changes as necessary**.

See below for a decription of these paremeters :

| Parameter | Purpose |
| --- | --- |
| `mig_controller_remote_cluster_url` | Endpoint of remote cluster mig-controller will connect to |
| `mig_controller_aws_access_key` | Base64 encoded AWS access key to auth with AWS services |
| `mig_controller_aws_secret_key` | Base64 encoded AWS secret access key to auth with AWS services |
| `mig_controller_sa_token` | Base64 encoded SA token used to authenticate with remote cluster |
| `mig_controller_aws_bucket_name` | Name of the S3 bucket to be used for temporary Migration storage |
| `mig_controller_aws_bucket_region` | Region of S3 bucket to be used for temporary Migration storage |
| `mig_controller_remote_cluster_exposed_registry_path` | Exposed registry path in remote cluster for direct image migration |

## Other considerations

* You must active sessions on the source and destination clusters before attempting to run any tests
* By default, the destination cluster (OCP4) is assumed to be the host for mig-controller

## Running mig-controller sample tests

On the **source** cluster, deploy **all** sample tests

```bash
$ ansible-playbook e2e_mig_sample.yml -e "with_migrate=false"
```

On the **destination** cluster (mig-controller host) , migrate **all** sample tests

```bash
$ ansible-playbook e2e_mig_sample.yml -e "with_deploy=false"
```

The migrations will be tracked to completion, you can also check the status of each _phase of the migration_ using : 

```bash
$ oc describe migmigration robot-shop-mig-1650513983 -n openshift-migration
Name:         robot-shop-mig-1650513983
Namespace:    openshift-migration
Labels:       controller-tools.k8s.io=1.0
              migration.openshift.io/migplan-name=robot-shop-migplan-1650513983
              migration.openshift.io/migration-uid=d17fdd65-717e-4dd4-aab8-b608e925385b
Annotations:  openshift.io/touch: b4c9fd5e-c128-11ec-b354-0a580a800262
API Version:  migration.openshift.io/v1alpha1
Kind:         MigMigration
Metadata:
  Creation Timestamp:  2022-04-21T04:07:35Z
  Generation:          42
  Managed Fields:
    API Version:  migration.openshift.io/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .:
          f:controller-tools.k8s.io:
      f:spec:
        .:
        f:migPlanRef:
          .:
          f:name:
          f:namespace:
        f:quiescePods:
        f:stage:
    Manager:      OpenAPI-Generator
    Operation:    Update
    Time:         2022-04-21T04:07:35Z
    API Version:  migration.openshift.io/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:openshift.io/touch:
        f:labels:
          f:migration.openshift.io/migplan-name:
          f:migration.openshift.io/migration-uid:
        f:ownerReferences:
          .:
          k:{"uid":"3815dea2-0230-454e-8f40-66074239c6dd"}:
      f:status:
        .:
        f:conditions:
        f:itinerary:
        f:observedDigest:
        f:phase:
        f:pipeline:
        f:startTimestamp:
    Manager:    manager
    Operation:  Update
    Time:       2022-04-21T04:07:35Z
  Owner References:
    API Version:     migration.openshift.io/v1alpha1
    Kind:            MigPlan
    Name:            robot-shop-migplan-1650513983
    UID:             3815dea2-0230-454e-8f40-66074239c6dd
  Resource Version:  23520370
  UID:               d17fdd65-717e-4dd4-aab8-b608e925385b
Spec:
  Mig Plan Ref:
    Name:        robot-shop-migplan-1650513983
    Namespace:   openshift-migration
  Quiesce Pods:  true
  Stage:         false
Status:
  Conditions:
    Category:              Advisory
    Durable:               true
    Last Transition Time:  2022-04-21T04:08:31Z
    Message:               The migration has completed successfully.
    Reason:                Completed
    Status:                True
    Type:                  Succeeded
  Itinerary:               Final
  Observed Digest:         1a85178362c75faede780394a10333e42b232993485cc084eba92b37b0e200d8
  Phase:                   Completed
  Pipeline:
    Completed:  2022-04-21T04:07:49Z
    Message:    Completed
    Name:       Prepare
    Started:    2022-04-21T04:07:35Z
    Completed:  2022-04-21T04:08:04Z
    Message:    Completed
    Name:       Backup
    Progress:
      Backup openshift-migration/robot-shop-mig-1650513983-initial-msfhn: 67 out of estimated total of 67 objects backed up (13s)
    Started:    2022-04-21T04:07:49Z
    Completed:  2022-04-21T04:08:20Z
    Message:    Completed
    Name:       StageBackup
    Progress:
      Backup openshift-migration/robot-shop-mig-1650513983-stage-dxsv6: 9 out of estimated total of 9 objects backed up (1s)
    Started:    2022-04-21T04:08:04Z
    Completed:  2022-04-21T04:08:24Z
    Message:    Completed
    Name:       StageRestore
    Progress:
      Restore openshift-migration/robot-shop-mig-1650513983-stage-h48g5: 8 out of estimated total of 8 objects restored (1s)
      All the stage pods are restored, waiting for restore to Complete
    Started:    2022-04-21T04:08:20Z
    Completed:  2022-04-21T04:08:24Z
    Message:    Completed
    Name:       DirectImage
    Progress:
      0 total ImageStreams; 0 running; 0 successful; 0 failed
    Started:    2022-04-21T04:08:24Z
    Completed:  2022-04-21T04:08:31Z
    Message:    Completed
    Name:       Restore
    Progress:
      Restore openshift-migration/robot-shop-mig-1650513983-final-7f62s: 66 out of estimated total of 66 objects restored (5s)
      All the stage pods are restored, waiting for restore to Complete
    Started:        2022-04-21T04:08:24Z
    Completed:      2022-04-21T04:08:31Z
    Message:        Completed
    Name:           Cleanup
    Started:        2022-04-21T04:08:31Z
  Start Timestamp:  2022-04-21T04:07:35Z
Events:
  Type    Reason   Age                 From                     Message
  ----    ------   ----                ----                     -------
  Normal  Running  10m                 migmigration_controller  Step: 2/49
  Normal  Running  10m                 migmigration_controller  Step: 3/49
  Normal  Running  10m (x5 over 10m)   migmigration_controller  Step: 4/49
  Normal  Running  10m                 migmigration_controller  Step: 5/49
  Normal  Running  10m (x2 over 10m)   migmigration_controller  Step: 6/49
  Normal  Running  10m                 migmigration_controller  Step: 7/49
  Normal  Ready    10m (x13 over 10m)  migmigration_controller  The migration is ready.
  Normal  Running  10m                 migmigration_controller  Step: 8/49
```

_**Note:**_ The migration name will be printed during the ansible playbook run
