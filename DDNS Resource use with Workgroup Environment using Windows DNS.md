# Integrating Express Cluster DDNS resource with Windows DNS Servers in a Workgroup Environment

This article provides a comprehensive guide on configuring an Express Cluster DDNS (Dynamic Domain Name System) resource to work with Windows DNS Servers in a workgroup environment. It covers step-by-step instructions and best practices to ensure seamless integration and operation of DDNS services within a non-domain network setup.

# Reference

## EXPRESSCLUSTER X
- https://www.nec.com/en/global/prod/expresscluster/en/doc/manuals/W52_SG_EN_03.pdf

## System configuration
- Servers: 2 Nodes with Mirror Disk and DDNS resource
- OS: Windows Server 2022
- SW: EXPRESSCLUSTER X 5.2
- Failover Group Resource
    - DDNS
    - MD
       - Cluster Partition: D: Drive
       - Data Partition:    E: Drive

- DNS Servers: 2 Nodes (Primary DNS and Secondary DNS Server)
- OS: Windows Server 2022
- SW: DNS Server Role has been installed on both the Servers.

## Note
  - All primary servers, secondary servers, and DNS servers should be reachable via their IP addresses.
  - Each server belongs to a different network, you can use ddns resource with Dynamic DNS Server instead of fip   address.

```
  e.g. 

  (Network)
  |
  |      +--------------------------------+
  |      | Cluster node 1 (Windows Server)|
  |------+ EXPRESSCLUSTER 5.2             |
  |      | IP: 10.0.7.178/24              |
  |      +--------------------------------+
  ~
  |      +--------------------------------+
  |      | Cluster node 2 (Windows Server)|
  |------+ EXPRESSCLUSTER 5.2             |
  |      | IP: 10.0.8.179/24              |
  |      +--------------------------------+
  ~
  |      +-----------------------+
  |      | DNS server1 (Windows) |
  |------+ IP: 10.0.7.185/24     |
  |      +-----------------------+
  ~
  |      +-----------------------+
  |      | DNS server2 (Windows) |
  |------+ IP: 10.0.8.186/24     |
  |      +-----------------------+

```


### **1. Setting Up the Primary DNS on Server1**

#### **Step 1: Install DNS Server Role**

1. **Open Server Manager:**
   - Click the **Start** button.
   - Type **Server Manager** and press Enter.

2. **Add Roles and Features:**
   - In Server Manager, click **Add roles and features**.
   - Click **Next** through the wizard until you reach the **Server Roles** page.

3. **Select DNS Server Role:**
   - Check the **DNS Server** role.
   - Click **Next**, and then click **Install**. Wait for the installation to complete.

#### **Step 2: Configure the Primary DNS Server**

1. **Open DNS Manager:**
   - Press **Win + R**, type `dnsmgmt.msc`, and press Enter. This opens the DNS Manager.

2. **Create a New Forward Lookup Zone:**
   - In the DNS Manager, expand the server node, right-click **Forward Lookup Zones**, and select **New Zone**.
   - Click **Next** to start the New Zone Wizard.

3. **Configure Zone Type:**
   - Select **Primary zone** and click **Next**.
   - Choose **To all DNS servers in this domain** (if it’s a domain environment) or **Only this server** (for a standalone workgroup), and click **Next**.

4. **Name the Zone:**
   - Enter the zone name (e.g., `ecx.local`) and click **Next**.

5. **Set Zone File:**
   - Choose **Create a new file with this file name** and accept the default name or specify a new one. Click **Next**.

6. **Dynamic Updates:**
   - Choose the option for dynamic updates (e.g., **Allow only secure dynamic updates**) and click **Next**.

7. **Finish Setup:**
   - Review your settings and click **Finish**.


#### **Step 3: Configure Forwarders (if necessary)**

1. **Open DNS Manager:**
   - In the DNS Manager, right-click your DNS server node and select **Properties**.

2. **Configure Forwarders:**
   - Go to the **Forwarders** tab.
   - Click **Edit**, and add IP addresses of external DNS servers if you want to resolve external domains.

### **2. Setting Up the Secondary DNS Server on Server2**

#### **Step 1: Install DNS Server Role**

1. **Repeat the installation steps for the secondary DNS server:**
   - Follow the same procedure as described for the primary server to install the DNS Server role.

#### **Step 2: Configure the Secondary DNS Server**

1. **Open DNS Manager:**
   - As before, press **Win + R**, type `dnsmgmt.msc`, and press Enter.

2. **Create a New Secondary Zone:**
   - Right-click **Forward Lookup Zones** and select **New Zone**.
   - Click **Next** to start the New Zone Wizard.

3. **Configure Zone Type:**
   - Select **Secondary zone** and click **Next**.

4. **Specify the Primary DNS Server:**
   - Enter the IP address of the primary DNS server where the zone is hosted and click **Next**.

5. **Name the Zone:**
   - Enter the zone name (the same as on the primary server, e.g., `ecx.local`) and click **Next**.

6. **Finish Setup:**
   - Review your settings and click **Finish**.

#### **Step 3: Verify Zone Transfer Settings**

1. **Open DNS Manager:**
   - In the DNS Manager, right-click your secondary zone and select **Properties**.

2. **Check Zone Transfer Settings:**
   - Go to the **Zone Transfers** tab.
   - Ensure **Allow zone transfers** is checked and configure any necessary restrictions (e.g., **Only to the following servers** if you want to restrict transfers to specific servers).

### **3. Configure Clients to Use DNS Servers**

1. **Set DNS Addresses Manually:**
   - On each client machine, go to **Control Panel** > **Network and Sharing Center** > **Change adapter settings**.
   - Right-click the network connection, select **Properties**, then select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
   - Select **Use the following DNS server addresses** and enter the IP addresses of your primary and secondary DNS servers.

### **4. Testing and Verification**

1. **Test DNS Resolution:**
   - Open a command prompt on a client machine and use `nslookup` to test DNS resolution:
     ```bash
     nslookup ecx.local
     ```

2. **Verify Secondary Server:**
   - Ensure the secondary DNS server can resolve queries and has replicated the zone data from the primary server.

3. **Monitor and Troubleshoot:**
   - Regularly check DNS logs and performance to ensure everything is functioning correctly.

### **EXPRESSCLUSTER X Setup a basic cluster**
Please refer [Basic Cluster Setup](https://www.nec.com/en/global/prod/expresscluster/en/doc/manuals/W52_IG_EN_02.pdf)

### **Create a Base Cluster**
- **Create a failover group that includes a mirror disk resource and DDNS resource**.
1. Right click on failover and click Add Resource in builder window.
1. Choose DDNS resource.
   - **Info**
      - Type: Dynamic DNS resource
      - Name: DDNS
      - Comments: Add optional comments if required.

   - **Dependancy**: Default

   - **Recovery Operation**: As per the Standerd
   - **Details**:
      - Common:
         - Virtual Host Name: milestone.ecx.com
         - IP Address: 10.0.7.178
         - DDNS Server: 10.0.7.185,10.0.8.186
         - Port No.: 53
         - Cache TTL: 0 Sec.
         - Execute Dynamic Update Periodically: Checked
         - Update Interval: 60 Min.
         - Delete the Registered IP Address: Checked
   - **Extension**: Default
1. Click OK.
1. Click Next (for default values) to learn more about parameters please refer the Express Cluster Reference Guide. Click Next.
1. Click **Finish**.
1. Upload the cluster configuration and start the cluster WebUI.

Verify the functionality of the DDNS resource by conducting failover and failback tests within the ECX cluster. This process involves:

1. **Failover Testing:**
   - Simulate a failure of the primary node or resource in the ECX cluster to ensure that the DDNS resource correctly shifts to the secondary node.
   - Monitor the transition to confirm that DNS records are updated appropriately and that the secondary node takes over DNS resolution without disruption.

2. **Failback Testing:**
   - After the failover test, restore the primary node or resource to its operational state.
   - Verify that the DDNS resource transitions back to the primary node and updates DNS records accordingly.
   - Ensure that normal DNS operations are resumed with minimal downtime or issues.

This testing ensures that the DDNS resource can handle node failures and recoveries effectively, maintaining reliable DNS services throughout the cluster’s lifecycle.





