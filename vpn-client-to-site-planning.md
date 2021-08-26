---

copyright:
  years: 2021
lastupdated: "2021-08-26"

keywords:

subcollection: vpc

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:important: .important}
{:new_window: target="_blank"}
{:pre: .pre}
{:beta: .beta}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}

# Planning considerations for VPN servers (Beta)
{: #client-to-site-vpn-planning}

Client VPN for VPC is available to all {{site.data.keyword.cloud}} users. After the Beta period ends, you will be given a time period to migrate your VPN servers to the standard pricing plan to avoid disruption of service.
{: beta}

Review the following considerations before creating a client-to-site VPN server.
{: shortdesc}

## Scaling considerations for Beta
{: #scaling-considerations}

The aggregation bandwidth of each VPN server is 200 Mbps, and the maximum number of active clients is 500. If you require more bandwidth, or have more clients that need to connect with the VPN server, you can create multiple VPN servers in the same VPC, or in different VPCs in different regions.

## Existing VPC configuration considerations
{: #existing-vpc-configuration-considerations}

Decide if you need to access service endpoints and IaaS endpoints from your client. These endpoints securely connect to {{site.data.keyword.cloud_notm}} services over the {{site.data.keyword.cloud_notm}} private network. If you need to access these endpoints, you must specify the DNS server addresses `161.26.0.10` and `161.26.0.11` when you provision the VPN server. See [Service endpoints](/docs/vpc?topic=vpc-service-endpoints-for-vpc#cloud-service-endpoints) and [IaaS endpoints](/docs/vpc?topic=vpc-service-endpoints-for-vpc#infrastructure-as-a-service-iaas-endpoints) for details.

You must also decide whether you need to resolve private DNS names from your client. {{site.data.keyword.cloud_notm}} DNS Services provides private DNS to VPC users. If you need to access these endpoints, you must specify the DNS server addresses `161.26.0.7` and `161.26.0.8` when you provision the VPN server. See [About DNS Services](/docs/dns-svcs?topic=dns-svcs-about-dns-services) for details.

   When you specify this DNS server, you must also create a VPN route after the VPN server is provisioned, with destination `161.26.0.0/16` and the `translate` action.
   {: important}

## VPN server provisioning considerations
{: #vpn-server-considerations}

Consider the following when provisioning a VPN server. 

### Client IPv4 address pool
{: #client-ip-address-pool}

When you create a VPN server, you are prompted for a client IPv4 address pool (CIDR range). The client is assigned an IP address for its session from this address pool. Keep in mind that the client IP pool property must not overlap with VPC address prefixes. The VPN service validates the client IP address if the client IP pool overlaps with an existing address prefix.

Review the following requirements:

* Each active VPN client is assigned an IP address from a configurable IP pool. You must select the IP CIDR range carefully to make sure it does not overlap with the VPC prefix and personal device local CIDR.

* Depending on the use case, the client IP pool cannot overlap with a customer's local device IP address. The client IP pool also cannot overlap with the destination network; for example, if the VPN server is used to access the {{site.data.keyword.cloud_notm}} classic network, the client IP cannot overlap with the {{site.data.keyword.cloud_notm}} classic network.

* You must ensure that the block size is at least `/22` (`1024` free IP addresses). It is recommended to use a CIDR block that contains twice the number of IP addresses that are required to enable the maximum number of concurrent connections.

### Subnets: high-availability versus stand-alone mode 
{: #vpn-type}

When you create a VPN server, you can specify either high-availability or stand-alone mode. 

   * If you select high-availability mode, you must deploy the VPN server acress two subnets in different zones. Use this mode for production deployments. Best for multi-zone deployments and solutions where client VPN access is critical.
   * If you select stand-alone mode, you deploy the VPN server in a single subnet and zone. Use this mode for a pilot, non-production deployment where multi-zone resiliency is not required.

### VPN server authentication
{: #server-authentication}

You must specify a VPN server certificate during provisioning. You can create a certificate using the {{site.data.keyword.cloud_notm}} Certificate Manager, or use one of your own.

   If using the CLI or API, you must specify the certificate's CRN. To obtain the certificate CRN, see [Locating the certificate CRN ](/docs/vpc?topic=vpc-client-to-site-authentication#locating-cert-crn).
   {: note}

If the VPN server is using user ID and passcode authentication only, you only need to specify the VPN server certificate, which includes the public/private key and CA certificate. The VPN service gets the public and private keys from the certificate instance and stores them in the VPN server. This CA certificate is also copied in the client profile so that the client can use the CA certificate to verify the VPN server.  

If certificate authentication is enabled, you must specify the client CA certificate. The public key and private key are not required. The VPN service gets the client CA certificate from the certificate manager. In turn, the client presents its public key when it connects with the VPN server, and the VPN server uses the CA certificate to verify the public key.  

   If the client and VPN server certificate are signed by the same CA, then the administrator can use the same certificate instance when they provision the VPN server.
   {: note}
   
For more information, see [Setting up client-to-server authentication](/docs/vpc?topic=vpc-client-to-site-authentication).

### VPN client authentication
{: #client-authentication}

As the VPN server administrator, you must choose at least one authentication method and configure it during the VPN server provisioning. You can choose a client certificate, two-factor authentication with a user ID and passcode, or both types of client authentication.

   Multiple VPN clients can share one client certificate.
   {: note}

If you plan to use the client certificate, the user must edit the client profile provided by the server administrator and include the client certificate (also called the public key) and the private key.  Note that modification of the client profile is not necessary if using only two-factor (user ID and passcode) client authentication.

VPN users don't use their password directly to connect to the VPN server. They get the passcode from the IBM Access Manager (IAM) via a browser, and if the MFA is enabled, the MFA enforcement is always done via the browser. The user must configure the MFA properly to make sure the MFA enforcement can be done on the browser. After a user gets the passcode, they input the passcode on the OpenVPN client and initiate the connection.

The VPN server receives the username and passcode from the VPN client and makes an IAM call to verify the passcode and permission with IAM policy.  

   * The passcode is an one-time password. The user MUST re-generate the passcode for re-connection, even if the re-connection is initiated by the VPN server.
   * The SoftLayer MFA is not supported because SoftLayer MFA enforcement is not done via the browser.
   
If you use user ID/passcode authentication, maintenance activities force users to re-authenticate by fetching and re-entering the code. The connection is restored only after the new code is entered. This is applicable using stand-alone or HA mode. 
{: important}

### Client certificate revocation lists
{: #client-certificate-revocation-lists}

Optionally, you can import a certificate revocation list (CRL), which is a time-stamped list of certificates that have been revoked by a certificate authority (CA). A certificate in a certificate revocation list (CRL) might not be expired, but is no longer trusted by the certificate authority that issued the certificate. The VPN client uses this list to validate digital certificates.   

After you import a CRL, the VPN client uses this list to validate digital certificates. The CRL is saved as a string (not a file) in the system. If you need to download the CRL in the future, it is renamed as  `<vpn_server_name>.pem.`    

For more information, see [Setting up client-to-server authentication](/docs/vpc?topic=vpc-client-to-site-authentication).

### Transport protocol
{: #transport-protocol}

The transport layer oversees the delivery of data from a process on one device to a process on another device. Transport layer protocols act as liaisons between the application layer protocols and the services that are provided by the network. Client VPN for VPC supports the following protocols:

   UDP is recommended for optimal performance; TCP for reliability.
   {: note}
   
   * **User Datagram Protocol (UDP)** 

      The User Datagram Protocol (UDP) is a simple, lightweight protocol with minimum overhead. If a process wants to send a small message and doesn't care about reliability, it can use UDP. Sending a message by using UDP takes much less time than using TCP. It performs little error checking and does not add any advantages to IP services except to provide process-to-process communication instead of host-to-host communication.  

   * **Transmission Control Protocol (TCP)**

      Transmission Control Protocol (TCP) is a reliable but complex transport-layer protocol. TCP adds connection-oriented features and reliability to IP services.

      TCP is a stream delivery service that guarantees delivery of data streams sent from one host to another without duplication or lost data. Since packet transfer is not reliable, a technique known as positive acknowledgment with retransmission is used to guarantee reliability of packet transfers. This fundamental technique requires the receiver to respond with an acknowledgment message as it receives the data.

      The sender keeps a record of each packet it sends, and waits for acknowledgment before sending the next packet. The sender also keeps a timer from when the packet was sent, and retransmits a packet if the timer expires. This timer is needed in case a packet becomes lost or corrupted.
      
### Full versus split tunnel mode
{: #full-versus-split-tunnels}

When a VPN connection is set up, an encrypted tunnel is created over the internet to the VPN server. The VPN connection appears as a virtual network interface to the computer in addition to the existing LAN interface. You can now use both interfaces simultaneously by sending the private traffic destined to the VPC inside the VPN tunnel and the public traffic (internet traffic) over the other interface (outside the VPN tunnel). When the traffic is split between the VPN interface and other interfaces, _split tunneling_ is said to be in use. When split tunneling is not in use, all the traffic uses the VPN interface, resulting in the internet traffic being sent into the VPN tunnel, which is _full-tunnel_.

Other considerations:

   * Use full-tunnel (default) mode if you have a security concern when the client accesses the internet without a VPN tunnel. Full-tunnel is usually necessary for compliance with regulatory standards; however, this approach can be costly and also increases the load of the VPN server.

   * In split-tunnel mode, routes are pushed to the VPN client by the VPN server. This way, the OpenVPN client knows which traffic should be sent into the VPN tunnel. You should be careful when adding routes and avoid routing loops. For example, if the VPN server's public IP address is `3.3.3.3`, then you cannot add a route `3.3.3.0/24` because this route would send traffic to `3.3.3.3` that should not go through the VPN tunnel. Ideally, you should configure the private subnet only as a route destination, such as a VPC subnet, a CSE subnet, an on-premises private subnet, and so on.

   * The VPN route is pushed to your VPN client. If your VPN client already has a route with the same destination, the route "push" fails and the traffic cannot reach your VPN server. You must address the route conflict and then re-connect the VPN client. A common issue is if you add a VPN route with destination `0.0.0.0/0` on a split-tunnel-mode VPN server, and this route needs to be pushed to your VPN client. Typically, the VPN client already has a route with destination `0.0.0.0/0`; therefore, this VPN route will conflict with your VPN client route. To avoid conflict, use a full-tunnel-mode VPN server, or remove the route `0.0.0.0/0` on your host.

No matter which tunnel mode you choose, you must use the API `/vpn_servers/{id}/routes` to define how the VPN server forwards the traffic from the VPN client. For example, if you want the internet traffic from the client to go through the VPN tunnel, a `0.0.0.0/0` route must be configured using the VPN service routes API.


### Supported VPN client software
{: #vpn-client-software}

You must provide VPN client software for your users. The following client software versions are verified for use with this beta release:

* For macOS Catalina and later:<br />[OpenVPN Connect v3](https://openvpn.net/vpn-client/#tab-macos){: external}, OpenVPN Connect v2, and Tunnelblick 3.8.4
* Windows 8 and later:<br />[OpenVPN Connect v3](https://openvpn.net/vpn-client/#tab-windows){: external}, OpenVPN Connect v2
* RHEL 7.x and later:<br />[OpenVPN Connect v3](https://openvpn.net/vpn-client/#tab-linux){: external}, OpenVPN Connect v2, and OpenVPN command-line client (version 2.4.4 and later)
* Ubuntu 18.04 and later:<br />[OpenVPN Connect v3](https://openvpn.net/vpn-client/#tab-linux){: external}, OpenVPN Connect v2, and OpenVPN command-line client (version 2.4.10 and later)

VPN client users can choose other OpenVPN-2.4-compatible client software. However, software that is not listed is not guaranteed to work.
{: tip}