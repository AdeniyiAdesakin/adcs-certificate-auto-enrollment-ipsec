# Active Directory Certificate Services: Computer Auto-enrollment and IPsec Authentication

**Enterprise PKI | Computer Certificates | Group Policy Auto-enrollment | Certificate-Based IPsec | Wireshark Validation**

## Project Overview

I deployed Active Directory Certificate Services (AD CS) in a Windows Server domain, configured automatic computer-certificate enrollment through Group Policy, and used the issued certificates to authenticate IPsec communication between two servers.

The project began with a standalone root certification authority on the `MS1` member server. After confirming that a standalone CA does not provide Active Directory certificate templates, I deployed an enterprise root CA in the domain. This made the built-in Computer template available for domain-based certificate enrollment.

I then created a Group Policy Object that automatically requested computer certificates for domain systems. I verified the issued certificates on `MS1`, Windows 10, and Windows 11, and reviewed the issued-certificate list on the certification authority.

Finally, I created matching server-to-server IPsec rules on `ADDS-Server` and `MS1`. The rules used certificates from the enterprise CA to authenticate both endpoints. I generated network traffic, inspected the active IPsec security associations with PowerShell, and used Wireshark to confirm the presence of Encapsulating Security Payload (ESP) traffic.

## Business Scenario

An organization needs a trusted method for identifying domain computers and protecting network communication between important servers.

Manually requesting and installing a certificate on every computer would be difficult to manage as the environment grows. To simplify the process, an enterprise certification authority can issue certificates from Active Directory templates while Group Policy automatically enrolls eligible domain computers.

The issued certificates can then be used for services such as IPsec, VPN authentication, secure wireless access, Remote Desktop, and other applications that require trusted computer identities.

## Certificate Deployment Path

`Enterprise Root CA → Computer Certificate Template → Group Policy Autoenrollment → Domain Computers → Certificate-Authenticated IPsec`

## Project Objectives

- Install the Active Directory Certificate Services server role.
- Configure the Certification Authority role service.
- Compare standalone and enterprise certification authorities.
- Identify why certificate templates were unavailable on the standalone CA.
- Deploy an enterprise root CA for domain certificate enrollment.
- Configure computer-certificate autoenrollment through Group Policy.
- Assign the Computer certificate template through an automatic certificate request.
- Link the certificate GPO to the Active Directory domain.
- Force a Group Policy refresh on domain computers.
- Verify issued certificates through the local computer certificate store.
- Review issued certificates from the Certification Authority console.
- Configure matching server-to-server IPsec rules.
- Use computer certificates to authenticate both IPsec endpoints.
- Inspect active IPsec security associations with PowerShell.
- Confirm ESP-protected traffic with Wireshark.

## Lab Environment

| Component | Configuration |
| --- | --- |
| Active Directory domain | `adeniyi.com` |
| Initial CA server | `MS1` member server |
| Initial CA configuration | Standalone root CA |
| Working domain CA | `ADDS-Server` |
| Working CA configuration | Enterprise root CA |
| AD CS role service | Certification Authority |
| Certificate template | Computer |
| Autoenrollment scope | Active Directory domain |
| Certificate recipients | `MS1`, Windows 10, and Windows 11 |
| IPsec endpoints | `ADDS-Server` and `MS1` |
| IPsec rule type | Server-to-server |
| IPsec authentication | Computer certificate |
| Packet-analysis tool | Wireshark |
| Administration tools | Server Manager, Group Policy Management, MMC, Windows Defender Firewall with Advanced Security, PowerShell |

## Skills Demonstrated

- Active Directory Certificate Services
- Public key infrastructure fundamentals
- Standalone and enterprise CA configuration
- Certificate-template administration
- Computer-certificate autoenrollment
- Group Policy administration
- Microsoft Management Console
- Local computer certificate-store management
- Windows Defender Firewall with Advanced Security
- Certificate-based IPsec authentication
- Main Mode and Quick Mode security-association inspection
- Wireshark packet analysis
- Windows Server troubleshooting
- Technical documentation

## Implementation

### Part 1: Installed Active Directory Certificate Services

#### 1. Started the Add Roles and Features Wizard

On the `MS1` member server, I opened **Server Manager > Manage > Add Roles and Features**.

<p align="center">
  <img src="https://i.imgur.com/5NoW5Og.png" width="750" alt="Opening Add Roles and Features from Windows Server Manager">
</p>

I reviewed the Before You Begin page and continued.

<p align="center">
  <img src="https://i.imgur.com/2UAe7QL.png" width="750" alt="Add Roles and Features Wizard Before You Begin page">
</p>

#### 2. Selected the Installation Type and Server

I selected **Role-based or feature-based installation**.

<p align="center">
  <img src="https://i.imgur.com/u8VLb7Q.png" width="750" alt="Selecting role-based or feature-based installation">
</p>

I selected the target member server from the server pool.

<p align="center">
  <img src="https://i.imgur.com/XJxTUpr.png" width="750" alt="Selecting the member server as the AD CS installation target">
</p>

#### 3. Selected Active Directory Certificate Services

On the Server Roles page, I selected **Active Directory Certificate Services**.

<p align="center">
  <img src="https://i.imgur.com/TsrkD3i.png" width="750" alt="Selecting Active Directory Certificate Services">
</p>

I added the required AD CS management features.

<p align="center">
  <img src="https://i.imgur.com/4paGCrD.png" width="750" alt="Adding the management features required by AD CS">
</p>

I confirmed that Active Directory Certificate Services was selected.

<p align="center">
  <img src="https://i.imgur.com/PtGe0bA.png" width="750" alt="Active Directory Certificate Services selected in Server Manager">
</p>

I retained the default feature selections.

<p align="center">
  <img src="https://i.imgur.com/XoH829y.png" width="750" alt="Reviewing the Windows Server feature selections">
</p>

I reviewed the AD CS role information.

<p align="center">
  <img src="https://i.imgur.com/481IPqT.png" width="750" alt="Reviewing Active Directory Certificate Services role information">
</p>

#### 4. Selected the Certification Authority Role Service

I selected **Certification Authority** as the AD CS role service.

<p align="center">
  <img src="https://i.imgur.com/23CMI19.png" width="750" alt="Selecting the Certification Authority role service">
</p>

I reviewed the installation selections and selected **Install**.

<p align="center">
  <img src="https://i.imgur.com/DBgqn2h.png" width="750" alt="Confirming the AD CS installation selections">
</p>

The wizard confirmed that the AD CS role installation completed successfully.

<p align="center">
  <img src="https://i.imgur.com/bFYd2VU.png" width="750" alt="Successful installation of Active Directory Certificate Services">
</p>

### Part 2: Configured and Evaluated the Standalone Root CA

#### 5. Started Post-Deployment Configuration

From the Server Manager notification flag, I selected **Configure Active Directory Certificate Services on the destination server**.

<p align="center">
  <img src="https://i.imgur.com/H2tvP3E.png" width="750" alt="Starting AD CS post-deployment configuration">
</p>

I confirmed the administrative credentials used for the configuration.

<p align="center">
  <img src="https://i.imgur.com/AjQLj99.png" width="750" alt="Confirming credentials for AD CS configuration">
</p>

I selected the **Certification Authority** role service.

<p align="center">
  <img src="https://i.imgur.com/fEkWtxi.png" width="750" alt="Selecting Certification Authority during AD CS configuration">
</p>

#### 6. Configured MS1 as a Standalone Root CA

For the initial deployment, I selected **Standalone CA**.

<p align="center">
  <img src="https://i.imgur.com/zhh9FuN.png" width="750" alt="Selecting Standalone CA as the setup type">
</p>

I selected **Root CA**.

<p align="center">
  <img src="https://i.imgur.com/A7wxQZv.png" width="750" alt="Selecting Root CA as the certification authority type">
</p>

I created a new private key for the certification authority.

<p align="center">
  <img src="https://i.imgur.com/BVg9zE6.png" width="750" alt="Creating a new private key for the certification authority">
</p>

I retained the cryptographic settings used in the original lab.

<p align="center">
  <img src="https://i.imgur.com/hCsD1ZL.png" width="750" alt="Reviewing the certification authority cryptographic settings">
</p>

I retained the generated CA name.

<p align="center">
  <img src="https://i.imgur.com/bJavoc6.png" width="750" alt="Reviewing the certification authority common name">
</p>

I retained the five-year CA certificate validity period used in the lab.

<p align="center">
  <img src="https://i.imgur.com/eAgN0Iu.png" width="750" alt="Configuring a five-year validity period for the CA certificate">
</p>

I retained the default certificate database and log locations.

<p align="center">
  <img src="https://i.imgur.com/OfrT7PP.png" width="750" alt="Reviewing the certification authority database locations">
</p>

I reviewed the configuration and selected **Configure**.

<p align="center">
  <img src="https://i.imgur.com/Ca90hpy.png" width="750" alt="Confirming the standalone root CA configuration">
</p>

The configuration completed successfully.

<p align="center">
  <img src="https://i.imgur.com/y9g5lgY.png" width="750" alt="Successful configuration of the standalone root CA">
</p>

#### 7. Identified the Certificate-Template Limitation

After opening the Certification Authority console on `MS1`, I found that certificate templates were not available.

After some troubleshooting, I found this as an expected behavior for a standalone CA. Standalone certification authorities do not use the Active Directory certificate templates required for domain auto-enrollment. Only an enterprise CA can issue certificates based on templates stored in Active Directory Domain Services.

<p align="center">
  <img src="https://i.imgur.com/jhcersu.png" width="750" alt="Selecting Enterprise CA for Active Directory-integrated certificate services">
</p>


### Part 3: Deployed the Enterprise Root CA

#### 8. Repeated the AD CS Deployment as an Enterprise CA

To support automatic computer-certificate enrollment, I repeated the AD CS installation and configuration on the domain server. During the Setup Type stage, I selected **Enterprise CA** instead of Standalone CA. And after completing the enterprise CA configuration, the Certification Authority console displayed the available certificate templates.

<p align="center">
  <img src="https://i.imgur.com/NjN9TI8.png" width="750" alt="Certificate templates available on the enterprise certification authority">
</p>


### Part 4: Configured Computer-Certificate Enrollment Through Group Policy

#### 9. Created the Certificate GPO

In **Group Policy Management**, I right-clicked **Group Policy Objects** and created a new GPO for computer-certificate enrollment.

<p align="center">
  <img src="https://i.imgur.com/BVzeQRb.png" width="750" alt="Creating a new Group Policy Object for computer certificates">
</p>

I assigned the GPO a descriptive name.

<p align="center">
  <img src="https://i.imgur.com/fGPX8tH.png" width="750" alt="Naming the computer-certificate Group Policy Object">
</p>

I right-clicked the new GPO and selected **Edit**.

<p align="center">
  <img src="https://i.imgur.com/AvngVAl.png" width="750" alt="Editing the computer-certificate Group Policy Object">
</p>

#### 10. Enabled Certificate Autoenrollment

In Group Policy Management Editor, I navigated to:

`Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies`

<p align="center">
  <img src="https://i.imgur.com/zio5k7z.png" width="750" alt="Opening Public Key Policies in Group Policy Management Editor">
</p>

I opened **Certificate Services Client - Auto-Enrollment** and enabled the policy.

<p align="center">
  <img src="https://i.imgur.com/FAULcor.png" width="750" alt="Opening Certificate Services Client Auto-Enrollment">
</p>

I enabled these options:

- **Renew expired certificates, update pending certificates, and remove revoked certificates**
- **Update certificates that use certificate templates**

<p align="center">
  <img src="https://i.imgur.com/Ev0qw79.png" width="750" alt="Enabling computer certificate autoenrollment options">
</p>

#### 11. Created an Automatic Computer Certificate Request

Under **Public Key Policies**, I right-clicked **Automatic Certificate Request Settings** and selected **New > Automatic Certificate Request**.

<p align="center">
  <img src="https://i.imgur.com/jEwd3C4.png" width="750" alt="Creating an automatic certificate request through Group Policy">
</p>

I started the Automatic Certificate Request Setup Wizard.

<p align="center">
  <img src="https://i.imgur.com/KX8YJJ1.png" width="750" alt="Starting the Automatic Certificate Request Setup Wizard">
</p>

I selected the built-in **Computer** certificate template.

<p align="center">
  <img src="https://i.imgur.com/l1roZ70.png" width="750" alt="Selecting the Computer certificate template">
</p>

I completed the automatic certificate request configuration.

<p align="center">
  <img src="https://i.imgur.com/WL6RXtw.png" width="750" alt="Completing the Automatic Certificate Request Setup Wizard">
</p>

#### 12. Linked the Certificate GPO

I right-clicked the `adeniyi.com` domain and selected **Link an Existing GPO**.

<p align="center">
  <img src="https://i.imgur.com/0GURmh2.png" width="750" alt="Linking an existing certificate GPO to the Active Directory domain">
</p>

I selected the certificate-enrollment GPO and completed the link.

<p align="center">
  <img src="https://i.imgur.com/a3kfNNm.png" width="750" alt="Selecting the computer-certificate Group Policy Object">
</p>

Linking the GPO at the domain level placed all domain computers within its scope, subject to normal Group Policy security filtering and inheritance.

#### 13. Refreshed Group Policy

I ran the original PowerShell command to force a Group Policy refresh:

```powershell
gpupdate /force
```

<p align="center">
  <img src="https://i.imgur.com/E8LgC0b.png" width="750" alt="Running gpupdate force after linking the certificate GPO">
</p>

### Part 5: Verified Certificate Enrollment

#### 14. Opened the Local Computer Certificate Store

On `MS1`, I pressed **Windows + R**, entered the original command, and selected **OK**:

```text
mmc
```

<p align="center">
  <img src="https://i.imgur.com/t1W0nvW.png" width="750" alt="Opening Microsoft Management Console with the mmc command">
</p>

In Microsoft Management Console, I selected **File > Add/Remove Snap-in**.

<p align="center">
  <img src="https://i.imgur.com/ni7Psfr.png" width="750" alt="Opening Add or Remove Snap-ins in Microsoft Management Console">
</p>

I selected **Certificates** and added the snap-in.

<p align="center">
  <img src="https://i.imgur.com/mM1B2KE.png" width="750" alt="Adding the Certificates snap-in to Microsoft Management Console">
</p>

I selected **Computer account**.

<p align="center">
  <img src="https://i.imgur.com/6QvxYJx.png" width="750" alt="Selecting Computer account for the Certificates snap-in">
</p>

I selected **Local computer** and completed the wizard.

<p align="center">
  <img src="https://i.imgur.com/7jgqyrH.png" width="750" alt="Selecting the local computer certificate store">
</p>

Under **Certificates (Local Computer) > Personal > Certificates**, I confirmed that `MS1` had received a computer certificate.

<p align="center">
  <img src="https://i.imgur.com/aaWVtQo.png" width="750" alt="Computer certificate issued to the MS1 member server">
</p>

#### 15. Verified Certificates on Windows 10 and Windows 11

I repeated the local computer certificate-store check on the Windows 10 client and confirmed that a computer certificate had been issued.

<p align="center">
  <img src="https://i.imgur.com/uM5Ykni.png" width="750" alt="Computer certificate issued to the Windows 10 domain client">
</p>

I repeated the verification on Windows 11 and confirmed that it also received a computer certificate.

<p align="center">
  <img src="https://i.imgur.com/Cdcg7WG.png" width="750" alt="Computer certificate issued to the Windows 11 domain client">
</p>

#### 16. Reviewed Issued Certificates on the CA

On the enterprise CA, I opened **Certification Authority > Issued Certificates** and reviewed the certificates issued to the domain computers.

<p align="center">
  <img src="https://i.imgur.com/Z25EYyR.png" width="750" alt="Issued computer certificates displayed in the Certification Authority console">
</p>

### Part 6: Configured Certificate-Based IPsec Authentication

#### 17. Started a Server-to-Server Connection Security Rule

On `ADDS-Server`, I opened **Windows Defender Firewall with Advanced Security**, right-clicked **Connection Security Rules**, and selected **New Rule**.

<p align="center">
  <img src="https://i.imgur.com/ifoA82Z.png" width="750" alt="Creating a new Windows connection security rule">
</p>

I selected **Server-to-server** as the rule type.

<p align="center">
  <img src="https://i.imgur.com/cFf6XwA.png" width="750" alt="Selecting a server-to-server connection security rule">
</p>

#### 18. Configured the IPsec Endpoints

On the Endpoints page, I selected **These IP addresses** and added the address for the local endpoint.

<p align="center">
  <img src="https://i.imgur.com/YSKsdsJ.png" width="750" alt="Selecting specific IP addresses for the first IPsec endpoint">
</p>

<p align="center">
  <img src="https://i.imgur.com/zB0w961.png" width="750" alt="Entering the local IPsec endpoint address">
</p>

I added the remote endpoint address for `MS1`.

<p align="center">
  <img src="https://i.imgur.com/FEDvX0V.png" width="750" alt="Entering the remote IPsec endpoint address">
</p>

I reviewed both endpoint addresses before continuing.

<p align="center">
  <img src="https://i.imgur.com/AqyhVmj.png" width="750" alt="Reviewing the local and remote IPsec endpoint addresses">
</p>

#### 19. Required Authentication in Both Directions

I selected **Require authentication for inbound and outbound connections**.

<p align="center">
  <img src="https://i.imgur.com/FARC08c.png" width="750" alt="Requiring authentication for inbound and outbound connections">
</p>

This setting requires both servers to authenticate before traffic matching the rule can pass between them.

#### 20. Selected Certificate-Based Authentication

On the Authentication Method page, I browsed for the trusted certification authority.

<p align="center">
  <img src="https://i.imgur.com/AtYWyMH.png" width="750" alt="Browsing for a trusted certification authority for IPsec">
</p>

I selected the certificate authority that issued the computer certificates.

<p align="center">
  <img src="https://i.imgur.com/47LY4IT.png" width="750" alt="Selecting the enterprise certification authority for IPsec authentication">
</p>

I confirmed the selected certificate-based authentication method.

<p align="center">
  <img src="https://i.imgur.com/V5sNP1I.png" width="750" alt="Confirming certificate authentication for the IPsec rule">
</p>

#### 21. Applied and Named the Rule

The original lab applied the connection security rule to the Domain, Private, and Public profiles.

<p align="center">
  <img src="https://i.imgur.com/ywrkgSR.png" width="750" alt="Selecting firewall profiles for the IPsec connection security rule">
</p>

I assigned the rule a descriptive name indicating that it secured communication between `ADDS-Server` and `MS1`.

<p align="center">
  <img src="https://i.imgur.com/R3oPw1S.png" width="750" alt="Naming the secure connection rule between ADDS Server and MS1">
</p>

#### 22. Created the Matching Rule on MS1

I repeated the configuration on `MS1`, reversed the local and remote endpoints, and used the same certificate authority for authentication.

<p align="center">
  <img src="https://i.imgur.com/EeTh7NO.png" width="750" alt="Matching IPsec connection security rule configured on MS1">
</p>

<p align="center">
  <img src="https://i.imgur.com/knwsKys.png" width="750" alt="IPsec rule endpoint and authentication configuration on MS1">
</p>

Matching connection security rules were required so both servers could negotiate and enforce the certificate-authenticated IPsec connection.

### Part 7: Validated the IPsec Connection

#### 23. Inspected the Quick Mode Security Association

I ran the original PowerShell command to inspect active Quick Mode security associations:

```powershell
Get-NetIPsecQuickModeSA
```

<p align="center">
  <img src="https://i.imgur.com/a34JlEc.png" width="750" alt="Inspecting the active IPsec Quick Mode security association">
</p>

Quick Mode security associations describe how matching IP traffic is protected, including the negotiated IPsec transforms and endpoints.

#### 24. Inspected the Main Mode Security Association

I ran the second original PowerShell command:

```powershell
Get-NetIPsecMainModeSA
```

<p align="center">
  <img src="https://i.imgur.com/h7GzwnR.png" width="750" alt="Inspecting the active IPsec Main Mode security association">
</p>

Main Mode security associations identify the authenticated IPsec peers and the protection suite used during the initial security negotiation.

#### 25. Generated Test Traffic

From `ADDS-Server`, I sent ping traffic to `MS1` to generate communication that matched the connection security rule.

<p align="center">
  <img src="https://i.imgur.com/WaI8uJK.png" width="750" alt="Generating test traffic between ADDS Server and MS1">
</p>

#### 26. Confirmed ESP Traffic in Wireshark

I used Wireshark to capture the traffic and confirmed that ESP packets were present while the IPsec rules were enabled.

<p align="center">
  <img src="https://i.imgur.com/tQUXfWN.png" width="750" alt="Wireshark capture showing ESP-protected traffic">
</p>

ESP traffic confirmed that IPsec was protecting the matching IP communication between the two servers.

#### 27. Compared Traffic After Disabling IPsec

I disabled the IPsec connection security rule.

<p align="center">
  <img src="https://i.imgur.com/bAwh02q.png" width="750" alt="Disabling the IPsec connection security rule">
</p>

I repeated the Wireshark capture and confirmed that ESP packets were no longer present.

<p align="center">
  <img src="https://i.imgur.com/TP6kTra.png" width="750" alt="Wireshark capture without ESP traffic after disabling IPsec">
</p>

The comparison showed that the connection security rules were responsible for applying IPsec protection to the matching traffic.


## Key Takeaways

This project demonstrated the relationship between Active Directory, enterprise certificate templates, Group Policy, computer certificates, and IPsec.

The standalone CA installation helped identify why certificate templates were unavailable. Reconfiguring the solution with an enterprise CA enabled domain computers to receive certificates automatically through Group Policy.

The IPsec portion showed how those certificates could be used beyond enrollment. Matching connection security rules authenticated the two servers, PowerShell exposed the negotiated security associations, and Wireshark provided packet-level evidence that ESP protection was active.


## Related Projects

- [Active Directory Domain Services and Windows Client Integration](https://github.com/AdeniyiAdesakin/Install-Active-Directory-Domain-Services-and-Join-Client-s-Computer-to-Active-Directory)
- [Group Policy Object Implementations](https://github.com/AdeniyiAdesakin/Group-Policy-Object-GPO-implementations)
- [Windows Server Printer Deployment and Secure File Sharing](https://github.com/AdeniyiAdesakin/Resource-Access-Sharing-a-printer-and-configuring-a-shared-folder)
- [Microsoft Entra ID and On-Premises Active Directory Synchronization](https://github.com/AdeniyiAdesakin/Sync-between-MS-Entra-ID-and-On-Premises-Active-Directory)

