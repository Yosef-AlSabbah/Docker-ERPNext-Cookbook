# Fix: Frappe Email Pull Scheduler Auto-Sync Issue

## 🧩 Problem

In a Dockerized Frappe/ERPNext production setup, the system was **not pulling incoming emails automatically**, although:

- Email Account was correctly configured.
- IMAP/SMTP settings were valid.
- Scheduler was running without errors.
- **Manual pull via the UI (Email Account > Pull Emails)** worked fine.
- The expected method `frappe.email.doctype.email_account.email_account.pull` was not being triggered automatically.

---

## 🔍 Root Cause

This issue was caused by a **startup sequence race condition**:

> The **Scheduler container started before the Queue workers**, which are responsible for executing background jobs.

As a result:
- Scheduled jobs like `frappe.email.doctype.email_account.email_account.pull` were enqueued when **no worker was available**, so they were silently dropped or never processed.
- The system appeared healthy, but emails didn’t sync unless pulled manually from the Email Account page.

---

## ✅ Solution

The fix was simple but effective:

> **Restart the queue worker containers after everything is up.**

```bash
docker restart erpnext-one-queue-short-1
docker restart erpnext-one-queue-long-1
````

Once restarted:

* Queue workers connected to Redis properly.
* The scheduler was able to enqueue jobs and get them executed.
* Email pulling resumed automatically as expected.

---

## 🛡️ Recommendation

To avoid this in future deployments:

1. ✅ Use `depends_on` with meaningful `condition` values:

   ```yaml
   scheduler:
     depends_on:
       redis-queue:
         condition: service_healthy
       queue-short:
         condition: service_started
       queue-long:
         condition: service_started
   ```

2. ✅ Add health checks for Redis and queue containers.

3. ✅ Consider delaying scheduler startup (using `sleep` or a wait script) until queues are fully ready.

4. ✅ Always confirm workers are connected and listening to the queue before relying on the scheduler.

---

## ✅ Final Status

After restarting the queue containers:

* Automatic email pulling via the scheduler started working.
* No changes were needed in the Email Account configuration.
* Email sending and receiving are now fully operational.
* Method `frappe.email.doctype.email_account.email_account.pull` is now triggered as expected by the scheduler.

---

## 🔗 References

* [Frappe Scheduler Docs](https://frappeframework.com/docs/user/en/tutorial/scheduler)
* [Email Account Setup](https://frappeframework.com/docs/user/en/setting-up/email/email-account)
* [Docker Compose `depends_on`](https://docs.docker.com/compose/compose-file/compose-file-v3/#depends_on)

---

> *Authored by Yousef M. Y. Al Sabbah – August 2025*
Would you like me to generate this now as a `.md` file for you to download and commit to GitHub?
```
