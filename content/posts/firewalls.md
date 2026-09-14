---
title: "Two Firewalls Are Better Than One"
date: 2026-03-22
summary: "Defense in depth applied to network access: OCI Security Lists and firewalld."
tags: [networking, security, firewall]
---

## The Principle

Defense in Depth is a security principle based on the idea of layering independent security measures so one failure doesn't cascade into a breach. This can be applied to my Oracle instance using firewalls: one at the infrastructure level, another on the server itself.

## Layer 1: The Cloud Firewall

Oracle Cloud's Security Lists provides a means to filter out unwanted traffic before it reaches the VM. This is enforced at the infrastructure level through the OCI console. Only needed ports are allowed: SSH (on a custom port), HTTP, and HTTPS.

The advantage here is that even if I misconfigured `firewalld` on the server, the cloud layer still blocks unauthorized traffic. Because the two layers live in separate control planes, an attacker with root on the VM can enumerate and change `firewalld`, but has no visibility into the OCI rules. Compromising the host doesn't expose or weaken the outer layer.

## Layer 2: firewalld

As mentioned, `firewalld` is the default firewall management daemon on RHEL-based distributions. To control traffic, I used service names instead of listing raw port numbers: `firewall-cmd --permanent --add-service=https`. I also removed the `cockpit` service enabled by default on Oracle Linux, it is a web-based control panel on port 9090 used by system administrators, not my use case for the moment.  

## Why Both?

Consider these failure scenarios:

If I accidentally open a port in `firewalld`, the OCI Security List still blocks it externally. The mistake has no impact.

If I misconfigure the OCI Security List too broadly, `firewalld` still enforces restrictions at the OS level. Again, no breach.

Both would need to fail simultaneously for an unauthorized port to be reachable. That's significantly less likely than either failing alone.

## The Traffic Path

Every incoming request passes through both layers in sequence:

Internet -> OCI Security List -> firewalld -> nginx -> Application

Each layer independently decides whether to allow or drop the connection. This is defense in depth in practice: independent configuration and compound failure required for compromise.

