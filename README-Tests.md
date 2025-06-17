# USB/Network Device Test Harness – Test Cycle Documentation

## Overview

This program is designed to **exercise, test, and log the behavior of a network device connected via USB** (using specified VID, PID, and IP address). It cycles the device (simulating physical replugging via software disable/enable), verifies driver status, and performs network connectivity tests, logging detailed results for each step.

---

## Test Cycle Sequencing

Each **test cycle** performs the following sequence of steps:

1. **Disable the USB device** using `devcon`.
2. **Wait for device to disappear** from the system, polling up to 10 seconds.
3. **Enable the USB device** using `devcon`.
4. **Wait for device to be recognized and running**, polling up to 10 seconds.
5. **Wait for network interface to appear and be ready** (polling for up to 4 attempts).
6. **Ping the device IP address** repeatedly until it responds (up to 120 attempts, 0.5s interval).
7. **Perform an HTTP GET** (unless `--no-get` is specified), retrying up to 2 times.
8. **Record and log network interface statistics** (sent/received bytes and packets).
9. **Log the outcome** ("OK" or "NOTOK") and a summary of all sub-tests.

This sequence repeats continuously, producing a new log entry for each cycle.

If there is a failure the test cycle is terminated.

After each test cycle a summary line is logged showing *OK* or *NOTOK* and providing
a summary of success and failures for each test, and the network activity (bytes sent/received and
packets sent/received.)

---

## Step-by-Step: Each Test Step

### 1. **Disable Device**
- **How:** Calls `devcon disable` on the device’s short hardware ID.

### 2. **Wait for Device to Disappear**
- **How:** Repeatedly checks device status with `devcon status`.
- **Success:** `devcon` reports the device as “disabled”.
- **Failure:** Device remains “running” after timeout.

### 3. **Enable Device**
- **How:** Calls `devcon enable` on the device.

### 4. **Wait for Device to be Running**
- **How:** Polls `devcon status` up to 10 seconds.
- **Success:** Device is “running”.
- **Failure:** Device not “running” after timeout.

### 5. **Wait for Network Interface**
- **How:** Attempts up to 4 times to detect the expected network interface (by subnet matching to target IP) using the `NetInfo` class.
- **Success:** Network interface appears, stats can be retrieved.
- **Failure:** Network interface not found.

### 6. **Ping Device**
- **How:** Uses `pythonping` (or Windows `ping` as fallback) to send ICMP echo requests to the device IP.
- **Success:** Ping succeeds (any response) within 120 attempts (0.5s per attempt).
- **Failure:** No ping response after all attempts.

### 7. **HTTP GET (Optional)**
- **How:** Uses `requests.get()` to fetch `http://<device_ip>/` (unless `--no-get` is set).
- **Success:** HTTP 200 OK received.
- **Failure:** No HTTP response or error after 2 attempts.

### 8. **Network Traffic Counters**
- **How:** Uses the `NetInfo` class (PowerShell-based or psutil, as available) to record sent/received bytes and packets before and after the test.
- **Success/Failure:** Used for additional diagnostics; not directly a pass/fail test.

### 9. **Logging the Result**
- Logs summary line for each test cycle, showing:
    - `OK` if all major tests succeeded
    - `NOTOK` if any step failed
    - Per-test success/failure counts (disable, enable, net, ping, get)
    - Current network interface traffic statistics

---

## External Programs and Their Purpose

- **`devcon.exe`:**  
  - Used for device enumeration, status checks, enable/disable operations.  
  - Provided by the Windows Driver Kit (WDK).

- **`powershell.exe`:**  
  - Used (via subprocess) to query network adapter statistics and driver info.

- **`ping` (Windows built-in) and `pythonping`:**  
  - Used for ICMP echo (ping) testing of the device’s IP address.

- **`requests` Python library:**  
  - Used for HTTP GET requests to the device web interface.

---

## Logfile Entries Per Test Cycle

Each cycle produces a series of log entries (with timestamps), typically including:

- Device enable/disable actions and elapsed time
- Device USB status checks (e.g., “USB Device Disconnected [N]”)
- Network interface status and statistics (activity: bytes/packets sent/received)
- Ping attempts and responses, with attempt count
- HTTP GET attempts and responses
- **Summary line** for the cycle, e.g.:

    ```
    2025-06-15 11:08:01,446 [INFO] [10:80] OK disable: 10:0 enable: 10:0 net: 10:0 ping: 9:1 get: 9:0 activity: 18816:5148 178:14
    ```
    Where:
    - `[10:80]`: Test cycle 10, elapsed time 80 seconds
    - `OK` (or `NOTOK`): Overall test result
    - `disable: 10:0`: Success:fail count for device disable
    - `enable: 10:0`: Success:fail for enable
    - `net: 10:0`: Success:fail for network interface found
    - `ping: 9:1`: Success:fail for ping
    - `get: 9:0`: Success:fail for HTTP GET
    - `activity: 18816:5148 178:14`: Network bytes sent:received and packets sent:received

---

## Example Log Sequence

```text
2025-06-15 11:04:37,585 [INFO] [6:146] NOTOK disable: 6:0 enable: 6:0 net: 6:0 ping: 5:1 get: 5:0 activity: 450:0 6:0
2025-06-15 11:04:57,774 [INFO] [7:18] OK disable: 7:0 enable: 7:0 net: 7:0 ping: 6:1 get: 6:0 activity: 12877:5064 126:12


## Summary
This test harness provides a robust, automated sequence to simulate USB device unplug/plug, validate driver and network stack readiness, and exercise application-level connectivity, logging granular status at every stage to facilitate debugging and analysis.
