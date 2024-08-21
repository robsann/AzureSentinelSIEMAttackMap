# Microsoft Sentinel (SIEM) Lab

This lab on Microsoft Azure utilizes Azure Log Analytics Workspace to monitor a Windows 10 VM with port 3389 (RDP) exposed. Data visualization is achieved through Microsoft Sentinel to display attacks from attacker worldwide.


----------------------------------------------------------------------------------------------------


## Components

### Microsoft Azure

1. **Windows 10 VM**: Exposed VM to Global Attackers.
2. **Log Analytics Workspace**: Collect data from VM.
3. **Microsoft Sentinel (SIEM)**: Visualize generated data.


----------------------------------------------------------------------------------------------------


## Procedure

<details>
<summary>
<h3>Step 1: Create the Virtual Machine</h3>
</summary>
<span style="color:gray">

1. Log in into Azure and go to Virtual machines.
2. Click on **Create** > **Azure virtual machine**
3. Create a virtual machine
4. On **Tab: Basics** fill the fields as shown below:
	1. On **Project details** fill the fields below:
		- **Subscription:** your_subscription
		- **Resource group:**
	2. Then click on **Create new** > **Name:** Honeypotlab
	3. On **Instance details** fill the fields below:
		- **Virtual machine name:** honeypot-vm
		- **Region:** (US) West US 3
		- **Availability options:** No infrastructure redundancy required
		- **Security type:** Standard
		- **Image:** Windows 10 Pro, version 22H2 - x64 Gen2 (free services eligible)
		- **VM architecture:** x64
		- **Run with Azure Spot discount:** (uncheck)
		- **Size:** Standard_D2s_v3 - 2 vcpus, 8 GiB memory ($70.08/month)
	4. On **Administrator account** fill the fields below:
		- **Username:** your_username
		- **Password:** your_password
		- **Confirm password:** your_password
	5. On **Inbound port rules** fill the fields below:
		- **Public inbound ports:** Allow selected ports
		- **Select inbound ports:** RDP (3389)
	6. On **Licensing** check the field below:
		- (check) I confirm I have an eligible Windows 10/11 licence with multiagent hosting rights.
	7. Then click on **Next: Disks >**
5. On **Tab: Disks** fill the fields as shown below:
    1. On **VM disk encryption** uncheck the field below:
		- **Encryption at host:** (uncheck)
    2. On **OS disk** fill the fields below:
        - **OS disk type:** Premium SSD (locally-redundant storage)
        - **Delete with VM:** (check)
        - **Key management:** Platform-managed key
        - **Enable Ultra Disk compatibility:** (uncheck)
    3. Then click on **Next: Networking >**
6. On **Tab: Networking** fill the fields as shown below:
    1. On **Network interface** configure the virtual network:
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
    2. On **Load Balancing** set the parameter below to none:
        - **Load balancing options:** None
    3. The click on **Review + create**
7. On **Tab: Review + create**
    1. **Subscription credits apply:** 0.0960 USD/hr
    2. Click **Create**

The resources created can be checked at Resources groups on Azure.

- **Honeypotlab**
    - **Resources:**
        - **honeypot-vm** (Virtual machines)
        - **honeypot-vm-ip** (Public IP addresses)
        - **honeypot-vm-nsg** (Network security group)
        - **honypot-vm878** (Network Interface)
        - **honeypot-vm_disk1_34ff...** (Disk)
        - **Honeypotlab-vnet** (Virtual networks)

</span>
</details>


<details>
<summary>
<h3>Step 2: Create the Log Analytics Workspace</h3>
</summary>
<span style="color:gray">

On Azure go to Log Analytics Workspaces then click on **Create**

1. On **Tab: Basics**, fill the fields below:
    1. **Project details**
        - **Subscription:** your_subscription
        - **Resource group:** Honeypotlab
    2. **Instance details**
        - **Name:** law-honeypot1
        - **Region:** West US 3
    3. Then click on **Review + Create**
2. On **Tab: Review + Create**
    - Click **Create**.

</span>
</details>


<details>
<summary>
<h3>Step 3: Configure Microsoft Defender for Clouds</h3>
</summary>
<span style="color:gray">

On Azure, go to **Microsoft Defender for Clouds | Environment Settings**:

1. On the table, click on **law-honeypot1:**
	- Azure
		- Tenant Root Group (1 of 1 subscriptions)
			- Azure subscription 1
			- **law-honeypot1**

2. On **Settings | Defender plans** set the parameters below:
	- **Turn On** the Foundational CSPM
	- **Turn On** the Servers
	- **Turn Off** the SQL servers on machines
	- Click on **Save**

3. On **Settings | Data collection**, set the parameters below:
	- **Store additional raw data - Windows security events**
		- Select **All Events**
	- Click on **Save**

</span>
</details>


<details>
<summary>
<h3>Step 4: Configure Log Generation</h3>
</summary>
<span style="color:gray">

1. On Azure, go to **Log Analytics Workspace**.
	1. On **law-honeypot1 | Virtual machines**, click on **honeypot-vm**, then click **Connect**.
2. On Azure, go to **Microsoft Sentinel**.
	1. Click on **Create**.
3. Add **Microsoft Sentinel** to a Workspace:
	1. Select **law-honeypot1 Workspace**.
	2. Then click **Add**.
4. On Azure, go to **Virtual machines**.
	1. Click on **honeypot-vm**.
	2. Then copy the **public IP address**.
5. Use **Remote Desktop Connection** to connect to the virtual machine using the public IP address and the credentials defined earlier.
	1. Disable all privacy settings on the device.
	2. Allow PC to be discoverable.
	3. Open `Event Viewer` and go to **Windows Logs** > **Security** and search for:
    	- **Event ID:** 4625
		- **Task Category:** Logon
		- **Keywords:** Audit Failure
6. From your local computer, ping virtual machine:
	```bash
	$ ping <VM_PUBLIC_IP_ADDRESS>
	```
	- The VM will not reply.
7. On the virtual machine, disable the Firewall.
	1. Open `wf.msc`
	1. Click on **Windows Defender Firewall Properties**
		- On **Tab: Domain Profile** set:
			- **Firewall state:** off
		- On **Tab: Private Profile** set:
			- **Firewall state:** off
		- On **Tab: Public Profile** set:
			- **Firewall state:** off
8. From your local computer, ping virtual machine again:
	```bash
	$ ping 20.125.146.48
	```
	- Now the VM should reply.
9. Open the PowerShell console on the VM and download the PowerShell script from [joshmadakor1](joshmadakor1/Sentinel-Lab/Custom_Security_Log_Exporter.ps1) with the command below:
	```sh
	PS \> Invoke-WebRequest "https://raw.githubusercontent.com/joshmadakor1/Sentinel-Lab/main/Custom_Security_Log_Exporter.ps1" -OutFile Custom_Security_Log_Exporter.ps1
	```
10. Open the `Custom_Security_Log_Exporter.ps1` script on PowerShell ISE as Administrator.
11. Get the free API access key for geolocation data from https://ipgeolocation.io, and replace it on the PowerShell script. Save it and run the script.
	1. Try to log in the VM with wrong credentials a couple of times to see the messages on the PowerShell console.
12. Copy the output file `failed_rdp.log` at `C:\ProgramData\failed_rdp.log` to your local machine to be opened on Azure later.

</span>
</details>


<details>
<summary>
<h3>Step 5: Configure Log Analytics Workspace</h3>
</summary>
<span style="color:gray">

On Azure, go to **Log Analytics Workspace**.

1. Click on **law-honeypot1**
2. On **law-honeypot1 | Tables**
	1. Click on **Create** > **New custom log (MMA-based)**
3. Create a custom log:
	1. On **Tab: Sample** fill the field below:
		- **Sample log:** "failed_rdp.log"
		- Click **Next**
	2. On **Tab: Record delimiter**, set the parameter below:
		- **Select record delimiter:** New line
		- Click **Next**
	3. On **Tab: Collection paths**, fill the fields below:
		- **Type:** Windows
		- **Path:** C:\ProgramData\failed_rdp.log
		- Click **Next**
	4. On **Tab: Details**, fill the field below:
		- **Custom log name:** FAILED_RDP_WITH_GEO
		- Click **Next**
	5. On **Tab: Review + Create**, check the information, then click **Create**.
4. On **law-honeypot1 | Logs**

	**New Query 1\***<br>
	FAILED_RDP_WITH_GEO_CL
	- Click **Run**
	- Should return no events

	**New Query 1\***<br>
	SecurityEvent
	- Click **Run**
	- Should return a lot of events

	**New Query 1\***<br>
	SecurityEvent | where EventID == 4625
	- Click **Run**
	- Should return a few events

	**New Query 1\***<br>
	FAILED_RDP_WITH_GEO_CL
	- Wait 10 minutes or more and before run
	- Click Run
	- Should return a few events

Extract fields using a query piped with parse and project.

- **New Query 1***
	```sql
	FAILED_RDP_WITH_GEO_CL
	| RawData with "latitude:" latitude_CF ",longitude:" longitude_CF ",destinationhost:" destinationhost_CF ",username:" username_CF ",sourcehost:" sourcehost_CF ",state:" state_CF ",country:" country_CF ",label:" label_CF ",timestamp:" timestamp_CF
	| TimeGenerated,Computer,timestamp_CF,latitude_CF,longitude_CF,destinationhost_CF,username_CF,sourcehost_CF,state_CF,country_CF,label_CF,Type,RawData
	```
</span>
</details>


<details>
<summary>
<h3>Step 6: Configure Microsoft Sentinel</h3>
</summary>
<span style="color:gray">

On Azure, go to **Microsoft Sentinel**.

1. Click on **law-honeypot1**.
2. Go to **Microsoft Sentinel | Workbooks**.
	1. Add **workbook**.
3. **New workbook**
	1. Click on **Edit**
		- **Remove text**
		- **Remove query**
		- **Add > Add query**
			- **Log Analytics Workspace Logs Query**
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
	2. **Save**
		- **Title:** Failed RDP World Map
		- **Subscription:** your_subscription
		- **Resource group:** Honeypotlab
		- **Location:** (US) West US 3
		- (uncheck) **Save content to an Azure Storage Account.**
		- Click on **Apply**
4. **Failed RDP World Map**.
	- Set **Auto refresh:** 5 minutes

</span>
</details>
