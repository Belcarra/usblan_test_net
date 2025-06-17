# usblan\_test\_net
# Tue, Jun 17, 2025  1:49:53 PM

This archive contains a Python script *usblantestnet.py* that implements a test cycle 
for testing USB clients that use the *Belcarra USBLAN* Windows Class driver.

The test cycle use *devcon.exe* (available in the Microsoft Windows DDK) and some short
PowerShell snippets to do a soft replug, then to wait for the device to reconnect,
the network adapter to become available, ping to the device to succeed and then optionally
fetch a file if the device supports HTTP.

The progress of each test cycle is logged with a summary showing the number of success
and failures for each run.

## Requirements

This script can be used in two ways:
- install Windows Python
- use the pre-compiled EXE

## usblantestnet


```
python usblantestnet.py
usage: usblantestnet.py [-h] [--logfile LOGFILE] [--version] [--wait-on-error] [--no-get] [--no-auto-replug] [vid] [pid] [ip]

USB/Network Device Test Harness

positional arguments:
  vid                USB Vendor ID (e.g., 15EC)
  pid                USB Product ID (e.g., E021)
  ip                 Target IP address to ping

options:
  -h, --help         show this help message and exit
  --logfile LOGFILE  Log file name
  --version          Show version and exit
  --wait-on-error    For for manual replug on error
  --no-get           Disable the HTTP GET test
  --no-auto-replug   Disable the automatic USB Disable
```

The VID, PID and IP address are required arguments. The script will use the VID and PID
to find the test device. N.b. *one* test device must be plugged in with the correct *USBLAN* 
driver for the device working.

The IP address is used to find the Network Adapter for the test devide and test that the 
network is operational.

Optional test options:
- *--wait-on-errors* - pause the test when an error occurs, this allows looking at the system with other tools for more information
- *--no-get* - the test device does not support fetching http://IPaddress/
- *--no-auto-replug" - do not do the automated disconnect, use if you want to do physical replugging of the device

## Logfile

```
20250617-00:57:27 [INFO] 
20250617-00:57:29 [INFO] [14:0] USB\VID_15EC&PID_E021: Disabled
20250617-00:57:31 [INFO] [14:2] USB\VID_15EC&PID_E021: USB Device Disconnected [1]
20250617-00:57:31 [INFO] [14:2] USB\VID_15EC&PID_E021: Enabled
20250617-00:57:32 [INFO] [14:4] USB\VID_15EC&PID_E021: USB Device Connected [3]
20250617-00:57:39 [INFO] [14:10] Ethernet 10: 10.1.11.58 activity: 0:0 0:0 0:0 [1]
20250617-00:58:39 [INFO] [14:70:61] Ethernet 10: Ping 10.1.11.58 [61]
20250617-00:58:39 [INFO] [14:70:61] Ethernet 10: HTTP GET success: http://10.1.11.58/ [1]
20250617-00:58:44 [INFO] [14:75:65] OK disable: 14:0 enable: 14:0 net: 14:0 ping: 14:0 get: 14:0 activity: 6614:5106 45:13 0:0
20250617-00:58:44 [INFO] 
20250617-00:58:46 [INFO] [15:0] USB\VID_15EC&PID_E021: Disabled
20250617-00:58:48 [INFO] [15:2] USB\VID_15EC&PID_E021: USB Device Disconnected [1]
20250617-00:58:48 [INFO] [15:2] USB\VID_15EC&PID_E021: Enabled
20250617-00:58:49 [INFO] [15:4] USB\VID_15EC&PID_E021: USB Device Connected [3]
20250617-00:58:56 [INFO] [15:10] Ethernet 10: 10.1.11.58 activity: 0:0 0:0 0:0 [1]
20250617-01:00:57 [ERROR] [15:131:121] Ethernet 10 Ping to 10.1.11.58 failed.
20250617-01:01:07 [ERROR] [15:141:131] NOTOK disable: 15:0 enable: 15:0 net: 15:0 ping: 14:1 get: 14:0 activity: 0:0 0:0 0:0
20250617-01:01:07 [INFO] 
20250617-01:01:09 [INFO] [16:0] USB\VID_15EC&PID_E021: REPLUG DEVICE NOW
```
The summary line shows the tests with number of success and failures, and network activity bytes sent/received, 
packets sent/received, discarded frames sent/received.

- disable - the device disconnect was correctly observed
- enable - the device reconnect resulted in status OK for the device
- net - ping succeeded or failed
- get - fetching the file succeeded or failed


## Windows Python

[Python for Windows](https://www.python.org/downloads/)

Installing for all users seems to be the best solution. Be sure to have it add to the system PATH
so that it can be easily run from any terminal window.

Additional packages required:
- pythonping
- requests

To install:
```
python -m pip install pythonping requests
```

## Cygwin

*Cygwin64* can be used. Ensure that python3 is installed.

See above for required packages.


## Windows Defender and EXE

Pre-compiled versions of the script (using nuitka) are available.

This may be the preferred method of using this if you do not want to install Python.

N.b. Windows Defender sometimes flags these. There are two mitigations available:

- Pause Real-time protection
- Submit to Microsoft

### Pause Protection

1. Open the *Virus & threat protection settings* in Windows.
2. Click on *Manage settings*.
3. Click on the radio-button to disable *Real-Time protection*.

N.b. Pausing protection appears to timeout after about an hour or two.


### Submit file to Microsoft

Use this URL to submit the EXE to Microsoft.

[Microsoft Submit File](https://www.microsoft.com/en-us/wdsi/filesubmission)

Generally Microsoft will review in less than 24 hours and if they agree that the EXE
is not a threat will update their definitions.


```
usbtestnet.exeSubmission ID: aaaeea06-34bb-46c3-ba68-f55affd2e880
Status: Completed
Submitted by: sl@belcarra.com
Submitted: Jun 16, 2025 12:49:40 AM
User Opinion: Incorrect detection
Analyst comments:

At this time, the submitted files do not meet our criteria for malware or potentially unwanted applications. The detection has been removed. Please follow the steps below to clear cached detections and obtain the latest malware definitions.

1. Open command prompt as administrator and change directory to c:\Program Files\Windows Defender
2. Run “MpCmdRun.exe -removedefinitions -dynamicsignatures”
3. Run "MpCmdRun.exe -SignatureUpdate"

Alternatively, the latest definition is available for download here: https://docs.microsoft.com/microsoft-365/security/defender-endpoint/manage-updates-baselines-microsoft-defender-antivirus

Thank you for contacting Microsoft.

Click here for more information
Thank you for your submission. To provide feedback about your submission experience to the Microsoft Defender team, click here. Please note that providing feedback will not open a new case or change a determination. To request further clarification on a file determination, please create a new submission
```

*N.b. After running the suggested commands it appears to take about a day for your local system
to stop flagging the EXE.*





