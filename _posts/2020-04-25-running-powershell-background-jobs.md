---
layout: post
title: Running Background Processes in Powershell
author: Andrew Kulpa
excerpt: In this article, the cmdlets we will go over are 'Start-Job', 'Get-Job', and 'Receive-Job' using a fairly simple example where we accumulate a count of all file types recursively in the current directory.
tags: [PowerShell]
---

Suppose we've been tasked to run a long-running process on a client machine. We have many other tasks we need to complete in their environment, but we know that this one will take what seems like *forever*. Although we can open another PowerShell console session, we miss out on any variables in the current session.

As you might have guessed by the title, the solution to this problem is to use PowerShell background jobs! In PowerShell, a **background job** is a command that runs in the background. A good way to see all the cmdlets that can be used for jobs is to run a `Get-Command` for the `Noun` `Job` like so:
```powershell
PS C:\Users\andrew.kulpa> Get-Command -Noun Job

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Debug-Job                                          3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Get-Job                                            3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Receive-Job                                        3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Remove-Job                                         3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Resume-Job                                         3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Start-Job                                          3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Stop-Job                                           3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Suspend-Job                                        3.0.0.0    Microsoft.PowerShell.Core
Cmdlet          Wait-Job                                           3.0.0.0    Microsoft.PowerShell.Core
```

In this article, the cmdlets we will go over are `Start-Job`, `Get-Job`, and `Receive-Job`.

## Starting Background Jobs

To start a background job, we use the `Start-Job` PowerShell cmdlet. A background job executes a given command outside the current session. The following is a fairly simple example where we accumulate a count of all file types recursively in the current directory:
```powershell
# You can also provide input using the $input variable and an -Input argument!
# This could then be used as the first argument instead of ".\".
PS C:\Users\andrew.kulpa\Documents> Start-Job -ScriptBlock {
>>   Get-Childitem ".\" -Recurse |
>>   where { -not $_.PSIsContainer } |
>>   group Extension -NoElement |
>>   sort count -Desc
>> }

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
7      Job7            BackgroundJob   Running       True            localhost            ...
```

As we saw, the output of the `Start-Job` cmdlet is an object that describes the background job. Most of these object parameters are quite useful. One thing we can note is that we should be able to set the `Name` of a job by invoking the cmdlet with a value passed to `-Name`.

## Checking and Receiving Jobs

In the last section we learned how to start a background job. Obviously with the job is running the background the job status could change at any time. We also want to get the output of the process! Let's do this by starting another job to try out passing a value to `-Name` and then explore multiple ways of using the `Get-Job` command.

First, let's run the same command with the value `fileCounts` passed into `-Name`:
```powershell
PS C:\Users\andrew.kulpa\Documents> Start-Job -Name fileCounts -ScriptBlock {
>>   Get-Childitem ".\" -Recurse |
>>   where { -not $_.PSIsContainer } |
>>   group Extension -NoElement |
>>   sort count -Desc
>> }

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
9      fileCounts      BackgroundJob   Running       True            localhost            ...
```

That wasn't much different, but we can clearly see that the value in `Name` is set to `fileCounts`! With that set, if we were to return to this machine we know what the background job was doing. Now lets try out the `Get-Job` cmdlet:
```powershell
PS C:\Users\andrew.kulpa\Documents> Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
7      Job7            BackgroundJob   Completed     False           localhost            ...
9      fileCounts      BackgroundJob   Completed     True            localhost            ...
```

It returns pretty much the same output as before, but it includes `Job7` and has a `State` of `Completed`. If we wanted to filter the results of `Get-Job` we can also filter by setting a value for either `-Id` or `-Name`:
```powershell
PS C:\Users\andrew.kulpa\Documents> Get-Job -Id 9

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
9      fileCounts      BackgroundJob   Completed     True            localhost            ...

PS C:\Users\andrew.kulpa\Documents> Get-Job -Name fileCounts

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
9      fileCounts      BackgroundJob   Completed     True            localhost            ...
```

That's great that we can now run a background job and check the status, but its likely we also want the output of the command that was ran. To do this, we'll use `Receive-Job` which has the same filters we used earlier. Let's run the cmdlet for the first job we ran:
```powershell
PS C:\Users\andrew.kulpa\Documents> Receive-Job -Id 7

Count Name
----- ----
    1 .txt
    1 .rtf
```

Awesome! Is that it? Maybe. Would running it again give us the same output? Lets try:
```powershell
PS C:\Users\andrew.kulpa\Documents> Receive-Job -Id 7
PS C:\Users\andrew.kulpa\Documents>
```

That's a big no. To keep the output of the job we need to add the `-Keep` parameter. Let's try that again with the `fileCounts` background job:
```powershell
PS C:\Users\andrew.kulpa\Documents> Receive-Job -Name fileCounts -Keep

Count Name
----- ----
    1 .txt
    1 .rtf


PS C:\Users\andrew.kulpa\Documents> Receive-Job -Name fileCounts -Keep

Count Name
----- ----
    1 .txt
    1 .rtf
```

Well there we go! We have successfully starting a background job, checked the status of the job, and received the output of the job. There are many cmdlets we have not gone over yet, so if you're interested in learning more check out the cmdlets we saw earlier and explore the [about_jobs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_jobs?view=powershell-7) documentation!
## Further Reading

Documentation:
  * [about_Jobs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_jobs?view=powershell-7)
  * [Start-Job](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/start-job?view=powershell-7)
  * [Get-Job](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-job?view=powershell-7)
  * [Receive-Job](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/receive-job?view=powershell-7)
