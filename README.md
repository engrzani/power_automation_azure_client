
## 🪪 Step 1 — Create an Azure Automation Account

**Purpose:** This account will securely host our PowerShell runbooks.

**Actions:**

1. Sign in to the [Azure Portal](https://portal.azure.com/).
2. Search for **Automation Accounts** → **Create**.
3. Fill details:

   * **Name:** `OnboardingAutomation`
   * **Region:** same as your M365 tenant (e.g., East US)
   * **Run As:** ✅ **System Assigned Managed Identity**
4. **Create** → Wait for deployment to finish.

---

## ⚙️ Step 2 — Install Required PowerShell Modules

**Purpose:** So the runbooks can call Microsoft Graph and Exchange Online.

**Actions:**

1. Open the created Automation Account.
2. Go to **Modules** → **Browse gallery**.
3. Install:

   ```
   Microsoft.Graph
   ExchangeOnlineManagement
   ```

> 📝 This might take 5–10 minutes to complete.

---

## 🛡️ Step 3 — Grant Minimum Permissions to the Managed Identity

**Purpose:** Give only the needed rights (as the client requested).

**Actions:**

1. Go to **Azure Active Directory → Enterprise applications → Managed identities.**
2. Find your Automation Account’s Managed Identity.
3. Assign these **roles**:

   * **User Administrator**
   * **Groups Administrator** (or `Group.ReadWrite.All` Graph App role)
   * **License Administrator**
   * **Exchange Administrator** (only for Send-As permissions)

---

## 📁 Step 4 — Import Runbooks

**Purpose:** Upload the `.ps1` files you downloaded in the zip.

**Actions:**

1. In your Automation Account → **Runbooks** → **Create a runbook**
2. Name: `CreateUniqueUserID`, Type: **PowerShell**
3. Paste contents of `CreateUniqueUserID.ps1`
4. **Save** → **Publish**

Repeat this for each file:

* `CreateUser.ps1`
* `CheckAndAssignLicense.ps1`
* `CreateOrEnsureGroup.ps1`
* `AddGroupMembers.ps1`
* `AssignSendAs.ps1`

---

## 🌐 Step 5 — Create Webhooks for Power Automate

**Purpose:** Allow Power Automate to trigger these runbooks with HTTP POST.

**Actions:**

1. Open each runbook
2. Click **Webhook → Create new webhook**
3. Copy and save the **webhook URL** securely
4. Expiry: Max allowed (1 year)

---

## ⚡ Step 6 — Update Power Automate Flows

**Purpose:** Integrate each runbook into your onboarding flow.

**Example connectors:**

* Use the **HTTP** action
* Method: `POST`
* URL: webhook URL
* Headers:

  ```
  Content-Type: application/json
  ```
* Body examples (from `PowerAutomate_HTTP_examples.json`):

```json
{ "FirstName": "John", "LastName": "Doe", "Domain": "contoso.com" }
```

Then parse the response JSON (use “Parse JSON” action).

---

## 💬 Step 7 — Teams Integration Enhancement

**Purpose:** Include the onboarding form’s attachment.

**Actions:**

1. After the form submission trigger:
2. Use **Get response details** to fetch the file content.
3. Use the **Teams → Post adaptive card in a chat or channel** action.
4. In card body, include:

   * Submitter info
   * Attachment link or file content (upload using **Create file** in OneDrive/SharePoint first, then include the link)

---

## 🛡️ Step 8 — Testing Checklist

1. Submit the onboarding form.
2. Verify:

   * Unique UPN is generated
   * User created
   * Business Basic license assigned (or fails gracefully if none available)
   * “Accounts Team 1” group exists with Will + Chai
   * Send-As permission given to this group on new mailbox
   * Teams message posts with attachment and submitter info

---

## 📌 Step 9 — Documentation

* The `README.md` inside the zip is your permanent documentation.
* Save webhook URLs and identities securely (Key Vault or SharePoint secure site).

Json template is attached as well
