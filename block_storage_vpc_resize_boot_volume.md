---

copyright:
  years: 2022
lastupdated: "2022-11-22"

keywords: block storage, boot volume, data volume, volume, data storage, virtual server instance, instance, expandable volume

subcollection: vpc


---

{{site.data.keyword.attribute-definition-list}}

# Increasing boot volume capacity
{: #resize-boot-volumes}

For boot volumes that are attached to an instance, you can increase the size of the boot volume during or after instance provisioning by using the UI, CLI, or API.
{: shortdesc}

## Increasing boot volume capacity concepts
{: #resize-boot-overview}

Boot volume capacity can be increased during instance provisioning or later, by directly updating the boot volume. This feature applies to instances created from stock and custom images. You can also specify a larger boot volume capacity when you create an instance template.

When you [provision an instance](#resize-boot-vol-new-instance-ui), you can expand the boot volume from its minimum size up to the maximum capacity of 250 GB.

For [existing instances](#resize-boot-vol-list-ui), you can modify the boot volume size and increase capacity up to 250 GB, depending on what the image allows. In this case, you select a boot volume from the list of block storage volumes to modify the capacity.

After you expand boot volume capacity, there's an extra step to get your OS to recognize the capacity increase. You must independently increase the size of the disk, grow the disk partition, and then increase the file system into the partition. For more information, see [Modifying a Linux OS for expanding boot volumes](/docs/vpc?topic=vpc-modifying-the-linux-os-expanded-boot-volume).

## Increase boot volume capacity in the UI
{: #resize-vpc-boot-volumes-ui}
{: ui}

Increase boot volume capacity for new or existing instances in the UI. For existing instances, you can increase the boot volume capacity by selecting a boot volume from the list of block storage volumes.

### Increase boot volume capacity during instance provisioning in the UI
{: #resize-boot-vol-new-instance-ui}

When you create new instance from either a stock or custom image, you can increase the size of the boot volume. For example, a stock image would show 100 GB by default. You can modify the size up to 250 GB. For more information about creating virtual server instances, see [Creating virtual server instances in the UI](/docs/vpc?topic=vpc-creating-virtual-servers&interface=ui). In that topic, the information for boot volume in Table 1 explains this process.

You can also specify a larger boot volume capacity when you create an instance template. For more information, see [Creating an instance template](/docs/vpc?topic=vpc-creating-auto-scale-instance-group&interface=ui#creating-instance-template).

### Increase boot volume capacity from the list of block storage volumes in the UI
{: #resize-boot-vol-list-ui}

For an existing instance, you can increase its boot volume capacity by selecting it from the list of block storage volumes.

1. Navigate to the list of block storage volumes. In the [{{site.data.keyword.cloud_notm}} console](/login){: external}, go to **menu icon ![menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > Storage > Block storage volumes**.

2. Select a boot volume from the list of volumes. The attachment type is _boot_.

3. In the boot volume details, click the **Size** pencil icon. Alternatively, select **Expand volume** from the Actions menu.

4. In the side panel, increase the boot volume size in the **Create size** field. The size must be more than the current size up to 250 GB.

5. Click **Expand boot volume size**.

## Increase boot volume capacity from the CLI
{: #expand-boot-vols-cli}
{: cli}

### Increase boot volume capacity when creating an instance from the CLI
{: #expand-new-boot-vols-cli}

Run the `ibmcloud is instance-create` command and specify a boot volume capacity in GBs.

This example creates an instance with a boot volume of 190 GB.

```zsh
ibmcloud is instance create vsi-1 vpc-1 us-south-1 bx2-2x8  subnet-1 --image ibm-ubuntu-20-04-3-minimal-amd64-1 --boot-volume '{"name": "my-boot-vol-1", "volume": {"capacity": 190, "profile": {"name": "general-purpose"}}}'

Creating instance cli-vsi-1 under account VPC1 as user myuser@mycompany.com...

ID                                    0716_84f99419-554d-4c05-bea0-7034d1c40ed3
Name                                  vsi-1
CRN                                   crn:v1:staging:public:is:us-south-1:a/efe5afc483594adaa8325e2b4d1290df::instance:0716_84f99419-554d-4c05-bea0-7034d1c40ed3
Status                                pending
Availability policy on host failure   restart
Startable                             true
Profile                               bx2-2x8
Architecture                          amd64
vCPU Manufacturer                     Intel
vCPUs                                 2
Memory(GiB)                           8
Bandwidth(Mbps)                       4000
Volume bandwidth(Mbps)                1000
Network bandwidth(Mbps)               3000
Metadata service enabled              false
Image                                 ID                                          Name
                                      9f6b534b-6061-40f4-ac42-5aba4dd0da7f         ubuntu-20-04-3-minimal-amd64-1

VPC                                   ID                                          Name
                                      r134-42ebadb6-65f8-4b2f-923b-50b0e44670df   vpc-1

Zone                                  us-south-1
Resource group                        ID                                 Name
                                      11caaa983d9c4beb82690daab08717e9   Default

Created                               2022-02-24T16:43:47+05:30
Boot volume                           ID   Name           Attachment ID                               Attachment name
                                      -    PROVISIONING   0716-ee0ca315-7a21-42e2-99f7-b68377bbffe0   my-boot-vol1
```
{: screen}

You can also specify a larger boot volume capacity when you create an instance template from an image or snapshot. For example,

```sh
ibmcloud is instance template create tpl-1 vpc-1 us-south-1 bx2-2x8  cli-subnet-1 --image ubuntu-20-04-3-minimal-amd64-1 --boot-volume '{"name": "my-boot-vol1", "volume": {"capacity": 190, "profile": {"name": "general-purpose"}}}'
```
{: screen}

For more information about creating virtual server instances from the CLI, see [Creating virtual server instances from the CLI](/docs/vpc?topic=vpc-creating-virtual-servers&interface=cli). For more information about the commands that are used for increasing boot volume size, see the [VPC CLI reference](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-referenc) 

### Increase capacity of an existing boot volume from the CLI
{: #expand-existing-boot-vol-cli}

From the CLI, locate the boot volume that you want to expand. You can use the `ibmcloud is volumes` command filter the results by specifying the resource group. Also, if you know the name or ID of the instance, you can view instance details and get information about the boot volume.

After you located the volume, use the `volume-update` command and provide the ID or name of the boot volume. Use the `--capacity` parameter to indicate the new size of the boot volume in GBs.

For example, this example increases the capacity of my-boot-vol1 to 200 GB. The existing capacity displays as the boot volume capacity is being expanded. Run an `ibmcloud is volume` command and specify the volume name to see the new capacity.

```zsh
ibmcloud is volume update my-boot-vol-1 --capacity 200
Updating volume my-boot-vol1 under account VPC1 as user myuser@mycompany.com...

ID                                     9d60ba27-170c-4e2a-9bf6-6dbb11f95c38
Name                                   my-boot-vol1
CRN                                    crn:v1:staging:public:is:us-south-1:a/efe5afc483594adaa8325e2b4d1290df::volume:9d60ba27-170c-4e2a-9bf6-6dbb11f95c38
Status                                 updating
Capacity                               190
IOPS                                   3000
Bandwidth(Mbps)                        393
Profile                                general-purpose
Encryption key                         -
Encryption                             provider_managed
Resource group                         Default
Created                                2022-02-24T16:43:47+05:30
Zone                                   us-south-1
Volume Attachment Instance Reference   Attachment type   Instance ID                                 Instance name   Auto delete   Attachment ID                               Attachment name
                                       boot              0716_84f99419-554d-4c05-bea0-7034d1c40ed3   vsi-1           true          0716-ee0ca315-7a21-42e2-99f7-b68377bbffe0   boot-vol-name

Operating system                       ubuntu-20-04-amd64
Source image                           ID                                          Name
                                       9f6b534b-6061-40f4-ac42-5aba4dd0da7f        ubuntu-20-04-3-minimal-amd64-1

Active                                 true
Busy                                   false
Tags                                   -
```
{: screen}

## Increase boot volume capacity with the API
{: #increase-vpc-volumes-api}
{: api}

### Increase boot volume capacity when you create an instance with the API
{: #expand-new-boot-vol-api}

When you create an instance by making a `POST \instances` request, you can specify larger boot volume capacity for any of these contexts: when you create the instance from an image, a source boot volume, or an instance template. Specify a boot volume name and capacity in the `boot-volume-attachment` property. The capacity for the boot volume must be at least the image's minimum provisioned size, which is the default if you don't specify capacity.

For example, when you create an instance from an image, you can make a call like this example to specify a boot volume with a capacity of 250 GB.

```curl
curl -X POST "$vpc_api_endpoint/v1/instances?version=2022-02-01&generation=2"\
-H "Authorization: $iam_token"\
-d '{
      "boot_volume_attachment": {
         "volume": {
           "capacity": 250",
           "encryption_key": {
             "crn": "crn:[...]"
           },
           "name": "my-boot-volume",
           "profile": {
            "name": "general-purpose"
           }
         }
       },
      "image": {
          "id": "9aaf3bcb-dcd7-4de7-bb60-24e39ff9d366"
       },
       .
       .
       .
   }'
```
{: codeblock}

For more information, see [Create an instance](/apidocs/vpc#create-instance) in the VPC API reference.

### Increase capacity of an existing boot volume with the API
{: #expand-existing-boot-vol-api}

With the API, locate the boot volume that you want to expand by making a `GET \volumes` call. Then, make a `PATCH \volumes` call with the ID of the boot volume and specify a new value for capacity.

For example, this call increases capacity of a boot volume to 250 GB.

```curl
curl -X PATCH "$vpc_api_endpoint/v1/volumes/$volume_id/?version=2022-02-12&generation=2"\
-H "Authorization: $iam_token" \
-d '{
      "capacity": 250,
   }'
```
{: codeblock}

## Next steps
{: #next-steps-resize-boot-vols}

Create more volumes or manage your existing block storage volumes.

* [Creating block storage volumes](/docs/vpc?topic=vpc-creating-block-storage).
* [Managing block storage volumes](/docs/vpc?topic=vpc-managing-block-storage).

Optionally, [increase the capacity of your data volumes](/docs/vpc?topic=vpc-expanding-block-storage-volumes) that are attached to a virtual server instance.
