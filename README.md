# Unsupported OS Migration: Windows 11 to Amazon EC2

A lab that simulates migrating a server whose operating system has no supported EC2 AMI (Windows 11). An EC2 instance with nested virtualization acts as the on-premises hypervisor. A Windows 11 VM is built on it with Hyper-V, exported, uploaded to S3, imported as an AMI with VM Import, and launched as a new EC2 instance.

![Architecture](docs/architecture.png)

## Flow

1. Build a VPC, public subnet, Internet Gateway route and RDP security group (Region `us-east-2`).
2. Launch a Windows `m8i.2xlarge` instance and enable **nested virtualization** in its CPU options.
3. Install Hyper-V and create a **generation 2** VM on an internal `NAT-Switch` (192.168.100.0/24).
4. Enable the vTPM (needs a key protector), install Windows 11, enable RDP, create a test file.
5. Export the VM and copy the VHDX to S3 (`aws s3 cp`).
6. Run `aws ec2 import-image` (BYOL, UEFI) to register an AMI.
7. Launch an EC2 instance from the AMI and check the test file.

## Repository layout

| Path | Content |
|---|---|
| `docs/architecture.png` | Architecture diagram |
| `docs/screenshots/` | Step-by-step screenshots (redact account IDs first) |
| `scripts/hyperv-host.ps1` | PowerShell commands for the Hyper-V host |
| `scripts/import-ami.sh` | AWS CLI commands for upload and import |

## Key points

- Windows 11 needs **generation 2** (UEFI) and a **vTPM**; `Enable-VMTPM` fails until `Set-VMKeyProtector -NewLocalKeyProtector` is run.
- Nested virtualization is an instance CPU option; without it Hyper-V cannot start a VM.
- `import-image` needs `--license-type BYOL` and `--boot-mode uefi` for this disk.
- The lab security group opens all TCP ports to `0.0.0.0/0`. Use it only for short-lived labs and restrict the source in real use.

## Notes

Scripts follow the commands shown in the lab screenshots; commands marked "reference" are the standard way to do a step that was done in the GUI. Replace bucket names, paths and IDs with your own.
