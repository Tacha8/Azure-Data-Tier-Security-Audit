# Azure Data Tier Security Audit 

## Overview

In this lab, I traced how an application accesses Azure storage and how its connection information is protected in Azure Key Vault. I reviewed the storage account configuration, checked the assigned identity permissions, inspected the Private Endpoint, and verified where the application retrieves its connection string.

I also investigated a second storage account in the same resource group and identified a public blob exposure. This showed me why securing the primary storage account is not enough if another copy of the data is left publicly accessible.

## Checkpoint 1: Identify the Storage Account

I opened the primary storage account and reviewed its JSON configuration to identify the Blob service endpoint.

The Blob endpoint is important because it identifies the storage service the application communicates with. It can also be useful when investigating connection strings, DNS activity, and application configuration.

**Screenshot:**

<img width="552" height="556" alt="image" src="https://github.com/user-attachments/assets/e67d957c-b7a0-416c-a152-de5e1a14aa59" />


## Checkpoint 2: Review Storage Account Security and RBAC

I reviewed the storage account's Configuration settings to check its authentication and data-access posture.

The following settings were visible:

- Secure transfer required: Enabled
- Allow Blob anonymous access: Disabled
- Allow storage account key access: Disabled

Disabling shared-key access helps prevent users or applications from authenticating with storage account keys instead of Microsoft Entra ID. Disabling anonymous Blob access also reduces the risk of unauthenticated access to storage data.

I also exported the role assignments CSV and identified the **Storage Blob Data Contributor** role assigned to the application workload identity. This role provides permissions to read and write Blob data.

This demonstrated that authentication controls and RBAC permissions serve different purposes. The authentication configuration determines which access methods are allowed, while the assigned role determines what the identity can do.

**Screenshots:**

<img width="950" height="638" alt="image" src="https://github.com/user-attachments/assets/2b072368-a07d-400a-ac86-bbfe4552e4fd" />


<img width="911" height="622" alt="image" src="https://github.com/user-attachments/assets/b22651da-eac3-44a9-a2e8-5f9ace8f5ddb" />


## Checkpoint 3: Inspect the Private Endpoint

I inspected the network interface associated with the storage account's Private Endpoint and confirmed that it had a private IPv4 address.

The storage account had public network access disabled, and the Private Endpoint provided the intended private connectivity path from the virtual network.

This matters because knowing a storage account's hostname or having a connection string does not automatically make the storage account reachable from outside the approved network path.

Private connectivity also does not replace identity-based authorization. The application still needs the appropriate permissions to access the data.

**Screenshot:**

<img width="755" height="250" alt="image" src="https://github.com/user-attachments/assets/3710e6ba-2f7e-4655-8c79-4d925e911118" />


## Checkpoint 4: Trace the Connection String to Key Vault

I opened Azure Key Vault and located the `app-db-connection` secret. The secret was enabled, and I used the authorized access provided for the lab to inspect it.

This demonstrated how an application can retrieve connection information from Key Vault instead of storing the actual value directly in its source code or configuration files.

In a secure design, the application can authenticate to Key Vault using a managed identity with the appropriate permissions. The secret value should not be committed to a public repository.

**Screenshot:**

<img width="721" height="386" alt="image" src="https://github.com/user-attachments/assets/18ec752d-223e-4b7e-a916-01f0b2b07822" />


## Checkpoint 5: Investigate the Public Backup Storage Account

I investigated a second storage account in the same resource group and found that its network configuration differed from the hardened primary storage account.

The backup account had public network access enabled, and Blob public access was allowed. The lab instructions then demonstrated how the public container could be enumerated without signing in to Azure.

This was the main security finding of the lab. Even though the primary storage account used stronger security controls, a separate copy of the data had been left exposed through another storage account.

This reinforced the importance of auditing every storage location, including temporary exports and backup accounts, rather than reviewing only the primary production resource.


## Key Takeaways

- Disabling shared-key access helps enforce identity-based authentication for Azure Storage.
- Disabling anonymous Blob access reduces the risk of unauthenticated data exposure.
- Azure RBAC determines which operations an identity can perform on storage data.
- Private Endpoints provide private connectivity to supported Azure services.
- Key Vault provides a centralized place to store application secrets.
- A connection string alone does not guarantee network connectivity or authorization.
- Every storage account and data copy must be reviewed for public exposure.
- Temporary backup and migration resources should be secured and removed when no longer needed.

## Conclusion

This lab helped me connect storage security, identity permissions, private networking, and secret management into one investigation.

The most important lesson was that security controls must cover every location where data exists. A properly secured production storage account does not prevent a data exposure if a separate backup account is configured for public access.

By tracing the application’s access path and investigating the second storage account, I gained practical experience reviewing Azure data-tier security and identifying a configuration that could expose data to unauthenticated users.
