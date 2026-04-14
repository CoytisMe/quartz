---
category: script
tags:
  - cmd
  - battery
  - power
  - report
---
~~~
powercfg /batteryreport /output C:\BatteryReport.html
~~~

~~~
powercfg /sleepstudy /duration 7 /output C:\sleepstudy-report.html
~~~



 
## Battery Report
Gives a full breakdown of recent active / sleep / screen off time alongside battery levels and estimated battery life at each moment. 

**`powercfg /batteryreport**
- Will create the report and throw it into System32

`/output <filepath>.<filename>.html`
- Lets you choose where it goes
	**You must put the file name ending in ‘.html’**
### Example
```cmd
powercfg /batteryreport /output C:\BatteryReport.html
```

## Sleep Study
Creates a System Power Report that provides much more detail on sleep states and battery drain. Shows a breakdown of each sleep and standby along with which apps interrupted them. 

It will also identify ‘Top Offenders’ being that apps that are causing the most drain on the battery during sleep and standby.

**`powercfg /batteryreport**
- Will create the report and throw it into System32

**`/output <filepath>.<filename>.html`**
- Lets you choose where it goes
	**You must put the file name ending in ‘.html’**

**`/duration <Number of Days>`**
- If you want it to go back further, maximum 28 days.
### Example
```cmd
powercfg /sleepstudy /duration 7 /output C:\sleepstudy-report.html
```