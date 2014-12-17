This is a fork of Invoke-Parallel that has been tweaked with additional parameters and functionality.

-noCloseOnTimeout

	Using this switch will cause timed out Threads to not be disposed and the Runspace not to be closed if any timeouts occurred. Calls to Thread.Dispose() and Runspace.Close() are synchronous and will never return if there is a non-responsive thread. WMI calls that fail in a particular way are known to cause the problem that this switch corrects.

	As a side effect, memory consumption within the PowerShell host process will balloon because the resources from timed out threads and the runspace itself are never freed. This can be mitigated by using a PSJob to execute Invoke-Parallel:
	```
	  $scriptblock = { <# your scriptblock here #> }
	  $tasks = @{} <# your input objects here #>
	  $myjob = Start-Job -Scriptblock { 
		param($tasks, $scriptblock) 
		. .\PathTo\Invoke-Parallel.ps1
		# $scriptblock gets passed into PSJob as [String], convert back to [ScriptBlock]
		$sb = [scriptblock]::Create($scriptblock) 
		$tasks | Invoke-Parallel -ScriptBlock $sb -noCloseOnTimeout -quiet
	  } -ArgumentList $tasks, $scriptblock
	  $results = Wait-Job $myJob | Receive-Job
	```
	Note: Using a PSJob means the Input and Output objects will be serialized to cross the process boundries. This may have side-effects. Be aware of how your objects behave with serialization before using a PSJob.

-quiet 

	Suppresses the calls to Write-Progress.

-PassThru

	Pass the original object from timed out tasks thru the pipeline. Use this if your Input and Output objects are of the same type and you need your results to contain all objects, including the ones that timed out.

Changes have been submitted upstream in https://github.com/RamblingCookieMonster/PowerShell/pull/1
