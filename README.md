# PAW C# SDK

The official C# SDK for [PAW (Puppy's Avatar World)](https://paw-api.amelia.fun) — a VRChat avatar database and search platform.

[![Discord](https://img.shields.io/discord/1331153121472282652?color=%23512BD4&label=Discord&logo=discord&logoColor=white&style=flat)](https://discord.gg/zHhs4nQYxX)
[![NuGet](https://img.shields.io/badge/NuGet-PAW__SDK-blue?logo=nuget)](https://www.nuget.org)
[![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet)](https://dotnet.microsoft.com)

## Overview

PAW SDK is a .NET 9 client library for interacting with the PAW API. It provides a simple, strongly-typed interface for searching, browsing, and requesting updates for VRChat avatars.

## Getting Started

### Prerequisites

- .NET 9.0 or later

### Quick Start

```csharp
using PAW_SDK;
using PAW_SDK.Models;

using var client = new PAWClient("YourApp/1.0.0");

var (avatars, pagination) = await client.SearchAsync(new SearchParams
{
    Query    = "Manuka",
    Type     = SearchType.Name,
    Order    = SearchOrder.Newest,
    Platforms = [Platform.PC],
    PageSize = 10
});

foreach (var avatar in avatars)
{
    Console.WriteLine($"{avatar.Name} — {avatar.Id}");
}
```

## Usage

### Creating a Client

```csharp
// Basic — user agent is required
using var client = new PAWClient("YourApp/1.0.0");

// With a custom timeout
using var client = new PAWClient("YourApp/1.0.0", timeout: TimeSpan.FromSeconds(10));
```

---

### Fetch a Single Avatar

```csharp
Avatar? avatar = await client.GetAvatarAsync("avtr_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx");

if (avatar is not null)
    Console.WriteLine($"{avatar.Name} by {avatar.AuthorName}");
```

---

### Search Avatars

```csharp
var (avatars, pagination) = await client.SearchAsync(new SearchParams
{
    Query    = "fox",
    Type     = SearchType.Name,     // Name | Description | Author | AuthorName | AiTags
    Order    = SearchOrder.Newest,  // Newest | Oldest
    Platforms = [Platform.PC],      // PC | Android | iOS
    Page     = 1,
    PageSize = 20
});

Console.WriteLine($"Page {pagination?.CurrentPage} of {pagination?.TotalPages}");

foreach (var avatar in avatars)
    Console.WriteLine(avatar.Name);
```

**Search Types**

| Value | Description |
|---|---|
| `SearchType.Name` | Search by avatar name |
| `SearchType.Description` | Search by avatar description |
| `SearchType.Author` | Search by author ID |
| `SearchType.AuthorName` | Search by author display name |
| `SearchType.AiTags` | AI-powered semantic tag search |

---

### Get Random Avatars

```csharp
IReadOnlyList<Avatar> avatars = await client.GetRandomAvatarsAsync();
```

---

### Get Recent Avatars

```csharp
// Recently added
var added = await client.GetRecentAvatarsAsync(RecentType.Added);

// Recently updated
var updated = await client.GetRecentAvatarsAsync(RecentType.Updated);

// Recently checked
var checked = await client.GetRecentAvatarsAsync(RecentType.Checked);
```

---

### Get Similar Avatars

Uses AI to find avatars similar to a given one.

```csharp
var (similar, pagination) = await client.GetSimilarAsync("avtr_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx");
```

---

### Request an Avatar Update

Queues a refresh of avatar data from VRChat. The API will ignore requests for the same avatar made within 3 days of the last check.

```csharp
UpdateResult result = await client.RequestUpdateAsync("avtr_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx");

if (result.Success)
    Console.WriteLine("Update queued!");
```

---

### Proxy Images

All `ThumbnailUrl` and `ImageUrl` fields on `Avatar` objects are already proxied through the PAW API. If you need to manually build or fetch a proxied image:

```csharp
// Build a proxy URL without downloading
Uri proxyUrl = client.BuildProxyUrl("https://api.vrchat.cloud/api/1/image/...");

// Download the image bytes directly
var (bytes, contentType) = await client.FetchProxiedImageAsync("https://api.vrchat.cloud/api/1/image/...");
```

---

### Error Handling

All methods throw `APIException` on non-successful API responses.

```csharp
using PAW_SDK.Exceptions;

try
{
    var avatar = await client.GetAvatarAsync("avtr_xxxx");
}
catch (APIException ex)
{
    Console.WriteLine($"API error {ex.StatusCode}: {ex.Message}");
}
```

---

## Models

### `Avatar`

| Property | Type | Description |
|---|---|---|
| `Id` | `string` | VRChat avatar ID |
| `Name` | `string` | Avatar display name |
| `Description` | `string?` | Avatar description |
| `AuthorId` | `string?` | Author's VRChat user ID |
| `AuthorName` | `string?` | Author's display name |
| `ThumbnailUrl` | `string?` | Proxied thumbnail image URL |
| `ImageUrl` | `string?` | Proxied full image URL |
| `Platform` | `string?` | Target platform (`pc`, `android`, `ios`) |
| `AddedAt` | `DateTimeOffset?` | When the avatar was added to PAW |
| `UpdatedAt` | `DateTimeOffset?` | When the avatar data was last updated |
| `CheckedAt` | `DateTimeOffset?` | When the avatar was last checked |

### `Pagination`

| Property | Type | Description |
|---|---|---|
| `CurrentPage` | `int` | Current page number |
| `PageSize` | `int` | Results per page |
| `TotalCount` | `int` | Total number of results |
| `TotalPages` | `int` | Total number of pages |
| `HasNextPage` | `bool` | Whether a next page exists |
| `HasPreviousPage` | `bool` | Whether a previous page exists |

## Technologies Used

- [.NET 8](https://dotnet.microsoft.com)
- [System.Text.Json](https://learn.microsoft.com/en-us/dotnet/api/system.text.json)
- [System.Net.Http](https://learn.microsoft.com/en-us/dotnet/api/system.net.http)

## Credits

Avatar data is sourced from the [PAW API](https://paw-api.amelia.fun), part of the PAW (Puppy's Avatar World) platform.
