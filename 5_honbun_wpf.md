# WPF

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
+    <UseWPF>true</UseWPF>
  </PropertyGroup>

</Project>
```

## System.Windows 名前空間

### Application クラス

```csharp
#pragma warning disable WPF0001

using System.Windows;

var app = new App();
app.Run();

class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        Console.WriteLine(Current.ThemeMode);
        Shutdown();
    }
}
```

### SystemColors クラス

```csharp
using System.Windows;

Console.WriteLine(SystemColors.AccentColor);
```

