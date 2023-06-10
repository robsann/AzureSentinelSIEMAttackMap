# Microsoft Sentinel (SIEM) Lab
This lab is built on Microsoft Azure.

## Components
<p></p>
<b>Microsoft Azure:</b>

1. **Windows 10 VM** - Exposed to Global Attackers
2. **Log Analytics Workspace** - Collect data from VM
3. **Microsoft Sentinel (SIEM)** - Data Visualization

## Procedure
## 1 - On Azure go to Virtual machines
- Click on **Create** > **Azure virtual machine**

### Create a virtual machine
- **Tab: Basics**
    - **Project details**
        - **Subscription:** your_subscription
        - **Resource group:**
            - Click on **Create new** > **Name:** Honeypotlab
    - **Instance details**
        - **Virtual machine name:** honeypot-vm
        - **Region:** (US) West US 3
        - **Availabilit options:** No infrastructure redundancy required
        - **Security type:** Standard
        - **Image:** Windows 10 Pro, version 22H2 - x64 Gen2 (free services eligible)
        - **VM architecture:** x64
        - **Run with Azure Spot discount:** (uncheck)
        - **Size:** Standard_D2s_v3 - 2 vcpus, 8 GiB memory ($70.08/month)
    - **Administrator account**
        - **Username:** your_username
        - **Password:** your_password
        - **Confirm password:** your_password
    - **Inbound port rules**
        - **Public inbound ports:** Allow selected ports
        - **Select inbound ports:** RDP (3389)
    - **Licensing**
        - (check) I confirm I have an eligible Windows 10/11 license with multi-nent hosting rights.
    - Click on **Next: Disks >**
- **Tab: Disks**
    - **VM disk encryption**
        **Encryption at host:** (uncheck)
    - **OS disk**
        - **OS disk type:** Premium SSD (locally-redundant storage)
        - **Delete with VM:** (check)
        - **Key management:** Platform-managed key
        - **Enable Ultra Disk compatibility:** (uncheck)
    - Click on **Next: Networking >**
- **Tab: Networking**
    - **Network interface**
        - **Virtual network:**
            - Click on **Create new**
                - **Name:** Honeypotlab-vnet
                - **Address space:** (uncheck)
                - **Subnets:** (check) default - 10.0.0.0/24
        - **Subnet:** (new) default (10.0.0.0/24)
        - **Public IP:** (new) honeypot-vm-ip
        - **NIC network security group:** Advanced
        - **Configure network security group:**
            - Click on **Create new**
                - **Create network security group**
                    - **Name:** honeypot-vm-nsg
                    - **Inbound rules**
                        - Remove default-allow-rdp rule
                        - Click + Add an inbound rule
                            - Add inbound security rule
                                - **Source:** Any
                                - **Source port ranges:** *
                                - **Destination:** Any
                                - **Service:** Custom
                                - **Destination port rages:** *
                                - **Protocol:** Any
                                - **Action:** Allow
                                - **Priority:** 100
                                - **Name:** DANGER_ANY_IN
                        - |100: DANGER_ANY_IN
                        - |Any
                        - |Custom (Any/Any)
                    - Click **OK**
        - **Delete public IP and NIC when VM is deleted:** (check)
        - **Enable accelerated networking:** (check)
    - **Load Balancing**
        - **Load balancing options:** None
    - Click on **Review + create**
- **Tab: Review + create**
    - **Subscription credits apply:** 0.0960 USD/hr
    - Click **Create**

## 2 - On Azure go to Resources groups
- Just check the resources created.
- **Honeypotlab**
    - **Resources:**
        - honeypot-vm (Virtual machines)
        - honeypot-vm-ip (Public IP addresses)
        - honeypot-vm-nsg (Network security group)
        - honypot-vm878 (Network Interface)
        - honeypot-vm_disk1_34ff... (Disk)
        - Honeypotlab-vnet (Virtual networks)

## 3 - On Azure go to Log Analytics workspaces
- Click on **Create**
### Create Log Analytics workspace
- **Tab: Basics**
    - **Project details**
        - **Subscription:** your_subscription
        - **Resource group:** Honeypotlab
    - **Instance details**
        - **Name:** law-honeypot1
        - **Region:** West US 3
    - Click on **Review + Create**
- **Tab: Review + Create**
    - Click **Create**

## 4 - On Azure go to Microsoft Defender for Clouds | Environment Settings
On the table click on **law-honeypot1:**
- Azure
  - Tenant Root Group (1 of 1 subscriptions)
    - Azure subscription 1
      - **law-honeypot1**
#### Settings | Defender plans
- **Turn On** the Foundational CSPM
- **Turn On** the Servers
- **Turn Off** the SQL servers on machines
- Click on **Save**
#### Settings | Data collection
- **Store additional raw data - Windows security events**
    - Select **All Events**
- Click on **Save**

## 5 - On Azure go to Log Analytics workspace
### law-honeypot1 | Virtual machines
- Click on **honeypot-vm**
    - Click on **Connect**

## 6 - On Azure go to Microsoft Sentinel
- Click on **Create**
### Add Microsoft Sentinel to a workspace
- Select **law-honeypot1 Workspace**
- Click **Add**

## 7 - On Azure go to Virtual machines
- Click on **honeypot-vm**
  - Copy **public IP address** 20.125.146.48

## 8 - Use Remote Desktop Connection to connect to the virtual machine using the public IP address and the credentials defined
- Disable all privacy settings on the device.
- Allow PC to be discoverable.
- Open `Event Viewer` and go to Windows Logs > Security and search for:
    - Event ID: 4625; Task Category: Logon; Keywords: Audit Failure

## 9 - From local computer ping virtual machine
`$ ping 20.125.146.48`
- The VM will not reply.

## 10 - On the virtual machine disable the Firewall.
- Open `wf.msc`
    - Click on **Windows Defender Firewall Properties**
        - **Tab: Domain Profile**
            - Firewall state: off
        - **Tab: Private Profile**
            - Firewall state: off
        - **Tab: Public Profile**
            - Firewall state: off

## 11 - From local computer ping virtual machine
`$ ping 20.125.146.48`
- Now the VM should reply.

## 12 - On the PowerShell console download the powershell script from `joshmadakor1/Sentinel-Lab/Custom_Security_Log_Exporter.ps1` with the command below
`PS \> Invoke-WebRequest "https://raw.githubusercontent.com/joshmadakor1/Sentinel-Lab/main/Custom_Security_Log_Exporter.ps1" -OutFile Custom_Security_Log_Exporter.ps1`

## 13 - Open the `Custom_Security_Log_Exporter.ps1` script on PowerShell ISE as Administrator

## 14 - Get the free API access key from https://ipgeolocation.io, replace it on the powershell script, save it, and run the script
- Try to login the VM with wrong credentials a couple of times to see the messages on the PowerShell console.

## 15 - Copy the output file `failed_rdp.log` at `C:\ProgramData\failed_rdp.log` to your local machine to be opened on Azure later.

## 16 - On Azure go to Log Analytics workspace
- Click on **law-honeypot1**
### law-honeypot1 | Tables
- Click on **Create** > **New custom log (MMA-based)**
### Create a custom log
- **Tab: Sample**
    - **Sample log:** "failed_rdp.log"
    - Click **Next**
- **Tab: Record delimiter**
    - **Select record delimiter:** New line
    - Click **Next**
- **Tab: Collection paths**
    - **Type:** Windows &emsp;&emsp;&emsp;&emsp;
    **Path:** C:\ProgramData\failed_rdp.log
    - Click **Next**
- **Tab: Details**
    - **Custom log name:** FAILED_RDP_WITH_GEO
    - Click **Next**
- **Tab: Review + Create**
    - Click **Create**

### law-honeypot1 | Logs
<p></p>
<b>New Query 1*</b> <br/>
FAILED_RDP_WITH_GEO_CL

- Click **Run**
- Should return no events

<p></p>
<b>New Query 1*</b> <br/>
SecurityEvent

- Click **Run**
- Should return a lot of events

<p></p>
<b>New Query 1*</b> <br/>
SecurityEvent | where EventID == 4625

- Click **Run**
- Should return a few events

<p></p>
<b>New Query 1*</b> <br/>
FAILED_RDP_WITH_GEO_CL

- Wait 10 minutes or more and before run
- Click Run
- Should return a few events

## 17 - Extract fields using a query piped with parse and project.
**New Query 1***
```
FAILED_RDP_WITH_GEO_CL
| RawData with "latitude:" latitude_CF ",longitude:" longitude_CF ",destinationhost:" destinationhost_CF ",username:" username_CF ",sourcehost:" sourcehost_CF ",state:" state_CF ",country:" country_CF ",label:" label_CF ",timestamp:" timestamp_CF
| TimeGenerated,Computer,timestamp_CF,latitude_CF,longitude_CF,destinationhost_CF,username_CF,sourcehost_CF,state_CF,country_CF,label_CF,Type,RawData
```
## 18 - On Azure go to Microsoft Sentinel
- Click on **law-honeypot1**

### Microsoft Sentinel | Workbooks
- Add **workbook**
### New workbook
- Click on **Edit**
    - **Remove text**
    - **Remove query**
    - **Add > Add query**
        - **Log Analytics workspace Logs Query**
            ```
            FAILED_RDP_WITH_GEO_CL
            | parse RawData with "latitude:" latitude_CF ",longitude:" longitude_CF ",destinationhost:" destinationhost_CF ",username:" username_CF ",sourcehost:" sourcehost_CF ",state:" state_CF ",country:" country_CF ",label:" label_CF ",timestamp:" timestamp_CF
            | project TimeGenerated,Computer,timestamp_CF,latitude_CF,longitude_CF,destinationhost_CF,username_CF,sourcehost_CF,state_CF,country_CF,label_CF,Type,RawData
            | where TimeGenerated > datetime(2023-06-01 02:03:46) and TimeGenerated < datetime(2023-06-03 02:03:46)
            | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
            | where destinationhost_CF != "samplehost"
            | where sourcehost_CF != ""
            ```
        - **Run Query**
        - **Set Visualization:** Map
        - Click on **Map Settings**
            - **Layout Settings**
                - **Location info using:** Latitude/Longitude
                - **Latitude:** latitude_CF
                - **Longitude:** longitude_CF
                - **Size by:** event_count
                - **Aggregation for location:** Sum of values
            - **Color Settings**
                - **Coloring Type:** Heatmap
                - **Color by:** event_count
                - **Aggregation for color:** Sum of values
            - **Metric Settings**
                - **Metric Label:** label_CF
                - **Metric Value:** event_count
                - **Create 'Others' group after:** 10
                - **Aggregate 'Others' metrics by:** Sum of values
                - (uncheck) **Custom formatting**
            - Click on **Apply**
        - Click on **Save and Close**
    - **Time Range:** Last 30 days
    - **Size:** Full
- **Save**
    - **Title:** Failed RDP World Map
    - **Subscription:** your_subscription
    - **Resource group:** Honeypotlab
    - **Location:** (US) West US 3
    - (uncheck) **Save content to an Azure Storage Account.**
    - Click on **Apply**

### Failed RDP World Map
- Set **Auto refresh:** 5 minutes
