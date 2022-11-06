# Lab - Microsoft Sentinel (SIEM) - Map with live cyberattacks

> This project was inspired by Josh Madakor’s [video](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=2722s) where he goes through setting up a SIEM in Azure with Microsoft Sentinel.
> 

<br>

## [Here's my YouTube video ←](https://www.youtube.com/watch?v=lFmOtSKN6Jk)

<a href="https://www.youtube.com/watch?v=lFmOtSKN6Jk"><img src="https://imgur.com/mcMgofS.jpg" height="70%" width="70%"></a>

<br>

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

<br>

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

<br>

## Project overview:

Here’s a brief description of our implementation’s components.

<p align="center">
<img src="https://imgur.com/RVUq7cw.png" height="75%" width="75%" alt="Project overview"/>
</p>

<br>

- First, we’ll create an Azure (free) subscription that will give you 200 USD worth of credits.

<br>

- We’ll set up a vulnerable VM in Azure which will essentially function as our “honeypot” by disabling our internal and external firewall rules.

<br>

## What is a Honeypot?

A honeypot is a security mechanism that creates a virtual trap to lure attackers. An intentionally compromised computer system allows attackers to exploit vulnerabilities so you can study them to improve your security policies. You can apply a honeypot to any computing resource from software and networks to file servers and routers.

<p align="center">
<img src="https://imgur.com/tLG4VcT.png" height="75%" width="75%" alt="Honeypot"/>
</p>

[Source](https://www.imperva.com/learn/application-security/honeypot-honeynet/)

<br>

- Next, we’ll create a log repository in Log Analytics Workspaces used to ingest our logs from the VM.

<br>

- We’ll use PowerShell to extract the IP addresses from our Windows logs and send them to a third-party API ([IP Geolocation](https://ipgeolocation.io/)) to determine their latitude, longitude, state, country, etc., and then send it back to our VM where we’ll use these details to create a custom log.

<br>

### [PowerShell Script by /joshmadakor1 ←](https://github.com/YayoPalacios/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1)

```powershell
# Get API key from here: https://ipgeolocation.io/
$API_KEY      = "d4600b4efdef42b39828f5155041a457"
$LOGFILE_NAME = "failed_rdp.log"
$LOGFILE_PATH = "C:\ProgramData\$($LOGFILE_NAME)"

# This filter will be used to filter failed RDP events from Windows Event Viewer
$XMLFilter = @'
<QueryList> 
   <Query Id="0" Path="Security">
         <Select Path="Security">
              *[System[(EventID='4625')]]
          </Select>
    </Query>
</QueryList> 
'@

<#
    This function creates a bunch of sample log files that will be used to train the
    Extract feature in Log Analytics workspace. If you don't have enough log files to
    "train" it, it will fail to extract certain fields for some reason -_-.
    We can avoid including these fake records on our map by filtering out all logs with
    a destination host of "samplehost"
#>
Function write-Sample-Log() {
    "latitude:47.91542,longitude:-120.60306,destinationhost:samplehost,username:fakeuser,sourcehost:24.16.97.222,state:Washington,country:United States,label:United States - 24.16.97.222,timestamp:2021-10-26 03:28:29" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-22.90906,longitude:-47.06455,destinationhost:samplehost,username:lnwbaq,sourcehost:20.195.228.49,state:Sao Paulo,country:Brazil,label:Brazil - 20.195.228.49,timestamp:2021-10-26 05:46:20" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:52.37022,longitude:4.89517,destinationhost:samplehost,username:CSNYDER,sourcehost:89.248.165.74,state:North Holland,country:Netherlands,label:Netherlands - 89.248.165.74,timestamp:2021-10-26 06:12:56" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:40.71455,longitude:-74.00714,destinationhost:samplehost,username:ADMINISTRATOR,sourcehost:72.45.247.218,state:New York,country:United States,label:United States - 72.45.247.218,timestamp:2021-10-26 10:44:07" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:33.99762,longitude:-6.84737,destinationhost:samplehost,username:AZUREUSER,sourcehost:102.50.242.216,state:Rabat-Salé-Kénitra,country:Morocco,label:Morocco - 102.50.242.216,timestamp:2021-10-26 11:03:13" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-5.32558,longitude:100.28595,destinationhost:samplehost,username:Test,sourcehost:42.1.62.34,state:Penang,country:Malaysia,label:Malaysia - 42.1.62.34,timestamp:2021-10-26 11:04:45" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:41.05722,longitude:28.84926,destinationhost:samplehost,username:AZUREUSER,sourcehost:176.235.196.111,state:Istanbul,country:Turkey,label:Turkey - 176.235.196.111,timestamp:2021-10-26 11:50:47" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:55.87925,longitude:37.54691,destinationhost:samplehost,username:Test,sourcehost:87.251.67.98,state:null,country:Russia,label:Russia - 87.251.67.98,timestamp:2021-10-26 12:13:45" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:52.37018,longitude:4.87324,destinationhost:samplehost,username:AZUREUSER,sourcehost:20.86.161.127,state:North Holland,country:Netherlands,label:Netherlands - 20.86.161.127,timestamp:2021-10-26 12:33:46" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:17.49163,longitude:-88.18704,destinationhost:samplehost,username:Test,sourcehost:45.227.254.8,state:null,country:Belize,label:Belize - 45.227.254.8,timestamp:2021-10-26 13:13:25" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-55.88802,longitude:37.65136,destinationhost:samplehost,username:Test,sourcehost:94.232.47.130,state:Central Federal District,country:Russia,label:Russia - 94.232.47.130,timestamp:2021-10-26 14:25:33" | Out-File $LOGFILE_PATH -Append -Encoding utf8
}

# This block of code will create the log file if it doesn't already exist
if ((Test-Path $LOGFILE_PATH) -eq $false) {
    New-Item -ItemType File -Path $LOGFILE_PATH
    write-Sample-Log
}

# Infinite Loop that keeps checking the Event Viewer logs.
while ($true)
{
    
    Start-Sleep -Seconds 1
    # This retrieves events from Windows EVent Viewer based on the filter
    $events = Get-WinEvent -FilterXml $XMLFilter -ErrorAction SilentlyContinue
    if ($Error) {
        #Write-Host "No Failed Logons found. Re-run script when a login has failed."
    }

    # Step through each event collected, get geolocation
    #    for the IP Address, and add new events to the custom log
    foreach ($event in $events) {


        # $event.properties[19] is the source IP address of the failed logon
        # This if-statement will proceed if the IP address exists (>= 5 is arbitrary, just saying if it's not empty)
        if ($event.properties[19].Value.Length -ge 5) {

            # Pick out fields from the event. These will be inserted into our new custom log
            $timestamp = $event.TimeCreated
            $year = $event.TimeCreated.Year

            $month = $event.TimeCreated.Month
            if ("$($event.TimeCreated.Month)".Length -eq 1) {
                $month = "0$($event.TimeCreated.Month)"
            }

            $day = $event.TimeCreated.Day
            if ("$($event.TimeCreated.Day)".Length -eq 1) {
                $day = "0$($event.TimeCreated.Day)"
            }
            
            $hour = $event.TimeCreated.Hour
            if ("$($event.TimeCreated.Hour)".Length -eq 1) {
                $hour = "0$($event.TimeCreated.Hour)"
            }

            $minute = $event.TimeCreated.Minute
            if ("$($event.TimeCreated.Minute)".Length -eq 1) {
                $minute = "0$($event.TimeCreated.Minute)"
            }


            $second = $event.TimeCreated.Second
            if ("$($event.TimeCreated.Second)".Length -eq 1) {
                $second = "0$($event.TimeCreated.Second)"
            }

            $timestamp = "$($year)-$($month)-$($day) $($hour):$($minute):$($second)"
            $eventId = $event.Id
            $destinationHost = $event.MachineName# Workstation Name (Destination)
            $username = $event.properties[5].Value # Account Name (Attempted Logon)
            $sourceHost = $event.properties[11].Value # Workstation Name (Source)
            $sourceIp = $event.properties[19].Value # IP Address
        

            # Get the current contents of the Log file!
            $log_contents = Get-Content -Path $LOGFILE_PATH

            # Do not write to the log file if the log already exists.
            if (-Not ($log_contents -match "$($timestamp)") -or ($log_contents.Length -eq 0)) {
            
                # Announce the gathering of geolocation data and pause for a second as to not rate-limit the API
                #Write-Host "Getting Latitude and Longitude from IP Address and writing to log" -ForegroundColor Yellow -BackgroundColor Black
                Start-Sleep -Seconds 1

                # Make web request to the geolocation API
                # For more info: https://ipgeolocation.io/documentation/ip-geolocation-api.html
                $API_ENDPOINT = "https://api.ipgeolocation.io/ipgeo?apiKey=$($API_KEY)&ip=$($sourceIp)"
                $response = Invoke-WebRequest -UseBasicParsing -Uri $API_ENDPOINT

                # Pull Data from the API response, and store them in variables
                $responseData = $response.Content | ConvertFrom-Json
                $latitude = $responseData.latitude
                $longitude = $responseData.longitude
                $state_prov = $responseData.state_prov
                if ($state_prov -eq "") { $state_prov = "null" }
                $country = $responseData.country_name
                if ($country -eq "") {$country -eq "null"}

                # Write all gathered data to the custom log file. It will look something like this:
                #
                "latitude:$($latitude),longitude:$($longitude),destinationhost:$($destinationHost),username:$($username),sourcehost:$($sourceIp),state:$($state_prov), country:$($country),label:$($country) - $($sourceIp),timestamp:$($timestamp)" | Out-File $LOGFILE_PATH -Append -Encoding utf8

                Write-Host -BackgroundColor Black -ForegroundColor Magenta "latitude:$($latitude),longitude:$($longitude),destinationhost:$($destinationHost),username:$($username),sourcehost:$($sourceIp),state:$($state_prov),label:$($country) - $($sourceIp),timestamp:$($timestamp)"
            }
            else {
                # Entry already exists in custom log file. Do nothing, optionally, remove the # from the line below for output
                # Write-Host "Event already exists in the custom log. Skipping." -ForegroundColor Gray -BackgroundColor Black
            }
        }
    }
}

```

<br>

<p align="center">
<img src="https://imgur.com/X6NC5Lz.png" height="75%" width="75%" alt="PS"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/Xv0fDbu.png" height="75%" width="75%" alt="ipgeolocation"/>
</p>

<br>

## Quick note on IP Geolocation:

If you elect the **Free tier** option, you’re limited to **30,000** requests per month or **1000** a day. Though it may sound like a lot, it isn’t.

However, if you want to get the complete experience, I would recommend subscribing to the **Bronze tier** for this lab. This tier will ensure every one of our requests gets parsed and displayed on our map without any limitations.

You can take advantage of your subscription and go through the lab a couple of times to familiarize yourself with what’s going on behind the scenes.

<br>

<p align="center">
<img src="https://imgur.com/uiGGmW6.png" height="30%" width="30%" alt="Bronze tier"/>
</p>

<br>

- We’ll then setup Microsoft Sentinel where we’ll create a workbook to visualize the attackers’ geo-data and display it “live” on our map.

<br>

<p align="center">
<img src="https://imgur.com/H61gHB9.png" height="95%" width="95%" alt="Map"/>
</p>

<br>

- You can let it run for days and observe the activity on your VM/honeypot - as long as you keep an eye on your Azure credits, you should be fine.

<br>

<p align="center">
<img src="https://imgur.com/bmGziLy.png" height="95%" width="95%" alt="Map"/>
</p>

<br>

## When we're finished …

- Be sure to remove **all** your resources associated with this lab to avoid going over your free credits.

<br>

<p align="center">
<img src="https://imgur.com/MupdGU8.png" height="85%" width="85%" alt="Delete RG"/>
</p>

<br>

- Using the portal. we can click on **All Resources** or **Resource Groups** and start selecting and deleting the resources. Bear in mind this will take a few minutes to complete.

<br>

<p align="center">
<img src="https://imgur.com/xHwjixr.png" height="75%" width="75%" alt="Notification"/>
</p>

<br>

- You will get a confirmation on your Notifications tab once completed.

<br>

<p align="center">
<img src="https://imgur.com/sCA6WCu.png" height="75%" width="75%" alt="Notification"/>
</p>

<br>

## Further learning:
To learn more about these concepts, you can check out the following articles:

- [https://docs.microsoft.com/en-us/azure/sentinel/](https://docs.microsoft.com/en-us/azure/sentinel/)
- [https://docs.microsoft.com/en-us/azure/azure-monitor/logs/quick-create-workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/quick-create-workspace)
- [https://www.div0.sg/post/beginners-tale-honeypots](https://www.div0.sg/post/beginners-tale-honeypots)
- [https://www.varonis.com/blog/what-is-siem](https://www.varonis.com/blog/what-is-siem)
