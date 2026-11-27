# Hands-on Lab: HVR Workshop 1 — Create a Replication Channel

*Workshop 1 of 5. Adapted from the HVR workshop guide for the LabStudios HVR lab; validate with the provisioned lab resources before delivery.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to prepare. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you for another workshop.

---

## Objective

Use the HVR web UI to build a **one-to-many replication channel** from an **Oracle** source to **PostgreSQL** and **MariaDB** targets, activate it, start a simulated **HammerDB** load on the source, and watch the changes flow to both targets.

## Requirements

- **A web browser** — Chrome or Firefox
- Outbound HTTPS access (port 443) from your machine / corporate VPN

## Housekeeping

- You'll be provided your own **HVR lab environment**: an HVR hub plus an Oracle source database and PostgreSQL and MariaDB target databases, all running on a VM assigned only to you.
- Everything you need to log in is in your **1Password credential link**, provided by your instructor:
  - **HVR Hub URL**, **HVR Username** and **HVR Password** — your HVR web UI login
  - **Gateway URL**, **Guacamole Username** and **Guacamole Password** — your lab desktop, where you start the HammerDB load
  - **HVR Oracle …**, **HVR PostgreSQL …** and **HVR MariaDB …** — the connection details for your lab databases
- DO NOT use someone else's credentials — they are unique to your environment.

## What you'll do in this lab

1. Log in to your HVR hub
2. Create a channel
3. Add the Oracle source location
4. Add the PostgreSQL and MariaDB target locations
5. Select tables, then activate the channel, refresh the targets and start replication
6. Start a HammerDB load and watch the changes replicate
7. Explore the Locations and Tables pages

---

## About this workshop

HVR is designed to make it easy to get replication running between sources and targets. It walks you through getting a channel running from start to finish: it asks for a channel name, description and type, then helps you set up your locations as sources or targets. You then choose the tables to replicate, and HVR activates the channel, starts the capture job, refreshes the target locations and starts the integrate job.

A few HVR terms used throughout these workshops:

- **Channels** — a group of locations and their tables or files that are involved in replication. A channel can have many locations, each acting as a source or a target. You can also have more than one channel to group related objects, for example an HR channel and a Sales channel.
- **Locations** — the source or target endpoints. A location can be a database or a file/directory.
- **Actions** — define the behavior of replication. Every replication channel needs at least two actions: **Capture** on the source location and **Integrate** on the target locations.

**The hub.** An HVR hub is an installation of HVR on a server machine. The hub orchestrates replication in channels and holds the hub server, hubs, scheduler, log files and router files. You connect to the hub server from a browser. An organization can run several hubs, for example one each for Development, UAT and Production. For these workshops, your hub has been created for you.

---

## Part 1: Log in to your HVR hub

1. Open the **HVR Hub URL** from your 1Password credential link in a web browser.
2. Log in with the **HVR Username** (`hubowner`) and **HVR Password** from your credential link, then click **Log in**.
3. The first time you log in, HVR shows a screen to start setting up a replication channel. Click **Create New Channel**.

## Part 2: Create a channel

Your environment has an Oracle source database and two target databases, PostgreSQL and MariaDB.

1. On the left side of the page, enter the channel name and description. The middle area guides you through building the channel.
   - **Channel name:** `ora2mdbpg`
   - **Description:** `Oracle to MariaDB and PostgreSQL`
2. HVR offers four channel types: **One to One**, **One to Many**, **Many to One** and **Multidirectional**. Click **One to Many** to go to step 2, **Select Locations**.

## Part 3: Add the Oracle source location

1. Under the **Source Location** box, click **Create New Location**.
2. Click the **Oracle** tile. This opens step 2, **Select Agent Connection**.
3. Choose to connect through the **HVR agent** (not **Connect directly from hub**), and keep **Regular connection** selected — this is not a RAC cluster.
4. Enter the agent details:
   - **Agent Host:** `oracle`
   - **Agent Port:** `4343`

   > **No agent user is needed.** The agent in your lab accepts connections only from your own hub, so skip **Configure Agent Service** and don't add an agent user.
5. Click **Confirm Connection Method** **once**, and wait for step 3 to load.

   > **Tip:** Clicking **Confirm Connection Method** more than once can leave the **ORACLE_HOME** and **SID** lists in the next step empty. If that happens, click **Edit** on the connection method and confirm it again with a single click.
6. In step 3, **Configure Location Connection**:
   - **ORACLE_HOME:** open the drop-down and select `/opt/oracle/product/26ai/dbhomeFree`.
   - Choose **Oracle TNS** and enter the **HVR Oracle TNS** value from your credential link: `localhost:1521/FREEPDB1`.
   - **User:** `tpcc` (**HVR Oracle User**)
   - **Password:** the **HVR Oracle tpcc Password** from your credential link

   > The `tpcc` schema lives in the `FREEPDB1` pluggable database, so choose **Oracle TNS** rather than a local connection to the `FREE` SID — a local connection refuses the `tpcc` login (`ORA-01017`). If HVR asks for the container (CDB root) login, use `system` (**HVR Oracle Admin User**) and the **HVR Oracle Admin Password**.
7. Click **Confirm Connection Details**.
8. In step 4, choose the capture method. **Direct Redo Access** should already be selected; if not, select it. Click **Confirm Capture**.
9. Name the location:
   - **Location name:** `oracle`
   - **Description:** `Oracle source database`
10. Click **Save Location**.

You're back on step 2, **Select Locations**, with `oracle` chosen as the source. The topology pane on the left shows the Oracle location as the source.

## Part 4: Add the target locations

### PostgreSQL

1. Under the **Target Location** box, click **Create New Location**, then choose **PostgreSQL**.
2. Choose **Connect directly from hub**, then click **Confirm Connection Method**.
3. In step 3, **Configure Location Connection**, enter the details from your credential link:

   | Field | Value |
   |---|---|
   | **Host** | `postgres` (**HVR PostgreSQL Host**) |
   | **Port** | `5432` (leave the default) |
   | **Database** | `pg` (**HVR PostgreSQL Database**) |
   | **User** | `postgres` (**HVR PostgreSQL User**) |
   | **Password** | the **HVR PostgreSQL Password** |

4. Click **Confirm Connection Details**.
5. In step 4, **Configure Integrate for PostgreSQL Location**, make sure every option except **Test connection before confirming** is unchecked, then click **Confirm Integrate**.
6. Enter `postgres` for the location name and `PostgreSQL` for the description, then click **Save Location**.

You now see both the Oracle source and the PostgreSQL target.

### MariaDB

The MariaDB location is set up the same way as PostgreSQL.

1. Under the **Target Location** box, click **Create New Location**, then choose the **MariaDB** tile.
2. Make sure **Connect directly from hub** is selected, then click **Confirm Connection Method**.
3. In step 3, enter the details from your credential link:

   | Field | Value |
   |---|---|
   | **Host** | `mariadb` (**HVR MariaDB Host**) |
   | **Port** | `3306` (leave the default) |
   | **Database** | `maria` (**HVR MariaDB Database**) |
   | **User** | `root` (**HVR MariaDB User**) |
   | **Password** | the **HVR MariaDB Password** |

4. Click **Confirm Connection Details**.
5. In step 4, **Configure Integrate for MariaDB Location**, you'll use a staging directory, which gives the best load performance:
   - **Integrate Staging Directory** should be checked; if not, check it.
   - Click the folder browse button to the right of **Staging Directory**, select **Home** at the top left of the pop-up (the top bar shows `/home/hvr`), then click **Select**.
6. Click **Confirm Integrate**.
7. Enter `mariadb` for the location name and `MariaDB target location` for the description, then click **Save Location**.
8. With all three locations set up, click **Confirm Locations**.

## Part 5: Configure, select tables and activate

1. In step 3, **Configure Channel**, make sure **Standard Replica** is checked, then click **Save Channel and Continue**.
2. Click **Select Tables**. HVR lists the tables in the `tpcc` schema you used for the Oracle location.
3. Select all the tables (or check the **NAME** box, which selects them all).
4. Enter `DEMO` for the **Group name** in the lower right, then click **Save**.
5. Click **Confirm** to go to the final step.
6. The final step activates replication (creates the objects and job components capture and integrate need), refreshes the data into both targets with a bulk load (creating the tables first if they don't exist), and starts the replication jobs. Make sure **Activate Replication**, **Refresh Data into Target** and **Start Replication Jobs** are all checked.
7. Click **Complete Channel Creation**.

This takes a minute or two. When it finishes, you have a running channel replicating from the Oracle source to both targets. If a red activate message appears, refresh your browser and it goes away.

## Part 6: Start a HammerDB load

HammerDB simulates a TPC-C workload on the Oracle source so HVR has changes to replicate. It runs on your lab desktop.

1. Open the **Gateway URL** from your credential link in a new browser tab, and log in with your **Guacamole Username** and **Guacamole Password**. Your lab desktop opens in the browser.
2. Double-click the **HammerDB** icon on the desktop to start the load.

   > The load runs for **5 minutes** at about **100 transactions per second** (roughly 1,800 row changes per second on the source). To run it again later, double-click the icon again after the previous run finishes.
   >
   > For reference, `lab-connections.txt` on the desktop lists this VM's hosts, ports and users. It shows no passwords; those are in your credential link.
3. Return to the HVR browser tab. After a minute or two, the **Integrated Changes (split by Table)** chart on the right side of the channel page shows data moving to your targets. The **Jobs** list below it shows the capture and both integrate jobs running, with their latency.

Your replication channel is complete, with data flowing from the Oracle source to both the MariaDB and PostgreSQL targets.

## Part 7: Explore locations and tables

### Locations

1. Click the cylinder icon in the navigation bar on the left. This opens the **Locations** page, which lists every location created on this hub. You can also add locations from here.
2. Click the `oracle` location name to see its details. From there you can edit the location, test its connection, or add it to an existing channel.

### Tables

As channels grow, you'll want to see which tables are in which channels.

1. Click the table icon in the navigation bar on the left. This opens the **Tables in all channels** page.
2. Use the drop-downs across the top to filter the results to what you need.
3. Click the `history` table name to see its details, including a history of its compares and refreshes.

---

## Conclusion

In this workshop, you were introduced to the HVR user interface. HVR guided you through a complete channel setup: creating a channel and its locations, choosing tables, activating, and starting replication. You also saw how to view the details of a location and of the tables in replication.

Next: [Workshop 2 — Monitoring](hvr_workshop_2_hands_on_lab.md)
