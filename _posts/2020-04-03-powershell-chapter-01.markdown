---
layout: post
title:  "PowerShell : Chapter 01"
date:   2020-04-03 23:20:58 +0530
categories : powershell
tags: powershell
---

## Important Commands
{% highlight console %}
PS C:\> Get-Help *process

Name                              Category  Synopsis
----                              --------  --------
Get-Process                       Cmdlet    Gets the processes that are running on the local computer or a remote co...
Stop-Process                      Cmdlet    Stops one or more running processes.
Wait-Process                      Cmdlet    Waits for the processes to be stopped before accepting more input.
Debug-Process                     Cmdlet    Debugs one or more processes running on the local computer.
Start-Process                     Cmdlet    Starts one or more processes on the local computer.
{% endhighlight %}

If we use cmdlet `Get-Process`, it lists out the running processes, however we can achieve the same result by using `ps` command. This is achieved by something called as `alias`. Powershell defines a number of `alias` for the comdlets. The list of corresponding `alias` and cmdlet can be viewed as follows :
 
{% highlight console %}
PS C:\> Get-Alias

CommandType     Name                                                Definition
-----------     ----                                                ----------
Alias           %                                                   ForEach-Object
Alias           ?                                                   Where-Object
Alias           ac                                                  Add-Content
Alias           asnp                                                Add-PSSnapIn
Alias           cat                                                 Get-Content
Alias           cd                                                  Set-Location
Alias           ps                                                  Get-Process
{% endhighlight %}

`Get-Command` cmdlet can be used to get basic information about cmdlets. In order to check what all parameters the `Get-Command` can check by 
{% highlight console %}
PS C:\> Get-Help Get-Command -Parameter * | more
{% endhighlight %}

If we want to see all the *cmdlets* with *services* mentioned in their *names*, then we can check by
{% highlight console %}
PS C:\> Get-Command -CommandType cmdlet -Name *service*

CommandType     Name
-----------     ----
Cmdlet          Get-Service
Cmdlet          New-Service
Cmdlet          New-WebServiceProxy
Cmdlet          Restart-Service
Cmdlet          Resume-Service
Cmdlet          Set-Service
Cmdlet          Start-Service
Cmdlet          Stop-Service
Cmdlet          Suspend-Service
{% endhighlight %}

`Verb` parameter can also be used with `Get-Command` fo look for cmdlets having specific verbs.
{% highlight console %}
 
PS C:\> Get-Command -Verb start

CommandType     Name
-----------     ----
Cmdlet          Start-Job
Cmdlet          Start-Process
Cmdlet          Start-Service
Cmdlet          Start-Sleep
Cmdlet          Start-Transaction
Cmdlet          Start-Transcript


PS C:\> Get-Help Start-Process -Examples | more
{% endhighlight %}

Now, we can use `Start-Process` cmdlet to start a `notepad.exe`.

{% highlight console %}
PS C:\> Start-Process notepad.exe
{% endhighlight %}

We can use `Stop-Process` cmdlet to stop all the running processes

{% highlight console %}
PS C:\> Get-Process notepad

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id ProcessName
-------  ------    -----      ----- -----   ------     -- -----------
     62       7     1400       5640    71     0.13    560 notepad
     62       7     1452       5760    71     0.92    984 notepad
     60       7     1392       5500    75     0.08   1364 notepad


PS C:\> Get-Process notepad | Stop-Process
PS C:\> Get-Process notepad
Get-Process : Cannot find a process with the name "notepad". Verify the process name and call the cmdlet again.
At line:1 char:12
+ Get-Process <<<<  notepad
    + CategoryInfo          : ObjectNotFound: (notepad:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
{% endhighlight %}
