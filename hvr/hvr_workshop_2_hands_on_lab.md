# Hands-on Lab: HVR Workshop 2 — Monitoring

*Workshop 2 of 5. Adapted from the HVR workshop guide for the LabStudios HVR lab; validate with the provisioned lab resources before delivery.*

This workshop uses the `ora2mdbpg` channel you built in [Workshop 1](hvr_workshop_1_hands_on_lab.md). Log in to your hub with the **HVR Hub URL**, **HVR Username** and **HVR Password** from your 1Password credential link. If the charts are empty, start another HammerDB load from your lab desktop (Workshop 1, Part 6).

---

## About this workshop

HVR gives you several tools to monitor your replication:

- **Topology** — an overview of every replication flow the hub manages. In one view it shows the direction of the data flows, an indication of data volumes, a relative indication of latency, and whether jobs are running. The chart is interactive, so you can retrieve more detail such as current latency and volume.
- **Statistics** — as data flows between sources and targets, HVR keeps detailed logs and harvests them into historical time-series metrics about the replication flows. The series use fixed intervals (minute, 10 minutes, hourly or daily), with metrics aggregated to each interval, such as the total number of changes or the minimum and maximum latency.
- **Events** — a record of the operations HVR has performed. Use it for auditing and for monitoring running jobs such as refreshes and compares. It also records definition changes you made to the repository: defining actions, adding tables, creating location groups, creating the channel and defining locations.
- **Log files** — HVR keeps log files at several levels, including the hub, jobs and events.

---

## Part 1: Topology

The icons on the left side of the HVR page are the navigation bar. Each icon takes you to a different part of the UI.

1. Click the star-like circle icon at the very bottom of the navigation bar. This opens the **Topology** page, an overview of your replication environment.
2. Inspect the drop-downs at the top left:
   - the top drop-down chooses the channel to view;
   - below it are all the locations that have been created;
   - next is the timeframe;
   - the bottom slider sets the latency colors, by time or by SLA.

   Explore each drop-down to see its options.
3. The Topology page lets you drill into the activity of the capture and integrate jobs through **live tiles**. By default, the live hub tile at the bottom left shows the integrate volume. If it shows no volume, set the graph range to the last ten minutes with the graph range drop-down at the upper left.
4. The gray circles in the center of the topology graph open more live tiles showing the volume and latency of each job. Click the gray circle with the arrow to open the **Capture Volume** tile, then try the other gray circles.
5. Click the **training** tile at the bottom left. It takes you to the next section, Statistics.

## Part 2: Statistics

You can also reach the **Statistics** page from the chart icon in the navigation bar.

1. Use the drop-downs across the top to select **Channels**, **Locations**, **Table**, **Time Range** and **Metrics**. Explore each one and watch the charts change with your selection.
2. To zoom into a time period, move the cursor into a chart until a plus sign (+) appears, then click and drag left or right to highlight the period and release.
3. Hover over a section of a chart to see the values for that point.

## Part 3: Events

1. Click the flag icon in the navigation bar. This opens the **Events** page.

   As you work in HVR, records are created to support auditing and reviewing HVR operations: channel activations, refreshes, hub property changes, compares, and job starts and suspends. As on the Statistics page, the drop-downs across the top filter the events. Most events have a log file you can open with **View Log**.
2. Click an event to drill into its details, for example the refresh that ran when you activated your channel.
3. Explore the drop-downs and drill into a few different events.

## Part 4: Log files

HVR keeps log files for events, operations and jobs, past and running. They're useful for reviewing the state of replication or finding out why a job stopped: HVR logs errors there, along with information about the transactions being replicated.

1. Click the icon with the circle and arrow in the navigation bar. This opens the **Jobs** page, which shows every job and its status. Explore the drop-downs across the top to filter the jobs shown.
2. Click the three dots on the right side of the page and select **View Hub log**. This opens the main HVR hub log, which rolls up information from the other jobs' logs, such as capture, integrate and stats.
3. To view a single job's log, click **View Log** at the right of the running capture job's row. The capture log shows details such as the transactions captured.
4. Explore the logs of the other jobs on the page.
5. Click the gray **X** on the right side of the log window to close it.

---

## Conclusion

In this workshop you learned the different ways HVR monitors and logs its operations and events. You also explored how HVR keeps statistics you can review at different levels of detail to analyze how your replication is running. In the next workshop, you'll learn how to confirm that your data is in sync between the source and targets.

Next: [Workshop 3 — Refresh and Compare](hvr_workshop_3_hands_on_lab.md)
