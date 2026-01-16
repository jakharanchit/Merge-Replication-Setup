# Prerequisites Checklist

> **Purpose**: Complete this checklist before beginning replication setup. Each item must be verified on ALL machines participating in replication.

---

## 1. Network & Connectivity

### IP Configuration
- [ ] Publisher has a **static IP address** or resolvable hostname
- [ ] All Subscribers can resolve Publisher hostname via DNS or hosts file
- [ ] Run `ping [PublisherHostname]` from each Subscriber - responds successfully

### Port Accessibility
- [ ] **TCP 1433** (SQL Server) is open inbound on Publisher
- [ ] **UDP 1434** (SQL Browser) is open inbound on Publisher
- [ ] **TCP 445** (SMB) is open inbound on Publisher (for snapshot folder)
- [ ] Firewall allows `sqlservr.exe` through on all machines

### Connection Stability
- [ ] Network connection is stable (Ethernet preferred over Wi-Fi)
- [ ] Latency between Publisher and Subscribers is acceptable (<100ms recommended)

**Verification Command:**
```cmd
# From Subscriber, test SQL connectivity
telnet [PublisherIP] 1433

# Or using PowerShell
Test-NetConnection -ComputerName [PublisherIP] -Port 1433
```


---

## 2. SQL Server Configuration

### Instance Settings
- [ ] SQL Server **TCP/IP protocol** is enabled (SQL Server Configuration Manager)
- [ ] SQL Server is listening on **port 1433** (or documented custom port)
- [ ] **SQL Server Browser** service is running (if using named instances)
- [ ] Authentication mode is set to **Mixed Mode** (SQL + Windows)


### Server Name Verification
- [ ] Run `SELECT @@SERVERNAME` - result matches Windows computer name
- [ ] If mismatch, fix using:
  ```sql
  sp_dropserver 'OldName';
  sp_addserver 'NewName', 'local';
  -- Restart SQL Server
  ```

### Version Compatibility
- [ ] Publisher version: _________________ (SQL Server Standard/Enterprise)
- [ ] Subscriber version: _________________ (must be same or higher)
- [ ] Replication feature installed on all instances

**Verification Query:**
```sql
SELECT @@VERSION;
SELECT SERVERPROPERTY('Edition');
SELECT SERVERPROPERTY('IsFullTextInstalled');
```

---

## 3. Service Accounts

### Dedicated Account Created
- [ ] Windows account created: `________________` (e.g., `sql_repl_agent`)
- [ ] Password set to **never expire**
- [ ] Account exists on Publisher
- [ ] Account has **Log on as a service** right
- [ ] Account has **Log on as a batch job** right

### SQL Server Login
- [ ] Account added as SQL Server login on Publisher
- [ ] Login has `sysadmin` role (for setup) or minimal required permissions:
  - [ ] `db_owner` on publication database
  - [ ] `db_owner` on distribution database

**PowerShell to create local user:**
```powershell
$Password = Read-Host -AsSecureString "Enter Password"
New-LocalUser -Name "sql_repl_agent" -Password $Password -PasswordNeverExpires
```

> 🔐 For detailed security configuration, see [Security Guide](../setup/02-security-guide.md)

---

## 4. Snapshot Folder

### Folder Configuration
- [ ] Folder created on Publisher: `________________` (e.g., `C:\ReplData`)
- [ ] Folder is **shared** with share name: `________________` (e.g., `ReplData`)
- [ ] UNC path works: `\\[PublisherName]\[ShareName]`


### Share Permissions
- [ ] Service account has **Change** permission on share
- [ ] `SYSTEM` account has **Full Control**
- [ ] `Administrators` have **Full Control**

### NTFS Permissions
- [ ] Service account has **Read & Execute**
- [ ] Service account has **List folder contents**
- [ ] Service account has **Read**
- [ ] Service account has **Write** (if using pull subscriptions)


**Verification Test:**
```
# From Subscriber, open Run (Win+R):
\\PublisherName\ReplData
# Should open folder (may prompt for credentials)
```

---

## 5. Storage Requirements

### Publisher
- [ ] Free disk space for snapshot: _________ GB (minimum 2x database size)
- [ ] Database files are not on system drive (recommended)
- [ ] Transaction log has adequate space for replication tracking

### Subscriber
- [ ] Free disk space: _________ GB (minimum 1.5x expected data size)
- [ ] SQL Express database size limit: **10 GB** (plan accordingly)

---

## 6. Time Synchronization

- [ ] All servers synchronized to same NTP source
- [ ] Time difference between servers: _________ (must be <5 minutes)
- [ ] Time zone settings are consistent or properly configured

**Verification:**
```cmd
w32tm /query /status
```

---

## 7. Additional Considerations

### Firewall Rules Created
- [ ] Inbound rule for SQL Server (TCP 1433)
- [ ] Inbound rule for SQL Browser (UDP 1434)
- [ ] Inbound rule for File Sharing (TCP 445)
- [ ] Program rule for `sqlservr.exe`

### Hosts File (if DNS unavailable)
- [ ] Publisher hostname mapped on all Subscribers
- [ ] File location: `C:\Windows\System32\drivers\etc\hosts`
- [ ] Example entry: `192.168.1.100  SQLPUBLISHER`

---

## Next Steps

Once all items are checked:
1. **[Implementation Guide](../setup/01-implementation-guide.md)** → Begin setup
2. **[Troubleshooting](../operations/02-troubleshooting-guide.md)** → If verification failed
