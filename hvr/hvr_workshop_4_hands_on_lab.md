# Hands-on Lab: HVR Workshop 4 — Alerts and Access Control

*Workshop 4 of 5. Adapted from the HVR workshop guide for the LabStudios HVR lab; validate with the provisioned lab resources before delivery.*

This workshop uses the hub you set up in [Workshop 1](hvr_workshop_1_hands_on_lab.md). Log in to your hub with the **HVR Hub URL**, **HVR Username** and **HVR Password** from your 1Password credential link.

---

## About this workshop

**Alerts.** HVR raises alerts through maintenance tasks that are scheduled outside HVR to regularly check the health of the system, the current latency, and whether there have been errors or warnings since the last check.

HVR is designed to recover automatically from interruptions to replication and always resumes where it left off. Once configured, HVR can be left to manage replication while you periodically look at the Topology chart or Statistics. But some failures can't be resumed without intervention, for example a network that's unavailable for an extended time, a system crash caused by a faulty part, or a software bug in HVR or a technology HVR integrates with. Alerts tell you when that happens.

**Access control.** Controlling access to every application is critically important: the wrong access could allow malicious or accidental changes with deep consequences. HVR lets you create users and control their access to the HVR UI.

In this workshop you'll explore the Alerts page and create an alert, then create an HVR user and control that user's access.

---

## Part 1: Create an alert

1. At the bottom of the navigation bar on the left are three icons. Click the middle one, the gear icon. This opens the **System** page, with information on the current hub, **Permissions**, **Users** and **Alerts**.
2. Click the **Alerts** tab on the far right. No alerts have been defined yet, so none are listed.
3. Click **Create New Alert** on the right side of the page. The **New Alert** window opens; this is where you define an alert and how its notification is sent.
4. Fill in the alert. These are placeholder values: your lab has no mail server, so no notification is actually sent.
   - **Alert name:** `test` (it must be lower case)
   - Leave **Send Alerts on both Errors and Warnings** selected.
   - Leave the notification type set to **Email**.
   - **Recipients:** `demo@demo.com`
   - **Server:** `localhost`
   - **User:** `demo`
   - **Password:** any placeholder, for example `demo`
5. Scroll to the bottom of the window and click **Save**.

You're back on the **System › Alerts** page, with your new alert listed.

## Part 2: Create a user and grant access

1. On the **System** page, click the **Users** tab.
2. Click **Add User** on the right side of the page.
3. In the pop-up:
   - Leave **Authentication** set to **Local user**.
   - **Username:** `test`
   - **Full name:** `Test User`
   - **Password:** choose a password for this test user and note it — you'll use it to log in as `test` below.
4. Click **Save**. The new user is listed on the **Users** page.
5. Click the **Permissions** tab, to the left of **Users**.
6. Click **Add Access** on the right side of the page. The **User Access** pop-up opens.
7. Select **User**, then choose `test` from the drop-down.
8. Select **ReadOnly**, the last choice.
9. Click **Save** at the lower right. The `test` user is listed on the **Permissions** page with the access you gave it.

## Part 3: Test the read-only user

1. Log out of HVR: click the profile icon at the very bottom of the navigation bar and select the logout option. The HVR login screen appears.
2. Log in as `test` with the password you chose.
3. On the channels page, notice that you can see channel information but have no buttons to run a compare or refresh, as you did as `hubowner`. Explore the other pages from the navigation bar and confirm that only read-only access is available.
4. Log out, then log back in with your **HVR Username** (`hubowner`) and **HVR Password**.

---

## Conclusion

In this workshop, you learned how to set up alerts that notify you when there's an issue with your replication. You also learned how to create users and control their access to the HVR UI.

Next: [Workshop 5 — Repairing an Out-of-Sync Table](hvr_workshop_5_hands_on_lab.md)
