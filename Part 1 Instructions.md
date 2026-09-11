# POE Part 1 guidelines

## What you need to have

- A working trigger functions app with Blob and Azure table storage running of the azurite emulator.
- Validated endpoints
- An extensive postman collection covering all tests with an exported postman collection.json on GitHub (create requests in named folders, keep it neat and when you are done right click and export. Click the + to add requests).
- A Docker Image and container that works of the port you exposed it to in your Dockerfile
- A docker file (right click on the function app and look for container support)
- A tagged and pushed docker image for your backend logic.
- A video (5 to 15 minutes long)
- A detailed readme with setup, references, commands, endpoint docs and a section detailing what each team member has worked on.

<img width="1917" height="1020" alt="image" src="https://github.com/user-attachments/assets/f3eccd7d-47cf-402f-bcf9-169026da3757" />


<img width="1866" height="971" alt="image" src="https://github.com/user-attachments/assets/1381f1ec-c598-4085-8e85-c8d735a78b65" />


<img width="1917" height="1020" alt="image" src="https://github.com/user-attachments/assets/01fb426c-9303-4599-90b8-8a71e9676650" />


<img width="1917" height="1020" alt="image" src="https://github.com/user-attachments/assets/3a51be62-0527-4c45-9e02-95572b1d3934" /> 

For connection strings in code:
<img width="1120" height="276" alt="image" src="https://github.com/user-attachments/assets/2dabfacd-e01a-42bc-b563-06dbbc026cf7" />



## Video Instructions

### Steps

- Start by making sure you have tagged your docker image and pushed it to DockerHub
- In the video start from DockerHub showing your tagged image in a public repo
- You can pull this image in Docker Desktop and run it locally or just continue using the one you have (the tagged one). I would pull a new one and run it (make sure the others are not running to avoid conflicts with port numbers).
- Once you have pulled and successfully run your image you can show it working by going to containers and clicking the link that takes you to your Functions 4.0 app
- From there you can go to Visual Studio to give a quick breakdown of your trigger functions and discuss localsettings.json.
- If you have pushed it talk about why it is not best practice but also not an issue for this assignment (using azurite so same conn str across the board).
- Next you can go to postman and speak about the tests you have done while demoing a few of them individually to demonstrate that they work.
- Show your variables and talk about how each folder is meant to test different services etc.
- For Azure you can do a post and then a get for the same right in postman but for blob you can execute the test in postman and show the file in Azure Storage Explorer.
- After all tests have run successfully and you have shown everything you need to show you can do a quick GitHub walkthrough to close out the video.
- Talk about collaboration and show evidence in your commit history.
- Show your postman collection on GitHub in its folder, show off the readme (this is our readme which contains everything you need to know about setting up this project locally with the appropriate software and a breakdown of what each group member worked on. You can also find detailed documentation about our endpoints).
- You should be able to finish the video here

### Commits

- Make sure to make up the 5 commits per collaborator minimum for a good mark

---

## README Instructions

For the readme:

- Detail the setup for VS 2026, docker desktop (all steps including getting WSL), where to go to get a dockerhub account, postman (how to create and get started with a collection).
- A "what we each worked on" section.
- An overview of what technologies you have used (tech stack).
- Why Azurite Emulation.
- Write about the localsettings.json file (why it is not a security risk if you pushed it or why the connection string is in the code -> Use Dev storage is default and does not present a security risk).
- Note that the aforementioned is not best practice and you should leave it out completely.
- The gitignore takes care of this you can just do a section on how to do a localsettings by using yours as an example in the readme.
- We need references of course (at the end).
- Endpoint documentation, testing (how to test on postman by importing your collection), how to build the dockerfile in VS 2026, running of your PC vs running of docker.

## Commands
### Packages to download:


````
Microsoft.Azure.Functions.Worker
Microsoft.Azure.Functions.Worker.Sdk
Microsoft.Azure.Functions.Worker.Extensions.Http
Microsoft.Azure.Functions.Worker.Extensions.Tables
Azure.Data.Tables
Azure.Storage.Blobs
````
````
dotnet add package Microsoft.Azure.Functions.Worker
dotnet add package Microsoft.Azure.Functions.Worker.Sdk
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Http
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Tables
dotnet add package Azure.Data.Tables
dotnet add package Azure.Storage.Blobs
````

````
docker build -t poe_part1:v1 .
docker run -d -p 8080:8000 --name poe_part1 poe_part1:v1
docker tag poe_part1:v1 <username_dockerhub>/poe_part1:v1
docker push <username_dockerhub>/poe_part1:v1
````

````
docker run -d -p 7071:80 -e AzureWebJobsStorage="DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://host.docker.internal:10000/devstoreaccount1;QueueEndpoint=http://host.docker.internal:10001/… --name poe_part1_dev guts024/poe_part1_dev:v1
````


This will paste incorrectly. It should look like this:

<img width="1917" height="1017" alt="image" src="https://github.com/user-attachments/assets/4dba130a-47b1-40a3-9dd8-9db92fb2434e" />


Do 7071 instead of 8080 for this last one
- How to tag and push + commands (all commands can be found on Teams or in my readmes)
- Validate all endpoints with helper methods for bad requests, not found, internal server errors etc.
- Document how to get Azurite and run it. Maybe you can demo this in your video


## From the budget app

# Code Snippets — BudgetApp Functions

## BudgetFunctions.cs (Table Storage)

### Function Class Setup & Table Constant

```csharp
public class BudgetFunctions
{
    private readonly ILogger _logger;
    private const string TableName = "BudgetItems";

    public BudgetFunctions(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<BudgetFunctions>();
    }
```

---

### POST /api/budget — Create Budget Item

```csharp
[Function("CreateBudgetItem")]
public async Task<HttpResponseData> CreateBudgetItem(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "budget")] HttpRequestData req,
    [TableInput(TableName, Connection = "AzureWebJobsStorage")] TableClient tableClient)
{
    await tableClient.CreateIfNotExistsAsync();
```

**JSON deserialization with error handling:**

```csharp
    string body = await new StreamReader(req.Body).ReadToEndAsync();
    BudgetItemCreateRequest? data;

    try
    {
        data = JsonSerializer.Deserialize<BudgetItemCreateRequest>(body,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
    }
    catch (JsonException)
    {
        return await BadRequest(req, "Request body is not valid JSON.");
    }
```

**Validation checks:**

```csharp
    if (data is null) return await BadRequest(req, "Request body is required.");
    if (string.IsNullOrWhiteSpace(data.TransactionId)) return await BadRequest(req, "TransactionId is required.");
    if (string.IsNullOrWhiteSpace(data.Category)) return await BadRequest(req, "Category is required.");
    if (data.Amount <= 0) return await BadRequest(req, "Amount must be greater than 0.");
```

**Duplicate check (404 catch = doesn't exist yet):**

```csharp
    try
    {
        await tableClient.GetEntityAsync<BudgetItem>(data.Category, data.TransactionId);
        return await BadRequest(req, $"Item '{data.TransactionId}' already exists in '{data.Category}'.");
    }
    catch (RequestFailedException ex) when (ex.Status == 404) { }
```

**Entity creation (PartitionKey = Category, RowKey = TransactionId) & 201 response:**

```csharp
    var item = new BudgetItem
    {
        PartitionKey = data.Category,
        RowKey = data.TransactionId,
        TransactionId = data.TransactionId,
        Category = data.Category,
        Description = data.Description,
        Amount = data.Amount
    };

    await tableClient.AddEntityAsync(item);

    var response = req.CreateResponse(HttpStatusCode.Created);
    await response.WriteAsJsonAsync(item);
    return response;
}
```

---

### GET /api/budget — Get All Items

```csharp
[Function("GetAllBudgetItems")]
public async Task<HttpResponseData> GetAllBudgetItems(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "budget")] HttpRequestData req,
    [TableInput(TableName, Connection = "AzureWebJobsStorage")] TableClient tableClient)
{
    await tableClient.CreateIfNotExistsAsync();

    var items = new List<BudgetItem>();
    await foreach (var item in tableClient.QueryAsync<BudgetItem>())
    {
        items.Add(item);
    }

    var response = req.CreateResponse(HttpStatusCode.OK);
    await response.WriteAsJsonAsync(items);
    return response;
}
```

---

### GET /api/budget/category/{category} — Filter by Category

**Note the OData injection guard (`'` → `''`):**

```csharp
[Function("GetBudgetItemsByCategory")]
public async Task<HttpResponseData> GetBudgetItemsByCategory(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "budget/category/{category}")] HttpRequestData req,
    [TableInput(TableName, Connection = "AzureWebJobsStorage")] TableClient tableClient,
    string category)
{
    await tableClient.CreateIfNotExistsAsync();

    string safeCategory = category.Replace("'", "''");
    var query = tableClient.QueryAsync<BudgetItem>(filter: $"PartitionKey eq '{safeCategory}'");

    var results = new List<BudgetItem>();
    await foreach (var item in query)
    {
        results.Add(item);
    }

    var response = req.CreateResponse(HttpStatusCode.OK);
    await response.WriteAsJsonAsync(results);
    return response;
}
```

---

### PUT /api/budget/{category}/{id} — Update Item

```csharp
[Function("UpdateBudgetItem")]
public async Task<HttpResponseData> UpdateBudgetItem(
    [HttpTrigger(AuthorizationLevel.Anonymous, "put", Route = "budget/{category}/{id}")] HttpRequestData req,
    [TableInput(TableName, Connection = "AzureWebJobsStorage")] TableClient tableClient,
    string category,
    string id)
{
    await tableClient.CreateIfNotExistsAsync();

    string body = await new StreamReader(req.Body).ReadToEndAsync();
    BudgetItemUpdateRequest? data;

    try
    {
        data = JsonSerializer.Deserialize<BudgetItemUpdateRequest>(body,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
    }
    catch (JsonException)
    {
        return await BadRequest(req, "Invalid JSON payload.");
    }

    if (data is null || data.Amount <= 0) return await BadRequest(req, "Valid Amount (>0) is required.");
```

**Fetch, mutate, replace (ETag.All = unconditional overwrite):**

```csharp
    try
    {
        var existingResponse = await tableClient.GetEntityAsync<BudgetItem>(category, id);
        var item = existingResponse.Value;

        item.Description = data.Description;
        item.Amount = data.Amount;

        await tableClient.UpdateEntityAsync(item, ETag.All, TableUpdateMode.Replace);

        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(item);
        return response;
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        return await NotFound(req, $"Budget item '{id}' in category '{category}' was not found.");
    }
}
```

---

### DELETE /api/budget/{category}/{id}

```csharp
[Function("DeleteBudgetItem")]
public async Task<HttpResponseData> DeleteBudgetItem(
    [HttpTrigger(AuthorizationLevel.Anonymous, "delete", Route = "budget/{category}/{id}")] HttpRequestData req,
    [TableInput(TableName, Connection = "AzureWebJobsStorage")] TableClient tableClient,
    string category,
    string id)
{
    await tableClient.CreateIfNotExistsAsync();

    try
    {
        await tableClient.DeleteEntityAsync(category, id);
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(new { message = $"Successfully deleted item '{id}'." });
        return response;
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        return await NotFound(req, $"Item '{id}' in category '{category}' not found.");
    }
}
```

---

### Shared Helper Methods (400 / 404)

```csharp
private static async Task<HttpResponseData> BadRequest(HttpRequestData req, string message)
{
    var response = req.CreateResponse(HttpStatusCode.BadRequest);
    await response.WriteAsJsonAsync(new { error = message });
    return response;
}

private static async Task<HttpResponseData> NotFound(HttpRequestData req, string message)
{
    var response = req.CreateResponse(HttpStatusCode.NotFound);
    await response.WriteAsJsonAsync(new { error = message });
    return response;
}
```

---

## ReceiptFunctions.cs (Blob Storage)

### Function Class Setup & Container Constant

```csharp
public class ReceiptFunctions
{
    private readonly ILogger _logger;
    private const string ContainerName = "receipts";

    public ReceiptFunctions(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<ReceiptFunctions>();
    }
```

---

### POST /api/receipts/upload?fileName=... — Upload Receipt

**Query string parsing for fileName:**

```csharp
[Function("UploadReceipt")]
public async Task<HttpResponseData> UploadReceipt(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "receipts/upload")] HttpRequestData req)
{
    var queryParams = System.Web.HttpUtility.ParseQueryString(req.Url.Query);
    string? fileName = queryParams.Get("fileName");

    if (string.IsNullOrWhiteSpace(fileName))
        return await BadRequest(req, "Provide a 'fileName' query parameter, e.g. ?fileName=receipt.pdf");
```

**Read raw binary body into MemoryStream (for size validation + rewind):**

```csharp
    using var memoryStream = new MemoryStream();
    await req.Body.CopyToAsync(memoryStream);

    if (memoryStream.Length == 0)
        return await BadRequest(req, "Request body is empty - attach raw binary file.");

    memoryStream.Position = 0;
```

**Upload to blob (overwrite: true) & return 201 with metadata:**

```csharp
    var containerClient = await GetBlobContainerClientAsync();
    var blobClient = containerClient.GetBlobClient(fileName);

    await blobClient.UploadAsync(memoryStream, overwrite: true);

    var response = req.CreateResponse(HttpStatusCode.Created);
    await response.WriteAsJsonAsync(new
    {
        fileName,
        sizeInBytes = memoryStream.Length,
        uploadedAtUtc = DateTime.UtcNow,
        blobUri = blobClient.Uri.ToString()
    });
    return response;
}
```

---

### GET /api/receipts — List All Blobs

```csharp
[Function("ListReceipts")]
public async Task<HttpResponseData> ListReceipts(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "receipts")] HttpRequestData req)
{
    var containerClient = await GetBlobContainerClientAsync();
    var blobs = new List<object>();

    await foreach (BlobItem blob in containerClient.GetBlobsAsync())
    {
        blobs.Add(new
        {
            name = blob.Name,
            sizeInBytes = blob.Properties.ContentLength,
            createdOn = blob.Properties.CreatedOn
        });
    }

    var response = req.CreateResponse(HttpStatusCode.OK);
    await response.WriteAsJsonAsync(blobs);
    return response;
}
```

---

### GET /api/receipts/download/{fileName} — Download Blob

**Exists check + streaming download (no memory buffering):**

```csharp
[Function("DownloadReceipt")]
public async Task<HttpResponseData> DownloadReceipt(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "receipts/download/{fileName}")] HttpRequestData req,
    string fileName)
{
    var containerClient = await GetBlobContainerClientAsync();
    var blobClient = containerClient.GetBlobClient(fileName);

    if (!await blobClient.ExistsAsync())
        return await NotFound(req, $"Receipt '{fileName}' was not found.");

    BlobDownloadStreamingResult downloadResult = await blobClient.DownloadStreamingAsync();

    var response = req.CreateResponse(HttpStatusCode.OK);
    response.Headers.Add("Content-Type", downloadResult.Details.ContentType ?? "application/octet-stream");
    await downloadResult.Content.CopyToAsync(response.Body);

    return response;
}
```

---

### Blob Container Client Factory (Azurite default)

**Key snippet for the README's localsettings/Azurite discussion — falls back to `UseDevelopmentStorage=true`:**

```csharp
private static async Task<BlobContainerClient> GetBlobContainerClientAsync()
{
    string connectionString = Environment.GetEnvironmentVariable("AzureWebJobsStorage")
        ?? "UseDevelopmentStorage=true";

    var containerClient = new BlobContainerClient(connectionString, ContainerName);
    await containerClient.CreateIfNotExistsAsync(PublicAccessType.None);
    return containerClient;
}
```

---

### Duplicate Helpers in ReceiptFunctions

> Same `BadRequest` and `NotFound` helpers appear here as static methods.

```csharp
private static async Task<HttpResponseData> BadRequest(HttpRequestData req, string message)
{
    var response = req.CreateResponse(HttpStatusCode.BadRequest);
    await response.WriteAsJsonAsync(new { error = message });
    return response;
}

private static async Task<HttpResponseData> NotFound(HttpRequestData req, string message)
{
    var response = req.CreateResponse(HttpStatusCode.NotFound);
    await response.WriteAsJsonAsync(new { error = message });
    return response;
}
```

---

## Quick Reference Table for README/Video

| Function | Method | Route | Storage | Success | Error Cases |
|---|---|---|---|---|---|
| `CreateBudgetItem` | POST | `/api/budget` | Table | 201 | 400 (bad JSON, missing fields, duplicate) |
| `GetAllBudgetItems` | GET | `/api/budget` | Table | 200 | — |
| `GetBudgetItemsByCategory` | GET | `/api/budget/category/{category}` | Table | 200 | — |
| `UpdateBudgetItem` | PUT | `/api/budget/{category}/{id}` | Table | 200 | 400 (bad JSON/amount), 404 |
| `DeleteBudgetItem` | DELETE | `/api/budget/{category}/{id}` | Table | 200 | 404 |
| `UploadReceipt` | POST | `/api/receipts/upload?fileName=` | Blob | 201 | 400 (no fileName, empty body) |
| `ListReceipts` | GET | `/api/receipts` | Blob | 200 | — |
| `DownloadReceipt` | GET | `/api/receipts/download/{fileName}` | Blob | 200 | 404 |

### Key Talking Points for the Video

- **`AuthorizationLevel.Anonymous`** — no auth required for demo/assignment.
- **`TableInput` binding** with `Connection = "AzureWebJobsStorage"` — points to Azurite locally.
- **`CreateIfNotExistsAsync()`** called on every request — idempotent safety for fresh Azurite instances.
- **OData injection guard** — `category.Replace("'", "''")` on the filter query.
- **`ETag.All` + `TableUpdateMode.Replace`** — full replace, no optimistic concurrency.
- **`UseDevelopmentStorage=true`** — the Azurite fallback string, ties directly to your README's localsettings justification.
- **Streaming download** vs. buffered upload — good contrast to mention in the video.
- **Duplicated helpers** — worth noting as a refactor opportunity if asked.

## Handling Merge Conflicts

A merge conflict happens when Git cannot automatically combine changes from two branches, usually because the same lines in a file were edited in both branches. Do not panic. Communicate with the person who made the other changes and decide which version is correct.

### What a conflict looks like

When you open a conflicted file, you will see markers like this:

```text
<<<<<<< HEAD
// Your branch's version
=======
// Incoming branch's version
>>>>>>> feature/other-branch
```

- `<<<<<<< HEAD` starts your current branch's changes.
- `=======` separates the two versions.
- `>>>>>>> branch-name` ends the incoming branch's changes.

To resolve it, delete the markers and keep the correct code. You can keep yours, keep theirs, or combine both.

---

### Method 1: Resolve by changing the files manually

This is the most common method and works well in Visual Studio 2026, VS Code, or any editor.

1. Pull the latest changes from the target branch:
   ```bash
   git checkout your-branch
   git pull origin main
   ```

2. Git will list the conflicted files. Open each one in Visual Studio 2026, VS Code, or another editor.

3. Search for `<<<<<<<`.

4. Decide what the final code should be:
   - Keep only your changes.
   - Keep only the incoming changes.
   - Combine both if needed.

5. Delete all conflict markers:
   - `<<<<<<<`
   - `=======`
   - `>>>>>>>`

6. Save the file.

7. Test the app locally to make sure nothing broke.

8. Stage and commit the resolved files:
   ```bash
   git add .
   git commit -m "Resolve merge conflict in BudgetFunctions.cs"
   git push
   ```

---

### Method 2: Resolve directly on GitHub

GitHub can resolve conflicts directly in the browser, but this normally works best for pull requests and simple conflicts.

1. Open the pull request on GitHub.

2. If there are conflicts, GitHub shows a **Resolve conflicts** button. Click it.

3. GitHub opens a web editor showing the conflicted files with the same markers.

4. Edit the file:
   - Remove `<<<<<<<`, `=======`, and `>>>>>>>`.
   - Keep the correct code.

5. Click **Mark as resolved** for each file.

6. Once all conflicts are resolved, click **Commit merge**.

7. Finish merging the pull request.

8. If the conflict is large or complicated, resolve it locally instead.

> Note: You cannot easily run the project locally before committing when resolving on GitHub, so be careful and double-check the code.

---

### Method 3: Resolve with GitHub Desktop

1. Open **GitHub Desktop** and select your repository.

2. Click **Fetch origin**.

3. Click **Pull origin**.

4. If conflicts appear, GitHub Desktop will warn you and list the conflicted files.

5. Click **Open in Visual Studio Code** (or your preferred editor).

6. Edit each conflicted file:
   - Remove the conflict markers.
   - Keep the correct code.
   - Save the file.

7. Return to GitHub Desktop.

8. If the file is not automatically marked as resolved, right-click it and choose **Mark as resolved**.

9. Enter a commit message, for example:
   ```text
   Resolve merge conflict in ReceiptFunctions.cs
   ```

10. Click **Commit merge**.

11. Click **Push origin**.

12. Run the app and your Postman tests again to confirm everything still works.

---

### After resolving a conflict

- Always run the project locally.
- Run your Postman collection to check all endpoints.
- Check that no conflict markers remain:
  ```bash
  git grep "<<<<<<<"
  ```
- If you get stuck, you can abort the merge and ask for help:
  ```bash
  git merge --abort
  ```
- Never force push over someone else's work unless the team agrees.

---

### How to avoid merge conflicts

- Pull from the main branch often.
- Commit small changes frequently.
- Work on separate files or sections where possible.
- Communicate with your team before editing shared files.
- Merge pull requests regularly instead of letting branches get too far behind.
