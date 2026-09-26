Azure Managed Identity Adoption – Zero Trust Application
📌 Project Overview

This project demonstrates how an application can securely access Azure resources without storing passwords, connection strings, storage keys, or service-principal secrets inside the application code.

The solution uses Azure Managed Identity, Microsoft Entra ID, and Azure RBAC to provide secure, identity-based access to Azure Key Vault and Azure Blob Storage.

The project follows a Zero Trust approach:

Never trust automatically. Always verify the application's identity and permissions before granting access.

🎯 Problem Statement

Traditional applications often store credentials such as:

Database passwords
Storage account keys
API keys
Service principal secrets
Connection strings

inside configuration files, environment variables, or application code.

This creates security risks because leaked credentials can potentially be reused to access cloud resources.

This project addresses the problem by removing the need for the application to store Azure credentials.

💡 Proposed Solution

We assign a System-Assigned Managed Identity to the Azure App Service.

The application uses this identity when communicating with Azure services.

Azure then verifies:

Who is requesting access?
Does that identity have permission?
Which resource is it allowed to access?

Only the required permissions are granted using Azure RBAC.

🏗️ Architecture
                         ┌───────────────────┐
                         │       User        │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │   Azure App       │
                         │     Service       │
                         │  Spring Boot App  │
                         └─────────┬─────────┘
                                   │
                         Managed Identity
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Microsoft Entra ID│
                         │   Authentication  │
                         └─────────┬─────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
          ┌──────────────────┐          ┌──────────────────┐
          │   Azure Key Vault │          │ Azure Blob       │
          │                  │          │ Storage          │
          │ Database Secret  │          │ Audit Log        │
          └──────────────────┘          └──────────────────┘
🔐 Key Security Concept

The main idea of this project is:

The application has an identity instead of carrying a password.

For example, instead of putting a storage key inside the application:

Storage Account Key = XXXXXXXX

the application uses its Azure Managed Identity.

Azure checks the identity and its RBAC permissions before allowing access.

⭐ What Makes This Project Different?

Traditional RBAC-based applications, such as university ERP systems, generally use roles to control what users can do inside an application.

This project focuses on what the application itself is allowed to access.

Traditional Application
User
  ↓
Application
  ↓
Role
  ↓
Allowed actions
Our Application
Application
     ↓
Managed Identity
     ↓
Microsoft Entra ID
     ↓
RBAC Permission
     ↓
Azure Resource

Therefore, the project demonstrates application-to-resource security, rather than only user-to-application authorization.

🚀 Technologies Used
Technology	Purpose
Java	Application development
Spring Boot	Backend application
Azure App Service	Application hosting
Azure Managed Identity	Application identity
Microsoft Entra ID	Identity authentication
Azure RBAC	Authorization
Azure Key Vault	Secure secret storage
Azure Blob Storage	Secure cloud storage
Azure SDK	Azure service integration
Maven	Project build and dependency management
Git & GitHub	Version control
🔑 Azure Services
1. Azure App Service

Hosts the Spring Boot application in Azure.

Application:

app-secops-zero-trust
2. Managed Identity

A System-Assigned Managed Identity is enabled for the App Service.

This gives the application an Azure identity without requiring credentials to be stored in the source code.

3. Azure Key Vault

Stores sensitive information securely.

Example:

DatabaseConnectionString

The application retrieves the secret through its managed identity.

4. Azure Blob Storage

Stores application data such as:

audit-log.txt

The application accesses the blob using its Azure identity and RBAC permissions.

🔒 Access Control

The application is given only the permissions required for its operation.

For example:

App Service Managed Identity
          │
          ├── Key Vault
          │      └── Read required secret
          │
          └── Blob Storage
                 └── Read required blob

This follows the principle of least privilege.

🧩 Application Endpoints

The Spring Boot application provides the following endpoints:

Endpoint	Purpose
/	Application home/status
/health	Health check
/secret	Demonstrates Key Vault access
/storage	Demonstrates Blob Storage access

Example:

GET /health

Response:

UP

The /secret endpoint demonstrates that the application can retrieve a value from Azure Key Vault without having the Key Vault credential stored in the application.

The /storage endpoint demonstrates access to Azure Blob Storage through the application's Azure identity.

💻 Authentication in the Application

The application uses Azure's credential mechanism rather than hardcoding Azure credentials.

Example:

DefaultAzureCredential credential =
        new DefaultAzureCredentialBuilder().build();

This allows the application to obtain credentials from the Azure environment without embedding a password or service-principal secret in the source code.

🔄 How the Application Works
Step 1 – Application starts

The Spring Boot application runs on Azure App Service.

Step 2 – Azure provides identity

The App Service has a System-Assigned Managed Identity.

Step 3 – Application requests access

The application requests access to Key Vault or Blob Storage.

Step 4 – Identity is verified

Microsoft Entra ID authenticates the application's identity.

Step 5 – RBAC is checked

Azure checks whether the identity has the required permission.

Step 6 – Resource is accessed

If permission is available:

Application → Key Vault ✅
Application → Blob Storage ✅

If permission is not available:

Application → Unauthorized Resource ❌
🛡️ Zero Trust Approach

The project demonstrates Zero Trust principles through:

Verify Identity

The application must authenticate using its managed identity.

Verify Permission

Azure RBAC determines whether the application can access the requested resource.

Least Privilege

The application receives only the permissions required for its functionality.

No Stored Azure Credentials

Azure credentials are not hardcoded into the application.

📊 Security Comparison
Traditional Approach	This Project
Store credentials	Use Managed Identity
Storage keys may be stored	No storage key required
Service principal secret may be required	No stored service principal secret
Application carries credentials	Azure provides application identity
Broad permissions may be configured	Least-privilege RBAC
Credential rotation required	Managed Identity avoids application-managed credentials
🧪 Demonstration

The project can be demonstrated using the following flow:

1. Open Azure App Service
          ↓
2. Show Managed Identity = Enabled
          ↓
3. Show Key Vault
          ↓
4. Show RBAC permission
          ↓
5. Show Storage Account
          ↓
6. Show Blob Storage
          ↓
7. Open /health
          ↓
8. Open /secret
          ↓
9. Open /storage

This demonstrates that the deployed application can securely communicate with Azure resources using its identity.
