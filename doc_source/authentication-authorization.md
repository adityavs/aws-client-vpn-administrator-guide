# Client authentication and authorization<a name="authentication-authorization"></a>

Client VPN provides authentication and authorization capabilities\.

**Topics**
+ [Authentication](#client-authentication)
+ [Authorization](#client-authorization)

## Authentication<a name="client-authentication"></a>

Authentication is implemented at the first point of entry into the AWS Cloud\. It is used to determine whether clients are allowed to connect to the Client VPN endpoint\. If authentication succeeds, clients connect to the Client VPN endpoint and establish a VPN session\. If authentication fails, the connection is denied and the client is prevented from establishing a VPN session\.

Client VPN offers two types of client authentication: Active Directory authentication and mutual authentication\. You can choose to use either one or both authentication methods\.

### Active Directory authentication<a name="ad"></a>

Client VPN provides Active Directory support by integrating with AWS Directory Service\. With Active Directory authentication, clients are authenticated against existing Active Directory groups\. Using AWS Directory Service, Client VPN can connect to existing Active Directories provisioned in AWS or in your on\-premises network\. This allows you to use your existing client authentication infrastructure\. If you are using an on\-premises Active Directory, you must configure an Active Directory Connector \(AD Connector\)\. You can use one Active Directory server to authenticate the users\. For more information about Active Directory integration, see the [AWS Directory Service Administration Guide](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/)\.

To create a Client VPN endpoint, you must provision a server certificate in AWS Certificate Manager\. For more information about creating and provisioning a server certificate, see the steps in [Mutual authentication](#mutual)\.

Client VPN supports multi\-factor authentication \(MFA\) when it's enabled for AWS Managed Microsoft AD or AD Connector\. If MFA is enabled, clients must enter a user name, password, and MFA code when they connect to a Client VPN endpoint\. For more information about enabling MFA, see [Enable Multi\-Factor Authentication for AWS Managed Microsoft AD](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ms_ad_mfa.html) and [Enable Multi\-Factor Authentication for AD Connector](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ad_connector_mfa.html) in the *AWS Directory Service Administration Guide*\. 

### Mutual authentication<a name="mutual"></a>

With mutual authentication, Client VPN uses certificates to perform authentication between the client and the server\. Certificates are a digital form of identification issued by a certificate authority \(CA\)\. The server uses client certificates to authenticate clients when they attempt to connect to the Client VPN endpoint\. The server and client certificates must be uploaded to AWS Certificate Manager \(ACM\)\. For more information about provisioning and uploading certificates in ACM, see the [AWS Certificate Manager User Guide](https://docs.aws.amazon.com/acm/latest/userguide/)\. 

You only need to upload the client certificate to ACM when the Certificate Authority \(Issuer\) of the client certificate is different from the Certificate Authority \(Issuer\) of the server certificate\.

You can create a separate client certificate and key for each client that will connect to the Client VPN endpoint\. This enables you to revoke a specific client certificate if a user leaves your organization\.

A Client VPN endpoint supports 1024\-bit and 2048\-bit RSA key sizes only\.

The following procedure uses OpenVPN easy\-rsa to generate the server and client certificates and keys, and then uploads the server certificate and key to ACM\. For more information, see the [Easy\-RSA 3 Quickstart README](https://github.com/OpenVPN/easy-rsa/blob/v3.0.6/README.quickstart.md)\. The following procedures require OpenSSL\.

**To generate the server and client certificates and keys and upload them to ACM**

1. \(Linux\) Clone the OpenVPN easy\-rsa repo to your local computer and navigate to the `easy-rsa/easyrsa3` folder\.

   ```
   $ git clone https://github.com/OpenVPN/easy-rsa.git
   ```

   ```
   $ cd easy-rsa/easyrsa3
   ```

   \(Windows\) Download the latest release for Windows at [https://github\.com/OpenVPN/easy\-rsa/releases](https://github.com/OpenVPN/easy-rsa/releases)\. Unzip the folder and run the EasyRSA\-Start\.bat file\.

1. Initialize a new PKI environment\.

   ```
   $ ./easyrsa init-pki
   ```

1. Build a new certificate authority \(CA\)\.

   ```
   $ ./easyrsa build-ca nopass
   ```

   Follow the prompts to build the CA\.

1. Generate the server certificate and key\.

   ```
   $ ./easyrsa build-server-full server nopass
   ```

1. Generate the client certificate and key\.

   Make sure to save the client certificate and the client private key because you will need them when you configure the client\.

   ```
   $ ./easyrsa build-client-full client1.domain.tld nopass
   ```

   You can optionally repeat this step for each client \(end user\) that requires a client certificate and key\.

1. Copy the server certificate and key and the client certificate and key to a custom folder and then navigate into the custom folder\.

   Before you copy the certificates and keys, create the custom folder by using the `mkdir` command\. The following example creates a custom folder in your home directory\.

   ```
   $ mkdir ~/custom_folder/
   $ cp pki/ca.crt ~/custom_folder/
   $ cp pki/issued/server.crt ~/custom_folder/
   $ cp pki/private/server.key ~/custom_folder/
   $ cp pki/issued/client1.domain.tld.crt ~/custom_folder
   $ cp pki/private/client1.domain.tld.key ~/custom_folder/
   $ cd ~/custom_folder/
   ```

1. Upload the server certificate and key and the client certificate and key to ACM\. The following commands use the AWS CLI\.

   ```
   $ aws acm import-certificate --certificate file://server.crt --private-key file://server.key --certificate-chain file://ca.crt --region region
   ```

   ```
   $ aws acm import-certificate --certificate file://client1.domain.tld.crt --private-key file://client1.domain.tld.key --certificate-chain file://ca.crt --region region
   ```

   To upload the certificates using the ACM console, see [Import a Certificate](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate-api-cli.html) in the *AWS Certificate Manager User Guide*\.
**Note**  
Be sure to upload the certificates and keys in the same Region in which you intend to create the Client VPN endpoint\.  
If you're using the AWS CLI version 2, use the `fileb://` prefix instead of the `file://` prefix\. For more information, see the [AWS CLI version 2 migration information](https://docs.aws.amazon.com/cli/latest/userguide/cliv2-migration.html#cliv2-migration-binaryparam) in the *AWS Command Line Interface User Guide*\.

## Authorization<a name="client-authorization"></a>

Client VPN supports two types of authorization: security groups and network\-based authorization \(using authorization rules\)\.

### Security groups<a name="security-groups"></a>

Client VPN automatically integrates with VPC security groups\. The security groups are associated with the Client VPN network interfaces\. When you create a Client VPN endpoint, you can specify the security groups from a specific VPC to apply to the Client VPN endpoint\. When you associate a subnet with a Client VPN endpoint, we automatically apply the VPC's default security group\. You can change the security groups after you create the Client VPN endpoint\. 

You can enable Client VPN users to access your applications in a VPC by adding a rule to your applications' security groups to allow traffic from the security group that was applied to the association\. Conversely, you can restrict access for Client VPN users, by not specifying the security group that was applied to the association\. For more information, see [Apply a security group to a target network](cvpn-working-target.md#cvpn-working-target-apply)\. The security group rules that you require might also depend on the kind of VPN access you want to configure\. For more information, see [Scenarios and examples](scenario.md)\.

For more information about security groups, see [Security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) in the *Amazon VPC User Guide*\.

### Network\-based authorization<a name="auth-rules"></a>

Network\-based authorization is implemented using authorization rules\. For each network that you want to enable access, you must configure authorization rules that limits the users who have access\. For a specified network, you configure the Active Directory group that is allowed access\. Only users who belong to the specified Active Directory group can access the specified network\. If you are not using Active Directory, or you want to open access to all users, you can specify a rule that grants access to all clients\. For more information, see [Authorization rules](cvpn-working-rules.md)\.