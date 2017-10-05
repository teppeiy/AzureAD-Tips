https://blogs.technet.microsoft.com/enterprisemobility/2016/07/29/azure-ad-mailbag-hybrid-identity-and-adfs-part-2/

Question 1:

Both AD FS and Azure AD give me SaaS App SSO. What should I use when?

 

Answer 1:

Azure AD is an Identity and Access Management (IAM) platform that brings additional capabilities for Software as a Service (SaaS) applications beyond Single Sign On (SSO). In general, it is recommended to use Azure AD for SaaS apps SSO configuration because it provides:

Automated provisioning for SaaS applications that support it (Salesforce, ServiceNow, etc.). This provides a simple and secure way to manage identities in various SaaS applications.
User friendly portals https://myapps.microsoft.com/ for information workers (IW) to discover and launch applications they have access to. Similarly, SaaS apps are available for users from the office portal (https://portal.office.com), which is a great option for users who are already used to it.
The application gallery enables simpler configuration wizards optimized for each application. You can also “bring your own” federated applications that support SAML protocol or WS-Federation.
Azure AD Provides built-in reports for SaaS applications including:  Logins to the applications, and anomalous logins using machine learning powered by the cloud backend.  Without Azure AD, you need additional logic and complex parsing of the IdP audits to replicate (a) above. The machine learning models and techniques to produce (b) are not cost effective to produce on premises.
Azure AD Identity Protection provides risk-based conditional access, evaluating risk of logins evaluation from multiple data sources using machine learning at cloud scale in addition to supporting conditional access based on location and MFA.
Common management control plane with:
Microsoft services such as Office 365
Password SSO application, in addition to federation
Internal applications using Azure AD Application proxy
Custom Azure AD line of business (LOB) applications
Independent token signing certificates per app, reducing the rollover impact.
When combined with group management features, Azure AD provides more options to assign access to the SaaS apps:
Delegate a business owner to manage user assignment
Allow users to self-service requests for access with optional approval process
Provides attribute based control using groups with dynamic membership
In an on premises deployment, the capabilities above are usually available with 3rd party solutions.

Setting up Azure AD as the trust decouples the application from the on-premises credential approach in your tenant, which gives you flexibility to move from federation to Azure AD password hash sync and reduce on-premises infrastructure in the future.
In addition to the functional reasons above, there are also some practical considerations:

As a cloud service, Azure AD is continuously releasing fixes, new features and enabling new scenarios for administrators, developers, and end users.
When Apps in the Gallery change or break their API’s, Azure AD engineers wake up in the middle of the night to fix it, not you
The SaaS application gallery is constantly updated with new applications. As a customer, you can submit requests to the  Azure AD team to add new applications.
You can keep adding applications without worrying about upgrading your on-prem capacity.
 

So, when should you keep the SaaS application RP trusts in AD FS? Simply, when Azure AD does not support it. There are some advanced use cases that can only be implemented by AD FS such as:

Advanced claim transformations such as transformation of attributes, regular expressions, or claim extractions from LDAP, SQL Server, or custom attribute stores
Token customizations such as SHA256 signatures, specific NameID policies, etc.
Support for SAML 1.1 tokens for WS-Federation applications.
Custom triggering of multifactor authentication rules that are not supported by conditional access.
Custom authorization logic that can’t be modeled as a security group or conditional access policies.
As mentioned above, Azure AD is constantly releasing more capabilities to close the SSO functional gaps with AD FS, especially around conditional access capabilities.