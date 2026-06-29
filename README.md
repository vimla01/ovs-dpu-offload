# DPU-Accelerated OVS Offload Blueprint

This repository documents my understanding, research, and learning progress on the OPI Summer 2026 project "DPU-Accelerated OVS Offload"

The idea of the project is to build a reference setup for running VM networking through OVS where the actual datapath is offloaded to a DPU. First on plain KVM with RHEL, then on OpenShift Virtualization. NVIDIA BlueField-3 is the reference hardware since its software stack (OVS-DOCA, vDPA, DPF) is the most complete right now. After that the same setup gets ported to a second vendor, either Intel IPU or Marvell Octeon, and the gaps in that vendor's stack get documented and ideally contributed upstream.

## Why this matters
Normally a VM's network traffic goes through OVS running in software on the host CPU. That costs CPU cycles and adds latency. A DPU can run that whole datapath in hardware instead. The VM still just sees a normal virtio-net device, it has no idea the heavy lifting is happening on the DPU. That's where vDPA comes in, it lets the guest use a standard virtio interface while the real work happens on the accelerated hardware underneath.

## Track 1: BlueField-3 (reference)

### KVM on RHEL
- Set up BF3 in DPU mode, OVS-DOCA with hardware offload enabled (switchdev/eswitch mode)
- Set up VF/SF representors so guests can attach
- Attach libvirt guests through vhost-vdpa
- Actually confirm the offload is happening in hardware and not silently falling back to software (using flow dumps and hardware counters, not just trusting the config)
- Run throughput tests, offloaded vs not offloaded
- Test live migration of a VM that's using the offloaded path

### OpenShift Virtualization
- Get NVIDIA DPF running
- OVN-Kubernetes offloaded through OVS-DOCA on the DPU
- KubeVirt VMs running on top of that offloaded network

## Track 2: second vendor

### KVM on RHEL, vendor stack
- Marvell: octeon_ep vDPA driver (in-kernel) and the Octeon SDK
- Intel: IPDK / P4 and IDPF
- Write down exactly what's different from the BF3 setup, drivers, tools, config steps

### OpenShift Virtualization
- Right now the OPI / Red Hat DPU Operator is mostly built for network function offload and service function chaining, not the full OVN datapath offload that DPF gives you on BF3
- Part of this track is figuring out exactly what's missing and contributing toward closing that gap

## Where I actually am right now

I currently do not have access to DPU hardware (NVIDIA BlueField, Intel IPU, or Marvell Octeon), so I am using software-based environments and documentation-driven exploration to understand the architecture and prepare for implementation.

What I've done so far:
- Basic OVS bridges and flows in software, no offload involved yet
- Playing around with vdpa-sim (the kernel's software vDPA simulator) to get used to the actual vdpa/vhost-vdpa commands, even without real hardware behind them
- Basic libvirt VM setup with virtio-net
- Reading through OVS-DOCA and DPF docs to understand the architecture before touching anything

## Stuff I still need to figure out

- How exactly OVS-DOCA proves a flow is offloaded vs falling back to software
- How DPF actually wires into OVN-Kubernetes under the hood
- The real differences between mlx5_vdpa, octeon_ep and IDPF as driver models
- How much work the OPI DPU Operator actually needs to support full datapath offload

## Background I'm building on

- Linux on RHEL: systemd, kernel modules, networking stack
- KVM/libvirt/QEMU, virtio, vhost, vDPA basics
- OVS: bridges, flows, OpenFlow, switchdev, VF/SF representors
- Kubernetes: CRDs, operators, CNI
- Networking: VLAN, VXLAN/Geneve, SR-IOV, throughput testing

## Some references I've been using

- OPI project site: https://opiproject.org/
- NVIDIA BlueField-3 / DOCA / DPF docs
- OVN-Kubernetes docs
- OPI DPU Operator repo on GitHub
