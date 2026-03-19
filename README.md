
﻿This script is a periodic system monitoring + logging automation tool.
 
# Features

* 📊** CPU Monitoring – ** Tracks real-time CPU usage

* 🧠 ** Memory Analysis – ** Logs RAM utilization percentage

* 💾** Disk Usage Report – ** Displays usage of all mounted partitions

* 🌐 ** Network Monitoring – **Tracks sent and received data (in MB)

* ⚙️ ** Process Tracking – **Captures:

	* Process ID (PID)
	
	* Process Name
	
	* User
	
	* Status
	
	* Start Time
	
	* CPU & Memory consumption

* ⏱️ ** Automated Scheduling – ** Runs at user-defined time intervals

* 📁 ** Dynamic Log Generation – ** Creates timestamp-based log files

-----------------------------------------------------------------------------------------------------

# Tech Stack
 ## * Python 3

 * Libraries used:

	* **psutil **– System and process monitoring
	
	* **schedule –** Task scheduling
	
	* **os, sys, time –** System operations
		
-----------------------------------------------------------------------------------------------------

# How It Works
1. The script collects system metrics using psutil

2. It scans all active processes and gathers detailed information

3. Logs are generated in a specified directory with timestamps

4. Using the schedule module, the script runs periodically based on user input


-----------------------------------------------------------------------------------------------------


# Usage

**python ScriptName.py <TimeInterval_in_minutes> <DirectoryName>**

Example:

	python AutomatedSurveillanceSystem.py 5 Logs
	
* Runs every 5 minutes    
* Stores logs in the Logs/ directory


-----------------------------------------------------------------------------------------------------


# Command Line Options
Option 
	Description  
	--h  
	Displays help information  
	--u  
	Shows usage instructions
    
-----------------------------------------------------------------------------------------------------


# Detailed Explanation of the Project : 

## **Modules used**

import psutil  
import sys  
import os  
import time  
import schedule  


### **Psutil**
Provides system and process information:  
	 • CPU, RAM, Disk, Network, Processes


### **sys**

Used for command line arguments  
     • sys.argv


### **Os**

Used for file system operations:  
     • check folder exists (os.path.exists)  
     • create folder (os.mkdir)  
     • join path (os.path.join)


### **Time**

Used for:  
     • current time formatting (strftime)  
     • sleeping (sleep)  
     • printing readable current time (ctime)  
     • converting epoch to readable date  

### **Schedule**

Used to run a function every N minutes  
     • schedule.every(5).minutes.do(CreateLog, "Folder")


-----------------------------------------------------------------------------------------------------


## Function:         CreateLog(FolderName)

This is the main logging function. It generates one complete log file.

### Step A: Prepare border and check folder

Border = "-"*50  
Ret = os.path.exists(FolderName)  
     • Border is just for formatting.  
     • os.path.exists(FolderName) checks if the folder already exists.


### Step B: If exists, confirm it is a directory

if(Ret == True):  
     Ret = os.path.isdir(FolderName)  
if(Ret == False):  
     print("Unable to create folder")  
     return

** Reason: **  
     • It’s possible a file exists with the same name as your folder.  
     • Example: If Marvellous is a file, you cannot create folder Marvellous.  
     • So it checks os.path.isdir().


### Step C: If folder doesn’t exist, create it
### else:
os.mkdir(FolderName)
print("Directory for log files gets created succesfully")


### Step D: Create a timestamp-based log filename

timestamp = time.strftime("%Y-%m-%d_%H-%M-%S")
FileName = os.path.join(FolderName,"Marvellous_%s.log" %timestamp)


This gives files like:
    **• Marvellous_2026-02-07_21-10-04.log**
So every run produces a unique log file (no overwrite).


### Step E: Open file in write mode

fobj = open(FileName, "w")

This creates a new file and writes fresh content.


### Step F: Write Header

fobj.write(Border+"\n")
fobj.write("---- Marvellous Platform Surveillance System -----\n")
fobj.write("Log created at : "+time.ctime()+"\n")
fobj.write(Border+"\n\n")
    
• time.ctime() gives human readable date/time.


-----------------------------------------------------------------------------------------------------




## System Report section
### CPU Usage


fobj.write("CPU Usage : %s %%\n" %psutil.cpu_percent())
     • psutil.cpu_percent() returns CPU utilization %
     • Note: first call may sometimes show 0 or inaccurate if not warmed; but okay for general monitoring.

### RAM Usage

mem = psutil.virtual_memory()
fobj.write("RAM usage : %s %%\n" %mem.percent)
     • virtual_memory() returns many fields, like total, used, free, percent.
     • You log mem.percent.


-----------------------------------------------------------------------------------------------------


## Disk Usage Report


for part in psutil.disk_partitions():
     try:
         usage = psutil.disk_usage(part.mountpoint)
         fobj.write("%s -> %s %% used\n" %(part.mountpoint,
         usage.percent))
     except:
             pass
			 
 • psutil.disk_partitions() returns all drives/partitions.
 ◦ Windows example: C:\, D:\
 ◦ Linux example: /, /home, /boot
 • For each partition:
 ◦ disk_usage(mountpoint) gives used %, total, free.


-----------------------------------------------------------------------------------------------------


## Network Usage Report


net = psutil.net_io_counters()
fobj.write("Sent : %.2f MB\n" % (net.bytes_sent / (1024 * 1024)))
fobj.write("Recv : %.2f MB\n" % (net.bytes_recv / (1024 * 1024)))


• net_io_counters() returns total bytes sent/received since boot.
• You convert bytes → MB by dividing by 1024*1024.
• %.2f prints 2 decimal points.


## Process Logging section

Data = ProcessScan()
for info in Data:
     fobj.write("PID : %s\n" %info.get("pid"))
     fobj.write("Name %s\n" %info.get("name"))
     …

Here:
     • ProcessScan() returns a list of dictionaries
     • Each dictionary contains process details
     • You write each process info into log


-----------------------------------------------------------------------------------------------------


## Function: ProcessScan()

This collects per-process details.


### Step A: Warm-up CPU percent

for proc in psutil.process_iter():
     try:
         proc.cpu_percent()
     except:
         pass
time.sleep(0.2)


• proc.cpu_percent() needs two measurements to calculate usage.
• First call “starts the measurement”
• After a short delay (sleep(0.2)), second call gives actual CPU %.


### Step B: Scan processes again and collect info

for proc in psutil.process_iter():
     try:
         info = proc.as_dict(attrs=["pid", "name",

"username","status","create_time"])

• process_iter() gives access to each running process.
• as_dict(attrs=[...]) collects selected fields only (fast + clean).


### Step C: Convert create_time to readable string

info["create_time"] = time.strftime("%Y-%m-%d %H:%M:%S",
time.localtime(info["create_time"]))

• create_time is stored as epoch timestamp (seconds since 1970).
• You convert it into readable date-time format.

If conversion fails, set "NA".


### Step D: Get CPU% and Memory%

info["cpu_percent"] = proc.cpu_percent(None)
info["memory_percent"] = proc.memory_percent()
	 • cpu_percent(None) uses the internal last measurement.
	 • memory_percent() returns percentage of RAM used by process.


### Step E: Handle common process exceptions

except (psutil.NoSuchProcess, psutil.AccessDenied,
psutil.ZombieProcess):
	 pass


### These happen because:
• Process may terminate while scanning
• Some processes need admin access
• Zombie processes exist on Linux

-----------------------------------------------------------------------------------------------------











































































Function: main()

This handles CLI options and starts scheduling.


Script banner


print("---- Marvellous Platform Surveillance System -----")


Command Line Arguments behavior


Case 1: --h help
python Demo.py --h
Shows what this script is used for.


Case 2: --u usage
python Demo.py --u
Shows how to run:
• ScriptName.py TimeInterval DirectoryName


Case 3: Actual automation run
python Demo.py 5 Marvellous


Means:
• Create log every 5 minutes
• Store logs in folder Marvellous




Scheduling logic


schedule.every(int(sys.argv[1])).minutes.do(CreateLog, sys.argv[2])


This registers the job:
• every N minutes → call CreateLog(DirectoryName)


Then the infinite loop keeps checking:


while True:
schedule.run_pending()
time.sleep(1)


• run_pending() runs jobs whose time has come.
• sleep(1) saves CPU (otherwise loop will run extremely fast).


Stop script using:
• Ctrl + C


Sample Log Output


Inside directory (example: Demo/):
• Marvellous_2026-02-07_21-00-00.log
• Marvellous_2026-02-07_21-05-00.log
• Marvellous_2026-02-07_21-10-00.log
...and so on.


Each contains:
• System snapshot + Process list, as


--------------------------------------------------------------------
--------Automated Platform Surveillance System------
Log created at : Tue Mar 18 12:04:34 2026
--------------------------------------------------------------------
CPU usage : 23 %
RAM usage : 45 %


Disk usage report
/ -> 60% used


Network usage Report
Sent : 120.45 MB
Recv : 98.32 MB
________________


Use Cases
                     * System performance monitoring

                     * Debugging performance issues

                     * Learning system-level programming

                     * Lightweight alternative to monitoring tools
