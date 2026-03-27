# Splunk Deployment Server – Post-Deployment Validation

This documentation provides **validation, functional testing, security considerations, and troubleshooting steps** after deploying a Splunk Deployment Server (DS) and connecting clients (Universal Forwarders or Heavy Forwarders).

---

## 1. Deployment Server Health Verification

### 1.1 Verify DS Status

Run the following command on the Deployment Server:

```
splunk status
```

**Expected Results:**

- `splunkd is running`
- Management port (8089) accessible
- Deployment Server enabled in `server.conf`

---

### 1.2 Verify Deployment Apps

List apps available for deployment:

```
splunk list deploy-apps
```

**Expected:**

- All intended apps are listed
- No errors or missing apps

---

## 2. Functional Testing

### 2.1 Client Registration Verification

On each client (UF or HF):

```
splunk show deploy-poll
```

**Expected:**

- Client connected to DS (IP:8089)
- Apps pulled successfully
- No error messages

---

### 2.2 App Deployment Test

1. Place a test app in DS:

```
cp -r /path/to/test_app $SPLUNK_HOME/etc/deployment-apps/
```

2. Force a deployment:

```
splunk reload deploy-server
```

3. On each client, check deployed apps:

```
ls $SPLUNK_HOME/etc/apps/
```

**Expected:**

- Test app present on all registered clients
- No errors in `splunkd.log`

---

### 2.3 Client Forwarding Test (Optional)

For UFs forwarding to an Indexer or HF:

1. Create a test log:

```
echo "DS UF Test Event" >> /tmp/uf_test.log
splunk add monitor /tmp/uf_test.log
```

2. Confirm event appears on Indexer:

```
index=main "DS UF Test Event"
```

**Expected:**  
Events successfully forwarded from clients.

---

## 3. Security Considerations

- Restrict DS access to trusted networks  
- Ensure management port 8089 is firewalled  
- Use secure passwords and secrets  
- Enable role-based access control (RBAC) for app deployment  
- Audit logs enabled to track deployments  
- Limit administrative access to DS  

---

## 4. Troubleshooting Guide – Deployment Server

### Issue: Client Not Connecting

**Possible Causes:**

- Incorrect DS IP or port in `deploy-poll`  
- Firewall blocking port 8089  
- Network connectivity issues  
- DS not running  

**Resolution Steps:**

```
splunk show deploy-poll
telnet <ds_ip> 8089
splunk status
```

---

### Issue: Apps Not Deployed

**Causes:**

- Incorrect app permissions
- App syntax errors
- Client failed to pull updates  

**Resolution:**

```
splunk validate files
tail -f $SPLUNK_HOME/var/log/splunk/splunkd.log
splunk reload deploy-server
```

---

### Issue: Forwarding Not Working (UF/HF)

**Causes:**

- outputs.conf misconfigured
- Forwarding port blocked (9997)
- Network connectivity issues  

**Resolution:**

```
splunk list forward-server
splunk restart
```

---

## 5. Operational Readiness Checklist

Before marking DS deployment complete:

- [ ] Deployment Server running (`splunkd is running`)  
- [ ] Deployment apps listed correctly  
- [ ] All clients connected and pulling apps  
- [ ] Test app successfully deployed on all clients  
- [ ] Forwarding verified (if clients forward events)  
- [ ] Logs reviewed for errors  
- [ ] Security controls implemented  

---

# Splunk Universal Forwarder – Post-Deployment Validation

This documentation provides **validation, functional testing, security considerations, and troubleshooting steps** after deploying Universal Forwarders (UFs).

---

## 1. UF Service Verification

Check UF status on each client:

```
splunk status
```

**Expected:**

- `splunkd is running`

---

## 2. Forwarding Verification

List forward servers:

```
splunk list forward-server
```

**Expected:**

- Indexer/HF IPs listed as active forwards  
- `useACK=true` enabled for reliability

---

## 3. Inputs Verification

List monitored inputs:

```
splunk list monitor
```

**Expected:**

- All intended files/directories monitored  
- No `disabled` inputs unless intentionally configured

---

## 4. Functional Testing

### 4.1 Event Forwarding Test

1. Create test log:

```
echo "UF Test Event" >> /tmp/uf_test.log
splunk add monitor /tmp/uf_test.log
```

2. Verify on Indexer/HF:

```
index=main "UF Test Event"
```

**Expected:**  
Events successfully forwarded from UF.

---

### 4.2 Deployment Server Polling (Optional)

If UF connected to DS:

```
splunk show deploy-poll
```

**Expected:**  
Client connected, apps deployed successfully.

---

## 5. Security Considerations

- Ensure forwarding ports (9997) and management ports (8089) are accessible only to trusted nodes  
- Use secure passwords or secrets  
- Apply role-based access if using DS management  
- Limit administrative access on UF hosts  
- Audit logs enabled for monitored events  

---

## 6. Troubleshooting Guide – Universal Forwarder

### Issue: UF Not Forwarding

- Check `outputs.conf` configuration  
- Verify connectivity to Indexer/HF on port 9997  
- Restart UF and check `splunkd.log`  

### Issue: UF Not Monitoring Files

- Check `inputs.conf` syntax  
- Confirm file/directory exists and has read permissions  
- Restart UF and verify monitoring  

### Issue: DS Polling Fails (if connected)

- Verify DS IP/port in `deploy-poll`  
- Confirm DS is running and accessible on port 8089  

---

## 7. Operational Readiness Checklist

- [ ] UF service running on all hosts  
- [ ] Forward servers configured and active  
- [ ] Inputs correctly monitored  
- [ ] Test events successfully forwarded  
- [ ] Deployment server connection verified (if applicable)  
- [ ] Logs reviewed for errors  
- [ ] Security controls applied  

---

## 8. Expected Stable Behavior

A properly functioning UF:

- Continuously forwards events to Indexer/HF  
- Monitors all configured inputs  
- Pulls apps from Deployment Server (if configured)  
- Automatically reconnects after network interruptions  
- Logs no persistent errors
