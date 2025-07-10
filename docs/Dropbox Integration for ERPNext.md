# Dropbox Integration for ERPNext

This guide covers the **straightforward setup** of Dropbox integration in ERPNext, plus a reference to detailed troubleshooting steps.

## Overview

* **Objective:** Connect ERPNext to Dropbox for file backups and storage.
* **Files:**

  * **Dropbox-Integration-Guide.md** (this file) – core setup instructions.
  * **Dropbox-Error-Troubleshooting.md** – error diagnosis and fix.

> ⚠️ *If you encounter an `Invalid redirect_uri` error, see [Troubleshooting Guide](./Dropbox-Error-Troubleshooting.md).*

---

## 1. Prerequisites

* An ERPNext instance reachable at `https://<your-domain>`.
* Docker‐Compose deployed ERPNext (Backend container available).
* A Dropbox account to register an API app.

---

## 2. Create a Dropbox API v2 App

1. Visit the [Dropbox App Console](https://www.dropbox.com/developers/apps) and click **Create App**.
2. Under **Choose an API**, select **Scoped Access**.
3. Under **Choose the type of access you need**, pick **App Folder** (recommended) or **Full Dropbox**.
4. Name your app (e.g., `ERPNext Integration`) and click **Create App**.

---

## 3. Configure App Settings

1. In your new app’s **Settings**:

   * Copy the **App key**.
   * Copy the **App secret**.
2. Under **OAuth 2** → **Redirect URIs**, click **Add**, then enter:

   ```text
   https://<your-domain>/api/method/
   frappe.integrations.doctype.dropbox_settings.dropbox_settings.dropbox_auth_finish
   ```

   * Ensure you include `https://` and the full path.
3. Click **Save** to apply the redirect URI.

---

## 4. Configure ERPNext

1. In your browser, go to:

   ```text
   https://<your-domain>/app/dropbox-settings
   ```
2. Paste the **App key** and **App secret** into their respective fields.
3. Click **Save** (do **not** click **Allow Dropbox Access** yet).

---

## 5. Authorize Dropbox Access

1. After saving, click **Allow Dropbox Access**.
2. If everything is correct, you’ll be redirected to Dropbox and back to ERPNext with a success message.

---

## Next Steps

* **If successful**, Dropbox integration is active—go to **Setup → Dropbox Settings** to confirm.
* **If you see an error**, proceed to the [Troubleshooting Guide](./Dropbox-Error-Troubleshooting.md).

*Last updated: July 2025*
