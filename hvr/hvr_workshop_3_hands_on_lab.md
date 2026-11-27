# Hands-on Lab: HVR Workshop 3 — Refresh and Compare

*Workshop 3 of 5. Adapted from the HVR workshop guide for the LabStudios HVR lab; validate with the provisioned lab resources before delivery.*

This workshop uses the `ora2mdbpg` channel you built in [Workshop 1](hvr_workshop_1_hands_on_lab.md). Log in to your hub with the **HVR Hub URL**, **HVR Username** and **HVR Password** from your 1Password credential link. The online options below are meant to be used while transactions are running, so start a HammerDB load from your lab desktop first (Workshop 1, Part 6).

---

## About this workshop

**Refresh** is HVR's term for a one-time data load.

- Refresh is often used for an initial load, because HVR automatically aligns the one-time load with incremental change data capture (CDC) to keep the systems in sync.
- In a heterogeneous environment, Refresh automatically maps source data types to compatible, lossless data types on the target.
- Refresh can also create the tables in the target database.

**Compare.** With replication in place, you expect the systems to be in sync, but are they? And if they aren't, which recent operations may have caused it? HVR's **Compare** capability identifies whether your data is in sync. Like Refresh and ongoing replication, Compare is based on the channel definition.

In this workshop you'll refresh the `item` table, then run a compare to confirm it's in sync between the source and the targets. Because transactions are active on the source, HVR has to account for the transactions in flight, so both the refresh and the compare use their online options.

---

## Part 1: Open your channel

1. If you're not already there, click the channels icon at the top of the navigation bar on the left.
2. This page shows every channel that has been set up. Click the `ora2mdbpg` channel link.
3. Your channel shows its running capture and integrate jobs. At the top right are three buttons: **Activate Replication**, **Compare Data** and **Refresh Data**.

## Part 2: Refresh the ITEM table

1. Click **Refresh Data**. The **Refresh Data into Target** screen opens.
2. Open the **Locations** drop-down and confirm that `oracle` is the source and that `mariadb` and `postgres` are both selected as targets.
3. Open the **Tables** drop-down and uncheck every table except `item`.
4. Select **Create Missing Target Tables and Alter or Recreate Tables with Incorrect Layout**.
5. Confirm that the **Bulk Load – Table Granularity** method is selected.
6. Expand **Online refresh consistency when selecting tables which are being changed** and select **Changes before refresh are skipped by both capture and integrate jobs**.
7. Click **Refresh Data** at the bottom right to start the refresh of the `item` table.
8. A pop-up tells you the job has started. Click its **View Refresh event** link. The Events page opens on the refresh event; when its state shows `DONE`, the refresh has completed.

## Part 3: Compare the ITEM table

Now confirm that the refreshed `item` table is in sync with the source.

1. Click the channels icon in the navigation bar to return to your channel, then click **Compare Data** at the top right.
2. On the **Compare Data** screen, open the **Locations** drop-down and confirm that `oracle` is the source and that `mariadb` and `postgres` are both selected.
3. Open the **Tables** drop-down and uncheck every table except `item`.
4. Confirm that **Online Compare** is selected.
5. Check **Do Compare Twice and Report Only Differences which occur twice**.
6. Confirm that **Wait** is selected with **5 seconds**.
7. Click **Compare Data** at the bottom right to start the compare.
8. A pop-up tells you the compare has started. Click its **View Compare event** link to open the compare event.
9. When the job's state shows `DONE`, the `item` table's state shows `DONE/IDENTICAL`: the targets are in sync with the source.

---

## Conclusion

You may need to refresh a table in replication from time to time. HVR lets you refresh a table and then compare it while transactions are still running on the source. This workshop walked you through refreshing a table and comparing it to confirm it's in sync with the source.

Next: [Workshop 4 — Alerts and Access Control](hvr_workshop_4_hands_on_lab.md)
