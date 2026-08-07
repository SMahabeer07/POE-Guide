# POE-Guide
For part 1 of the CLDV6212 POE we are looking at Azure Table Storage and Azure File Storage. Please note that you need to have a group of between 2 to 4 for this POE.

Group leaders can complete this form on their groups behalf: [CLDV6212 POE Groups](https://forms.cloud.microsoft/r/3TjeYvrGPx)

**NO ORGANIZATIONS BECAUSE WE MAY GET MODERATED**

We can stick to organizations for the ICE Task.

## VS Code
- Download VS Code [here](https://code.visualstudio.com)

Lets chat in class and come up with a good strategy we can apply across the board to make sure your first time in a group in a practical module goes smoothly and I will update this repo accordingly. Talking points include how to work with multiple branches, file structures, how to delegate etc.

We will also discuss the readme, classes and more. Today we will also be containerizing our first app so stick around for that.
- POE guide will be completed as individuals not in groups.

## POE Part 1

### Our scenario
We're building a simple **Budget Tracker** app. There are two components:

1. **Purchases** — a `Purchases` Table Storage table logging what's been bought (item name, amount, category, date), with full CRUD plus a "get by category" endpoint (e.g. pull up everything under "Groceries" or "Transport").
2. **Receipts** — a `receipts-share` Azure File Share for supporting documents: scanned receipts, invoices, statements. You'll build upload, list, and download endpoints against it.

### Marks breakdown for Part 1
Use this to sanity-check your submission before deadline — this is a summary of the actual rubric, not the full wording:

| Section | Marks | What "greatly exceeds" looks like |
|---|---|---|
| Submission, Structure & Documentation | 10 | `/docs` folder, README with step-by-step Azurite startup, architecture overview, individual role contributions, verified commit logs, professional unlisted YouTube walkthrough |
| Azure Tables & HTTP Functions (Purchases) | 30 | Full CRUD + category filtering, robust validation, custom error handling (404/400), clean separation of storage concerns |
| Azure Files Integration (Receipts) | 20 | Stream-based transfers, file metadata tracking (size, upload date), MIME-type validation, full error logging |
| Standalone Dockerfiles & Docker Hub Publishing | 20 | Optimized (multi-stage) Dockerfile, explicit port binding, env vars passed via CLI, public Docker Hub repo with a verified semantic version tag |
| Testing Suite & Postman Collection | 20 | Folders per resource, `{{baseUrl}}` environment variable, sample request bodies for every scenario, saved automated tests |
| Version Control & Repository Progress | 10 | Descriptive feature-based commits, feature branching, zero merge conflicts, distributed activity across all members |

**Penalty to know about:** no GitHub used, or a private repo set up incorrectly, is an automatic **-5 marks** — sort out repo access for every group member early.

### Submission, Structure & Documentation
Since this is 10 marks on its own, don't leave it as an afterthought:
- Keep a `/docs` folder in your repo for any architecture notes, diagrams, or extra setup instructions beyond this README.
- Your own README (per group) needs the local Azurite startup steps written out clearly enough that someone who missed class could follow them.
- Minimum **5 commits per student**, spread across development — not all clustered right before the deadline.
- Record an **unlisted YouTube video** walking through your code with voiceover.
- Each member's contribution should be identifiable, whether that's through commit history, a short section in the README, or both.

## Azurite Storage Setup
Use this link for Azurite setup: https://hub.docker.com/r/microsoft/azure-storage-azurite

- First make sure you have WSL. If you do not you can open a terminal (global) and paste:

wsl --install

- You need to install Docker Desktop next: https://docs.docker.com/desktop/setup/install/windows-install/
- Once you have both you can actually follow the guide by opening the Docker Desktop app, running the docker engine and opening the terminal. In the terminal paste the below commands in order:

docker pull mcr.microsoft.com/azure-storage/azurite

- If you want to run everything:

docker run -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite

- If you just want to run one service like Blob:

docker run -p 10000:10000 mcr.microsoft.com/azure-storage/azurite azurite-blob --blobHost 0.0.0.0

You can swap `azurite-blob`/`--blobHost` for `azurite-queue`/`--queueHost` or `azurite-table`/`--tableHost` to run the queue or table service instead.

Separately, you can also change the **bind address** — `0.0.0.0` (all interfaces) vs `127.0.0.1`/`localhost` (local machine only) — and the **port numbers** (`--blobPort`, `--queuePort`, `--tablePort`) independently of which service you're running.

For Blob and Table Storage, if you're using Azurite's **default ports**, this is your connection string:

"AzureWebJobsStorage": "UseDevelopmentStorage=true"


Here is a custom command to run the Azure Table Storage service on localhost, port 8000 (we're using 8000 here instead of Azurite's default 10002 just so it's obvious in class which service you're pointing at):

docker run -p 8000:8000 mcr.microsoft.com/azure-storage/azurite azurite-table --tableHost 127.0.0.1 --tablePort 8000

> ⚠️ **Important:** because this uses a non-default port, `UseDevelopmentStorage=true` will NOT work here — that shortcut only resolves to Azurite's default ports. You need the explicit connection string instead:

````
"AzureWebJobsStorage": "DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;TableEndpoint=http://127.0.0.1:8000/devstoreaccount1;
````

(That account name/key pair is the well-known Azurite dev default — safe to use locally, never in production.)

## Azure File Share
This service is not available via Azurite emulation so we need to use some of our credits on Azure. Hopefully you have not deleted your Storage Accounts. Please refer to the Azure File Share repo for instructions on how to set up a File Share and learn a thing or two about the concepts behind it. Here is the [link](https://github.com/SMahabeer07/Azure-File-Shares-CLDV6212).

Name your share `receipts-share`.

> 🔒 **Secrets note:** your File Share connection string is a real Azure credential, not a local dev default — keep it out of source control. Use `local.settings.json` (already gitignored by default in Function App templates) or environment variables.

For the "greatly exceeds" band on this section, plan for more than just upload/list/download working — you'll want file metadata tracking (size, upload date) and MIME-type validation on upload, not just a bare file write.

## Endpoints you need
- **Purchases (`Purchases` table):** create, get, get by category, update, delete.
- **Receipts (`receipts-share` File Share):** upload, list, download.

## Quick note
**Each endpoint needs to have validation** — at minimum: required fields present (e.g. no empty item name/amount), sensible data types/ranges (no negative amounts), and for file uploads, a file size limit and an allowed file type check.

Next we are going to dockerize our app — see the Dockerization guide below.

## Coding: Table Storage & File Share Functions

Below are the important snippets you'll need across your endpoints, with short explanations. These are **not** copy-paste solutions — you still need to work out the full method signatures, imports, and how these pieces fit together for each of your CRUD endpoints. Use the docs and your own testing to fill the gaps.

### Setting up a new function
Rename `Function1.cs` and change your function/class names to match what the endpoint actually does (e.g. `CreatePurchase`, `GetPurchasesByCategory`). Keep your Auth level **Anonymous** for this POE — we're not doing auth yet. You can keep GET and POST as your HTTP verbs, drop one, or swap for something else depending on what the endpoint needs to do.

### Defining your entity
```csharp
public class Expense
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public DateTime Timestamp { get; set; }
    public string ExpenseName { get; set; }
    public decimal Amount { get; set; }
    public string Category { get; set; }
}
```
This is your Table Storage row shape for a purchase. `PartitionKey` and `RowKey` together are what uniquely identify an entity in Table Storage — there's no auto-incrementing ID like SQL. Think about what makes sense as your PartitionKey (something you'll query by often — category is a strong candidate here) versus your RowKey (unique-within-partition). `Timestamp` is managed by Azure itself once the entity is written.

### Binding to the table
```csharp
[TableInput("Transactions", Connection = "AzureWebJobsStorage")] TableClient tableClient
```
This goes as an extra parameter after `HttpRequestData req`, separated by a comma — swap `"Transactions"` for `"Purchases"`. It's an **input binding** — Azure Functions hands you an already-configured `TableClient`, using whatever connection string is under the `AzureWebJobsStorage` key in your settings. You don't construct the client yourself here.

```csharp
await tableClient.CreateIfNotExistsAsync();
```
Put this early in your method body. It's what actually creates the table the first time your function runs against it — without this, your first request against a brand-new table/Azurite instance will fail.

### Reading the request body
You'll need to read the incoming HTTP body as a stream, then deserialize it. Research `StreamReader` on `req.Body` and `JsonSerializer.Deserialize<>()`. Pay attention to `JsonSerializerOptions { PropertyNameCaseInsensitive = true }` — worth having in mind when Postman's JSON casing doesn't match your C# property casing exactly.

### Building and pushing an entity
```csharp
var entity = new TableEntity(data["PartitionKey"].GetString(), data["RowKey"].GetString());
```
```csharp
await tableClient.UpsertEntityAsync(entity);
```
`TableEntity` here is being constructed from raw parsed JSON rather than a strongly-typed class — that's one valid approach, but you could also deserialize straight into your own `Purchase : ITableEntity` class. `UpsertEntityAsync` will **insert** if the PartitionKey/RowKey combo doesn't exist yet, or **replace** it if it does — that's different from `AddEntityAsync`, which throws if the entity already exists. Decide deliberately which one fits create vs. update in your CRUD set.

### Returning a response
```csharp
var response = req.CreateResponse(HttpStatusCode.OK);
await response.WriteStringAsync("Purchase added.");
return response;
```
Remember: **every endpoint needs validation before you get here**, and the rubric specifically wants proper status codes — `400 Bad Request` for bad input, `404 Not Found` where relevant — not just a blanket `200 OK` for everything.

### Querying with an optional filter
```csharp
var queryParams = System.Web.HttpUtility.ParseQueryString(req.Url.Query);
string partitionFilter = queryParams.Get("partition");
```
This is how you read a query string parameter like `?category=Groceries` off the URL — this is your "get by category" endpoint.

```csharp
string safeFilter = partitionFilter.Replace("'", "''");
queryResults = tableClient.QueryAsync<TableEntity>(
    filter: $"PartitionKey eq '{safeFilter}'"
);
```
Two things to notice: the OData filter string syntax (`PropertyName eq 'value'`) is how you filter Table Storage queries, and `.Replace("'", "''")` is escaping single quotes so a stray `'` in the input can't break (or inject into) the filter — the same idea as SQL injection defenses, adapted for OData. Don't skip this on your own filtered queries.

```csharp
await foreach (var entity in queryResults)
```
`QueryAsync<T>` returns an `AsyncPageable<T>`, not a plain list — hence `await foreach` rather than a normal `foreach`. Look up why this exists (paging behaviour under the hood) if you haven't hit `IAsyncEnumerable` before.

### Shaping the response as clean JSON
```csharp
var dict = new Dictionary<string, object>
{
    ["PartitionKey"] = entity.PartitionKey,
    ["RowKey"] = entity.RowKey,
    ["Timestamp"] = entity.Timestamp?.UtcDateTime.ToString("o")
};
```
This converts each `TableEntity` into a plain dictionary before serializing, rather than serializing the `TableEntity` object directly — the rubric's "clean DTO mapping" criterion is pointing at exactly this kind of thing. Worth deciding whether you want a proper DTO class instead of a raw dictionary for tidier code.

### File Share: choosing your write method
```csharp
string mountPath = Environment.GetEnvironmentVariable("FILE_SHARE_MOUNT_PATH");
bool isRunningInAzure = !string.IsNullOrEmpty(mountPath) && Directory.Exists(mountPath);
```
This checks at runtime whether it's running somewhere with the File Share **mounted as a local drive** (only happens when deployed in Azure with a mount configured) versus running **locally**, and branches accordingly:

- **In Azure (mounted):** plain `System.IO` — `File.WriteAllTextAsync(targetFilePath, content)` — because the file share just looks like a normal folder.
- **Locally:** the `ShareClient` from `Azure.Storage.Files.Shares`, talking to the File Share directly over HTTPS, since there's no local mount to write through.

Decide whether you actually need this dual-path branching for `receipts-share`, or whether you'll just use `ShareClient` consistently — that depends on how you're deploying and testing. Either way, look at `ShareClient`, `GetRootDirectoryClient()`, and `GetFileClient()` to understand how a file reference is built up before you `CreateAsync()` and `UploadAsync()` against it. For the "exceeds" band, also look at capturing file size and upload timestamp as metadata, and validating MIME type before accepting an upload.

## Dockerization Guide

This POE is marked on a **standalone container** (no Docker Compose) that's built, tagged, and published to Docker Hub. Follow these steps exactly.

### 1. Add a Dockerfile
In your Azure Functions project root, create a `Dockerfile` that packages your **compiled** C# project on the official Azure Functions base runtime. You guys use .NET 10 so it should look something like this:

````
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy the project file and restore dependencies
COPY ["MyApp.csproj", "."]
RUN dotnet restore

# Copy the rest of the source code and publish the application
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app

# Copy the published output from the build stage
COPY --from=build /app/publish .

# Set the entry point
ENTRYPOINT ["dotnet", "MyApp.dll"]
````

### 2. Build and tag your image
Tag it for your **public** Docker Hub repository, using semantic versioning (`:v1.0`):

docker build -t <dockerhub_username>/budget-tracker-functions:v1.0 .


### 3. Push to Docker Hub

docker login
docker push <dockerhub_username>/budget-tracker-functions:v1.0

Double-check the repo is set to **public** on Docker Hub, and confirm the version tag shows up on your repo page — that's part of what gets marked.

### 4. Run your function container standalone
No Compose — run it independently, passing environment variables so it can reach your Azurite container:

docker run -p 7071:80 --network host -e AzureWebJobsStorage="UseDevelopmentStorage=true" <dockerhub_username>/budget-tracker-functions:v1.0

> The `--network host` flag is what lets `UseDevelopmentStorage=true` (which points at `127.0.0.1` on Azurite's default ports) actually resolve — without it, `127.0.0.1` inside your function container refers to the container itself, not your Azurite container. Make sure Azurite is already running before you run this.

Test your endpoints against `http://localhost:7071/...` via Postman as before.

## Postman Collection & Testing Suite

This is scored separately from the Docker section, so treat it as its own deliverable:

- Build **one Postman collection** covering every Table Storage endpoint (create, get, get by category, update, delete) and every File Share endpoint (upload, list, download).
- Use a Postman **environment** with a `{{baseUrl}}` variable — no hardcoded URLs in any request. This lets you flip between local (`http://localhost:7071`) and any deployed URL without editing every request.
- Organize requests into **folders** (e.g. "Purchases", "Receipts") rather than one flat list.
- Include a **sample request body** for every request that needs one (create/update especially) — don't leave students to guess the JSON shape from scratch each time.
- Add basic **automated tests** per request (Postman's Tests tab) — even simple checks like status code `200`/`201` and expected fields present in the response — and save them as part of the collection, not just run once manually.
- Export the finished collection (and environment) as JSON files to submit alongside your code.

## Version Control
Minimum 5 commits **per student**, not per group — and generic messages like "fix" or "test" won't be scored in "Greatly exceeds the required standard". Aim for descriptive, feature-based commits (e.g. `feat: add get purchases by category endpoint`).

Also keep in mind that we are using feature branches and merging into main at the end when our backend has been completed which makes 5 commits a person very easy regardless of group size.
