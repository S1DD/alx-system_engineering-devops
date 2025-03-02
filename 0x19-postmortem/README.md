# Post-Mortem Incident Report: Nexus App Outage of 2023

![Nginx-PostgreSQL](https://www.dropbox.com/scl/fi/d24o0nn7wypysny1owe4m/Nginx-PostgreSQL.png?rlkey=tfe5junq6eno7f59jxfwf2gw7&st=i6bo512f&raw=1)


## Date:
October 31, 2023

## Duration:
4.5 hours (2:00 PM UTC – 6:30 PM UTC)

## Severity:
SEV-1

### **Issue Summary:**

On October 31, 2023, Nexus Banking app experienced a critical outage lasting **4.5 hours**. At **2:00 PM UTC**, users were unable to log in, send money, or view their balances due to a **dual failure**:

1. **Database Migration Error:** A migration script for PostgreSQL mistakenly ran the command `DROP TABLE transactions` instead of `ALTER TABLE`, resulting in the deletion of all transaction records.
2. **NGINX Misconfiguration:** An expired SSL certificate combined with outdated routing rules blocked 70% of incoming user traffic, delaying detection of the issue.

As a result, **1.5 million users** were locked out, and **22,000 transactions** were delayed, including a critical **$10,000 wedding venue deposit**. Social media was flooded with complaints, and one review humorously stated, "Nexus made my money ✨disappear✨."

### **Timeline:**

- **1:45 PM:** PostgreSQL migration script is deployed to production.
- **2:00 PM:** Migration script executes `DROP TABLE transactions` instead of `ALTER TABLE`, erasing all transaction data.
- **2:05 PM:** NGINX server begins rejecting SSL requests due to an expired certificate. Engineers suspect a minor routing issue.
- **2:30 PM:** NGINX errors mask database alerts. Users start reporting login failures. The team assumes a connectivity problem.
- **3:00 PM:** Investigation reveals the missing transactions table. Backup restoration process begins.
- **3:30 PM:** Backup restoration fails due to corrupted backup files. NGINX issues resolved, but the app remains down.
- **4:00 PM:** The missing transactions table is rebuilt using PostgreSQL transaction logs, and recovery begins.
- **6:30 PM:** Services are fully restored. Social media backlash continues.

### **Root Cause and Resolution:**

#### **Root Cause:**
1. **Database Script Error:** A typo in the migration script resulted in the execution of `DROP TABLE transactions` instead of `ALTER TABLE transactions`, which deleted all transaction data in the PostgreSQL database.
2. **NGINX Misconfiguration:** The NGINX server had an expired SSL certificate, and the routing configuration hadn’t been updated since 2021. This misconfiguration blocked 70% of incoming traffic and delayed the detection of the issue.
3. **Backup Corruption:** The most recent usable backup was 48 hours old due to a storage issue. The corrupted backups prevented a quick restore, delaying the process.

#### **Resolution:**
1. **Database Fix:** The missing PostgreSQL transactions table was manually reconstructed from available transaction logs. All transaction data was carefully reviewed to ensure that no entries were lost.
2. **NGINX Fix:** The expired SSL certificate was renewed, and outdated routing configurations were updated to properly route user traffic.
3. **Backup Fix:** After discovering the backup corruption, a secondary backup system was used to restore the most recent usable data, minimizing the loss.

### **Corrective and Preventative Measures:**

#### **Broad Improvements:**
1. **NGINX Server Maintenance:** Automate SSL certificate renewals, add alerts for expiring certificates, and set up a quarterly review for routing configurations.
2. **Database Migration Safeguards:** Enforce a two-person approval process for all database migration scripts, and ensure staging and automated tests are conducted before any changes are made in production.
3. **Backup Integrity:** Implement daily verification of backups, ensure alerts for any failures, and store backups in a geographically redundant system.

#### **Specific Tasks:**
1. **NGINX Overhaul:**
   - Automate SSL certificate renewal and configure alerts for expiring certificates.
   - Review and update routing configurations quarterly.
   
2. **Database Safeguards:**
   - Require two engineers to review and approve migration scripts before deployment.
   - Ensure that all migrations are tested in staging environments before deployment.
   - Implement automated validation for database schema changes.
   
3. **Backup Process:**
   - Introduce automated integrity checks for backups, with alerts for corruption.
   - Ensure backup systems are geographically redundant and can quickly restore data.
   
4. **Training:**
   - Conduct mandatory workshops for engineers on safe migration practices, PostgreSQL management, and NGINX configuration.
   - Provide additional training on backup management and troubleshooting to engineering teams.

### **Conclusion:**

The outage on October 31 was caused by a combination of human error (incorrect database script), outdated infrastructure (NGINX SSL and routing misconfigurations), and faulty backup systems. While the issue was resolved in 4.5 hours, the user impact was significant. Going forward, the team will implement safeguards to prevent such incidents, including automated monitoring and validation processes, along with improved backup procedures.

**Key Quote:**  
“We turned ‘Move fast and break things’ into ‘Move fast and delete money.’ Lessons learned.” – DevOps Lead
