# C# 13関連の新機能

## params Collections

```csharp
ReadOnlySpan<object?> span = ["Hello", "world!"];
Console.WriteLine("{0}, {1}", span);
```

## allows ref struct

### 従来

```csharp
// BitConverter（引数がbyte[]）
Console.WriteLine(BitConverter.ToString([0xFF, 0xAA, 0x55, 0x00]));
// ReadOnlySpan<byte>を引数にした自作バージョン
Console.WriteLine(ToHexString_Unsafe([0xFF, 0xAA, 0x55, 0x00]));

static unsafe string ToHexString_Unsafe(ReadOnlySpan<byte> source)
{
    if (source.Length == 0) return string.Empty;

    var p = (IntPtr)(&source);
    return string.Create(source.Length * 3 - 1, p, (chars, p) =>
    {
        ReadOnlySpan<byte> bytes = *(ReadOnlySpan<byte>*)p;
        chars[0] = ToHexChar(bytes[0] >> 4);
        chars[1] = ToHexChar(bytes[0] & 0xF);
        for (int i = 1; i < bytes.Length; i++)
        {
            chars[i * 3 - 1] = '-';
            chars[i * 3] = ToHexChar(bytes[i] >> 4);
            chars[i * 3 + 1] = ToHexChar(bytes[i] & 0xF);
        }
    });
}

static char ToHexChar(int value) => (char)(value < 10 ? value + '0' : value - 10 + 'A');
```

### .NET 9

```csharp
Console.WriteLine(ToHexString_Generic([0xFF, 0xAA, 0x55, 0x00]));

static string ToHexString_Generic(ReadOnlySpan<byte> source)
{
    if (source.Length == 0) return string.Empty;

    return string.Create(source.Length * 3 - 1, source, (chars, bytes) =>
    {
        chars[0] = ToHexChar(bytes[0] >> 4);
        chars[1] = ToHexChar(bytes[0] & 0xF);
        for (int i = 1; i < bytes.Length; i++)
        {
            chars[i * 3 - 1] = '-';
            chars[i * 3] = ToHexChar(bytes[i] >> 4);
            chars[i * 3 + 1] = ToHexChar(bytes[i] & 0xF);
        }
    });
}

static char ToHexChar(int value) => (char)(value < 10 ? value + '0' : value - 10 + 'A');
```
