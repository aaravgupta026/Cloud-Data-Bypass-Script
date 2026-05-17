# Server-to-Server Cloud Archival Pipeline

## 📌 Overview
Transferring massive datasets (150GB+) between competing cloud providers (e.g., Google Drive to AWS S3) typically requires downloading the data to local hardware first. This traditional method bottlenecks local network bandwidth, risks data corruption upon connection drops, and consumes hours of time.

This project outlines a lightweight, server-to-server data pipeline utilizing Google Colab's ephemeral Linux environment. By mounting the source drive directly to a cloud virtual machine, this script bypasses local hardware entirely, zips loose directories on the fly, and uses the AWS CLI to route data directly to AWS S3 Glacier Deep Archive over enterprise fiber-optic backbones.

## 🏗️ System Architecture

`Google Drive (Source)` ➔ `Google Colab VM (Processing & Zipping)` ➔ `AWS S3 (Deep Archive)`

## 🚀 Key Features
* **Zero Local Bandwidth Consumption:** Executes a 100% cloud-side transfer.
* **Automated Cold Storage Tagging:** Programmatically forces the `--storage-class DEEP_ARCHIVE` flag to bypass expensive standard S3 storage tiers.
* **On-the-Fly Compression:** Utilizes the Colab VM's local scratch disk (~70GB) to compress thousands of loose files into a single `.zip` archive before AWS ingestion, optimizing transfer speeds and minimizing API put requests.

## 🛠️ Usage Instructions

### Prerequisites
1. An active AWS Account with an IAM User Access Key and Secret Key configured for CLI access.
2. An S3 Bucket created in your preferred region.
3. A Google Account with the source files in Google Drive.

### Execution Steps
1. Open the `cloud_data_bypass.ipynb` notebook in Google Colab.
2. Run the first cell to mount your Google Drive and install the AWS CLI.
3. Run `!aws configure` and input your AWS credentials when prompted in the terminal output. **(Never hardcode your keys into the script).**
4. Choose the appropriate transfer method:
   * **Method A:** For pre-zipped or standard folders, modify the directory paths and run the direct `aws s3 cp` command.
   * **Method B:** For directories containing thousands of loose files (e.g., photos), use the Zip-on-the-Fly block to compress, upload, and automatically clear the VM's temporary storage.

## ⚠️ Important Constraints
* **Session Timeouts:** Google Colab VMs will disconnect if left idle. For massive transfers (50GB+), keep the browser tab active to prevent the VM from wiping its state mid-transfer.
* **Temporary Storage Limits:** The Colab scratch disk holds ~70GB. When using the Zip-on-the-Fly method, batch your directories to stay under this limit before running the cleanup `!rm` command.
