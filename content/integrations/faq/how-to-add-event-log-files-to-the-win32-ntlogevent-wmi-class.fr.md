---
title: Comment ajouter des logs à la classe WMI `Win32_NTLogEvent`
kind: faq
---

Tous les logs ne se trouvent pas dans la classe Win32_NTLogEvent WMI. Étant donné que l'intégration de Event Viewer ne peut prendre en compte que les événements de cette classe, modifiez le Registre Windows pour ajouter des logs en dehors de la portée de cette classe.

The first step is to confirm whether or not the logfile can be accessed through the Win32_NTLogEvent using the following WMI query in Powershell. (This is the same query the Agent runs to collect these events)
```
$ Get-WmiObject -Query "Select EventCode,SourceName,TimeGenerated,Type,InsertionStrings,Message,Logfile from Win32_NTLogEvent WHERE ( LogFile = '<LogFileName>' )" | select -First 1
```

If there are no results, the log file cannot be accessed and you need to add it through the Windows Registry.

Locate the event logs you want to monitor in the Event Viewer. Locate the log file and click “properties” under the “Actions” section to find the Log path and Full Name. For example, here is how to set up monitoring the “Operational” event Log file located in the Microsoft/Windows/TaskScheduler folder:
{{< img src="integrations/faq/image1.png" alt="image1" responsive="true" popup="true">}}

Ouvrez le registre Windows. (recherchez regedit.exe, le nom par défaut de l'éditeur de registre). Dans l'éditeur de registre, recherchez le dossier EventLog à partir du chemin suivant:
```
\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\
```

Create a new key with the name of the event log you’re wanting to monitor. Using the syntax of path-to-folder/LogFileName (i.e. the Full Name found in the Event Viewer)
{{< img src="integrations/faq/image2.png" alt="image2" responsive="true" popup="true">}}

{{< img src="integrations/faq/image3.png" alt="image3" responsive="true" popup="true">}}

Next, you’ll need to add three values to this key. First, add the path to the log file as a String Value (REG_SZ) named “File”:
{{< img src="integrations/faq/image4.png" alt="image4" responsive="true" popup="true">}}

Next, add the Full Name of the Log file as a String Value (REG_SZ) named “Primary Module”:
{{< img src="integrations/faq/image5.png" alt="image5" responsive="true" popup="true">}}

Finally, add the path to the Windows Event Log Api DLL (wevtapi.dll), which should be at `%SystemRoot%\system32\wevtapi.dll` as an Expandable String Value with the name “DisplayNameFile”:
{{< img src="integrations/faq/image6.png" alt="image6" responsive="true" popup="true">}}

The changes should be immediate. To confirm that the event log is accesible through the Win32_NTLogEvent WMI class, try the above query again. Then you can resume [adding events to the Event Viewer integration][1] config file

Note: if there still aren’t events when running the query, check the event viewer to confirm that there are any events in the log file. Also, make sure that the event log isn’t disabled and that there are recent events available.
{{< img src="integrations/faq/image8.png" alt="image8" responsive="true" popup="true">}}

[1]: /integrations/faq/how-to-monitor-events-from-the-windows-event-logs