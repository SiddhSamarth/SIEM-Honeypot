# SIEM-Honeypot

## Objective

The SIEM & Honeypot project aimed to Utilize Azure Sentinel (SIEM) to get Logs from Honeypot & Visualise Malicious Log-in attempts coming from different locations in a World map.

## Skills Learned

- Deploying a Honeypot virtual machine to attract and analyze attacks.
- Setting up Azure Sentinel (SIEM) to collect logs from the Honeypot for comprehensive threat monitoring.
- Utilizing KQL to write queries.
- Visualizing logs in Workbook (Map format) to discern the origins of attacks on the Honeypot.

## Tools Used

- Security Information and Event Management (SIEM) system for log ingestion and Visualization.
- Honeypot to attract malicious attackers.

## Steps

**1. To start, we will first log into Azure portal. As we discussed earlier First we will create honeypot to do that in Search tab above we will type "Virtual Machine" & create the virtual machine.
    Create -> Azure Virtual Machine -> customizations you want -> Create Username & Password for Admin account -> create network security group to allow all traffic -> Review & Create**

![Screenshot 2024-03-04 234052](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/1342bdaf-ce3e-4a35-8ead-049a6ec71670)
![Screenshot 2024-03-04 234951](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/bc4e9a09-08b9-4a82-90dd-a9a58ed26619)

**2. Now we will make a Log Analytic Workspace where the logs will saved. 
    Azure -> Search "Log Analytic" -> Create -> add resource group -> give name to log space -> review & create.**
   
![Screenshot 2024-03-04 235211](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/ee9f7075-63d0-4a09-8f65-2e9b19facd13)

**3. We will now enable gathering VM logs in Defender, head over to Microsoft Defender for Cloud -> Environment Setting -> Workspace -> ON Servers -> OFF SQL server -> Save
    Then go to Data collection -> select All Events -> Save**

![Screenshot 2024-03-05 001507](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/caa11ce0-ab97-4853-ba3c-732e3a1999ed)
![Screenshot 2024-03-05 001441](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/81aec98b-bf77-46da-a6f8-77f1237fcfda)

**4. Now we will go to Log space & connect our VM with Log space
    Virtual machines -> Honeypot -> connect**

![Screenshot 2024-03-05 001850](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/88d96a25-d124-4ff6-a74d-20b47bc5b349)

**5. Now we will create & setup Azure Sentinel (SIEM) to analyse & visualize the log data
    Portal -> search "Sentinel" -> select log space -> select resource group -> review & create**

![Screenshot 2024-03-05 002034](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/b286d821-f1ea-48d6-a25b-17e3ac299b5c)

**6. Now we will Disable the firewall in Honeypot to get attacked by attackers. We will first log into VM by coping Public IP address of VM & logging through RDP.
    Azure portal -> search "Virtual Machine" -> honeypot -> copy IP -> log in through RDP**

![Screenshot 2024-03-05 002503](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/cc4fe38e-8522-43c4-937c-0801c1efb47c)
![Screenshot 2024-03-05 002625](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/cfb2e48d-e35e-4273-b5de-943137024431)
![Screenshot 2024-03-05 003355](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/e58143d2-427f-406e-8ed7-9e38c9be17d3)
![Screenshot 2024-03-05 003517](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/c06b941d-481e-4e37-927f-ab407423e7ac)

**7. Now we will get geolocation of IP address of attacker from geolocation.io then we will use Powershell script to normalize (format) in a Log format.
    --Event ID 4625 - failed logon attempts--**
 
![Screenshot 2024-03-05 004203](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/2a8dc283-7bed-4f17-a88d-3c24e384e3c0)
![Screenshot 2024-03-05 004906](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/88887bfb-689a-4595-904b-378edfc5d8f3)

**8. As we can see the custom log data we created with longitude & latitude is ready**

![Screenshot 2024-03-05 005801](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/ef598bc9-ecd2-4e8c-80b4-2b7501a7f709)


**9. Now we will create custom log in Log Analytic Workspace to get the custom log we created with longitude & latitude
    Portal -> Log space -> Settings -> Tables -> Create -> MMA based -> add sample file (which we created) -> select collection path -> Give name -> Create**

![Screenshot 2024-03-05 011352](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/bb3f7616-c12d-4d44-b312-8e45f4ce5688)


**10. Now we will extract fields from RAW data log to use for Map visualization
      Portal -> log analytics -> Logs -> query "Custom log name" -> select "RawData" field -> Extract fields**

![Screenshot 2024-03-05 044726](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/0e703048-f78d-4c62-8da9-1991f0125664)


**11. Select numeric data of latitude & give it field name & type, also correct the sample field if needed**

![Screenshot 2024-03-05 044921](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/8674a31b-0a7c-4b4f-9b17-5311d2fc9248)
![Screenshot 2024-03-05 045038](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/e87e7f84-1c0b-4545-83ed-e0175e31b05d)

 
**12. Get all the fields done. & check if the new fields are showing in Logs now**
 
![Screenshot 2024-03-05 045412](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/3d33742a-3eb2-4105-ae48-ac11b5c167c3)


**13. Now we will visualize this logs into map to get an idea from where we are getting attacked.
	    Portal -> sentinel -> workbook -> edit -> remove all widgets -> add query**
	
![Screenshot 2024-03-05 045852](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/cee99952-8d2c-44ad-a81c-2f2793867179)


**we will include all fields required & exclude sample logs also null fields. Then we will visualize it in map**

![Screenshot 2024-03-05 051215](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/ed3cdc1c-4f3b-411c-8d90-51a0993ab257)


**14. Finally we will save the map, select done editing to save all progress. Now we will wait to get attacked by attackers by which point we can visualize the attacks from all over the world.**

![Screenshot 2024-03-05 051113](https://github.com/laaaaaarry/SIEM-Honeypot/assets/125237930/a2626270-26f4-49a8-9fdf-03628973e62f)


## Conclusion : 
- In this lab, we learned to setup a honeypot to attract attackers.
- To configure Azure Sentinel to collect & visualize Logs in Map format.
- We encountered around 20000+ logon attempts in this lab.
