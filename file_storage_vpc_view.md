---

copyright:
  years: 2022, 2023
lastupdated: "2023-05-30"

keywords:

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Viewing file shares and mount targets
{: #file-storage-view}

View all file shares and mount targets in the UI, CLI, or API. View details of a single file share or mount target.
{: shortdesc}

{{site.data.keyword.filestorage_vpc_full}} is available for customers with special approval to preview this service in the Frankfurt, London, Dallas, Toronto, Washington, Sao Paulo, Sydney, Osaka, and Tokyo regions. Contact your IBM Sales representative if you are interested in getting access.
{: preview}

## View file shares and mount targets in the UI
{: #file-storage-view-shares-targets-ui}
{: ui}

### View all file shares in the UI
{: #fs-view-all-shares-ui}

1. In the [{{site.data.keyword.cloud_notm}} console](/login){: external}, go to **menu icon ![menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > Storage > File Shares**.

2. The File Shares for VPC list page shows all file shares that are created in that zone. Overflow menu options are used to manage the file shares. The following table describes the information and actions on the list page.

| Field | Value |
|-------|-------|
| Region | Account region for the file share. Select a different region to see file shares for zones in that region. |
| Name  | The file share name. It can be the original file share or a replica file share. Click the name to see details about that file share. |
| Status | For a list of statuses for file shares, see [File storage lifecycle states](/docs/vpc?topic=vpc-file-storage-managing#file-storage-vpc-status). |
| Mount targets | Number of mount targets that are associated with the file share. You can have one mount target per VPC per file share. |
| Size | Size of the file share, in GBs. |
| Replication role | Relationship to the source file share. "Replica of" indicates that the file share a replica of the source share, which is linked. "Source of" indicates that the share the source of the replica, which is linked. "None" indicates that replication is not configured for the file share. |
| Encryption type | Shows encryption type of the file share, either provider-managed or customer-managed. [Customer-managed encryption](/docs/vpc?topic=vpc-file-storage-vpc-encryption) uses your own root keys to protect your data. The UI also identifies the key management service (KMS), {{site.data.keyword.keymanagementserviceshort}} or {{site.data.keyword.hscrypto}}. |
| Actions menu| Options for managing the file share, depending on its state. For a file share in a _stable_ state, you can rename the share, create a replica, or delete a file share. **Delete** and **Create replica** are disabled if you set up replication to a replica file share. For more information, see [Creating replica file shares](/docs/vpc?topic=vpc-file-storage-create-replication&interface=ui). |
{: caption="Table 1. File shares list page." caption-side="bottom"}

### View details of a file share in the UI
{: #fs-view-single-share-ui}

1. Go to the list of all file shares. From the [{{site.data.keyword.cloud_notm}} console](/login){: external}, go to **menu icon ![menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > Storage > File Shares**.

2. Click the name of a file share to see the details page.

The editable name and [status](/docs/vpc?topic=vpc-file-storage-managing#file-storage-vpc-status) of the file share is shown. If you applied user or access management tags to the file share, they are listed. Click **Add tags** to apply new [tags](/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-about-fs-tags) to the share.

The following table describes the information on files shares details page.

| Field | Value |
|-------|-------|
| **File share details** | |
| Name  | The file share name. Click the pencil icon to change the name. |
| Zone | Zone for the file share (for example, Dallas 2). |
| Max IOPS | Maximum IOPS for the IOPS tier, custom, or dp2 [profile](/docs/vpc?topic=vpc-file-storage-profiles) associated with the file share. |
| Resource group | Resource groups associated with the file share in your account. |
| Replication role | Source file share or replica. |
| Encryption | Specifies provider-managed or [customer-managed encryption](/docs/vpc?topic=vpc-file-storage-vpc-encryption). |
| Encryption instance | For customer-managed encryption, link to the {{site.data.keyword.keymanagementserviceshort}} or {{site.data.keyword.hscrypto}} instance. |
| Key ID |  Copiable customer root key ID. |
| ID | For customer-managed encryption, the UUID generated when you created the file share. |
| Size | File share size in GB. |
| Created | Date the file share was created. |
| **Profile, size, and IOPS**| |
| Size | File share size in GB. |
| IOPS tier | IOPS tier [profile](/docs/vpc?topic=vpc-file-storage-profiles) defining the file share performance. For example, a 3 IOPS/GB general-purpose profile. |
| Max IOPS | Maximum IOPS for the specified profile. |
| **Mount targets** | Number of mount targets associated with the file share. You can have one mount target per VPC per file share. You can create more mount targets for other VPCs. |
| Name | Name of the mount target. |
| Virtual private cloud | Click the name to go to the details page for that VPC, where you can see a [list of file shares](#fs-view-shares-vpc) that have a mount target to the VPC. |
| Status | Status of the mount target on the VPC. |
| **File share replication relationship** | Shows the replica file share in relationship to the source file share. If you change to the source file share size or profile that is applied, the replica is updated at the next replication interval. \n Click **Remove replication relationship** to break the replication relationship. The replica file share is now an independent read/write file share. If no replica file shares were created, click **Create replica** to [create one](/docs/vpc?topic=vpc-file-storage-create-replication).|
| Replication frequency | Hover over the information icon to see an explanation of the cron replication frequency. |
| Replication role | Source or replica file share. |
| File share Name | Click the file share name to see its details. |
| Status | Replication status; for example, _suspended_ or _available_. |
| Actions menu | Options for managing the file share, depending on its state. Options include add auditing, expand share, edit IOPS profile, perform failover, create replica, and delete share. Expand share, create replica, and delete are disabled if you set up replication to a replica file share. For more information, see [Creating replica file shares](/docs/vpc?topic=vpc-file-storage-create-replication&interface=ui). |
{: caption="Table 2. File shares details page" caption-side="top"}

### View all file shares for a VPC in the UI
{: #fs-view-shares-vpc}

You can see all file shares that have a mount target to a VPC by viewing the VPC details page.

1. Go to a VPC:

    1. From the [file shares details page](#fs-view-single-share-ui), click the VPC link in the list of mount targets.
    2. From the UI, go to **menu icon ![menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > > Network > VPCs**. Click the name of a VPC in the list.

2. On the VPC details page, scroll to **File shares in this VPC**.


## View file share and mount targets from the CLI
{: #file-storage-view-shares-targets-cli}
{: cli}

As of 30 May 2023, you can use `--mount-targets` instead of `--targets` option. To see and use the updated option, set the feature environment variable `IBMCLOUD_IS_FEATURE_FILESHARE_CHANGE_TO_MOUNT_TARGETS` to true.
{: beta}

   ```text
   export IBMCLOUD_IS_FEATURE_FILESHARE_CHANGE_TO_MOUNT_TARGETS=true
   ```
   {: pre}

### View mount targets for a file share from the CLI
{: #fs-view-mount-shares-cli}

Run the `ibmcloud is share-mount-targets` command and specify the file share ID to see all mount targets for a file share.

```sh
ibmcloud is share-mount-targets SHARE_ID [--output JSON] [-q, --quiet]
```
{: pre}


### View all file shares from the CLI
{: #fs-view-all-shares-cli}

Run the `ibmcloud is shares` command to list all file shares in a region.

```sh
ibmcloud is shares [--resource-group-id RESOURCE_GROUP_ID | --resource-group-name RESOURCE_GROUP_NAME | --all-resource-groups] [--output JSON] [-q, --quiet]
```
{: pre}

For example,

```text
ibmcloud is shares
Listing shares in all resource groups and region us-south under account VPC as user myuser@mycompany.com...
ID                                          Name                Lifecycle State   Zone         Profile      Size(GB)   Resource Group
866da826-6f30-444f-b55e-0d697cf8b4bb        new-share           stable            us-south-1   tier-3iops   40         Default
81222eee-b3e0-4dc3-b429-aee9e5c0abf2        new-share-1         stable            us-south-1   tier-3iops   40         Default
```
{: screen}

### View details of a file share from the CLI
{: #fs-share-details-cli}

Run the `ibmcloud is share` command and specify the file share by ID or name. This example identifies the file share by ID.

```text
ibmcloud is share 72332eee-b3e0-4dc3-b429-aee9e5c0abf2
Getting file share 72332eee-b3e0-4dc3-b429-aee9e5c0abf2 under account VPC as user myuser@mycompany.com...

ID                72332eee-b3e0-4dc3-b429-aee9e5c0abf2
Name              new-share-1
CRN               crn:v1:bluemix:public:is:us-south-1:a/7f75c7b025e54bc5635f754b2f888665::share:r006-72332eee-b3e0-4dc3-b429-aee9e5c0abf2
Lifecycle State   stable
Zone              us-south-1
Profile           tier-3iops
Size(GB)          40
IOPS              3000
Encryption        provider_managed
Mount Targets     ID                                          Name           VPC ID                                      VPC Name
                  3c413e2f-2e7e-47e9-a9ad-12d23068d2c4        my-target121   0d37163a-701d-4ad6-9ece-a3e34cf28935        cdl
                  585075b5-b466-4b98-b733-2c5c2d906823        my-target122   dd1c6831-cc4d-4c40-aab2-6b0c17f41424        cli-vpc-1

Resource Group    ID                                 Name
                  bdd96715c2a44f2bb60df4ff14a543f5   Default

Created           2023-05-30T15:21:35+05:30
```
{: screen}

### View a source file share for a replica file share from the CLI
{: #fs-view-source-share}

Run the `ibmcloud is share-replica-source` command and specify the replica share by ID or name.

```sh
ibmcloud is share-replica-source REPLICA-SHARE [--output JSON] [-q, --quiet]
```
{: pre}

For example,

```text
ibmcloud is share-replica-source replica-share-4
Getting source file share for replica file share replica-share-4 under account VPC as user myuser@mycompany.com...

ID                   0e8ee997-bf1d-4165-9573-726c3a4d3b36
Name                 share-4
CRN                  crn:v1:staging:public:is:us-south-1:a/efe5afc483594adaa8325e2b4d1290df::share:0e8ee997-bf1d-4165-9573-726c3a4d3b36
Lifecycle state      stable
Zone                 us-south-1
Profile              tier-5iops
Size(GB)             40
IOPS                 3000
Encryption           provider_managed
Mount targets        ID                          Name   VPC ID   VPC Name
                     No mounted targets found.

Resource group       ID                                 Name
                     11caaa983d9c4beb82690daab08717e9   Default

Created              2023-05-07T18:13:04+05:30
Last sync at         2023-05-06T05:53:28+05:53
Latest job           succeeded
Replication share    ID                                          Name                Resource type
                     0bb9c083-2ac5-43d5-962a-757f69d5e6c8        replica-p-share-4   share

Replication role     source
Replication status   active
Source share         ID   Name   Resource type
```
{: screen}

## View file shares and mount targets with the API
{: #file-storage-view-shares-targets-api}
{: api}

### View all file shares with the API
{: #fs-view-all-shares-api}

Make a `GET /shares` request to list all file shares for a region.

```curl
curl -X GET \
"$vpc_api_endpoint/v1/shares?version=2023-05-30?limit=50&generation=2"\
-H "Authorization: $iam_token"
```
{: pre}

A successful response looks like the following example. In the example, the `limit` query parameter specifies a limit of 50 file shares, all though there is only one in the response.

```json
{
  "first": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/shares?limit=50"
  },
  "limit": 50,
  "shares": [
    {
      "created_at": "2023-05-30T13:02:17Z",
      "crn": "crn:[...]",
      "encryption": "provider_managed",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/51bba578-0dce-4f8a-aa6e-f06c899e2c8e",
      "id": "51bba578-0dce-4f8a-aa6e-f06c899e2c8e",
      "iops": 3000,
      "lifecycle_state": "stable",
      "name": "share-name1",
      "profile": {
        "family": "tiered",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/share/profiles/tier-10iops",
        "name": "tier-10iops",
        "resource_type": "share_profile"
      },
      "resource_group": {
        "crn": "crn:[...]",
        "href": "https://resource-controller.cloud.ibm.com/v2/resource_groups/60fc731a-1794-4f5d-ba51-ea24b5357207",
        "id": "60fc731a-1794-4f5d-ba51-ea24b5357207",
        "name": "Default"
      },
      "resource_type": "share",
      "size": 40,
      "mount_targets": [
        {
          "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/51bba578-0dce-4f8a-aa6e-f06c899e2c8e/mount_targets/d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
          "id": "d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
          "name": "mount-target-name1",
          "resource_type": "share_target",
          "vpc": {
            "crn": "crn:[...]",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/c2d941de-27f5-432c-b4d0-37a8491c3216",
            "id": "c2d941de-27f5-432c-b4d0-37a8491c3216",
            "name": "vpc-name1",
            "resource_type": "vpc"
          }
        }
      ],
      "zone": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
        "name": "us-south-1"
      }
    }
  ]
}
```
{: codeblock}

### View a single file share with the API
{: #fs-single-file-shares-api}

Make a `GET /shares/{share_id}` request to get details about a single file share.

```curl
curl -X GET \
"$vpc_api_endpoint/v1/shares/$share_id?version=2023-05-30&generation=2"\
-H "Authorization: $iam_token"
```
{: pre}

A successful response looks like the following example. In this example, the share was created based on a `dp2` profile.

```json
{
  "created_at": "2023-05-07T22:58:49.000Z",
  "crn": "crn:[...]",
  "encryption": "provider_managed",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/199d78ec-b971-4a5c-a904-8f37ae710c63",
  "id": "199d78ec-b971-4a5c-a904-8f37ae710c63",
  "iops": 14400,
  "lifecycle_state": "stable",
  "mount_targets": [
    {
      "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/199d78ec-b971-4a5c-a904-8f37ae710c63/mount_targets/d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
      "id": "d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
      "name": "my-share-mount-target",
      "resource_type": "share_mount_target",
      "vpc": {
        "crn": "crn:[...]",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/c2d941de-27f5-432c-b4d0-37a8491c3216",
        "id": "c2d941de-27f5-432c-b4d0-37a8491c3216",
        "name": "my-vpc",
        "resource_type": "vpc"
      }
    }
  ],
  "name": "my-share",
  "profile": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/share/profiles/dp2",
    "family": "defined-performance",
    "name": "dp2",
    "resource_type": "share_profile"
  },
  "replication_role": "none",
  "replication_status": "none",
  "replication_status_reasons": [],
  "resource_group": {
    "crn": "crn:[...]",
    "href": "https://us-south.iaas.cloud.ibm.com/v2/resource_groups/678523bcbe2b4eada915d32640909956",
    "id": "678523bcbe2b4eada915d32640909956",
    "name": "Default"
  },
  "resource_type": "share",
  "size": 4800,
  "user_tags": [],
  "zone": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
    "name": "us-south-1",
    "resource_type": "zone"
  }
}
```
{: codeblock}

### List all mount targets of a file share with the API
{: #fs-list-targets-api}

Make a `GET /shares/{share_id}/mount_targets` request to list all mount targets of a file share.

For example,

```curl
curl -X GET \
"$vpc_api_endpoint/v1/shares/$share_id/mount_targets?version=2023-05-30?limit=50&generation=2"\
-H "Authorization: $iam_token"
```
{: pre}

A successful response looks like the following example:

```json
{
  "first": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/199d78ec-b971-4a5c-a904-8f37ae710c63/mount_targets?limit=50"
  },
  "limit": 50,
  "mount_targets": [
    {
      "created_at": "2023-05-30T01:59:46.000Z",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/199d78ec-b971-4a5c-a904-8f37ae710c63/mount_targets/r134-1b5571cb-536d-48d0-8452-81c05c6f7b80",
      "id": "r134-1b5571cb-536d-48d0-8452-81c05c6f7b80",
      "lifecycle_reasons": [],
      "lifecycle_state": "stable",
      "mount_path": "fsf-dal1099a-fz.adn.networklayer.com:/nxg_s_voll_mz0716_a4cc07a3_4425_4adf_aed6_0d7e142bee0c",
      "name": "my-target",
      "resource_type": "share_mount_target",
      "vpc": {
        "crn": "crn:[...]",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/4c0bb0df-5ca2-43ca-a3de-a4f86010a906",
        "id": "4c0bb0df-5ca2-43ca-a3de-a4f86010a906",
        "name": "my-vpc",
        "resource_type": "vpc"
      }
    }
  ],
  "total_count": 1
}
```
{: codeblock}

### View a single mount target with the API
{: #fs-get-target-api}

Make a `GET /shares/{share_id}/mount_targets/{mount_target_id}` request to information of a single mount target of a file share. This call includes mount path information. Use the mount path to attach a file share to an instance.

Fpr example,

```curl
curl -X GET \
"$vpc_api_endpoint/v1/shares/$share_id/mount_targets/$mount_target_id?version=2023-05-30&generation=2"\
-H "Authorization: $iam_token"
```
{: pre}

A successful response looks like the following example.

```json
{
    "created_at": "2023-05-30T01:59:46.000Z",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/shares//199d78ec-b971-4a5c-a904-8f37ae710c63/mount_targets/d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
    "id": "d5fd8173-f519-4ff7-8f63-0ead23ecf1f4",
    "lifecycle_reasons": [],
    "lifecycle_state": "stable",
    "mount_path": "fsf-dal1099a-fz.adn.networklayer.com:/nxg_s_vol_xyz_2891fd0a_64ea_4deb_9ed5_1159e37cb5aa",
    "name": "my-mount-target2",
    "resource_type": "share_mount_target",
    "vpc": {
      "crn": "crn:[...]",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/5821d0c4-a089-4957-b5fa-03b7ac636c15",
      "id": "5821d0c4-a089-4957-b5fa-03b7ac636c15",
      "name": "my-vpc",
      "resource_type": "vpc"
    }
  }
```
{: codeblock}

### View a source file share for a replica file share with the API
{: #fs-get-source-api}

Make a `GET /shares/{replica_id}/source` request and specify the replica share ID to retrieve the source file share details.

```curl
curl -X GET \
"$vpc_api_endpoint/v1/shares/$replica_id/source?version=2023-05-30&generation=2"\
-H "Authorization: $iam_token"\
```
{: pre}

A successful response provides details of the source file share. Notice that the replication role is `source`.

```json
{
    "created_at": "2023-05-30T22:58:49.000Z",
    "crn": "crn:[...]",
    "encryption": "provider_managed",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/207721a9-aff9-4b16-9823-fe68096aeac3",
    "id": "207721a9-aff9-4b16-9823-fe68096aeac3",
    "iops": 14400,
    "lifecycle_state": "stable",
    "mount_targets": [
      {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/shares/207721a9-aff9-4b16-9823-fe68096aeac3/mount_targets/ce244454-0919-45e2-b14b-f4285afcd856",
        "id": "ce244454-0919-45e2-b14b-f4285afcd856",
        "name": "my-share-mount-target",
        "resource_type": "share_mount_target",
        "vpc": {
          "crn": "crn:[...]",
          "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/c8b8fa2d-ccf7-4f42-9d38-df6d123c867d",
          "id": "c8b8fa2d-ccf7-4f42-9d38-df6d123c867d",
          "name": "my-vpc",
          "resource_type": "vpc"
        }
      }
    ],
    "name": "my-share-3",
    "profile": {
      "family": "defined-performance",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/share/profiles/dp2",
      "name": "dp2",
      "resource_type": "share_profile"
    },
    "replication_role": "source",
    "replication_status": "active",
    "replication_status_reasons": [],
    "resource_group": {
      "crn": "crn:[...]",
      "href": "https://resource-controller.cloud.ibm.com/v2/resource_groups/678523bcbe2b4eada915d32640909956",
      "id": "678523bcbe2b4eada915d32640909956",
      "name": "Default"
    },
    "resource_type": "share",
    "size": 4800,
    "user_tags": [],
    "zone": {
      "href": "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
      "name": "us-south-1",
      "resource_type": "zone"
    }
  }
```
{: codeblock}

## Next steps
{: #fs-view-next-steps}

* [Create new file shares and mount targets](/docs/vpc?topic=vpc-file-storage-create).
* [Manage your file shares](/docs/vpc?topic=vpc-file-storage-managing).
