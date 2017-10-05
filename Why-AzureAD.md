[Choose the right sign-in option to connect to Azure AD & Office 365](https://blogs.msdn.microsoft.com/samueld/2017/06/13/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365/)

https://blogs.technet.microsoft.com/enterprisemobility/2016/07/29/azure-ad-mailbag-hybrid-identity-and-adfs-part-2/

Question 1:

Both AD FS and Azure AD give me SaaS App SSO. What should I use when?

 

Answer 1:

Azure AD is an Identity and Access Management (IAM) platform that brings additional capabilities for Software as a Service (SaaS) applications beyond Single Sign On (SSO). In general, it is recommended to use Azure AD for SaaS apps SSO configuration because it provides:

1. Automated provisioning for SaaS applications that support it (Salesforce, ServiceNow, etc.). This provides a simple and secure way to manage identities in various SaaS applications.
2. User friendly portals https://myapps.microsoft.com/ for information workers (IW) to discover and launch applications they have access to. Similarly, SaaS apps are available for users from the office portal (https://portal.office.com), which is a great option for users who are already used to it.
3. The application gallery enables simpler configuration wizards optimized for each application. You can also “bring your own” federated applications that support SAML protocol or WS-Federation.
4. Azure AD Provides built-in reports for SaaS applications including:  Logins to the applications, and anomalous logins using machine learning powered by the cloud backend.  Without Azure AD, you need additional logic and complex parsing of the IdP audits to replicate (a) above. The machine learning models and techniques to produce (b) are not cost effective to produce on premises.
5. Azure AD Identity Protection provides risk-based conditional access, evaluating risk of logins evaluation from multiple data sources using machine learning at cloud scale in addition to supporting conditional access based on location and MFA.
6. Common management control plane with:
    1. Microsoft services such as Office 365
    2. Password SSO application, in addition to federation
    3. Internal applications using Azure AD Application proxy
    4. Custom Azure AD line of business (LOB) applications
7. Independent token signing certificates per app, reducing the rollover impact.
8. When combined with group management features, Azure AD provides more options to assign access to the SaaS apps:
    1. Delegate a business owner to manage user assignment
    2. Allow users to self-service requests for access with optional approval process
    3. Provides attribute based control using groups with dynamic membership  
In an on premises deployment, the capabilities above are usually available with 3rd party solutions.  

9. Setting up Azure AD as the trust decouples the application from the on-premises credential approach in your tenant, which gives you flexibility to move from federation to Azure AD password hash sync and reduce on-premises infrastructure in the future.  
In addition to the functional reasons above, there are also some practical considerations:

10. As a cloud service, Azure AD is continuously releasing fixes, new features and enabling new scenarios for administrators, developers, and end users.
11. When Apps in the Gallery change or break their API’s, Azure AD engineers wake up in the middle of the night to fix it, not you
12. The SaaS application gallery is constantly updated with new applications. As a customer, you can submit requests to the  Azure AD team to add new applications.
13. You can keep adding applications without worrying about upgrading your on-prem capacity.
 

So, when should you keep the SaaS application RP trusts in AD FS? Simply, when Azure AD does not support it. There are some advanced use cases that can only be implemented by AD FS such as:

1. Advanced claim transformations such as transformation of attributes, regular expressions, or claim extractions from LDAP, SQL Server, or custom attribute stores
2. Token customizations such as SHA256 signatures, specific NameID policies, etc.
3. ~~Support for SAML 1.1 tokens for WS-Federation applications.~~
4. ~~Custom triggering of multifactor authentication rules that are not supported by conditional access.~~
4. Custom authorization logic that can’t be modeled as a security group or conditional access policies.  
As mentioned above, Azure AD is constantly releasing more capabilities to close the SSO functional gaps with AD FS, especially around conditional access capabilities.