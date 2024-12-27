# コアランタイム

## System 名前空間

### Array クラス

```csharp
CreateArray(typeof(int[]));

static void CreateArray(Type type)
{
    if (type.IsArray)
    {
        // 旧API
        var array1 = Array.CreateInstance(type.GetElementType()!, 10);
        // 新API
        var array2 = Array.CreateInstanceFromArrayType(type, 10);
    }
}
```


### BitConverter クラス

```csharp
var array = BitConverter.GetBytes(Int128.MaxValue);
Console.WriteLine(BitConverter.ToString(array));

var buffer = new byte[16];
BitConverter.TryWriteBytes(buffer, Int128.MaxValue);
Console.WriteLine(BitConverter.ToString(buffer));

var minusOne = BitConverter.ToInt128([255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255]);
Console.WriteLine(minusOne);
```


### Convert クラス

```csharp
Span<byte> buffer = (stackalloc byte[2]);

System.Buffers.OperationStatus status = Convert.FromHexString("FE12", buffer, out int charsConsumed, out int bytesWritten);

Console.WriteLine(Convert.ToHexString(buffer));
Console.WriteLine(Convert.ToHexStringLower(buffer));
```

```csharp
var buffer = new char[4];
Convert.TryToHexString([0xFE, 0x12], buffer, out int charsWritten);
Console.WriteLine(buffer);
```

### Delegate クラス

```csharp
var sample = new Sample();

// イベントハンドラの登録数を確認
sample.TestEvent += sample.A;
Console.WriteLine(sample.HasSingleTarget());
sample.TestEvent += sample.B;
Console.WriteLine(sample.HasSingleTarget());

// イベントハンドラを列挙
foreach (var action in sample.EnumerateInvocations())
    Console.WriteLine(action.Method.Name);

class Sample
{
    public event Action? TestEvent;

    public bool HasSingleTarget()
        => TestEvent?.HasSingleTarget ?? false;

    public Delegate.InvocationListEnumerator<Action> EnumerateInvocations()
        => Delegate.EnumerateInvocationList(TestEvent);

    public void A() { }
    public void B() { }
}
```

### Environment クラス

```csharp
// Task.Delayだとカウントされないので、多少の重い処理をしておく
var sum = 0;
foreach (var i in Enumerable.Repeat(1, 1_000_000))
    sum += i;

var usage = Environment.CpuUsage;
Console.WriteLine(usage.TotalTime.TotalMilliseconds);
Console.WriteLine(usage.UserTime.TotalMilliseconds);
Console.WriteLine(usage.PrivilegedTime.TotalNanoseconds);
```


### Guid 構造体

```csharp
var guid7 = Guid.CreateVersion7();
Console.WriteLine(guid7);
Console.WriteLine(guid7.Version);
Console.WriteLine(guid7.Variant);

Console.WriteLine(Guid.AllBitsSet);
```

### Math クラス

```csharp
// 従来
long upper = Math.BigMul(long.MaxValue, long.MaxValue, out long lower);
Console.WriteLine($"upper: 0x{upper:X}");
Console.WriteLine($"lower: 0x{lower:X}");

// 新API
Int128 total = Math.BigMul(long.MaxValue, long.MaxValue);
Console.WriteLine($"total: 0x{total:X}");
```

```csharp
ulong result = Math.BigMul(uint.MaxValue, uint.MaxValue);
Console.WriteLine(result.ToString("X"));
```

### MemoryExtensions クラス

```csharp
using System.Buffers;

var searchValues = SearchValues.Create(["world"], StringComparison.OrdinalIgnoreCase);
ReadOnlySpan<char> input = "Hello, World!";
Console.WriteLine(input.ContainsAny(searchValues));
Console.WriteLine(input.IndexOfAny(searchValues));
```

```csharp
ReadOnlySpan<byte> input = "Hello World!"u8;

// input.Length > 0 && input[0] == (byte)'H' と同じ
Console.WriteLine(input.StartsWith((byte)'H'));
// input.Length > 0 && input[^1] == (byte)'!' と同じ
Console.WriteLine(input.EndsWith((byte)'!'));
```

```csharp
string input = "0,1,2";

// AsSpanでReadOnlySpanにすると新しいAPIが呼ばれる
foreach (var elem in input.AsSpan().Split(','))
    Console.WriteLine(input[elem]);
```


### ReadOnlySpan<T> 構造体

```csharp
ReadOnlySpan<string> strArray = ["a", "b", "c"];
// 直接キャストはCS0030 エラー
var castArray = (ReadOnlySpan<object>)strArray;
// CastUpメソッドを使えばキャスト可能
var upcastArray = ReadOnlySpan<object>.CastUp(strArray);
```


### TimeSpan 構造体

```csharp
// 従来のFrom***系はdoubleを引数にとるので、1.001秒を指定できなかった
var ticks1 = TimeSpan.FromSeconds(1.001).Ticks;
Console.WriteLine(ticks1);
// コンストラクタならもともと整数で指定可能
var ticks2 = new TimeSpan(0, 0, 0, 1, 1).Ticks;
Console.WriteLine(ticks2);
// 新API
var ticks3 = TimeSpan.FromSeconds(1, 1).Ticks;
Console.WriteLine(ticks3);
```

```csharp
// 以下はすべて 900610010010 となる例
double ticksFromDays = TimeSpan.FromDays(1, 1, 1, 1, 1, 1).Ticks;
double ticksFromHours = TimeSpan.FromHours(25, 1, 1, 1, 1).Ticks;
double ticksFromMinutes = TimeSpan.FromMinutes(1501, 1, 1, 1).Ticks;
double ticksFromSeconds = TimeSpan.FromSeconds(90061, 1, 1).Ticks;
double ticksFromMilliseconds = TimeSpan.FromMilliseconds(90061001, 1).Ticks;
```


### Uri クラス

```csharp
ReadOnlySpan<char> source = "a+x";
Span<char> buffer = (stackalloc char[5]);

if (Uri.TryEscapeDataString(source, buffer, out int charsWritten))
    Console.WriteLine(buffer[..charsWritten].ToString());
```


## System.Buffers 名前空間

### SearchValues クラス

```csharp
using System.Buffers;

var searchValues = SearchValues.Create(["Hello", "world"], StringComparison.OrdinalIgnoreCase);
Console.WriteLine(searchValues.Contains("hello"));
```


## System.Buffers.Text 名前空間

### Base64Url クラス

```csharp
using System.Buffers.Text;

// 既存のAPI。stringとの変換はSystem.Convertクラスで、
// UTF-8 バイト列との変換はSystme.Buffers.Text.Base64クラスで行う。
string base64 = Convert.ToBase64String([0xFF]);
Console.WriteLine($"Base64:    {base64}");

// 新API
string base64uri = Base64Url.EncodeToString([0xFF]);
Console.WriteLine($"Base64Url: {base64uri}");
```

```csharp
using System.Buffers.Text;

Span<char> buffer = stackalloc char[16];
Base64Url.TryEncodeToChars([0xFF], buffer, out int bytesWritten);
Console.WriteLine(buffer[..bytesWritten].ToString());
```


## System.Collections.Generic 名前空間

### Dictionary\<TKey,TValue\> クラス

```csharp
var dict = new Dictionary<int, string>(16);
Console.WriteLine(dict.Capacity);
dict.TrimExcess();
Console.WriteLine(dict.Capacity);
```

```csharp
// string -> utf-8 のマップ
var dict = new Dictionary<string, byte[]>();
dict["test"] = "test"u8.ToArray();
var alternate = dict.GetAlternateLookup<ReadOnlySpan<char>>();
var value = alternate["test".AsSpan()];
Console.WriteLine(BitConverter.ToString(value));
```

```csharp
using System.Diagnostics.CodeAnalysis;

var dict = new Dictionary<byte[], string>(new Utf8Comparer());
dict["test"u8.ToArray()] = "test";
var alternate = dict.GetAlternateLookup<ReadOnlySpan<byte>>();
Console.WriteLine(alternate["test"u8]);

// byte[] と ReadOnlySpan<byte> の AlternateLookup をサポートするクラス
class Utf8Comparer : IEqualityComparer<byte[]>, IAlternateEqualityComparer<ReadOnlySpan<byte>, byte[]>
{
    public byte[] Create(ReadOnlySpan<byte> alternate) => alternate.ToArray();

    public bool Equals(byte[]? x, byte[]? y)
    {
        if (x == null && y == null) return true;
        if (x == null || y == null) return false;
        return x.AsSpan().SequenceEqual(y);
    }

    public bool Equals(ReadOnlySpan<byte> alternate, byte[] other)
        => alternate.SequenceEqual(other);

    public int GetHashCode([DisallowNull] byte[] obj) => SimpleAdd(obj);
    public int GetHashCode(ReadOnlySpan<byte> alternate) => SimpleXor(alternate);

    // とりあえずのハッシュ関数（雑すぎ……）
    private static int SimpleAdd(ReadOnlySpan<byte> span)
    {
        int hash = 0;
        foreach (var item in span)
            hash += item;
        return hash;
    }
}
```

### OrderedDictionary\<TKey,TValue\> クラス

```csharp
var dict = new OrderedDictionary<string, int>();
dict["one"] = 1;
dict["three"] = 3;
Console.WriteLine(dict.GetAt(1).Value);
```


### PriorityQueue\<TElement,TPriority\> クラス

```csharp
var queue = new PriorityQueue<Person, int>();

queue.Enqueue(new Person("Subaru", 15), 2);
queue.Enqueue(new Person("Tomoka", 12), 1);
queue.Enqueue(new Person("Maho", 12), 1);

// Dequeueは優先度の数値の小さい順に取り出す
if (queue.TryDequeue(out Person? element, out int priority))
{
    Console.WriteLine($"Dequeue: {element}");
}

// Removeは特定の要素を削除できる
// 比較用のEqualityComparerも渡せる
if (queue.Remove(new Person("Subaru", 15), out Person? removedElement, out int remvoedPriority, EqualityComparer<Person>.Default))
{
    Console.WriteLine($"Remove: {removedElement}");
}

record Person(string Name, int Age);
```


## System.Collections.ObjectModel 名前空間

### ReadOnlySet\<T\> クラス

```csharp
HashSet<int> set = [1, 2, 3];
var readonlySet = new ReadOnlySet<int>(set);
readonlySet.Add(4);  // CS1061 エラー
```



### TypeDescriptor クラス

```csharp
using System.ComponentModel;

// 先にRegisterしていないとInvalidOperationException
TypeDescriptor.RegisterType<Person>();

PropertyDescriptorCollection properties = TypeDescriptor.GetPropertiesFromRegisteredType(typeof(Person));
foreach (PropertyDescriptor property in properties)
{
    Console.WriteLine(property.Name);
}

record Person(string Name, int Age);
```


## System.Diagnostics 名前空間

### ActivityListener クラス

```csharp
using System.Diagnostics;

var activitySource = new ActivitySource("source", tags: [new KeyValuePair<string, object?>("tag1", 1)]);

var listener = new ActivityListener();
listener.ShouldListenTo += (ActivitySource actsrc) => actsrc.Name == "source";
listener.Sample = (ref ActivityCreationOptions<ActivityContext> options) => ActivitySamplingResult.AllDataAndRecorded;
// 例外の捕捉
listener.ExceptionRecorder = (Activity activity, Exception exception, ref TagList tags) =>
{
    Console.WriteLine($"exception: {exception}");
};

ActivitySource.AddActivityListener(listener);

using (var activity = activitySource.StartActivity("act1", ActivityKind.Internal))
{
    activity?.AddException(new InvalidOperationException("test"));
}
```


### DiagnosticMethodInfo クラス

```csharp
using System.Diagnostics;

var methodInfo = DiagnosticMethodInfo.Create(Hello)!;
Console.WriteLine(methodInfo.DeclaringAssemblyName);
Console.WriteLine(methodInfo.DeclaringTypeName);
Console.WriteLine(methodInfo.Name);

static void Hello() => Console.WriteLine("Hello World!");
```


## System.Diagnostics.CodeAnalysis 名前空間

### FeatureGuardAttribute クラス

```csharp
using System.Diagnostics.CodeAnalysis;
using System.Runtime.CompilerServices;

// AOT非対応のAPIのつもり
if (TestFeature.IsSupported)
    TestFeature.Test();

public class TestFeature
{
    [FeatureGuard(typeof(RequiresDynamicCodeAttribute))]
    public static bool IsSupported => RuntimeFeature.IsDynamicCodeSupported;

    // （本来は）リフレクションを使うメソッド
    public static void Test() => Console.WriteLine("Test");
}
```

### FeatureSwitchDefinitionAttribute クラス

```csharp
using System.Diagnostics.CodeAnalysis;

if (TestFeature.IsSupported)
    TestFeature.Test();

public class TestFeature
{
    [FeatureSwitchDefinition("TestFeature.IsSupported")]
    public static bool IsSupported => AppContext.TryGetSwitch("TestFeature.IsSupported", out var isEnabled) ? isEnabled : true;

    public static void Test() => Console.WriteLine("Test");
}
```


## System.Diagnostics.Metrics 名前空間


### Gauge\<T\> クラス

```csharp
using System.Diagnostics.Metrics;

var meter = new Meter("testmeter");
var listener = new MeterListener();
// listenerの購読有効化
listener.InstrumentPublished += (instrument, listener) =>
{
    if (instrument.Meter.Name == "testmeter" && instrument is Gauge<long>)
    {
        listener.EnableMeasurementEvents(instrument, null);
    }
};

listener.SetMeasurementEventCallback<long>(
    (instrument, measurement, tags, state)
    => Console.WriteLine($"{measurement} by {instrument.Name}"));

listener.Start();

var gauge = meter.CreateGauge<long>("gauge");
gauge.Record(1);
gauge.Record(2);

// こっちはcounterが登録されてないので何も起きない
var counter = meter.CreateCounter<long>("counter");
counter.Add(3);
```

### Instrument\<T\> クラス

```csharp
using System.Diagnostics.Metrics;

var advice = new InstrumentAdvice<long>()
{
    HistogramBucketBoundaries = [0, 1, 2, 3, 4]
};
var meter = new Meter("testmeter");
var histogram = meter.CreateHistogram("histogram", advice: advice);

var boundaries = histogram.Advice!.HistogramBucketBoundaries!;
Console.WriteLine($"Boundaries: {string.Join(", ", boundaries)}");
```


## System.IO 名前空間

### File クラス

```csharp
// UTF-8（BOMBつき）テキスト
File.WriteAllBytes("test.txt", [0xEF, 0xBB, 0xBF]);
File.AppendAllBytes("test.txt", "Hello, world!"u8);
```

### TextWriter クラス

```csharp
using var writer1 = new StreamWriter(File.OpenWrite("test1.txt"));
using var writer2 = new StreamWriter(File.OpenWrite("test2.txt"));

using var combined = TextWriter.CreateBroadcasting(writer1, writer2);
combined.WriteLine("Hello, World!");
// test1.txtとtest2.txtに「Hello, World!」が書き込まれる
```


## System.IO.Compression 名前空間

### DeflateStream / GZipStream / ZLibStream クラス

```csharp
using System.IO.Compression;

var options = new ZLibCompressionOptions()
{
    CompressionLevel = 9
};

// plain.dat を compressed.dat に圧縮する
var source = File.OpenRead("plain.dat");
var gzip = new GZipStream(File.Create("compressed.dat"), options);
source.CopyTo(gzip);
```

## System.Linq 名前空間

### Enumerable クラス

```csharp
Person[] people = [new("Subaru", 15), new("Tomoka", 12), new("Maho", 12)];

foreach (var x in people.AggregateBy(x => x.Age, "", (sum, elem) => sum += $"{elem.Name}, "))
{
    Console.WriteLine($"Age {x.Key}: {x.Value}");
}

record Person(string Name, int Age);
```

```csharp
Person[] people = [new("Subaru", 15), new("Tomoka", 12), new("Maho", 12)];

foreach (var x in people.CountBy(x => x.Age))
{
    Console.WriteLine($"Age {x.Key}: {x.Value}");
}

record Person(string Name, int Age);
```


```csharp
Person[] people = [new("Subaru", 15), new("Tomoka", 12), new("Maho", 12)];

// foreach (var (index, item) in people.Select((item, index) => (index, item))) と同じ
foreach (var (index, item) in people.Index())
{
    Console.WriteLine($"{index}: {item.Name}");
}

record Person(string Name, int Age);
```


## System.Net.Mime 名前空間

### MediaTypeNames クラス

```csharp
using System.Net.Mime;

Console.WriteLine(MediaTypeNames.Application.GZip);
Console.WriteLine(MediaTypeNames.Multipart.Mixed);
Console.WriteLine(MediaTypeNames.Multipart.Related);
Console.WriteLine(MediaTypeNames.Text.EventStream);
```


## System.Numerics 名前空間

### IFloatingPoint<TSelf> インターフェイス

```csharp
using System.Numerics;

double value = 1.5;
int result = ToInt(value);
Console.WriteLine(result);

static int ToInt<T>(T x) where T : IFloatingPoint<T>
    => T.ConvertToInteger<int>(x);
```


### INumberBase<TSelf> インターフェイス

```csharp
using System.Numerics;

double a = double.MaxValue;
double b = -2.0;
double c = double.MaxValue;

Console.WriteLine(a * b + c);
Console.WriteLine(Math.FusedMultiplyAdd(a, b, c));
Console.WriteLine(CalcEstimate(a, b, c));

// Generic Mathの一部なのでジェネリック越しでしか呼べない
static T CalcEstimate<T>(T a, T b, T c) where T : INumberBase<T>
    => T.MultiplyAddEstimate(a, b, c);
```


## System.Reflection 名前空間

### Assembly クラス

```csharp
using System.Reflection;

var asm = Assembly.GetEntryAssembly();
Console.WriteLine(asm.FullName);
```

```csharp
using System.Reflection;

Assembly asm = Assembly.LoadFrom("Plugin.dll");
if (asm.EntryPoint is MethodInfo entryPoint)
{
    entryPoint.Invoke(null, [Array.Empty<string>()]);
}
```

```csharp
using System.Reflection;

Assembly asm = Assembly.LoadFrom("Plugin.dll");
if (asm.EntryPoint is MethodInfo entryPoint)
{
    Assembly.SetEntryAssembly(asm);  // ここを追加
    entryPoint.Invoke(null, [Array.Empty<string>()]);
}
```


## System.Reflection.Emit 名前空間


### PersistedAssemblyBuilder クラス

```csharp
using System.Reflection;
using System.Reflection.Emit;
using System.Reflection.Metadata;
using System.Reflection.Metadata.Ecma335;
using System.Reflection.PortableExecutable;

// DynamicApp.dll を作成
{
    var builder = new PersistedAssemblyBuilder(new AssemblyName("DynamicApp"), typeof(object).Assembly);

    ModuleBuilder module = builder.DefineDynamicModule("MyModule");

    // Hello, World! するだけの Main メソッドを定義
    TypeBuilder type = module.DefineType("Program", TypeAttributes.Public | TypeAttributes.Class);
    MethodBuilder main = type.DefineMethod("Main", MethodAttributes.Public | MethodAttributes.Static, typeof(void), [typeof(string[])]);

    // 処理本体
    ILGenerator il = main.GetILGenerator();
    il.Emit(OpCodes.Ldstr, "Hello, World!");
    il.Emit(OpCodes.Call, typeof(Console).GetMethod("WriteLine", [typeof(string)])!);
    il.Emit(OpCodes.Ret);

    type.CreateType();

    MetadataBuilder metadataBuilder = builder.GenerateMetadata(out BlobBuilder ilStream, out BlobBuilder mappedFieldData);

    ManagedPEBuilder managedPEBuilder = new(
        header: new PEHeaderBuilder(imageCharacteristics: Characteristics.ExecutableImage),
        metadataRootBuilder: new MetadataRootBuilder(metadataBuilder),
        ilStream: ilStream,
        mappedFieldData: mappedFieldData,
        entryPoint: MetadataTokens.MethodDefinitionHandle(main.MetadataToken)
    );

    BlobBuilder peBlobBuilder = new();
    managedPEBuilder.Serialize(peBlobBuilder);

    using var file = File.Create("DynamicApp.dll");
    peBlobBuilder.WriteContentTo(file);
}

// 作成した DynamicApp.dll をロードして Main メソッドを実行
Assembly asm = Assembly.LoadFrom("DynamicApp.dll");
if (asm.EntryPoint is MethodInfo entryPoint2)
{
    Console.WriteLine("DynamicApp.dll loaded successfully!");
    entryPoint2.Invoke(null, [Array.Empty<string>()]);
}
```


## System.Reflection.Metadata 名前空間

### AssemblyNameInfo クラス

```csharp
using System.Reflection.Metadata;

// System.Reflection.Assembly.GetExecutingAssembly().FullName) で取得できる
var info = AssemblyNameInfo.Parse("Playground, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null");

Console.WriteLine(info.FullName);
Console.WriteLine(info.CultureName);
Console.WriteLine(info.PublicKeyOrToken);
Console.WriteLine(info.Version);
```


### TypeName クラス

```csharp
using System.Reflection.Metadata;

var type = TypeName.Parse("System.Collections.Generic.Dictionary`2[System.Int32,System.String]");
foreach (var genericType in type.GetGenericArguments())
{
    Console.WriteLine(genericType.FullName);
}
```



## System.Runtime.CompilerServices 名前空間

### OverloadResolutionPriorityAttribute クラス

```csharp
using System.Runtime.CompilerServices;

string text = "Hello, World!";
C.Print(text);

static class C
{
    [OverloadResolutionPriority(-1)]
    public static void Print(string text) => Console.WriteLine("string");
    public static void Print(ReadOnlySpan<char> text) => Console.WriteLine("ReadOnlySpan<char>");
}
```


### ParamCollectionAttribute クラス

```csharp
// こういうメソッドを書くと、
static void Test(params List<char> chars) => Console.WriteLine(chars.Count);

// コンパイル→逆コンパイルでこんな感じになる
static void Test([ParamCollection] List<char> chars) => Console.WriteLine(chars.Count);
```


### RuntimeHelpers クラス

```コード
using System.Numerics;
using System.Runtime.CompilerServices;

var v = new Vector3();
int structSize = RuntimeHelpers.SizeOf(Type.GetTypeHandle(v));
Console.WriteLine(structSize);

var obj = new object();
int classSize = RuntimeHelpers.SizeOf(Type.GetTypeHandle(obj));
Console.WriteLine(classSize);
```

## System.Runtime.InteropServices 名前空間


### CollectionsMarshal クラス

```csharp
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

var dict = new Dictionary<string, byte[]>();
dict["foo"] = "foo"u8.ToArray();
var alternate = dict.GetAlternateLookup<ReadOnlySpan<char>>();

ref byte[] value = ref CollectionsMarshal.GetValueRefOrNullRef(alternate, "bar".AsSpan());
Console.WriteLine(Unsafe.IsNullRef(ref value));
```


### JsonMarshal クラス

```csharp
using System.Runtime.InteropServices;
using System.Text;
using System.Text.Json;

var json = """
    {"name":"Anseketamen"}
    """;
var nameProperty = JsonDocument.Parse(json).RootElement.GetProperty("name");
var utf8name = JsonMarshal.GetRawUtf8Value(nameProperty);
Console.WriteLine(Encoding.UTF8.GetString(utf8name));
```

## System.Security.Cryptography 名前空間


### CryptographicOperations クラス

```csharp
using System.Security.Cryptography;

var hash = CryptographicOperations.HashData(HashAlgorithmName.SHA1, [0]);
Console.WriteLine(BitConverter.ToString(hash));
```

### IncrementalHash クラス

```csharp
using System.Security.Cryptography;

var hash = IncrementalHash.CreateHMAC(HashAlgorithmName.SHA1, []);
hash.AppendData(new byte[16]);
Console.Write("original: ");
Console.WriteLine(BitConverter.ToString(hash.GetCurrentHash()));

var clone = hash.Clone();
Console.Write("clone:    ");
Console.WriteLine(BitConverter.ToString(clone.GetCurrentHash()));
```


## System.Text 名前空間

### StringBuilder クラス

```csharp
using System.Text;

var builder = new StringBuilder();
builder.Append("Hello, world!");

ReadOnlySpan<char> replaceFrom = ['!'];
ReadOnlySpan<char> replaceTo = ['\uD83C', '\uDF1E'];

builder.Replace(replaceFrom, replaceTo);
Console.WriteLine(builder.ToString());
```

## System.Text.Json 名前空間

### JsonElement 構造体

```csharp
using System.Text.Json;

var json = "{\"Name\":\"John\",\"Age\":30}";

var element1 = JsonSerializer.Deserialize<JsonElement>(json);
var element2 = JsonSerializer.Deserialize<JsonElement>(json);

Console.WriteLine(JsonElement.DeepEquals(element1, element2));
Console.WriteLine(element1.GetPropertyCount());
```

### JsonReaderOptions クラス

```csharp
using System.Text.Json;

var jsons = """
{"Name":"Subaru","Age":15}
{"Name":"Tomoka","Age":12}
"""u8;

var options = new JsonReaderOptions()
{
    AllowMultipleValues = true
};

var reader = new Utf8JsonReader(jsons, options);
while (reader.Read())
{
    if (reader.TokenType == JsonTokenType.StartObject)
    {
        var person = JsonSerializer.Deserialize<Person>(ref reader);
        Console.WriteLine(person);
    }
}

record Person(string Name, int Age);
```



## System.Text.Json.Schema 名前空間

### JsonSchemaExporter クラス

```csharp
using System.Text.Json;
using System.Text.Json.Schema;

var options = new JsonSerializerOptions(JsonSerializerOptions.Default)
{
    WriteIndented = true
};

var node = options.GetJsonSchemaAsNode(typeof(Person));
Console.WriteLine(node.ToJsonString(options));

record Person(string Name, int Age);
```


## System.Text.Json.Serialization 名前空間

### JsonStringEnumMemberNameAttribute クラス

```csharp
using System.Text.Encodings.Web;
using System.Text.Json;
using System.Text.Json.Serialization;

var options = new JsonSerializerOptions
{
    Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};

var json = JsonSerializer.Serialize(School.Nanashiba, options);
Console.WriteLine(json);

[JsonConverter(typeof(JsonStringEnumConverter))]
enum School
{
    Keishin = 1,
    [JsonStringEnumMemberName("七芝")]
    Nanashiba = 2,
}
```


## System.Text.Json.Serialization.Metadata 名前空間

### JsonTypeInfo クラス

```csharp
using System.Text.Json.Serialization;
using System.Text.Json.Serialization.Metadata;

JsonTypeInfo? typeInfo = DictionaryContext.Default.GetTypeInfo(typeof(Dictionary<string, int>))!;
Console.WriteLine(typeInfo.Kind);
Console.WriteLine(typeInfo.ElementType?.FullName);
Console.WriteLine(typeInfo.KeyType?.FullName);

[JsonSerializable(typeof(Dictionary<string, int>))]
partial class DictionaryContext : JsonSerializerContext { }
```


### JsonParameterInfo クラス

```csharp
using System.Text.Json.Serialization;
using System.Text.Json.Serialization.Metadata;

JsonTypeInfo? typeInfo = PersonContext.Default.GetTypeInfo(typeof(Person))!;
foreach (var property in typeInfo.Properties)
{
    JsonParameterInfo? parameterInfo = property.AssociatedParameter;
    if (parameterInfo != null)
    {
        Console.WriteLine($"Type: {parameterInfo.ParameterType}, Name: {parameterInfo.Name}");
    }
}

[JsonSerializable(typeof(Person))]
partial class PersonContext : JsonSerializerContext { }

record Person(string Name, int Age);
```



## System.Text.RegularExpressions 名前空間

### Regex クラス

```csharp
using System.Text.RegularExpressions;

string input = "test1,test2-test3;test4";
var regex = new Regex("[-,;]");
foreach (var match in regex.EnumerateSplits(input))
{
    Console.WriteLine(input[match]);
}
```



## System.Threading 名前空間

### Interlocked クラス

```csharp
byte x = 0xFF;
Interlocked.Exchange(ref x, 0x00);
Console.WriteLine(x);
```

```csharp
var c = new C();
c.Dispose();
c.Dispose();

public class C : IDisposable
{
    private bool _disposed = false;

    public void Dispose()
    {
        if (Interlocked.Exchange(ref _disposed, true))
            return;

        Console.WriteLine("Disposed");
        GC.SuppressFinalize(this);
    }
}
```

### Lock クラス

```csharp
class C
{
    private readonly Lock _lock = new();

    public void Write(ReadOnlySpan<byte> data)
    {
        (using var scope = _lock.EnterScope())
        {   
            // 書き込み
        }

        // 既存のlockキーワードも使用可能
        lock (_lock)
        {
            // 書き込み
        }
    }
}
```


```csharp
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

byte[] data = [1, 2, 3, 4];

// ロック前
PrintDump("unlocked:   ", data);

// ロック中
lock (data)
{
    PrintDump("locked:     ", data);
}

// ロック解除後
PrintDump("unlocked:   ", data);

int hashCode = data.GetHashCode();
Console.WriteLine($"hash code:  {hashCode:X8}");

// HashCode計算後
PrintDump("after hash: ", data);

static void PrintDump(string title, Span<byte> data)
{
    // 配列の手前20バイトまでを表示
    ref byte start = ref MemoryMarshal.GetReference<byte>(data);
    ref byte shifted = ref Unsafe.Subtract(ref start, 20);
    Span<byte> dump = MemoryMarshal.CreateSpan(ref shifted, 24);
    Console.Write(title);
    Console.WriteLine(BitConverter.ToString(dump.ToArray()));
}
```


## System.Threading.Channels 名前空間

### Channel クラス

```csharp
using System.Threading.Channels;

// Age の若い順に優先度を設定
var options = new UnboundedPrioritizedChannelOptions<Person>()
{
    Comparer = Comparer<Person>.Create(static (x, y) => x.Age.CompareTo(y.Age))
};

var channel = Channel.CreateUnboundedPrioritized(options);

// 読み出しタスク
var reader = channel.Reader;
_ = Task.Run(async () =>
{
    while (true)
    {
        Console.WriteLine(await reader.ReadAsync());
    }
});

channel.Writer.TryWrite(new Person("Subaru", 15));
channel.Writer.TryWrite(new Person("Tomoka", 12));
channel.Writer.TryWrite(new Person("Maho", 12));

// 読み終わるまでちょっと待つ
await Task.Delay(100);

record Person(string Name, int Age);
```


## System.Threading.Tasks 名前空間

### Task クラス

```csharp
var tasks = Task.WhenEach(WaitAsync(3000), WaitAsync(2000), WaitAsync(1000));
await foreach (var task in tasks)
{
    var millis = await task;
    Console.WriteLine($"Waited {millis} milliseconds");
}

// 待機したミリ秒を返す
static Task<int> WaitAsync(int milliseconds)
    => Task.Delay(milliseconds).ContinueWith(t => milliseconds);
```


### TaskCompletionSource クラス

```csharp
static Task NopAsync()
{
    var tcs = new TaskCompletionSource();
    tcs.SetFromTask(Task.CompletedTask);

    return tcs.Task;
}
```


## System.Timers 名前空間

### ElapsedEventArgs クラス

```csharp
using System.Timers;

var e = new ElapsedEventArgs(new DateTime(1975, 10, 29));
Elapsed(e);

static void Elapsed(ElapsedEventArgs e)
{
    Console.WriteLine(e.SignalTime);
}
```
