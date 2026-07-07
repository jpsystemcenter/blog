---
title: Common Log Collection Methods Used in SCOM
date: 2026-05-15 00:00:00
tags:
  - SCOM
  - Troubleshooting
  - トラブルシューティング
  - Log Collection
---

This article explains common logs collected for investigation and how to obtain them when performing troubleshooting in SCOM.

<!-- more -->
Hello everyone.

Have you ever encountered various issues while using System Center Operations Manager (SCOM)?
When addressing such issues, many of you may contact us for support.
In general, investigating the reported issue requires collecting data including logs.
In this article, we explain the logs we commonly request from customers when troubleshooting from an SCOM perspective, and how to collect them.

This article was translated from the below article.
[SCOM で使用する一般的なログ収集方法](https://jpsystemcenter.github.io/blog/SCOM/SCOM_Troubleshooting_log/)

## Event Logs
Event log analysis is the most common type of log analysis in SCOM troubleshooting.
In general, many issues that occur in SCOM and their details are also recorded in event logs.
Therefore, for many issues, checking event logs may help identify the cause.
When collecting event logs, collect them from the following computers depending on the situation.
- SCOM Management Servers:
Regardless of the troubleshooting type, always collect event logs from SCOM Management Servers.
If your environment has multiple SCOM Management Servers, collect event logs from all of them.
If the issue occurs while monitoring computers in another domain through a gateway server, also collect event logs from the gateway server.
- Monitored Windows computers where the issue occurred:
If monitoring issues are occurring, collect event logs from all monitored computers where the issue is occurring.
(You may select and collect from 2 ~ 3 issued computers if there's too many issued monitored computers.)
If troubleshooting is for an SCOM-wide issue rather than an issue on a specific monitored computer, collecting logs from these computers is unnecessary.
- SQL Server host running the SCOM databases:
If a database issue is suspected, collect event logs from this computer.
Typically, event logs from these servers are required when SCOM processing is slow, data is missing, or SCOM is not functioning.
If the SQL Server hosting the SCOM database exists on an SCOM Management Server, you do not need to pay special attention to this point.

Use the following steps to collect event logs.
1. Open Event Viewer on the target computer.
A reliable way to open it is to press [win] + [R], directly specify "eventvwr.msc", and run it.
2. In the console tree on the left, click the event log to save.
From an SCOM perspective, collect the following three event logs.
   - Application (*1)
   - System (*1)
   - Operations Manager (*2)

   (*1) is under [Windows Logs].
   (*2) is under [Applications and Services Logs].
   On servers other than SCOM Management Servers / Gateway servers, and where the SCOM agent is not installed, event log (*2) may not exist.
   In that case, you do not need to collect event log (*2) from that computer.
3. In the right pane, click [Save All Events As...].
4. Specify any name for the event log to save.
For file types, collect both "Event Files (*.evtx)" and "CSV (Comma Separated) (*.csv)".

When sending event logs to us, please group each computer's event logs into one folder, then place those folders into one parent folder and compress it before sending.
If there are multiple target computers, please make them easy to identify, for example by including computer names in event log file names or in folder names that store them.

## Alert Information
You can output alert information stored in the SCOM database regardless of alert status (open, resolved, etc.).
Although not as common as event logs, this is often useful information for troubleshooting.
To collect alert information, use Operations Manager Shell and export a CSV list.
1. Sign in to an SCOM Management Server with an account that has SCOM administrator privileges.
2. Create a folder named "temp" directly under the C drive in advance.
3. Run [Operations Manager Shell] from the [Start] menu. It's usually located under [Microsoft System Center].
4. Run the following command.
```
Get-SCOMAlert | export-csv -path "c:/temp/Alert.csv" -encoding UTF8
```
5. Send the output file "Alert.csv" to us.


## Alert Information (Exception: Previously Resolved Alerts)
In most SCOM troubleshooting cases, the target is alerts currently occurring or alerts that occurred recently, so you can generally obtain the necessary alert information using the above Get-SCOMAlert command procedure.
In rare cases, it is necessary to investigate alerts that were already resolved in the past. Resolved alerts are deleted from the Operations Manager database (which stores recent data) after a specified period (7 days by default), and the data is retained only in the Operations Manager DW database (which stores long-term data). Because this alert information cannot be obtained with the Get-SCOMAlert command, it must be retrieved directly from the database.

1. On the SCOM database server, launch SSMS (SQL Server Management Studio) and connect to the SCOM database.
2. Open [Tables] under the [OperationsManagerDW] database, and note the table name [Alert].[Alert_xxxx].
3. Click [New Query], enter the following SQL in the query window, then click [Execute] to run it.
```
SELECT *
FROM [OperationsManagerDW].[Alert].[Alert_xxxx]
where [Alert].Alert_xxxx.RaisedDateTime > '2025-01-01 00:00:00.000'
order by [Alert].Alert_xxxx.RaisedDateTime desc
```
* Replace the Alert_xxxx part with the table name noted in step 2.
* This query extracts alerts that occurred on or after 2025/01/01. Adjust datetime on "where" statement as needed.

4. Results are displayed at the bottom of the screen. Right-click any of the column name in the result grid and click [Save Results As...].
5. Send the exported CSV file to us.

## Environment Information
You can obtain a list of information about Management Servers and monitored servers registered in SCOM.
When investigating various issues, the information output by these commands can sometimes lead to faster resolution.

1. Sign in to an SCOM Management Server with an account that has SCOM administrator privileges.
2. Create a folder named "temp" directly under the C drive in advance.
3. Run [Operations Manager Shell] from the [Start] menu. It's usually located under [Microsoft System Center].
4. Run the following commands.
```
Get-SCOMManagementServer | export-csv -path "c:/temp/SCOMMS.csv" -encoding UTF8
Get-SCOMGatewayManagementServer | export-csv -path "c:/temp/Gateway.csv" -encoding UTF8
Get-scomagent | export-csv -path "c:/temp/agents.csv" -encoding UTF8
Get-scxagent | export-csv -path "C:\temp\SCXAgent.csv" -encoding utf8
Get-SCOMAgentlessManagedComputer | export-csv -path "C:\temp\Agentless.csv" -encoding utf8
```
5. Send each CSV output in the temp folder individually, or send a compressed archive of the temp folder.

## Installation Logs
Investigating installation logs is used when installation-related operations cannot be performed successfully, such as SCOM installation or push installation of agents.
If an installation-related issue occurs, these logs need to be investigated.
The logs are stored in the following folder.
> C:\Users\<Username used when installing SCOM>\AppData\Local\SCOM

## Management Packs
As described in this article, SCOM monitoring behavior is basically defined in Management Packs.
For example, in the following situations, investigation on the Management Pack side is required.
- Items expected to be discovered are not discovered.
- Alerts that were not output before are now being output.
- In addition, this alert repeatedly resolves and reoccurs.
- A command you were not aware of appears to be running on a monitored server, and you want to investigate whether it may be executed by SCOM.

Using the following steps, you can export all Management Packs in your environment in XML format and also output an HTML file listing the Management Packs in use.
1. Log on to the Operations Manager Management Server with a user that has administrator privileges.
2. Create a folder named "MPs" directly under C: drive.
3. Select [Operations Manager Shell] from the [Start] screen.
4. In the [Operations Manager Shell] window prompt, run the following commands.
```
Get-SCOMManagementPack | Export-SCOMManagementPack -Path:C:\MPs
Get-SCOMManagementPack | ConvertTo-HTML > C:\MPs\MpList.html
```

By running the first command in 4\., you can export all Management Packs introduced in your environment in XML format.
The Management Packs exported in XML format contain all definitions configured in those Management Packs.
In general, SCOM monitoring runs scripts created in Powershell, VBS, Javascript, etc. on target computers.
By checking these implementations, troubleshooting related to Management Packs becomes possible.

You can also use this procedure to check whether your Management Packs are up to date.
The issue you reported may be a known issue for that version and may be resolved by updating the Management Pack.
In such a case, because the reported issue is likely to be resolved by updating the Management Pack, please update the Management Pack.
In most cases, such known issues are documented in Management Pack documentation available from the site where the Management Pack can be obtained.

Please also note that some customers may have introduced third-party Management Packs in addition to those provided by Microsoft.
If investigation of Management Packs determines that the reported issue is caused by a third-party Management Pack, we are unable to provide detailed investigation.
For third-party Management Packs, you need to contact the vendor that developed that Management Pack.

## Trace Logs
All logs described above are materials collected after an issue occurs to investigate the cause retrospectively.
For most issues, the cause can be identified using these materials, but in some cases no information is recorded in logs collected after the issue occurs, making investigation difficult.
In such cases, logs at the time of issue occurrence must be collected in real time, and the processing performed inside SCOM must be investigated chronologically from those logs.
This can be done by collecting trace logs built into SCOM.
Use the following steps to start collecting SCOM trace logs.
1. Sign in with administrator privileges to the computer where trace logs will be collected.
2. Open Command Prompt as administrator.
3. Use the cd command to move to the following directory depending on the computer from which you collect trace logs.
The directories below are used when installed in default folders.
If installation paths for SCOM Management Servers or SCOM agents were changed from defaults, move to the appropriate folder.
| Target computer for trace log collection | Default directory to move to with cd command |
| --- | --- |
| SCOM 2016 Management Server | C:\Program Files\Microsoft System Center 2016\Operations Manager\Server\Tools |
| SCOM 2019 or later Management Server | C:\Program Files\Microsoft System Center\Operations Manager\Server\Tools|
| SCOM agent | C:\Program Files\Microsoft Monitoring Agent\Agent\Tools |
4. Run the following command to stop any trace collection already running.
   ```
   StopTracing.cmd
   ```
   For SCOM trace logs, when the "Microsoft Monitoring Agent" service transitions from stopped to running, collection of error-level trace logs starts automatically.
   Example conditions for "service transitions from stopped to running" include:
   - Manually restarting the service itself
   - Restarting the target server
5. Using Windows Explorer, move to the below directory.
   > C:\Windows\Logs\OpsMgrTrace
6. Delete all files in the below folder.
   > C:\Windows\Logs\OpsMgrTrace
7. Run the following command according to the trace log level to collect.
   - Verbose trace logs:
   ```
   StartTracing.cmd VER
   ```
   - Debug trace logs:
   ```
   StartTracing.cmd DBG
   ```
   In general, debug trace logs contain more information than verbose trace logs.
   Therefore, while debug logs are more likely to collect useful information for investigation, the log size growth rate also increases proportionally.
   As a rough guideline, if immediate reproduction is possible, we recommend debug-level trace logs; if when reproduction will occur is unknown, we recommend verbose-level trace logs.
8. Reproduce the issue.
Collect logs until reproduction is confirmed, then proceed to step 9.
9. Run the following command to stop trace log collection again.
```
StopTracing.cmd
```
10. Run the following command to format collected trace logs into text file format.
```
FormatTracing.cmd R
```
11. When sending materials to us, compress the entire "C:\Windows\Logs\OpsMgrTrace" folder as a zip file and send it to us.
If trace logs are collected on multiple computers, we would appreciate it if you include computer names in zip file names, etc., so collected computers can be identified.


(*1)
Inside the "C:\Windows\Logs\OpsMgrTrace" folder, collected diagnostic logs are stored as files with the "etl" extension.
Each ".etl" file stores logs up to 100 MB.
If logs exceed 100 MB, older logs are overwritten by newer logs in sequence.
The time required to reach 100 MB varies by environment, so it is difficult to provide a general estimate.
We ask that you stop tracing immediately using step 9 after reproducing the issue.
The trace log collection size can be increased or decreased by directly editing the contents of "StartTracing.cmd", which starts trace log collection.
Specifically, in the following line shown at line 99 of the target file, the numeric value following the "-cir" parameter specifies the upper limit of trace log size in MB.
> if not [%1]==[TracingGuidsConfigService] tracelogsm.exe -start %1 -flag %2 -level %3 -f %OpsMgrTracePath%\%1.etl -b 64 -ft 10 **-cir 100** -guid %4

(*2)
When collecting SCOM diagnostic logs in step 7, we have confirmed that no significant performance impact occurs in particular.

(*3)
If operations that stop the SCOM agent service (such as server restart) are performed after setting up trace logging, trace collection settings are reset, and error-level trace collection at OS startup begins.
Therefore, if you perform operations that stop the SCOM agent service during trace collection, you need to repeat steps 1 through 8 again.

Reference:
[Use diagnostic tracing - Operations Manager | Microsoft Learn](https://docs.microsoft.com/ja-jp/system-center/scom/manage-overview-management-pack?view=sc-om-2022)
(This is our documentation describing SCOM diagnostic logs.)

## System Information
In SCOM troubleshooting, information about the server itself (Windows Server version, installed software, list of services, etc.) may be required.
This information is collected using the System Information tool.
1. Sign in to the target server.
2. Create a folder named "temp" directly under the C drive in advance.
3. Start Command Prompt with administrator privileges, run the below command to launch MSInfo32.
```
msinfo32
```
4. Save the information from [File] - [Save]. (ComputerName.NFO)
5. Then enter and run the following command.
```
msinfo32 /report C:\temp\msinfo32.txt
```
Please send the files output in steps 4 and 5 to us.