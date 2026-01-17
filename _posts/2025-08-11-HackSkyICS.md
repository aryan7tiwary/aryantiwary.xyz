---
toc: true
classes: wide
title: "HackSkyICS – AI-Powered ICS Cybersecurity Attack Simulation & Anomaly Detection"
date: 2025-08-10T12:20:30-05:30
categories:
  - blog
tags:
  - Industrial Control System
  - Information Technology
  - Operational Technology
  - Anomaly Detection
  - SCADA
  - Autonomous Defense
---
>When we joined the Kaspersky Industrial Cybersecurity Hackathon, we wanted to show that AI can spot industrial cyberattacks as they happen — even subtle ones that humans might miss — using realistic, hands-on simulations.

---

# 1. The Idea & Vision
Industrial Control Systems (ICS) run the world quietly in the background — they control our power grids, water treatment plants, manufacturing lines, and more. But these systems weren’t built with cybersecurity in mind. In today’s world, that’s a dangerous gap.

When we joined the Kaspersky Industrial Cybersecurity Hackathon, we wanted to show that AI can spot industrial cyberattacks as they happen — even subtle ones that humans might miss — using realistic, hands-on simulations.

That’s how HackSkyICS came to life. Our main goal:
* Simulate a real ICS network using virtualization.
* Launch actual ICS-specific cyberattacks on it.
* Detect anomalies in real-time using machine learning and an LLM.

The focus was purely on detection, not on stopping or reversing the attack. And despite that scope, we ended up as one of the Top 15 teams in the hackathon.

# 2. Quick Primer – ICS, IT, and OT
If you come from an IT-only background, here’s a quick context:
* IT (Information Technology) → Your usual servers, databases, web apps, and emails. Downtime is bad, but mostly an inconvenience.
* OT (Operational Technology) → The tech that makes the physical world work — turbines, pumps, PLCs, conveyor belts. Downtime here can cause massive damage or even risk lives.
* ICS (Industrial Control Systems) sits inside OT. It’s a combination of hardware, software, and networks that control industrial processes — typically via PLCs and SCADA systems.

The tricky part: many ICS protocols like Modbus and DNP3 have zero built-in security. They trust anything that talks to them, which is why attacks like [Stuxnet](https://en.wikipedia.org/wiki/Stuxnet) were possible.

![security-event](/assets/images/HackSky/security-events-management.png)

# 3. Building HackSkyICS – The Core Architecture
We went heavy on virtualization to create a controlled yet realistic environment. The setup:
* Attack VM – Kali Linux with Metasploit, Modbus exploitation scripts, and network scanning tools.
* ICS VM – Ubuntu Server running OpenPLC + ModbusPal, simulating a water treatment plant.
* Monitoring VM – Python + Node.js environment running our ML models and anomaly detection logic.
* Display VM (HMI) – React-based SCADA dashboard showing live sensor readings and system status.

All of them were connected to a virtual network (192.168.100.0/24) to mimic a real plant setup.
We used OpenPLC for its open-source flexibility and native Modbus support, and ModbusPal for generating realistic sensor values.

![Architecture](/assets/images/HackSky/architecture.png)

# 4. The Attack Simulation
One of our main demo scenarios involved a parameter manipulation attack.

From the Kali attack VM, we connected to the Modbus registers controlling voltage in the water treatment system. Then we subtly increased the value — not enough to trigger obvious alarms, but enough to cause long-term damage in a real system.

![Attack-Interface](/assets/images/HackSky/attack.png)

From the operator’s SCADA dashboard, everything looked “mostly fine.” No flashing alerts, just slightly off readings. This is exactly the kind of scenario where anomaly detection shines.

![network-protocol-distribution](/assets/images/HackSky/network-protocol-distribution.png)

# 5. Our AI/ML Detection Model
We built HackSkyICS to use two key ML approaches:
* Isolation Forest (scikit-learn) – An unsupervised method that detects data points that don’t fit normal patterns. Perfect for catching unknown or zero-day attacks.
* Autoencoder (PyTorch) – A deep learning model trained on normal sensor patterns. If it struggles to “rebuild” the incoming data, that’s a strong anomaly signal.

![Detection](/assets/images/HackSky/detection.png)

Why both?
* ICS attacks can be subtle drifts in data (great for Isolation Forest).
* Or they can be completely new patterns (Autoencoder catches these).

We trained using both clean operational data and simulated attack data. The result:

    Accuracy: 92.42%

    Detection time: <500ms

    Alerts in real-time to the operator dashboard.

![AI-Anomaly-Detection](/assets/images/HackSky/AI-Anomaly-Detection.png)

# 6. The LLM Integration
On top of ML detection, we added a fine-tuned LLM trained on ICS/Modbus logs.

The idea was simple: operators shouldn’t have to read endless log entries. They should be able to just ask:

    “Is something wrong with the voltage in Line 3?”

The LLM could check logs, correlate ML alerts, and respond in plain language:

    “Voltage anomaly detected. Register 40005 shows deviation from baseline by +12%.”

In our voltage manipulation attack, the LLM caught it and reported it clearly.

![admin-control-panel](/assets/images/HackSky/admin-control-panel.png)

# 7. Challenges We Faced
* VM Networking – Making four VMs communicate smoothly on the same subnet took a lot of trial and error.
* Real-time Performance – ML inference had to be fast enough to not delay dashboard updates.
* Data Availability – Industrial attack datasets aren’t public, so we had to create and label our own.
* Resource Limits – Running this all on a laptop while keeping CPU load low was tricky.

# 8. Why HackSkyICS is Different
A lot of ICS security demos either:

    Show the attack, or

    Show the detection model.

We combined both into one integrated platform where:

    - The attack is real (via Kali + Modbus manipulation).
    - The detection is AI-driven.
    - The data is visualized instantly on a SCADA-like interface.

It’s a realistic training ground for anyone learning OT cybersecurity.

# 9. Takeaways from the Kaspersky Hackathon
What made us stand out was that HackSkyICS wasn’t just a model — it was a complete working environment where people could see an attack happen and then watch the system detect it in real-time.

That live element impressed the judges, and it’s what helped us land in the Top 15.

# 10. The Bigger Picture
ICS attacks are getting more common, and they’re dangerous because they target the physical world.

HackSkyICS is a proof-of-concept that shows:
* How AI can catch anomalies that humans or simple rules might miss.
* How realistic simulation environments can be built without needing physical ICS hardware.
* How cyber ranges for OT can be designed for training purposes.

For the future, adding prevention or automated response is possible — but for now, HackSkyICS is about seeing the threat as it happens and learning how to respond.