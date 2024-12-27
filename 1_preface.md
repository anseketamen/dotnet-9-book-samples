# まえがき

```csharp
using System.Diagnostics;

var processorName = Process.Start(new ProcessStartInfo()
{
    FileName = "powershell",
    Arguments = "(Get-ComputerInfo).CsProcessors.Name",
    RedirectStandardOutput = true,
})!.StandardOutput.ReadToEnd().Trim();

Console.WriteLine($"OS: {Environment.OSVersion}");
Console.WriteLine($"Processor: {processorName}");
Console.WriteLine($"Runtime Version: {Environment.Version}");
```
