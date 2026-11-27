# Hands-on Lab: HVR Workshop 5 — Repairing an Out-of-Sync Table

*Workshop 5 of 5. Adapted from the HVR workshop guide for the LabStudios HVR lab; validate with the provisioned lab resources before delivery.*

This workshop uses the `ora2mdbpg` channel you built in [Workshop 1](hvr_workshop_1_hands_on_lab.md). You'll need both:

- your HVR hub: the **HVR Hub URL**, **HVR Username** and **HVR Password** from your 1Password credential link;
- your lab desktop: the **Gateway URL**, **Guacamole Username** and **Guacamole Password** from the same link.

---

## About this workshop

Sometimes target tables no longer match their source tables. This condition is called **out of sync (OOS)**, and it can happen for many reasons. HVR lets you compare the source and target tables and, if a table is out of sync, repair the target table to bring it back in sync with the source.

In this workshop you'll create an out-of-sync condition on the PostgreSQL `history` table by running a script on your lab desktop that deletes rows from it. You'll then use HVR Compare to confirm the table is out of sync, and HVR Repair to bring it back in sync with the source.

---

## Part 1: Make the PostgreSQL history table out of sync

1. Open the **Gateway URL** in a browser tab and log in with your **Guacamole Username** and **Guacamole Password** to open your lab desktop.
2. If HammerDB is still open, close it with the **X** at the top right of its window.
3. Double-click the **labs** folder on the left side of the desktop.
4. Double-click `delete_history.sh`. A window asks how you want to run the script; click **Run**.
5. Close the labs folder window with the **X** at its top right.

## Part 2: Confirm the table is out of sync

1. In your HVR browser tab, make sure you're logged in as `hubowner` and on the `ora2mdbpg` channel. If not, open it from the channels icon at the top of the navigation bar.
2. Click **Compare Data** at the top right of the channel page.
3. In the **Compare Data** window, open the **Locations** drop-down and click the **x** next to `mariadb` to remove it, so only the `postgres` target is compared.
4. Open the **Tables** drop-down and uncheck every table except `history`.
5. Make sure **Online Compare** is selected, with the first choice, **Combine Initial Differences…**, selected.
6. Click **Compare Data** at the bottom of the window to start the compare.
7. The compare creates an event you can monitor. Click the link in the green pop-up message.
8. When the compare finishes, the `history` table's **State** column shows `DONE/DIFFERENT`, and **Rows Only on Source** shows the number of rows missing from or different in the target.

## Part 3: Repair the table

1. Click the channels icon in the navigation bar to return to the `ora2mdbpg` channel.
2. Click **Refresh Data** at the top right of the channel page. The **Refresh Data into Target** window opens.
3. Open the locations and click the **x** next to `mariadb` to remove it from the refresh.
4. Open the **Tables** drop-down and make sure only `history` is selected.
5. Confirm that **No Initial Creation or Alter of Tables** is checked.
6. Select **Repair – Row by Row Granularity**.
7. Click **Refresh Data** at the bottom of the window to start the repair.
8. As with the compare, a green pop-up tells you the refresh has started. Click its link to view the status of the repair on the **Refresh** event.
9. To view the refresh's log file, click the **View Log** link in the information line across the top of the Refresh page. Close the log with the **X** on the right side of the log window.

## Part 4: Confirm the table is back in sync

Re-run the compare from Part 2, this time comparing twice:

1. Go to the channel page and click **Compare Data**.
2. Open the **Locations** drop-down and remove `mariadb`.
3. Open the **Tables** drop-down and make sure only `history` is selected.
4. Confirm that **Online Compare** is selected, this time with **Do Compare Twice and Report Only Differences Which Occur Twice**.
5. Confirm that **Wait** is selected with a wait time of 5 seconds.
6. Click **Compare Data** at the lower right of the window.
7. Click the link in the green pop-up to open the compare event, and confirm that the `history` table is back in sync with the Oracle source.

---

## Conclusion

Target tables can sometimes fall out of sync with their source tables. HVR lets you confirm whether a table is out of sync and, if it is, repair the target table to bring it back in sync with the source.

## Congratulations!

You've set up data replication from scratch between three systems running different database technologies. You configured replication with HVR, inspected the topology of your setup, and looked at the statistics for the ongoing replication. If replication runs into issues, you know HVR can proactively send out notifications.

You also refreshed a table and validated that the tables in the databases are truly in sync, and saw that along the way HVR kept an audit log of every operation you performed. Finally, you learned how to repair a target table that has fallen out of sync with its source.

You can show what you achieved in these workshops to your co-workers, or keep exploring this setup for the rest of the time your lab environment is available.
