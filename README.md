# Minid

[![NuGet](https://img.shields.io/nuget/v/minid.svg)](https://www.nuget.org/packages/minid) 
[![NuGet](https://img.shields.io/nuget/dt/minid.svg)](https://www.nuget.org/packages/minid)
[![License](https://img.shields.io/:license-mit-blue.svg)](https://benfoster.mit-license.org/)

![Build](https://github.com/benfoster/minid/workflows/Build/badge.svg)
[![Coverage Status](https://coveralls.io/repos/github/benfoster/minid/badge.svg?branch=main)](https://coveralls.io/github/benfoster/minid?branch=main)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=benfoster_minid&metric=alert_status)](https://sonarcloud.io/dashboard?id=benfoster_minid)
[![GuardRails badge](https://api.guardrails.io/v2/badges/benfoster/minid.svg?token=59af717aeba71bc862995cd302659f0e511ebe43ff28ee6df11fe8669b15dc1d&provider=github)](https://dashboard.guardrails.io/gh/benfoster/repos/146839)

Minid generates human-readable, URL-friendly, unique identifiers that are computed in-memory.

- Safe without encoding (uses only characters from ASCII)
- Avoids ambiguous characters (i/I/l/L/o/O/0)
- Easy for humans to read and pronounce
- Supports full UUID range (128 bits)
- Safe for URLs and file names
- Case-insensitive
- 30% smaller than Guid's default string format

### Example

The `Guid` `8108afcc-980f-438d-bdd7-51375fcf073a` converted to a Minid `Id` is encoded as `473cr1y0ghbyc3m1yfbwvn3nxx`.

## Motivation

The motivation for this library came from a need to return readable "resource" identifiers in our APIs that could be computed quickly (no database lookups). In .NET the [Guid Struct](https://learn.microsoft.com/en-us/dotnet/api/system.guid?view=net-7.0) meets the uniqueness and computational requirements but its string representation is not optimal in terms of size and format.

With [Base32 encoding](https://en.wikipedia.org/wiki/Base32) we can encode a 128-bit `Guid` into a 26-character string. My original implementation was based on a Base32 encoder ported from [this Java implementation](https://github.com/google/google-authenticator-android/blob/master/java/com/google/android/apps/authenticator/util/Base32String.java).

I later updated this to use [Kristian Hellang](https://github.com/khellang)'s "CompactGuid" implementation which was far more performant and had better support for ambiguous characters 🙇‍♂️

## Usage

```
dotnet add package minid
```

You can then use the provided `Id` struct to generate identifiers in your applications. 

```
public class Customer
{
    public Id Id { get; }

    public Customer()
    {
        Id = Id.NewId();
    }
}
```

To get the Base32-encoded value call `ToString()` on the `Id` value:

```
string encoded = customer.Id.ToString();
```

You can also initialise the `Id` type from an existing Guid:

```
var existingId = new Id(existingGuid);
```

### Serialization and Model-binding

A type converter is included so that you can bind directly to the `Id` type in your applications and both System.Text.Json and Newtonsoft will also take care of serializing and deserializing it correctly.


### Why a dedicated type?

We originally started to use Base32-encoding to format our API resource identifiers in responses. Internally we used the underlying Guid value. This became problematic when correlating client and internal requests. Effectively we had introduced a second implicit identifier and our codebase was littered with conversion to/from the encoded values. 

In our case it was far better to use the same encoded representation throughout our platform. Given we were mostly using document/no-sql databases which typically store Guid values as strings, we benefited from the reduction in size.

If you don't want to depend on this type you can use `string` but this will allocate more memory. My recommendation is to only convert to `string` when you need to.

## Benchmarks

// * Summary *

BenchmarkDotNet=v0.13.2, OS=macOS Monterey 12.5 (21G72) [Darwin 21.6.0]
Apple M1 Max, 1 CPU, 10 logical and 10 physical cores
.NET SDK=6.0.400
  [Host]     : .NET 6.0.8 (6.0.822.36306), Arm64 RyuJIT AdvSIMD
  DefaultJob : .NET 6.0.8 (6.0.822.36306), Arm64 RyuJIT AdvSIMD


|       Method |      Mean |    Error |   StdDev |   Gen0 | Allocated |
|------------- |----------:|---------:|---------:|-------:|----------:|
|        NewId | 112.24 ns | 0.430 ns | 0.381 ns |      - |         - |
|   IdToString | 151.08 ns | 0.632 ns | 0.528 ns | 0.0381 |      80 B |
| GuidToString | 217.03 ns | 0.606 ns | 0.537 ns | 0.0458 |      96 B |
|      ParseId |  52.88 ns | 0.235 ns | 0.220 ns |      - |         - |
