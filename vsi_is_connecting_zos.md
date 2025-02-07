---

copyright:
  years: 2022

lastupdated: "2022-09-23"

keywords: connecting, zos, s390x, zosmf, virtual server instance

subcollection: vpc


---

{{site.data.keyword.attribute-definition-list}}

<!-- DO NOT MOVE WAZI AAS TO PUBLISH-->

# Connecting to z/OS virtual server instances
{: #vsi_is_connecting_zos}

A z/OS virtual server instance is typically a private backend infrastructure component that must never be directly accessed from the outside world. Even environments that are used for development, testing, or demonstration must follow good practices for logical isolation.
{: shortdesc}

You can refer to the solution tutorial [Public front end and private backend in a Virtual Private Cloud](/docs/vpc?topic=solution-tutorials-vpc-public-app-private-backend) to learn how to use subnets to isolate and control traffic to backend infrastructure such as a z/OS virtual server instance.

Additionally, good practices be can be used to securely connect to a VPC subnet by containing a z/OS virtual server instance.

*  [Securely access remote instances with a bastion host](/docs/vpc?topic=solution-tutorials-vpc-secure-management-bastion-server)
*  [Using client-to-site VPN servers (Beta)](/docs/vpc?topic=vpc-vpn-client-to-site-overview)

After you connected to the VPC network by using the client-to-site VPN server, you can access the z/OS virtual server instance by using the private IP address.

## Before you begin
{: #vsi_is_connecting_zos_prereq}

Make sure that you locate the private IP address before you connect to the z/OS instances. Private IP addresses are IP addresses that are provided by the system and are only reachable within the VPC network. You can find the private IP address of your z/OS virtual server instance under the *Reserved IP address* column on the console after the instance is created successfully.

## Step 1. Configuring the password
{: #configure-password}

Before you can log in to a z/OS virtual server instance, you must change the password for the default user ID `ibmuser`.

If you are using your z/OS Wazi aaS custom images, you do not need to configure the password. You can safely ignore this step and then connect to the z/OS virtual server instance instead. 
{: note}

1. Log in to the z/OS UNIX shell environment by using your SSH private key and the default user ID. Note that `vsi ip address` in the following code snippets represent the private IP address of your z/OS virtual server instance.

   ```sh
   ssh -i <path to your private key file> ibmuser@<vsi ip address>
   ```
   {: pre}

   You receive a response similar to the following example. When prompted to continue connecting, type `yes`.
   
   ```json
   The authenticity of host 'xxx.xxx.xxx.xxx (xxx.xxx.xxx.xxx)' can't be established.
   ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxx.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added 'xxx.xxx.xxx.xxx' (ECDSA) to the list of known hosts.
   ```
   {: screen}

2. Use the `tsocmd` shell command to configure your own password for the z/OS instance. For example, the new password is `system12`.

   ```sh
   tsocmd 'ALTUSER IBMUSER PASSWORD(system12) NOEXPIRE RESUME'
   ```
   {: pre}

   You must follow the password policies to enforce more secure connection:
    * Must be 8 characters.
    * Valid values for passwords include numerals, capital letters, lowercase letters, and special characters (`.,<,+,|,%,!,*,-,%,>,?,:,=`).
    * The minimum period between password changes is 0 day.
    * The password expiration time is 30 days. For this release, you do not receive any password expiration warning messages. 
    * The z/OS virtual server instance retains the last five passwords for each user and prevents these passwords from being reused.
    * Inactive user IDs are not automatically revoked.                        
                                                                
   For more information about the commands, see [tsocmd - Run a TSO/E command from the shell (including authorized commands)](https://www.ibm.com/docs/en/zos/2.4.0?topic=scd-tsocmd-run-tsoe-command-from-shell-including-authorized-commands){: external} and [ALTUSER (Alter user profile)](https://www.ibm.com/docs/en/zos/2.4.0?topic=syntax-altuser-alter-user-profile){: external}.

## Step 2. Getting connected
{: #getting-connected-zos}

You can interact with your z/OS virtual server instance with the following approaches:

* Use the TN3270 terminal emulator to log on to the Time Sharing Option/Extensions (TSO/E). The default port for the secured TN3270 is `992` and you need to provide a self-signed certificate-authority(CA) certificate as a parameter to have a secured connection to the 3270 emulation program. You can find your self-signed CA certificate in a file that is called `common_cacert` in the home directory of the `IBMUSER` user ID in UNIX System Services.

   * For macOS, you need to transfer the `common_cacert` file before you connect to the secured telnet port and run the following commands:

   ```sh
   scp ibmuser@<vsi ip address>:/u/ibmuser/common_cacert <my local dir>/common_cacert
   c3270 -cafile <my local dir>/common_cacert -port 992 <vsi ip address>
   ```
   {: codeblock}

   * For Windows, you need to import the self-signed CA cert into Windows first and then create the IBM Personal Communications to read the CA cert from the truststore in Windows. Complete the following steps:

    1. Transfer the `common_cacert` file from the z/OS system to your workstation.
    2. Open **Internet Options** in your workstation.
    3. Under **Content**, select **Certificates**.
    4. Select **Import** to open the Certificate Import Wizard page and click **Next**.
    5. Click **Browse** to select the `common_cacert` file and click **Next**.
    6. Under the Certificate Store page, select **Place all certificates in the following store** and click **browse** to select the **Trusted Root Certificate Authorities**, and then click **Next**.
    7. On the Completing the Certificate Import Wizard page, click **Finish**.
    8. After you transfer the CA cert to the truststore successfully, you need to create a secure session in PCOMM. Under the **Host Definition** tab of the **Link Parameters** configuration, enter the IP address or the hostname of your z/OS virtual server instance with port `992`.
    9. Under the **Security Setup** tab of the **Link Parameters** configuration, check the **Enable Security** box. 

    The default user ID is `IBMUSER` and the password is the one you configured in the previous step. Then, you can interact with the z/OS in the TSO native mode, by using the Interactive System Productivity Facility (ISPF), or by using z/OS UNIX shell and utilities. For more information, see [Interacting with z/OS: TSO, ISPF, and z/OS UNIX interfaces](https://www.ibm.com/docs/en/zos-basic-skills?topic=concepts-interacting-zos-tso-ispf-zos-unix-interfaces){: external}.

    The unsecured port `23` for 3270 connection is closed. You must use the secured port `992`. The VSI server certificate only contains the private IP address information of the z/OS virtual server instance. Accessing the instance by using its floating IP together with this VSI server certificate fails.
    {: important}

* Use the web browser to access the IBM z/OS Management Facility (z/OSMF). For example, access the url `https://<vsi ip address>:10443/zosmf/LogOnPanel.jsp`. For more information about the z/OSMF, see [IBM z/OS Management Facility](https://www.ibm.com/products/zos-management-facility){: external}.

* Use the serial console from the {{site.data.keyword.cloud_notm}} UI or CLI. The following instructions show how you can connect the serial console to the z/OS virtual server instance from the {{site.data.keyword.cloud_notm}} UI.

   1. In the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/vpc-ext){: external}, go to **menu icon ![menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > Compute > Virtual server instance**.
   2. In the **Virtual server instances for VPC** list, click the overflow button of the instance that you need to access, then click **Open serial Console**. Alternatively, on the instance details page, click **Action** on the upper right then click **Open Serial Console**.
   3. (For serial console only) If the serial console is being used, you are prompted  to confirm whether to force open a session. This action disconnects the other user's session.
   4. Enter the credentials following the prompts to log in to your instances.
   5. (For z/OS Master Console only) Use the **Ctrl + L** key combination to open a new z/OS Master Console, where you can issue four commands `start`, `stop`, `ipl`, and `oprmsg` for z/OS virtual server instance operations.

   For more information about the serial console and how you can use the serial console with {{site.data.keyword.cloud_notm}} CLI, see [Accessing virtual server instances by using VNC or serial consoles](/docs/vpc?topic=vpc-vsi_is_connecting_console). Note that the VNC console is not supported on z/OS virtual server instances.

* Use the SSH private key through a floating IP address to connect the z/OS UNIX shell environment. However, you might want to unbind your floating IP from your z/OS virtual server instance for security considerations and use a [bastion host](/docs/vpc?topic=solution-tutorials-vpc-secure-management-bastion-server) or [client-to-site VPN server](/docs/vpc?topic=vpc-vpn-client-to-site-overview) for all connections. The default secured port is 22. For more information about the floating IP address, see [Reserving a floating IP address](/docs/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#reserving-a-floating-ip-address)

If you want to access another z/OS virtual server instance that is created by using a different SSH key, you must ask the instance owner to add your SSH public key into the `authorized_keys` file on that z/OS virtual server instance. For more information about the authorized_key file, see [Format of the authorized_keys file](https://www.ibm.com/docs/en/zos/2.4.0?topic=daemon-format-authorized-keys-file){: external} and [Configuring new users to access your z/OS virtual server instance](#new_sshkey_zos).

If your z/OS virtual server instance is created by using the z/OS 2.4 dev and test stock image, you can refer to [Reserved configurations](https://www.ibm.com/docs/en/wazi-aas/1.0.0?topic=vpc-configurations-in-zos-stock-images){: external} when adding additional ports into the security group of your z/OS virtual server instance.

## Step 3. Configuring new users to access your z/OS virtual server instance
{: #new_sshkey_zos}

To allow others to access your z/OS virtual server instance by using their own z/OS user IDs and own SSH keys after the instance is created, you must create each user profile and add each SSH public key into the `authorized_keys` file on the z/OS virtual server instance.

1. You can use one of the following methods to create a new z/OS user ID in the RACF (Resource access control facility):
  
   * Issuing the `ADDUSER` command.
   * Enrolling the user through the TSO/E Information Center Facility (ICF) panels. For more information about administering the Information Center Facility, see [z/OS TSO/E Administration](https://www.ibm.com/docs/en/zos/2.4.0?topic=tsoe-zos-administration){: external}.

   Here is an example of using the `ADDUSER` command to create a user profile. Suppose you want to create a user profile for user Steve H., a member of Department A. You want to assign the following values:
   * `STEVEH` for the user ID
   * `DEPTA` for the default connect group
   * `DEPTA` for the owner of the STEVEH user profile
   * `R3I5VQX` for the initial password
   * `Steve H.` for the user's name

   Steve H. does not require any of the user profile segments except TSO. The TSO segment values that you want to set to start with are `123456` for the account number and `PROC01` for the logon procedure.
   {: note}

   To create a user profile with these values, you can run the following commands:
   
    ```sh
    ADDUSER STEVEH DFLTGRP(DEPTA) OWNER(DEPTA) NAME('Steve H.')
        PASSWORD(R315VQX) TSO(ACCTNUM(123456) PROC(PROC01))
    ```
    {: codeblock}

2. Then, you can use one of the following options to add new SSH public keys into the `authorized_keys` file.

   * Use the linux command such as `ssh-copy-id -i <new-ssh-public-key-file> ibmuser@<vsi ip>` to upload new SSH public keys on your local file system into the z/OS virtual server instance. For more information, see [ssh-copy-id for copying SSH keys to servers](https://www.ssh.com/academy/ssh/copy-id){: external}.

   * Log in to the z/OS virtual server instance, use the z/OS Unix System shell command such as `cat <new-ssh-public-key-file> >> ~/.ssh/authorized_keys` to concatenate the new SSH public key into the `authorized_keys` file. For more information, see [cat - Concatenate or display files](https://www.ibm.com/docs/en/zos/2.4.0?topic=descriptions-cat-concatenate-display-files){: external}.

   * Use the [Zowe CLI](https://docs.zowe.org/stable/user-guide/cli-installcli.html){: external} to connect to the z/OS virtual server instance, then create a CLI profile and add new SSH public keys into the `authorized_keys` file. For more information about the Zowe CLI profiles, see [Creating Zowe CLI profiles](https://ibm.github.io/zowe-cli-cics-deploy-plugin/cdp-Creating-Zowe-CLI-profiles){: external}.


For more information about the authorized_key file and z/OS user profiles, see [Format of the authorized_keys file](https://www.ibm.com/docs/en/zos/2.4.0?topic=daemon-format-authorized-keys-file){: external} and [Summary of steps for defining users](https://www.ibm.com/docs/en/zos/2.4.0?topic=users-summary-steps-defining){: external}.


## Next steps
{: #next-manage-vsi-zos}

After you connected to your virtual server instance, you can [manage your instance](/docs/vpc?topic=vpc-managing-virtual-server-instances). 

For instructions on managing z/OS virtual server instances, see [Configuring and managing z/OS virtual server instances](/docs/vpc?topic=vpc-vsi_is_configuring_zos).
