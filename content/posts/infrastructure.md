---
title: "Choosing the Platform: Oracle Cloud and Oracle Linux"
date: 2026-03-15
summary: "Why I picked Oracle Cloud's free tier and Oracle Linux for a production-style VPS."
tags: [infrastructure, cloud, linux]
---

## Why not a spare machine?

I don't have one and buying one for a learning project is expensive for little learning value (unless we're talking about a complete setup). Cloud VMs teach the same skills (networking, user management, firewalls, etc.) with the added challenge of managing the cloud-specific network layers.

## Oracle Cloud's Always Free Tier

OCI provides an ARM-based Ampere A1 instance in their permanently free tier with sufficient specs for a personal project: 24 GB of RAM, 4 OCPUs, and a large monthly outbound transfer allowance. It costs nothing and is always free, but the specs may change in the future pushing the VM out of the free tier boundary.

The ARM architecture means packages are `aarch64` instead of `x86_64`. This rarely matters since almost everything in Oracle repos ships an ARM build (building from source is also a possibility). This is worth knowing if I'm ever pulling container images.

## Why Oracle Linux

The main reason is that it is the only provided distro with the Ampere A1 instance at the moment. Even if that wasn't the case, Oracle Linux is a solid choice here because it is built to be compatible with the whole OCI infrastructure including utilities, cloud-init hooks, and provider configurations. It is worth mentioning that Oracle Linux is a rebuild from Red Hat Enterprise Linux, so they share the same package manager (dnf), same SELinux policies, and same directory conventions.

This matters because enterprise environments run RHEL, Rocky, AlmaLinux, or Oracle Linux. All of them share the same skill set under different branding. This will become relevant when I start preparing my Red Hat Certified System Administrator exam, a well-respected certification in the field. With that being said, Ubuntu becomes a secondary choice because of the differences between the two families of Linux distros, some examples are `ufw` vs `firewalld`, package manager `dnf` vs `apt` etc.

## SSH key

When setting up the instance, OCI offered either to generate an SSH key pair to connect to the machine, or to use my own public key instead. I chose the latter.
The reasoning behind this choice is that a private key should never exist on any machine except mine. Having them generate the key pair for me introduces a trust overhead: you trust their infrastructure, their memory management, and their cleanup process. The only way to be certain that no copy exists elsewhere is to generate it yourself and never share it outside your machine.

This is a principle I will try to follow throughout the project: don't trust defaults, and don't delegate security decisions to convenience.
