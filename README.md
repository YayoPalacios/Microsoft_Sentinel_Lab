# Lab - Microsoft Sentinel (SIEM) - Map with live cyberattacks

> This project was inspired by Josh Madakor’s [video](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=2722s) where he goes through setting up a SIEM in Azure with Microsoft Sentinel.
> 

## **What is security information and event management (SIEM)?**

A SIEM system is a tool that an organization uses to collect, analyze, and perform security operations on its computer systems. Those systems can be hardware appliances, applications, or both.

In its simplest form, a SIEM system enables you to:

- Collect and query logs.
- Do some form of correlation or anomaly detection.
- Create alerts and incidents based on your findings.

A SIEM system might offer functionality such as:

- **Log management**: The ability to collect, store, and query the log data from resources within your environment.
- **Alerting**: Takes a proactive look inside the log data for potential security incidents and anomalies.
- **Visualization**: Graphs and dashboards that provide visual insights into your log data.
- **Incident management:** The ability to create, update, assign, and investigate incidents that have been identified.
- **Querying data**: A rich query language, similar to that for log management, that you can use to filter and understand your data.

## **What is Microsoft Sentinel?**

Microsoft Sentinel is a cloud-native SIEM system that a security operations team can use to:

- Get security insights across the enterprise by collecting data from virtually any source.
- Detect and investigate threats quickly using built-in machine learning and Microsoft threat intelligence.
- Automate threat responses by using playbooks and by integrating Azure Logic Apps.

Unlike traditional SIEM solutions, to run Microsoft Sentinel, you do not need to install any servers either on-premises or in the cloud. Microsoft Sentinel is a service that you deploy in Azure. You can get up and running with Sentinel in just a few minutes in the Azure portal.

Microsoft Sentinel is tightly integrated with other cloud services. Not only can you quickly ingest logs, but you can also use other cloud services natively (for example, authorization and automation).

Microsoft Sentinel helps you enable end-to-end security operations. Including collection, detection, investigation, and response:

<p align="center">
<img src="https://imgur.com/Xa3VNpR.png" height="85%" width="85%" alt="MS Sentinel"/>
</p>

[Source: Microsoft Docs](https://docs.microsoft.com/en-us/learn/modules/intro-to-azure-sentinel/2-what-is-azure-sentinel)

## Project overview:

Here’s a brief description of our implementation’s components.

<p align="center">
<img src="https://imgur.com/RVUq7cw.png" height="75%" width="75%" alt="Project overview"/>
</p>

- First, we’ll create an Azure (free) subscription that will give you 200 USD worth of credits.

- We’ll set up a vulnerable VM in Azure which will essentially function as our “honeypot” by disabling our internal and external firewall rules.

## What is a Honeypot?

A honeypot is a security mechanism that creates a virtual trap to lure attackers. An intentionally compromised computer system allows attackers to exploit vulnerabilities so you can study them to improve your security policies. You can apply a honeypot to any computing resource from software and networks to file servers and routers.

<p align="center">
<img src="https://imgur.com/tLG4VcT.png" height="75%" width="75%" alt="Honeypot"/>
</p>

[Source](https://www.imperva.com/learn/application-security/honeypot-honeynet/)

- Next, we’ll create a log repository in Log Analytics Workspaces used to ingest our logs from the VM.

- We’ll use PowerShell to extract the IP addresses from our Windows logs and send them to a third-party API ([IP Geolocation](https://ipgeolocation.io/)) to determine their latitude, longitude, state, country, etc., and then send it back to our VM where we’ll use these details to create a custom log.

<p align="center">
<img src="https://imgur.com/X6NC5Lz.png" height="75%" width="75%" alt="PS"/>
</p>

<p align="center">
<img src="https://imgur.com/Xv0fDbu.png" height="75%" width="75%" alt="ipgeolocation"/>
</p>

## Quick note on IP Geolocation:

If you elect the Free tier option, you’re limited to 30,000 requests per month or 1000 a day. Though it may sound like a lot, it isn’t.

However, if you want to get the complete experience, I would recommend subscribing to the **Bronze tier** for this lab. This tier will ensure every one of our requests gets parsed and displayed on our map without any limitations.

You can take advantage of your subscription and go through the lab a couple of times to familiarize yourself with what’s going on behind the scenes.

<p align="center">
<img src="https://imgur.com/uiGGmW6.png" height="40%" width="40%" alt="Bronze tier"/>
</p>

- We’ll then setup Microsoft Sentinel where we’ll create a workbook to visualize the attackers’ geo-data and display it “live” on our map.

<p align="center">
<img src="https://imgur.com/H61gHB9.png" height="85%" width="85%" alt="Map"/>
</p>

- You can let it run for days and observe the activity on your VM/honeypot - as long as you keep an eye on your Azure credits, you should be fine.

<p align="center">
<img src="https://imgur.com/bmGziLy.png" height="85%" width="85%" alt="Map"/>
</p>

## When we're finished …

- Make sure to remove all your resources associated with this lab to avoid going over your free credits.

<p align="center">
<img src="https://imgur.com/MupdGU8.png" height="85%" width="85%" alt="Delete RG"/>
</p>

- Using the portal. we can click on **All Resources** or **Resource Groups** and start selecting and deleting the resources. Bear in mind this will take a few minutes to complete.

<p align="center">
<img src="https://imgur.com/xHwjixr.png" height="85%" width="85%" alt="Notification"/>
</p>

- You’ll get a confirmation on your Notifications tab once it’s done.

<p align="center">
<img src="https://imgur.com/sCA6WCu.png" height="85%" width="85%" alt="Notification"/>
</p>
