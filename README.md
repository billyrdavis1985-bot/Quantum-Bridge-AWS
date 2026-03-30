# ⚛️ AWS Braket Quantum Hardware Access

This repository contains the verified connection protocol for accessing **IonQ** and **Rigetti** Quantum Processing Units (QPUs) using the Amazon Braket SDK within a Google Colab environment.

## 🎯 Purpose
To provide a "ready-to-run" template that handles the complex authentication between Google Colab and AWS, allowing for the execution of quantum circuits on real hardware.

## 🛠️ Prerequisites
To use this notebook, you must have:
1. **AWS Account**: With Amazon Braket enabled.
2. **IAM Credentials**: An Access Key and Secret Key with `AmazonBraketFullAccess`.
3. **S3 Bucket**: Created in the `us-east-1` (N. Virginia) or `us-west-1` (N. California) region to store quantum task results.

## 🚀 Activation Steps
1. **Colab Secrets**: Store your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in the Colab "Secrets" (key icon) sidebar.
2. **Library Install**: The notebook automatically handles `!pip install amazon-braket-sdk`.
3. **Region Lock**: Hardware like **IonQ Forte-1** is typically region-locked. This notebook is pre-configured for `us-east-1`.

## 🔬 Hardware Targets
* **IonQ Forte-1**: High-fidelity trapped-ion QPU.
* **Rigetti Ankaa-2**: Superconducting qubit architecture.
* **SV1**: AWS high-performance state vector simulator for debugging.
