# Lab - Microsoft Sentinel (SIEM) - Map with live cyberattacks

> This article was inspired by Josh Madakor’s [video](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=2722s) where he goes through setting up a SIEM in Azure with Microsoft Sentinel.
> 

## **What is security information and event management (SIEM)?**

A SIEM system is a tool that an organization uses to collect, analyze, and perform security operations on its computer systems. Those systems can be hardware appliances, applications, or both.

In its simplest form, a SIEM system enables you to:

- Collect and query logs.
- Do some form of correlation or anomaly detection.
- Create alerts and incidents based on your findings.

A SIEM system might offer functionality such as:

- **Log management**: The ability to collect, store, and query the log data from resources within your environment.
- **Alerting**: A proactive look inside the log data for potential security incidents and anomalies.
- **Visualization**: Graphs and dashboards that provide visual insights into your log data.
- **Incident management:** The ability to create, update, assign, and investigate incidents that have been identified.
- **Querying data**: A rich query language, similar to that for log management, that you can use to query and understand your data.

## **What is Microsoft Sentinel?**

Microsoft Sentinel is a cloud-native SIEM system that a security operations team can use to:

- Get security insights across the enterprise by collecting data from virtually any source.
- Detect and investigate threats quickly by using built-in machine learning and Microsoft threat intelligence.
- Automate threat responses by using playbooks and by integrating Azure Logic Apps.

Unlike with traditional SIEM solutions, to run Microsoft Sentinel, you don't need to install any servers either on-premises or in the cloud. Microsoft Sentinel is a service that you deploy in Azure. You can get up and running with Sentinel in just a few minutes in the Azure portal.

Microsoft Sentinel is tightly integrated with other cloud services. Not only can you quickly ingest logs, but you can also use other cloud services natively (for example, authorization and automation).

Microsoft Sentinel helps you enable end-to-end security operations including collection, detection, investigation, and response:

<p align="center">
<img src="https://imgur.com/a/wiS8DUR" height="85%" width="85%" alt="MS Sentinel"/>
</p>

[Source: Microsoft Docs](https://docs.microsoft.com/en-us/learn/modules/intro-to-azure-sentinel/2-what-is-azure-sentinel)

## Project overview:

Here’s a brief description of our implementation’s components.

![Azure Sentinel (SIEM).png](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Azure_Sentinel_(SIEM).png)

- First, we’ll create an Azure (free) subscription which will give you 200 USD worth of credits.

- We’ll setup a vulnerable VM in Azure which will essentially function as our “honeypot” by disabling our internal and external firewall rules.

## What is a Honeypot?

A honeypot is a security mechanism that creates a virtual trap to lure attackers. An intentionally compromised computer system allows attackers to exploit vulnerabilities so you can study them to improve your security policies. You can apply a honeypot to any computing resource from software and networks to file servers and routers.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%201.png)

[Source](https://www.imperva.com/learn/application-security/honeypot-honeynet/)

- Next, we’ll create a log repository in Log Analytics Workspaces which will be used to ingest our logs from the VM.

- We’ll use PowerShell to extract the IP addresses from our Windows logs and send it to a third party API ([ipgeolocation](https://ipgeolocation.io/)) to determine their latitude, longitude, state, country, etc. and then send it back to our VM where we’ll use these details to create a custom log.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%202.png)

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%203.png)

## Quick note on ipgeolocation:

If you opt for the **Free tier,** you’re limited to **30,000** requests per month or **1000** a day, while it may sound like a lot, it really isn’t. 

Now, if you want to get the full experience, I’d recommend subscribing to the **Bronze tier** for this lab, this will ensure every one of our requests gets parsed and displayed on our map without any limitations.

You can take advantage of your subscription and go through the lab a couple of times to really familiarize yourself with what’s going on behind the scenes.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%204.png)

- We’ll then setup Microsoft Sentinel where we’ll create a workbook to visualize the attackers’ geo-data and display it “live” on our map.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%205.png)

- You can let it run for days and observe the activity on your VM/honeypot, as long as you keep an eye on your Azure credits, you should be fine.

![Progress6.png](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Progress6.png)

## Now, when we’re done …

- Make sure to get rid of all your resources associated to this lab to avoid going over your free credits.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%206.png)

- To do this, we can always use the portal: click on **All Resources** or **Resource Groups** and start selecting and deleting the resources, bear in mind this will take a few minutes to complete.

![Untitled.png](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%207.png)

- You’ll get a confirmation on your Notifications tab once it’s done.

![Untitled](Lab%20-%20Microsoft%20Sentinel%20(SIEM)%20-%20Map%20with%20live%20cy%20a06ed32bcba0468989787a04b4233c5e/Untitled%208.png)
