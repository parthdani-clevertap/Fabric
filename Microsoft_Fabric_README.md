# Microsoft Fabric

Learn how configuring Microsoft Fabric with CleverTap enables data sync for personalized engagement and growth.

---

## Overview

Configuring Microsoft Fabric with CleverTap enables seamless data import, ensuring synchronization and access to relevant information for analysis, personalized engagement, and data-driven growth.

> 📘 **Private Beta**
>
> Microsoft Fabric is a Private Beta release. Microsoft Fabric integration supports data import into CleverTap. Export capabilities will be available in a future release. Contact your Customer Success Manager for access.

---

## Quick Start Guide for Existing Users

<details>
<summary><b>Expand for quick setup if you have previously configured a Microsoft Fabric workspace and are familiar with the CleverTap dashboard.</b> If you are setting up Microsoft Fabric for the first time, skip to <a href="#prerequisites-for-integration">Prerequisites for Integration</a> below.</summary>

### Prerequisites

Before you begin, ensure you have the following details:

- **Connection Nickname**: A unique name to identify this configuration in CleverTap.
- **Host**: Microsoft Fabric SQL connection string endpoint (found under Warehouse/Lakehouse → Settings → SQL endpoint).
- **Port**: TDS port (defaults to `1433` if not specified).
- **Client ID**: Application (Client) ID from the Microsoft Entra app registration used for authentication.
- **Tenant ID**: Directory (Tenant) ID of your Microsoft Entra ID tenant.
- **Database**: The name of the Fabric Warehouse or Lakehouse SQL analytics endpoint.
- **Schema**: The specific schema within the database for CleverTap tables (defaults to `dbo`).

### Configure Microsoft Fabric Credentials in CleverTap

To set up the Microsoft Fabric credentials in CleverTap, perform the following steps:

1. Go to **CleverTap Dashboard > Settings > Partners > Microsoft Fabric**.
2. Click **Add Database**.
3. Enter the following details: Connection Nickname, Host, Port, Client ID, Tenant ID, Database, and Schema.
4. Click **Test Connection** to verify the configuration.
5. Click **Save** to store the connection.

After setting up the configuration, you can import data from Microsoft Fabric into CleverTap.

</details>

---

## Prerequisites for Integration

If you are setting up Microsoft Fabric for the first time, ensure you have the following before proceeding with the CleverTap configuration:

- CleverTap **Access** to configure Microsoft Fabric.

- **Microsoft Fabric Workspace Details**:

  - **Host**: SQL connection string endpoint (for example, `abc1def2ghij3klmn.datawarehouse.fabric.microsoft.com`). Do not include the protocol (`https://`) or the port number.

  - **Port**: TDS port used to reach Microsoft Fabric. Defaults to `1433` if not specified. Customers behind corporate proxies may need to consult their network administrator for the correct port.

  - **Database**: The name of the Fabric Warehouse or Lakehouse SQL analytics endpoint you plan to import data from.

  - **Schema**: The schema within the database that contains the tables CleverTap will read from. Defaults to `dbo` if not specified.

- **Microsoft Entra ID (Azure AD) Credentials**:

  - **Client ID**: The Application (Client) ID from a Microsoft Entra app registration configured as a service principal.

  - **Tenant ID**: The Directory (Tenant) ID of your Microsoft Entra ID tenant.

- **Microsoft Fabric Identity and Permissions**

  The service principal associated with the Client ID must have, at minimum:

  - Membership in a Fabric Workspace with **Contributor** (or higher) role.
  - **CONNECT** privilege on the target database.
  - **SELECT** permission on tables you plan to import.

> 📘 **Security**
>
> Use a dedicated Microsoft Entra app registration for CleverTap, grant least privilege, and set an expiry/rotation policy for the client secret.

> 📘 **IP Whitelisting**
>
> To ensure seamless communication between CleverTap and your Microsoft Fabric workspace, whitelist the required CleverTap IP ranges in your network and Fabric workspace settings. Refer to CleverTap IP Ranges ([Import FAQs](</Import FAQs link>)) for the complete list.

---

## Set Up Microsoft Fabric for Integration

You can set up Microsoft Fabric using one of the following ways:

- For new users who need to provision Microsoft Fabric resources and Azure credentials from scratch: [Create a New Microsoft Fabric Setup](#create-new-microsoft-fabric-setup).
- For users who already have configured resources in Microsoft Fabric: [Use Existing Microsoft Fabric Credentials](#use-existing-microsoft-fabric-credentials).

---

## Create New Microsoft Fabric Setup

If you do not already have a **Fabric Warehouse or Lakehouse**, **Microsoft Entra App Registration**, and **Schema** configured, you must create them before proceeding. These components are required to ensure CleverTap can securely access your data.

The following table explains each component and its role in the integration:

| Field | Description |
|---|---|
| **Fabric Warehouse / Lakehouse** | A data store in Microsoft Fabric that holds your tables. CleverTap connects to the SQL analytics endpoint of your Warehouse or Lakehouse to read data. |
| **Microsoft Entra App Registration** | An application identity registered in Microsoft Entra ID (Azure AD). It provides the Client ID and Tenant ID that CleverTap uses to authenticate via service principal. |
| **Client Secret** | A secret credential generated for the app registration, used alongside the Client ID to authenticate CleverTap to your Fabric workspace. |
| **Schema** | A logical grouping within a database that contains tables. CleverTap reads from tables within the schema you specify (default: `dbo`). |

To create each resource, perform the following steps:

1. [Create a Fabric Warehouse](#create-a-fabric-warehouse)
2. [Register an App in Microsoft Entra ID](#register-an-app-in-microsoft-entra-id)
3. [Create a Client Secret](#create-a-client-secret)
4. [Create a Schema (Optional)](#create-a-schema-optional)
5. [Configure Permissions](#configure-permissions)

---

### Create a Fabric Warehouse

CleverTap reads data through a Microsoft Fabric Warehouse or Lakehouse SQL analytics endpoint. Create a dedicated Warehouse for CleverTap data to keep your import workloads isolated.

> ⚠️ **Prerequisite**
>
> Creating a Warehouse requires access to a Microsoft Fabric workspace with a Fabric capacity (F64 or higher) or a Power BI Premium capacity (P1 or higher). Contact your Fabric administrator if you do not have this access.

To create a Warehouse via the Microsoft Fabric portal, perform the following steps:

1. Sign in to the [Microsoft Fabric portal](https://app.fabric.microsoft.com/).
2. Navigate to your target **Workspace** from the left sidebar.
3. Click **+ New Item** and select **Warehouse** under the Data Warehouse section.
4. Enter a descriptive **Name** (for example, `clevertap_warehouse`).
5. Click **Create**.
6. After the Warehouse is created, it opens in the Fabric portal. Note the Warehouse name — this is the **Database** value you will enter in CleverTap.

To obtain the **Host** value:

1. In your Workspace, locate the Warehouse you just created.
2. Click the **three dots (…)** next to the Warehouse name and select **Copy SQL connection string**.
3. The connection string contains the Host value in the format:
   ```
   abc1def2ghij3klmn.datawarehouse.fabric.microsoft.com
   ```
4. Copy only the server name portion (without the protocol or port). You will need this when configuring CleverTap.

Alternatively, you can find the connection string by opening the Warehouse, navigating to the **Settings** gear icon, and selecting the **SQL endpoint** page.

> 📘 **Note**
>
> You can also use a **Lakehouse SQL analytics endpoint** instead of a Warehouse. Every Lakehouse in Fabric automatically provisions a SQL analytics endpoint. To find its connection string, select the Lakehouse SQL analytics endpoint item in your workspace and navigate to **Settings > SQL endpoint**.

---

### Register an App in Microsoft Entra ID

CleverTap authenticates with Microsoft Fabric using a service principal. You must register an application in Microsoft Entra ID (formerly Azure Active Directory) to obtain the **Client ID** and **Tenant ID**.

To register an app, perform the following steps:

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Search for and select **Microsoft Entra ID** from the top search bar.
3. In the left menu, select **App registrations**.
4. Click **+ New registration** at the top of the page.
5. Configure the registration:
   - **Name**: Enter a descriptive name (for example, `CleverTap Fabric Integration`).
   - **Supported account types**: Select **Accounts in this organizational directory only** (Single tenant).
   - **Redirect URI**: Leave blank (not required for this integration).
6. Click **Register**.
7. On the app's **Overview** page, copy and securely store the following values:
   - **Application (client) ID** — this is the **Client ID** you will enter in CleverTap.
   - **Directory (tenant) ID** — this is the **Tenant ID** you will enter in CleverTap.

![Microsoft Entra App Registration Overview — showing Application (client) ID and Directory (tenant) ID](<!-- image placeholder: screenshot of Azure portal app registration overview page -->)

---

### Create a Client Secret

After registering the app, generate a client secret that CleverTap will use to authenticate.

To generate a client secret, perform the following steps:

1. In the Azure portal, navigate to your app registration (Microsoft Entra ID → App registrations → your app).
2. In the left menu, select **Certificates & secrets**.
3. Under the **Client secrets** tab, click **+ New client secret**.
4. Enter a **Description** (for example, `CleverTap Integration`) to identify the secret.
5. Set the **Expires** duration. For production integrations, a minimum of 6 months is recommended.
6. Click **Add**.
7. Copy the **Value** of the client secret immediately and store it securely. **The secret value is displayed only once.** You will provide this to CleverTap during connection setup.

> ⚠️ **Secret Expiration**
>
> Client secrets expire based on the duration you set during creation. When a secret expires, the CleverTap connection will fail, and imports will stop. Monitor secret expiration dates and regenerate secrets before they expire. Consider setting a calendar reminder.

---

### Create a Schema (Optional)

Microsoft Fabric Warehouses include a default `dbo` schema. If you want to organize CleverTap-related tables under a separate schema, you can create one.

To create a schema, perform the following steps:

1. Open your Warehouse in the Microsoft Fabric portal.
2. Click **New SQL query** from the top toolbar.
3. Run the following SQL command to create a schema:

```sql
-- Create a new schema for CleverTap data
CREATE SCHEMA IF NOT EXISTS clevertap_schema;
```

4. Verify the schema by expanding the **Schemas** folder in the Object Explorer, or by running:

```sql
-- List all schemas in the current database
SELECT name FROM sys.schemas ORDER BY name;
```

**Expected Output:**

```
+--------------------+
| name               |
+--------------------+
| clevertap_schema   |
| dbo                |
+--------------------+
```

> 📘 **Note**
>
> If you do not create a custom schema, CleverTap will read from the default `dbo` schema. Specify the schema name during the CleverTap connection setup.

---

### Configure Permissions

To ensure the service principal (app registration) can access your Fabric data, you must grant the required permissions. This step is essential for CleverTap to read data from your Microsoft Fabric workspace.

**Step 1: Enable Service Principals in Fabric Admin Portal**

1. Sign in to the [Fabric Admin portal](https://app.fabric.microsoft.com/admin-portal).
2. Navigate to **Tenant settings**.
3. Locate **Service principals can use Fabric APIs** and enable it for the security group containing your service principal.
4. Ensure **Users can access data stored in OneLake with apps external to Fabric** is also enabled under **OneLake settings**.

**Step 2: Add Service Principal to Workspace**

1. In the Microsoft Fabric portal, navigate to your target Workspace.
2. Click **Manage access** (or the **Settings** gear icon > **Manage access**).
3. Click **+ Add people or groups**.
4. Search for your app registration name (for example, `CleverTap Fabric Integration`).
5. Assign the **Contributor** role (minimum required for data read access).
6. Click **Add**.

**Step 3: Grant Database-Level Permissions (if required)**

If granular table-level permissions are needed, run the following SQL commands in your Warehouse:

```sql
-- Grant CONNECT access to the service principal
GRANT CONNECT TO [<service_principal_name>];

-- Grant SELECT access on a specific schema
GRANT SELECT ON SCHEMA::dbo TO [<service_principal_name>];
```

> 📘 **Granular Table Permissions**
>
> If you prefer to grant access to specific tables rather than the entire schema, use the following command instead:
> ```sql
> GRANT SELECT ON [dbo].[table_name] TO [<service_principal_name>];
> ```
> Repeat for each table that CleverTap needs to access.

---

## Use Existing Microsoft Fabric Credentials

If you already have Microsoft Fabric set up, perform the following steps to find each required detail.

- [Obtain Host from Warehouse or Lakehouse](#obtain-host-from-warehouse-or-lakehouse)
- [Find Client ID and Tenant ID](#find-client-id-and-tenant-id)
- [Create or Retrieve a Client Secret](#create-or-retrieve-a-client-secret)
- [Find Existing Database](#find-existing-database)
- [Find Existing Schema](#find-existing-schema)
- [Verify Existing Permissions](#verify-existing-permissions)

---

### Obtain Host from Warehouse or Lakehouse

To configure the integration, you need the Microsoft Fabric SQL connection string (Host).

Follow these steps:

1. In the Microsoft Fabric portal, navigate to your **Workspace**.
2. Locate the target **Warehouse** or **Lakehouse SQL analytics endpoint** in the item list.
3. Click the **three dots (…)** next to the item name.
4. Select **Copy SQL connection string**.
5. The copied value looks like: `abc1def2ghij3klmn.datawarehouse.fabric.microsoft.com,1433`.
6. Copy only the host portion (for example, `abc1def2ghij3klmn.datawarehouse.fabric.microsoft.com`). Do not include the port number or protocol.

Alternatively:

1. Open your Warehouse or Lakehouse in the Fabric portal.
2. Click the **Settings** gear icon.
3. Select the **SQL endpoint** page.
4. Copy the SQL connection string value displayed.

---

### Find Client ID and Tenant ID

The Client ID and Tenant ID are obtained from your Microsoft Entra ID (Azure AD) app registration.

**To find the Client ID and Tenant ID:**

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Navigate to **Microsoft Entra ID** > **App registrations**.
3. Select your app registration from the list (for example, `CleverTap Fabric Integration`).
4. On the **Overview** page, locate:
   - **Application (client) ID** — this is the **Client ID**.
   - **Directory (tenant) ID** — this is the **Tenant ID**.
5. Copy both values.

**To find the Tenant ID from the Azure portal (alternative method):**

1. In the Azure portal, select **Microsoft Entra ID** from the left menu or search bar.
2. On the **Overview** page, find the **Tenant ID** in the **Basic information** section.
3. Click the **Copy to clipboard** icon next to the Tenant ID.

---

### Create or Retrieve a Client Secret

To authenticate CleverTap with Microsoft Fabric, you need an active client secret for your app registration.

1. In the Azure portal, navigate to **Microsoft Entra ID** > **App registrations** > your app.
2. Select **Certificates & secrets** from the left menu.
3. Under **Client secrets**, check if an active (non-expired) secret exists.
4. If you need a new secret, click **+ New client secret**, enter a description and expiry, then click **Add**.
5. Copy the secret **Value** immediately and store it securely. **The value is displayed only once.**

> 📘 **Secret Rotation**
>
> If your organization requires periodic credential rotation, generate a new client secret before the current one expires. Update the CleverTap connection with the new secret to avoid import interruptions. Refer to [Microsoft Entra documentation on App Registration secrets](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app) for detailed guidance.

---

### Find Existing Database

The **Database** value is the name of your Fabric Warehouse or Lakehouse SQL analytics endpoint.

1. In the Microsoft Fabric portal, navigate to your **Workspace**.
2. Look for items of type **Warehouse** or **SQL analytics endpoint** in the item list.
3. The item name is your **Database** value.

Alternatively, you can find the database name from the SQL connection string. When you copy the connection string (from Settings > SQL endpoint), the **Initial Catalog** parameter contains the database name.

---

### Find Existing Schema

To find existing schemas in your Warehouse or Lakehouse, perform the following steps:

1. Open your Warehouse in the Microsoft Fabric portal.
2. In the **Object Explorer** panel, expand the database node to view schemas.
3. The default schema is `dbo`. Any custom schemas will also be listed.

Alternatively, run the following SQL query:

```sql
SELECT name FROM sys.schemas ORDER BY name;
```

---

### Verify Existing Permissions

Ensure the service principal associated with your app registration has the required permissions:

1. Confirm the service principal has **Contributor** (or higher) access to the Fabric Workspace via **Manage access**.
2. Verify database-level permissions by running:

```sql
-- Check current permissions for the service principal
SELECT dp.name, dp.type_desc, perm.permission_name, perm.state_desc
FROM sys.database_permissions perm
JOIN sys.database_principals dp ON perm.grantee_principal_id = dp.principal_id
WHERE dp.name = '<service_principal_name>';
```

If the required permissions (CONNECT and SELECT) are not present, refer to the [Configure Permissions](#configure-permissions) section above to grant them.

---

## Set Up CleverTap Dashboard for Integration

You have already prepared your Microsoft Fabric environment and gathered the required values (Host, Port, Client ID, Tenant ID, Database, and Schema). In this section, you will add a Microsoft Fabric connection in CleverTap, enter these parameters, validate the connection, and proceed to create an import.

![CleverTap Settings — Connect to Microsoft Fabric for import](<!-- image placeholder: screenshot of CleverTap dashboard showing Microsoft Fabric connection form with fields for Connection Nickname, Host, Port, Client ID, Tenant ID, Database, and Schema -->)

### Connection Details for Microsoft Fabric

To connect Microsoft Fabric with CleverTap, go to **Settings > Partners > Microsoft Fabric** and select **Add Database**. To create or retrieve details from your Microsoft Fabric workspace, refer to [Create a New Microsoft Fabric Setup](#create-new-microsoft-fabric-setup) or [Use Existing Microsoft Fabric Credentials](#use-existing-microsoft-fabric-credentials) and configure the following:

| Field | Description |
|---|---|
| **Connection Nickname** | A unique name to identify your configuration while setting up imports. Choose a descriptive name (for example, "Fabric Production" or "Fabric Analytics"). |
| **Host** | The Microsoft Fabric SQL connection string endpoint (domain only; do not include the protocol or port). For example: `abc1def2ghij3klmn.datawarehouse.fabric.microsoft.com`. Refer to [Obtain Host from Warehouse or Lakehouse](#obtain-host-from-warehouse-or-lakehouse). |
| **Port** | TDS port used to reach Microsoft Fabric. If a port is specified, CleverTap uses it; otherwise, the connection defaults to `1433`. |
| **Client ID** | The Application (Client) ID from your Microsoft Entra app registration. Refer to [Find Client ID and Tenant ID](#find-client-id-and-tenant-id). |
| **Tenant ID** | The Directory (Tenant) ID of your Microsoft Entra ID tenant. Refer to [Find Client ID and Tenant ID](#find-client-id-and-tenant-id). |
| **Database** | The name of the Fabric Warehouse or Lakehouse SQL analytics endpoint that CleverTap will read from. |
| **Schema** | The specific schema within the database that contains the tables CleverTap will read from. Defaults to `dbo` if not specified. |

---

### Test and Save the Connection

After entering all connection details, complete the setup:

1. Click **Test Connection** to verify that the Host, Port, Client ID, Tenant ID, and privileges are correct.
   - A successful test confirms that CleverTap can reach your Microsoft Fabric workspace and access the specified database and schema.
   - A failed test indicates an issue with one or more parameters. Refer to the [Troubleshooting](#troubleshooting) section below for common errors.
2. Click **Save** to store the connection details.
3. After saving, navigate to the **Import Connections** dashboard and click **Create Import** to set up your first data import.

---

## Troubleshooting

If the Test Connection fails, use the following table to identify and resolve common issues:

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Connection timeout | Network/firewall blocking the connection | Ensure CleverTap IPs are whitelisted. Verify the Host value does not include `https://` or the port number. Confirm TCP port `1433` is open. |
| Authentication failed | Invalid Client ID, Tenant ID, or expired client secret | Verify the Client ID and Tenant ID in the Azure portal. Generate a new client secret if the current one has expired and update the connection in CleverTap. |
| Database not found | Incorrect Database name or Warehouse is paused | Verify the Warehouse or Lakehouse name in the Fabric portal. Ensure the Warehouse is in a running state. |
| Schema not accessible | Missing permissions on the schema | Grant SELECT on the schema to the service principal. See [Configure Permissions](#configure-permissions). |
| Service principal not authorized | Service principal not added to Fabric Workspace | Add the service principal to the Workspace with Contributor role via **Manage access**. Ensure **Service principals can use Fabric APIs** is enabled in the Fabric Admin portal. |
| SELECT permission denied | Missing SELECT permission on tables | Grant SELECT on the schema or specific tables. See [Configure Permissions](#configure-permissions). |

---

## FAQs

**What happens when my client secret expires?**

When the client secret expires, CleverTap will no longer be able to authenticate with Microsoft Fabric, and all active imports will fail. To resolve this, generate a new client secret in the Azure portal under your app registration's **Certificates & secrets** page, then update the connection in CleverTap by editing the connection and replacing the expired credential.

**Can I use the same Microsoft Fabric connection for multiple imports?**

Yes. A single Microsoft Fabric connection in CleverTap can be used to create multiple import configurations, each reading from different tables within the same database and schema.

**What is the default port for Microsoft Fabric?**

Microsoft Fabric uses TDS (Tabular Data Stream) protocol on TCP port **1433**, which is the standard SQL Server port. If you do not specify a port during configuration, CleverTap defaults to `1433`.

**Can I connect to a Lakehouse instead of a Warehouse?**

Yes. Every Lakehouse in Microsoft Fabric automatically provisions a SQL analytics endpoint. You can use the SQL analytics endpoint connection string as the Host value and the Lakehouse name as the Database value. The SQL analytics endpoint provides read-only T-SQL access to your Lakehouse Delta tables.

**Where do I find the SQL connection string?**

You can find the SQL connection string in the Microsoft Fabric portal by clicking the three dots (…) next to your Warehouse or Lakehouse SQL analytics endpoint and selecting **Copy SQL connection string**. Alternatively, open the item's **Settings** and navigate to the **SQL endpoint** page.

**What permissions does the service principal need?**

At minimum, the service principal needs Contributor role membership in the Fabric Workspace and CONNECT + SELECT privileges on the target database and tables. Service principals must also be enabled in the Fabric Admin portal under **Tenant settings > Service principals can use Fabric APIs**.

---

## Next Steps

[Import Data to CleverTap](</Import Data link>)
