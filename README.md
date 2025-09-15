# UploadFile_MinIO

```csharp
private readonly IMinioClient minioClient;

    public ExampleController(IMinioClient minioClient)
    {
        this.minioClient = minioClient;
    }

var endpoint = "localhost:9000";
var accessKey = "minioadmin";
var secretKey = "minioadmin";
builder.Services.AddMinio(accessKey, secretKey);
```


```csharp
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file)
{
    string bucketName = "ubuntu";
    var fileName = "abc123.png";
    if (!await _minioClient.BucketExistsAsync(new BucketExistsArgs().WithBucket(bucketName)))
    {
        await _minioClient.MakeBucketAsync(new MakeBucketArgs().WithBucket(bucketName));
    }


    // Upload file
    using var stream = file.OpenReadStream();
    await _minioClient.PutObjectAsync(new PutObjectArgs()
        .WithBucket(bucketName)
        .WithObject(fileName)
        .WithStreamData(stream)
        .WithObjectSize(stream.Length)
        .WithContentType(file.ContentType)
    );
    // Trả về URL ảnh công khai
    var imageUrl = $"http://localhost:9000/{bucketName}/{fileName}";
    await _fileRepo.AddAync(bucketName, fileName, fileName);
    return Ok(new { url = imageUrl });
}
```


# GET: http://localhost:5084/api/upload/display
```csharp
[HttpGet("display")]
public async Task<IActionResult> DownloadFileAsFormFileAsync()
{
    // Create an async task for listing buckets.
    var getListBucketsTask = await _minioClient.ListBucketsAsync().ConfigureAwait(false);

    // Iterate over the list of buckets.
    foreach (var bucket in getListBucketsTask.Buckets)
    {
        Console.WriteLine(bucket.Name + " " + bucket.CreationDateDateTime);
    }

    var fileName = "abc123.png";
    var memStream = new MemoryStream();

    // Cấu hình GetObjectArgs
    var getObjectArgs = new GetObjectArgs()
        .WithBucket("ubuntu")
        .WithObject(fileName)
        .WithCallbackStream(stream =>
        {
            // Copy dữ liệu vào memory stream
            stream.CopyTo(memStream);
        });

    // Tải file từ MinIO
    await _minioClient.GetObjectAsync(getObjectArgs);
    
    memStream.Position = 0;
    return File(memStream, "image/png", fileDownloadName: fileName);
}
```
