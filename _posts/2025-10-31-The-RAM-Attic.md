---
title: "/dev/shm: The RAM Attic"
date: 2025-10-31T02:41:00+05:30
categories:
  - blog
tags:
  - linux
  - tmpfs
  - ipc
  - security
  - forensics
  - dev_shm
---

>There’s a small place on every Linux machine I’ve come to respect: `/dev/shm`. I first noticed it in one of [Ippsec's video](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA) during my regular bing watching.

---

> In this post I’ll walk through what `/dev/shm` is, why it matters from both a developer and defender perspective.

---


## What `/dev/shm` really is

`/dev/shm` is a **temporary filesystem (tmpfs)** mounted in RAM.  
Anything stored here lives entirely in memory, not on disk which means it’s:

- Extremely fast  
- Automatically cleared on reboot or unmount  
- Accessible to all users (by default `mode=1777`, like `/tmp`)  
- Often ignored in traditional monitoring setups

We can verify this ourself:

```bash
$ mount | grep shm
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)

$ df -h /dev/shm
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           7.8G   64K  7.8G   1% /dev/shm
```

Now let’s see why attackers (and defenders) both care so much about this innocent-looking path.

---

## 1) Why attackers use `/dev/shm` after they get a foothold

`/dev/shm` keeps appearing in post-exploitation because it checks every box for stealth and convenience.

- **Speed & low latency** — it’s RAM-backed (`tmpfs`), so binaries and data staged here execute faster than from disk. Great for quick, in-memory payloads.  
- **Ephemerality** — everything disappears on reboot. That’s perfect for short-lived tools or avoiding forensic traces.  
- **Low-privilege writeability** — default permissions (`1777`) let any user drop files here.  
- **Under-monitored** — most defenders forget to watch `/dev/shm`. Logs, FIM, and EDR agents often skip it.  
- **Unlink-and-run** — processes can execute a file from `/dev/shm`, then delete it. The binary vanishes from view while still running.  
- **Containers** — Docker and other containers have private `/dev/shm` mounts, making it a convenient hiding spot in containerized environments.

In short: `/dev/shm` is fast, volatile, and often invisible which makes it a perfect recipe for post-exploitation staging.

---

## 2) Why defenders need to take `/dev/shm` seriously

Anyone can ignore `/dev/shm`. It’s small, temporary, and seems harmless.  
But once I started digging, I realized how powerful it can be for attackers.

Here’s what defenders should note:

- **It’s writable by everyone** — any user, including compromised services, can drop files here.  
- **It bypasses disk-based detection** — file scanners, backup systems, and forensic tools often overlook memory-mounted paths.  
- **It enables persistence without persistence** — scripts or payloads can live only in memory and still cause serious impact.  
- **It can host sockets or shared memory segments** — not just files. Attackers can use these to communicate between processes stealthily.

For defenders, `/dev/shm` should be treated as a high-risk, high-speed scratchpad it needs to be monitored carefully.

---

## 3) How to make `/dev/shm` more secure

Completely disabling `/dev/shm` isn’t realistic — many system processes and apps depend on it.  
But we can harden and monitor it effectively.

Here’s how:

### **1. Restrict execution**

Add the `noexec` flag to the `/dev/shm` mount in `/etc/fstab`:

```
tmpfs /dev/shm tmpfs defaults,noexec,nosuid,nodev 0 0
```

Then remount:

```bash
sudo mount -o remount /dev/shm
```

This prevents binaries from being executed directly out of `/dev/shm`.

---

### **2. Tighten permissions**

Default permissions are `1777` (world-writable). We can reduce that surface if our workloads don’t require full public access:

```bash
sudo chmod 0700 /dev/shm
```

Some applications may rely on broader access.

---

### **3. Monitor and alert**

Use simple bash auditing or tools like `auditd`, `osquery`, or the SIEM agent:

```bash
auditctl -w /dev/shm -p war -k shm-monitor
```

This logs every write, access, and removal event in `/dev/shm`.

Or just check manually:

```bash
ls -alh /dev/shm
```

---

### **4. In containers**

For Docker containers, we can limit or disable shared memory mounts:

```bash
docker run --shm-size=1m --tmpfs /dev/shm:rw,noexec,nosuid,nodev mycontainer
```

---

## Final thoughts

`/dev/shm` is one of those Linux components that sits quietly until we realize how powerful it is.  
Attackers love it because it’s **fast, ephemeral, and overlooked**.  
Defenders should love it because **understanding it helps close a stealthy gap**.

In short: if we don’t watch `/dev/shm`, someone else eventually will.

---