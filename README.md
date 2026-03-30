# ⚛️ AWS Braket Quantum Hardware Access

A verified Google Colab-to-AWS Braket execution bridge for running 
quantum circuits on live QPU hardware.

This protocol handles the complete authentication stack between Google 
Colab and AWS, resolves common 403 Forbidden and session attribution 
errors, and provides a hardened `AwsSession` wrapper pattern for stable 
IonQ and Rigetti device access.

---

## 🎯 Purpose

Most Braket tutorials assume a local environment with pre-configured 
AWS credentials. This notebook solves the harder problem: running 
high-fidelity quantum circuits on real QPU hardware directly from 
Google Colab, with zero local setup.

---

## 🛠️ Prerequisites

| Requirement | Details |
|---|---|
| AWS Account | Amazon Braket service enabled |
| IAM Credentials | Access Key + Secret Key with `AmazonBraketFullAccess` |
| IAM Role | `AWSServiceRoleForAmazonBraket` configured |
| S3 Bucket | `us-east-1` for IonQ · `us-west-1` for Rigetti |
| Google Colab | Free tier sufficient |

---

## 🚀 Quickstart

### 1. Store credentials in Colab Secrets
Open the **key icon** in the Colab sidebar and add:
```
AWS_ACCESS_KEY_ID      →  your access key
AWS_SECRET_ACCESS_KEY  →  your secret key
```
Enable **"Notebook access"** for both.

### 2. Install dependencies
The notebook handles this automatically:
```bash
pip install amazon-braket-sdk boto3 --upgrade
```

### 3. The critical pattern
Standard `boto3.Session` objects lack the region attribute the 
Braket SDK requires. Always wrap before constructing any device:
```python
import boto3
from braket.aws import AwsSession, AwsDevice

boto_sess = boto3.Session(
    aws_access_key_id=aws_access,
    aws_secret_access_key=aws_secret,
    region_name="us-east-1"
)

# Required wrapper — do not skip this step
aws_session = AwsSession(boto_session=boto_sess)

device = AwsDevice(
    "arn:aws:braket:us-east-1::device/qpu/ionq/Forte-1",
    aws_session=aws_session
)
```

---

## 🔬 Hardware Targets

| Device | Type | Region | Notes |
|---|---|---|---|
| IonQ Forte-1 | Trapped-ion QPU | `us-east-1` | ~99% 3-qubit GHZ fidelity |
| Rigetti Ankaa-3 | Superconducting QPU | `us-west-1` | ~91% 3-qubit GHZ fidelity |
| Amazon SV1 | State vector simulator | `us-east-1` | Free · use for debugging |
| Amazon TN1 | Tensor network simulator | `us-east-1` | Large circuit testing |

> **Cost note:** QPU shots are billed per-task. Always validate 
> circuits on SV1 before submitting to hardware.

---

## 🧪 The Golden Circuit

The validation circuit used across all hardware targets:
```python
from braket.circuits import Circuit

# 3-qubit GHZ state — canonical cross-device benchmark
golden_circuit = Circuit().h(0).cnot(0, 1).cnot(1, 2)

task = device.run(golden_circuit, shots=100, 
                  s3_destination_folder=(bucket, prefix))
print(task.id)
```

Expected ideal output: `{'000': ~50, '111': ~50}`  
Hardware deviation from this distribution = noise fingerprint.

---

## 🛰️ Task Recovery

QPU jobs are asynchronous and can queue for minutes to hours. 
Recover any task by ARN without keeping a session alive:
```python
from braket.aws import AwsQuantumTask, AwsSession
import boto3

def recover_task(task_arn):
    region = task_arn.split(":")[3]
    boto_sess = boto3.Session(
        aws_access_key_id=aws_access,
        aws_secret_access_key=aws_secret,
        region_name=region
    )
    aws_session = AwsSession(boto_session=boto_sess)
    task = AwsQuantumTask(arn=task_arn, aws_session=aws_session)
    
    if task.state() == "COMPLETED":
        return task.result().measurement_counts
    return task.state()
```

---

## ⚠️ Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `AttributeError: Session has no attribute 'region'` | Raw boto3 session passed to AwsDevice | Wrap with `AwsSession(boto_session=...)` |
| `403 Forbidden` on device access | Missing IAM role | Enable `AWSServiceRoleForAmazonBraket` in IAM console |
| `ValidationException` on `search_devices` | Filter format incorrect | Pass `filters=[]` — never omit the parameter |
| Task stuck in `QUEUED` | QPU offline or high demand | Check device status · use SV1 for immediate testing |

---

## 📁 Repository Structure
```
├── AWS_Braket_Quantum_Access.ipynb   # Main execution notebook
└── README.md
```

---

## 🔗 Context

This repository is part of the **IRMB (Infinite Resilience Matrix 
Bridge)** research program — Phase 7G: Bell Coordination. The 
Braket access layer provides real QPU measurement data to a 
multi-LLM orchestration system studying quantum-AI coordination.

- Phase 7G Design 1: Quantum fleet abstraction & Golden Circuit validation  
- Phase 7G Design 2: Quantum Error Correction (QEC) — completed  
- Phase 7G Design 3: Live Bell/GHZ coordination experiments — active  

---

## 📄 License

MIT — see `LICENSE` for details.

---

*Built under the IRMB motto: **Full Force Eternal** · Romans 8:28*
