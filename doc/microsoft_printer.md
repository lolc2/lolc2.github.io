##### Abusing Microsoft Printer for C2

Attackers can exploit Microsoft Windows' Internet Printing Protocol (IPP) to establish a covert command and control (C2) channel. By leveraging shared printers, commands are encoded in document names within the print queue, which infected clients can retrieve and execute.

#### How It Works

The [IPPrintC2](https://github.com/mthcht/ThreatHunting-Keywords/tree/main/tools/I-K/IPPrintC2.csv) project demonstrates C2 communication through Windows print infrastructure.

**C2 Flow:** The operator submits print jobs with commands encoded in the document name (base64 strings or fragmented data). The agent monitors the print queue for new jobs, extracts commands from document names, executes them, and writes results back as new print jobs. This exploits the fact that printer additions require no administrative privileges.

**Detection patterns available:** https://github.com/mthcht/ThreatHunting-Keywords/tree/main/tools/I-K/IPPrintC2.csv

#### Key Detection Opportunities

1. **Print job anomalies:** Unusual printer job names containing base64 strings or long fragmented document names
2. **Unusual printer creation:** New printers created and jobs submitted in Microsoft-Windows-PrintService/Operational logs
3. **External printer IPs:** Event ID 300 for a printer pointing to an external IP address
4. **Polling behavior:** Repeated or scheduled polling of `/printers/` endpoints from client machines
5. **Process tree:** Unexpected processes (cmd.exe, powershell.exe, wscript.exe) spawned from `spoolsv.exe`
