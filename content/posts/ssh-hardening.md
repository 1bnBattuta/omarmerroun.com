---
title: "Hardening SSH"
date: 2026-03-18
summary: "Within minutes of going live, bots were already knocking. Here's how I locked them out."
tags: [linux, security, ssh]
---

## Wide Open!

Within minutes of the server going live, the auth logs started filling up. Numerous failed login attempts contained dozens of default usernames including root, admin, and test. These are automated scanners that sweep entire IP ranges looking for low-hanging fruit at every moment. What a hostile environment!

Securing SSH was thus on top of my priorities. I followed some good practices I read about concerning this topic, such as validating with `sshd -t` after every change and before restarting the service, and testing the connection from another terminal while keeping the existing session alive as a rollback path.

One critical principle to be followed throughout this section: Least Privileges. Meaning, privileges are to be set in a way to ensure desired functionality and only desired functionality; all additional privileges should be omitted.

## Disable Root Login

`PermitRootLogin no`
Normal system administration goes through a privileged user via sudo; root user should not be used under any circumstances to log into the system via SSH. The default configuration here is prohibit-password in many systems. However, keeping it that way serves no purpose and could introduce an attack vector if the threat actor manages to set SSH allowed keys for the root user. Other than that, automated scanners tend to favor root username over other credentials.

## Allowed Users

`AllowUsers <permitted_users>`
This is a whitelist, not a blacklist. It makes sense to allow only a selected set of users to connect to our machine via SSH. Imagine that an attacker has created a privileged user inside our machine through some vulnerability; they still can't establish an SSH session unless the user appears on this whitelist. In other words, uninvited guests can't just walk in even if they have keys.

## Disable Password Authentication

`PasswordAuthentication no`
Passwords are guessable, keys are not. As long as password login is enabled, brute-force attempts remain viable; disabling it closes that door entirely. From this point on, the only way in is with the private key that lives on my machine. This pairs naturally with the `AllowUsers` whitelist above: even if someone somehow ended up on the list, they still couldn't authenticate without the key.

## Move the Port

SSH service uses port 22. Many system administrators suggest moving SSH off the standard port to a custom one; others argue that a security-via-obscurity approach like this doesn't provide real security because a full port scan finds the new port in minutes. For my use case, I decided to move it because of the additional benefit of cutting some log noise from lazy scanners, making actual security events easier to spot in the logs.

## Idle Timeout

`ClientAliveInterval` and `ClientAliveCountMax`
Inactive sessions disconnect after a few minutes of silence. The server sends keepalive checks regularly; numerous unanswered checks inform the server to terminate the session. This reduces the window of session hijacking on unattended terminals.

## Order of Operations

Changing the port needs careful checking in order to prevent being locked out of the machine. First, I added the custom port to the firewall, then confirmed the connection worked on the new port from another terminal. Finally, I blocked traffic from the old port.

This can easily get you locked out if you removed the old port when the new one isn't open yet; you won't notice it until you close the current session and try to reconnect without success. If that ever happened, a system recovery needs to be done using the provided console in OCI.
 
