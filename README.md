This is a fork of Invoke-Parallel that has been tweaked to avoid hangs caused by timed out threads that cannot be closed by adding a -noCloseOnTimeout switch. When the switch is used and a thread times out, the calls to Thread.Dispose() and Runspace.Close() will be skipped. Those calls are synchronous and will never return if threads become non-responsive, as sometimes happens when WMI calls fail in a particular way.

As a side effect, memory consumption within the PowerShell host process will balloon because the resources from timed out threads and the runspace itself are never freed. This can be mitigated by using a PSJob to execute Invoke-Parallel:
```
  $scriptblock { <# your scriptblock here #> }
  $myjob = Start-Job -Scriptblock { 
    param($tasks, $scriptblock) 
    . .\PathTo\Invoke-Parallel.ps1
    $sb = [scriptblock]::Create($scriptblock) # $scriptblock gets passed as [String], convert back to [ScriptBlock]
    $tasks | Invoke-Parallel -ScriptBlock $sb -noCloseOnTimeout -quiet
  } -ArgumentList $tasks, $scriptblock
  $results = Wait-Job $myJob | Receive-Job
```
Note: Using a PSJob means the Input and Output objects will be serialized to cross the process boundries. This may have side-effects. Be aware of how your objects behave with serialization before using a PSJob.

Changes have been submitted upstream in https://github.com/RamblingCookieMonster/PowerShell/pull/1
