# Digital Forensics Lab — Operation Web Intrusion

An interactive, automated hands-on lab designed to teach the fundamentals of the **Digital Forensics Lifecycle** using **Autopsy**. 

This lab simulates a realistic corporate web server compromise where an attacker bypasses file upload security filters to upload a PHP backdoor, exfiltrates sensitive database tables, and executes anti-forensics scripts to wipe their tracks. Analysts must examine a virtual disk image to reconstruct the attack timeline, carve deleted files, identify extension signature mismatches, and identify the attacker's source IP.

---

## ⚙️ Quick Start Guide

To run this lab locally, you will need a **Kali Linux** virtual machine:

1.  **Clone or download** this repository onto your Kali Linux VM Desktop.
2.  Open your terminal, navigate to the directory, and make the setup script executable:
    ```bash
    chmod +x automate_lab.sh
    ```
3.  Execute the script with root privileges to build the virtual evidence disk:
    ```bash
    sudo ./automate_lab.sh
    ```
4.  The script will output `evidence.raw` on your **Desktop** along with its MD5 and SHA-256 integrity hashes.
5.  Launch Autopsy:
    ```bash
    sudo autopsy
    ```
6.  Open the web interface at `http://localhost:9999/autopsy`

---

## 👤 About the Author

Created and maintained by **Chaitanya Eshwar Prasad**. Connect with me or follow my work:

[![Website](https://img.shields.io/badge/Website-chaitanyaeshwarprasad.com-005F9E?style=for-the-badge&logo=google-chrome&logoColor=white)](https://chaitanyaeshwarprasad.com)
[![GitHub](https://img.shields.io/badge/GitHub-chaitanyaeshwarprasad-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/chaitanyaeshwarprasad)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Chaitanya%20Eshwar%20Prasad-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/chaitanya-eshwar-prasad)
[![YouTube](https://img.shields.io/badge/YouTube-@chaitanya.eshwar.prasad-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@chaitanya.eshwar.prasad)
[![Instagram](https://img.shields.io/badge/Instagram-@acep.tech.in.telugu-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/acep.tech.in.telugu)
[![YesWeHack](https://img.shields.io/badge/YesWeHack-chaitanya--eshwar--prasad-FF5E00?style=for-the-badge&logo=securityscorecard&logoColor=white)](https://yeswehack.com/hunters/chaitanya-eshwar-prasad)

---
*Disclaimer: This repository is for educational and training purposes only.*
