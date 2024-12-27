# Windows Forms

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
+    <UseWindowsForms>true</UseWindowsForms>
  </PropertyGroup>

</Project>
```


## System.Drawing 名前空間

### Bitmap クラス

```csharp
using System.Drawing.Imaging;
using System.Drawing.Imaging.Effects;

var bitmap = new Bitmap("cactus_3d.png");
bitmap.ApplyEffect(new BlurEffect(20, false));
bitmap.Save("cactus_3d_blur.png", ImageFormat.Png);
```

```csharp
using System.Drawing.Imaging;

var bitmap = new Bitmap("cactus_3d.png");
var palette = new ColorPalette(PaletteType.FixedBlackAndWhite);
bitmap.ConvertFormat(PixelFormat.Format1bppIndexed, DitherType.Solid, PaletteType.FixedBlackAndWhite, palette);
bitmap.Save("cactus_3d_bw.png");
```

## System.Windows.Forms 名前空間

### Application クラス

```csharp
#pragma warning disable WFO5001

Console.WriteLine(Application.IsDarkModeEnabled);
Console.WriteLine(Application.SystemColorMode);
Console.WriteLine(Application.ColorMode);
```

```csharp
#pragma warning disable WFO5001

Application.SetColorMode(SystemColorMode.Dark);

Application.EnableVisualStyles();
Application.Run(new FormMain());

public class FormMain : Form
{
    public FormMain()
    {
        Text = "Test";
        Size = new Size(300, 150);

        var button = new Button()
        {
            Text = "Test",
            Top = 10,
            Left = 10,
        };
        Controls.Add(button);
    }
}
```


### FolderBrowserDialog クラス

```csharp
class Program
{
    [STAThread]
    static void Main(string[] args)
    {
        var dialog = new FolderBrowserDialog()
        {
            Multiselect = true
        };

        if (dialog.ShowDialog() == DialogResult.OK)
        {
            foreach (var path in dialog.SelectedPaths)
            {
                Console.WriteLine(path);
            }
        }
    }
}
```


### Form クラス

```csharp
#pragma warning disable WFO5002

class Program
{
    [STAThread]
    static void Main(string[] args)
    {
        Application.EnableVisualStyles();
        Application.Run(new FormMain());
    }
}

class FormMain : Form
{
    public FormMain()
    {
        Size = new Size(300, 150);

        var button = new Button()
        {
            Text = "Open Sub Form",
            Location = new Point(10, 10),
            Size = new Size(100, 30)
        };
        button.Click += async (sender, e) =>
        {
            Console.WriteLine("sub form start");
            var form = new FormSub();
            await form.ShowAsync();
            // ウィンドウを閉じたら出力される
            Console.WriteLine("sub form end");
        };

        Controls.Add(button);
    }
}

class FormSub : Form
{
}
```

