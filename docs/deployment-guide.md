# Splunk Deployment Server – Deployment Procedure

## Configure Deployment Server

`Note: This method is intended for Deployment Server usage.`

---

### Method 1: Configuration File

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.

2. **Navigate to `$SPLUNK_HOME/etc/system/local`:**
   ```bash
   cd /opt/splunk/etc/system/local
   ```

3. **Create or Edit server.conf:**

   **Create:**
   ```bash
   touch server.conf
   ```

   **Edit:**
   ```bash
   vi server.conf
   ```

4. **Add Deployment Server Configuration:**
   ```ini
   [deploymentServer]
   disabled = 0
   mgmtPort = 8089
   ```

   - `disabled = 0` → Enables Deployment Server  
   - `mgmtPort` → Management port for client communication  

5. **Save Changes**

6. **Restart Splunk:**
   ```bash
   /opt/splunk/bin/splunk restart
   ```

---

## Configure Deployment Apps

`Note: Deployment apps are pushed from the Deployment Server to clients.`

1. **Place Apps in Deployment Directory:**
   ```bash
   cp -r /path/to/my_app $SPLUNK_HOME/etc/deployment-apps/
   ```

2. **Verify Deployment Apps:**
   ```bash
   /opt/splunk/bin/splunk list deploy-apps
   ```

---

## Configure Deployment Clients (UF or HF)

`Note: Execute this procedure on each Deployment Client.`

---

### Method 1: Command Line

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.

2. **Navigate to `$SPLUNK_HOME/bin`:**
   ```bash
   cd /opt/splunk/bin
   ```

3. **Configure Deployment Server:**
   ```bash
   ./splunk set deploy-poll <deployment_server_ip>:8089 -auth <username>:<password>
   ```

   - `<deployment_server_ip>` → Deployment Server hostname or IP  
   - `8089` → Management port  
   - `<username>:<password>` → Client authentication credentials  

4. **Restart Splunk:**
   ```bash
   ./splunk restart
   ```

5. **Verify Deployment Client Status:**
   ```bash
   ./splunk show deploy-poll
   ```

   **Expected Output:**
   ```
   Client is configured to pull apps from DS: <deployment_server_ip>:8089
   Apps deployed successfully
   ```

---

# Splunk Universal Forwarder – Deployment Procedure

## Configure Universal Forwarder

`Note: This procedure configures a Universal Forwarder to send data to Indexers or a Heavy Forwarder.`

---

### Method 1: Configuration File

1. **Access Command Line Interface**

2. **Navigate to `$SPLUNK_HOME/etc/system/local`:**
   ```bash
   cd /opt/splunkforwarder/etc/system/local
   ```

3. **Create or Edit outputs.conf:**

   **Create:**
   ```bash
   touch outputs.conf
   ```

   **Edit:**
   ```bash
   vi outputs.conf
   ```

4. **Add Forwarding Configuration:**
   ```ini
   [tcpout]
   defaultGroup = indexer_group
   indexAndForward = false

   [tcpout:indexer_group]
   server = <indexer1_ip>:9997,<indexer2_ip>:9997,<indexer3_ip>:9997
   autoLB = true
   useACK = true
   ```

   - `server` → Indexers or Heavy Forwarder endpoints  
   - `autoLB` → Enables load balancing  
   - `useACK` → Ensures reliable delivery  

5. **Save Changes**

---

### Method 2: Configuration File

1. **Create or Edit inputs.conf:**
   ```bash
   touch inputs.conf
   vi inputs.conf
   ```

2. **Add Data Input Configuration:**
   ```ini
   [monitor:///var/log/syslog]
   index = main
   sourcetype = syslog
   disabled = false
   ```

3. **Save Changes**

---

### Restart Universal Forwarder

```bash
/opt/splunkforwarder/bin/splunk restart
```

---

## Verify Forwarding

### 1. Verify Forward Servers

```bash
/opt/splunkforwarder/bin/splunk list forward-server
```

**Expected Output:**
```
Active forwards:
<indexer1_ip>:9997
<indexer2_ip>:9997
<indexer3_ip>:9997
```

---

### 2. Test Event Forwarding

```bash
echo "UF Test Event" >> /tmp/uf_test.log
/opt/splunkforwarder/bin/splunk add monitor /tmp/uf_test.log
```

Verify that the event appears on the target Indexer or Heavy Forwarder.

---

## Notes & Best Practices

- Use `useACK=true` for reliable forwarding  
- Use `autoLB=true` for load balancing across multiple indexers  
- Ensure ports are reachable:
  - Deployment Server: `8089`
  - Forwarding: `9997`  
- Monitor internal logs:
  ```bash
  $SPLUNK_HOME/var/log/splunk/splunkd.log
  ```
- Restrict network access to trusted clients only  
