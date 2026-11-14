---
title: Common Log Collection Methods Used in SCVMM
date: 2026-05-25 13:00:00
tags:
  - SCVMM
  - Troubleshoot
  - トラブルシューティング
  - Log Collection
---

Hello everyone,
This article explains the methods for acquiring logs and other information that are frequently collected when investigating troubles that occur in System Center Virtual Machine Manager (hereinafter, SCVMM).

<!-- more -->
This page is based on the below blog entry.
[SCVMM で使用する一般的なログ収集方法](https://jpsystemcenter.github.io/blog/SCVMM/SCVMM_Troubleshooting_log/)

## Information Required for Troubleshooting
When troubleshooting SCVMM, we request the following information from customers to understand their environment and SCVMM operations. This article explains the purpose of each collection and the procedures for collection.
<table>
  <thead>
    <tr>
      <th style="text-align:center">No.</th>
      <th style="text-align:center">Item</th>
      <th style="text-align:center">Collection Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">1</td>
      <td><a href="#No-1-Trace-Logs-and-Error-Information">Trace Logs and Error Information</a></td>
      <td>When reproducible malfunction behavior occurs, we collect trace logs during issue reproduction to trace the flow of processing and investigate the cause.</td>
    </tr>
    <tr>
      <td style="text-align:center">2</td>
      <td><a href="#No-2-Job-List">Job List</a></td>
      <td>When a SCVMM job does not terminate normally, we confirm the name, execution time, errors that occurred, and other details of running jobs in the job list.</td>
    </tr>
    <tr>
      <td style="text-align:center">3</td>
      <td><a href="#No-3-SCVMM-Environment-Information-SCVMM-Server-Information-Host-Information-Virtual-Machine-Information">SCVMM Environment Information</a></td>
      <td>This is information to understand the customer environment. Collection targets are three types: SCVMM server information, host information, and virtual machine information.</td>
    </tr>
    <tr>
      <td style="text-align:center">4</td>
      <td><a href="#No-4-Cluster-Host-Virtual-Machine-Property-Screens">Cluster/Host/Virtual Machine Property Screens</a></td>
      <td>This is information to understand the customer environment. Collection targets are the property screens of [Cluster/Host/Virtual Machine] where the issue occurred.</td>
    </tr>
    <tr>
      <td style="text-align:center">5</td>
      <td><a href="#No-5-SCVMM-Error-Log">SCVMM Error Log</a></td>
      <td>Used for investigation when errors occur in SCVMM or when the SCVMM console abnormally terminates due to an error.</td>
    </tr>
    <tr>
      <td style="text-align:center">6</td>
      <td><a href="#No-6-Event-Log">Event Log</a></td>
      <td>Confirm the event log during the time when the malfunction behavior occurred to verify the status of the OS and SCVMM at that time.</td>
    </tr>
  </tbody>
</table>

## No.1 Trace Logs and Error Information
When an unexpected issue occurs, trace logs collected while reproducing the issue help us investigate the processing flow and identify the cause. The exact error message and the time the error occurred are also important.

<b>■ Trace Log Collection Procedure</b>
Perform the following procedure on both the SCVMM management server and the affected virtualization host (Hyper-V server), collecting traces simultaneously.
1. Open Command Prompt as administrator.
2. Create a VMM folder (C:\VMMlogs).
3. Run the following commands in order.
 - Preparation: Remove existing data collector sets
```
logman delete VMM
logman delete WinRM
logman delete WMI
```
 (*) If a message indicates that the data collector set was not found, you can disregard it and continue.
 - Preparation: Create trace data collector sets
```
logman create trace VMM -v mmddhhmm -ow -o C:\VMMLogs\%computername%_VMMLog.ETL -p Microsoft-VirtualMachineManager-Debug -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 2048
logman create trace WinRM -v mmddhhmm -ow -o C:\VMMLogs\%computername%_WinRM.ETL -p Microsoft-Windows-WinRM 0xffff 0x5 -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 2048
logman create trace WMI -v mmddhhmm -ow -o C:\VMMLogs\%computername%_WMI.ETL -p WMI_Tracing 0xffff 0x5 -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 2048
```
 (*) Each trace file has a maximum size of 2 GB, as specified by the -max option. Before starting, ensure that the C: drive has sufficient free space, or change the output paths to a drive with sufficient capacity.
 - Start Collection (ETL files will be created under the C:\VMMlogs folder)
```
logman start VMM
logman start WinRM
logman start WMI
```

4. Reproduce the issue. <b><font color="red">* The trace logs during this process will be used for analysis.</font></b><br>

5. After reproducing the issue, run the following commands on both servers to stop trace collection and remove the data collector sets.
```
logman stop VMM
logman stop WinRM
logman stop WMI
logman delete WinRM
logman delete WMI
logman delete VMM
```

6. Execute the following command on each etl file to convert the trace file (a .txt file will be generated):
 ```
 netsh trace convert C:\VMMlogs\<Computer Name>_<Each ETL File>.etl
 ```
 (*) <Each ETL File> includes three types: WMI, WinRM, and VMMLog.
7. Compress the C:\VMMlogs folder and send it to us.<br><br>


<b>■ Error Information Collection Procedure</b>
Depending on the error occurrence situation, collect one of the following pieces of information:

<b>(1) When errors occur in job results</b>
A typical error occurrence situation in SCVMM is when operations or configuration changes are made and errors occur in jobs during execution.
In this case, collect information following these steps:
1. Log in to the SCVMM server with administrator privileges and open the SCVMM console.
2. Select the [Home] tab at the top of the console and click [Jobs] within the tab.
 ![](005.png)
3. From the displayed job list, capture a screenshot showing the execution result of the job you performed.
Capture screenshots of both the [Overview] tab and the [Details] tab.
Capture screens that show the following as an example:
 ![](006.png)
 ![](008.png)
4. For the job where you captured the screenshot in step 3, click [Copy Error] displayed below the error content, paste the copied error content into a text editor, and save it as a .txt file.
 ![](007.png)
5. Collect the files from steps 3 and 4 into one folder, compress them, and send them to us.

<b>(2) When errors are displayed in pop-ups before job execution and the job is not recorded</b>
There are cases where errors occur before job execution, such as when pop-ups appear during configuration saving and prevent the configuration from being saved.
In this case, since the job has not been executed, it is not possible to identify error content and execution time from the job for trace investigation.
In this situation, capture a screenshot showing the error content displayed in pop-ups and the OS date/time when the error occurred, and send it to us.

## No.2 Job List
When a job executed in SCVMM does not terminate normally, you can confirm the job executed, errors that occurred, and error messages by checking the job list.
The method for obtaining the job list is as follows:

<b>■ Collection Procedure</b>
1. Log in to the SCVMM management server with administrator privileges.
2. Create a folder named "temp" directly under the C drive in advance.
3. From the Start menu, execute [Virtual Machine Manager Command Shell].
4. Execute the following command:
 ```
 Get-SCJOB -All -Full | export-csv -path "C:\temp\scjob.csv" -encoding UTF8
 ```
5. Send the "scjob.csv" file output to the "temp" folder to us.<br><br>
 
 
## No.3 SCVMM Environment Information (SCVMM Server Information, Host Information, Virtual Machine Information)
To understand the configuration and usage of your SCVMM environment, we collect environment information using a PowerShell script. Follow the steps below and provide the generated ZIP file.
 
<b>■ Collection Procedure</b>
1. Sign in to the SCVMM management server using an account with administrator privileges.
2. Create the C:\temp folder if it does not already exist.
3. Download [Get-SCVMM-Environment-Data.zip](../SCVMM_Troubleshooting_log/Get-SCVMM-Environment-Data.zip), save it to a folder on the SCVMM management server, and extract it.
4. Open Virtual Machine Manager Command Shell from the Start menu.
5. Navigate to the folder containing the extracted script and run:
```
.\Get-SCVMM-Environment-Data.ps1
```
6. Enter the FQDN of the SCVMM management server—for example, scvmm2025.domain1.local—and press Enter.
7. When the script finishes, a ZIP file named SCVMM_Data_YYYYMMDD-HHMMSS.zip is generated under C:\temp.
8. Send the generated ZIP file to Microsoft Support.

※ Sample
The script collects information about the SCVMM server, managed computers, storage, and networks, and packages the results into a ZIP file.
![](009.png)

## No.4 Cluster/Host/Virtual Machine Property Screens
When warnings such as status abnormalities occur on clusters, hosts, or virtual machines managed by SCVMM, we request screenshots of the corresponding property screens to confirm the occurring warnings and configuration details.
The methods for acquiring each piece of information are as follows.<br>
<b>■ Collection Procedure</b>
1. Log in to the SCVMM console.
2. Open the [VMs and Services] screen.
3. Select the target where the warning occurred and click Properties from the right-click menu.
 ![](001.png)
4. On the Properties screen, open the [General] screen and [Status] screen, and capture screenshots.
 (1) <b>Cluster</b> [General] screen and [Status] screen
 ![](002.png)
 (2) <b>Host</b> [General] screen and [Status] screen
 When warnings are displayed on the Status screen, capture screenshots in a way that identifies the target.
 Also, copy the error message by clicking the [Copy Error] button and save it as error.txt.
 ![](003.png)
 (3) <b>Virtual Machine</b> [General] screen and [Status] screen
 When warnings are displayed on the Status screen, capture screenshots in a way that identifies the target.
 Also, copy the error message by clicking the [Copy Error] button and save it as error.txt.
 ![](004.png)
5. Send the captured screenshots and error.txt to us. <br><br>


## No.5 SCVMM Error Log
When errors occur in SCVMM or the SCVMM console abnormally terminates due to errors, information about the cause may be recorded in the error log.
The method for obtaining the error log is as follows:

<b>■ Collection Procedure</b>
Perform this procedure on the following servers:
 - SCVMM Server
 - Server where SCVMM console abnormally terminated
* If the SCVMM console was open on the SCVMM server, collect this log only from the SCVMM server.

1. Log in to the target server with administrator privileges.
2. Open Explorer and access the following folder:
 C:\ProgramData
 By default, the <b>"ProgramData" folder</b> is set as a hidden folder.
 To access this folder, enable the setting to display hidden folders in Explorer.
3. Compress the <b>"VMMLogs" folder</b> existing in the folder accessed in step 2, and send it to us.
If there are multiple collection targets, rename the compressed file of "VMMLogs" so that the collection target can be identified.
For example, if collected from the management server, rename it to "MS_VMMLogs", or if collected from a virtualization host, rename it to "Host_VMMLogs".<br><br>


## No.6 Event Log
Part of SCVMM processing is recorded in the event log, so we request the event log to be sent for problem analysis.
The method for obtaining the event log is as follows:

<b>■ Collection Procedure</b>
Perform this procedure on the following computers:
・SCVMM Server
・Virtualization host server (Hyper-V server) where the problem occurred

1. Open Event Viewer on the target computer.
A reliable way to open it is to press [win] + [R], directly specify "eventvwr.msc", and run it.
2. In the console tree on the left, click the event log to save.
From an SCVMM perspective, collect the following three event logs.
   - Application
   - System
   - Security
   - WinRM-Operational (SCVMM server only) (*1)
   - WMIActivity-Operational (*1)
   - Admin (*2)
   - Operational (*2)


   The logs marked with (*1) are displayed when expanding [Applications and Services Logs] -> [Microsoft] -> [Windows].
   If the event logs marked with (*1) are not displayed, collection of (*1) event logs from that server is not necessary.
   The logs marked with (*2) are displayed when expanding [Applications and Services Logs] -> [Microsoft] -> [VirtualMachineManager] -> [Server].
   If the event logs marked with (*2) are not displayed, collection of (*2) event logs from that server is not necessary.
3. In the right pane, click [Save All Events As...].
4. Specify any name for the event log to save.
For file types, collect both "Event Files (*.evtx)" and "CSV (Comma Separated) (*.csv)".
