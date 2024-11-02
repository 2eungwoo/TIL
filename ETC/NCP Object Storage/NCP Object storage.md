### Access-Key, Secret-Key 발급

![image](https://github.com/user-attachments/assets/81d2ac8b-a1f2-4578-a66f-8a3c5061209a)

### Object Storage 생성

![image](https://github.com/user-attachments/assets/35a01699-d2ba-4a61-beb9-cc6caee62b06)


### Application.yml

```json
ncp:
  object-storage:
    endpoint: https://kr.object.ncloudstorage.com
    region: kr-standard
    access-key: {access-key}
    secret-key: {secret-key}
    bucket: {storage-name}

```

### Config

```java
@Configuration
public class NCPStorageConfig {

    @Value("${ncp.object-storage.region}")
    private String region;

    @Value("${ncp.object-storage.endpoint}")
    private String endpoint;

    @Value("${ncp.object-storage.access-key}")
    private String accessKey;

    @Value("${ncp.object-storage.secret-key}")
    private String secretKey;

    @Bean
    public S3Client objectStorageClient() {
        AwsBasicCredentials credentials = AwsBasicCredentials.create(accessKey, secretKey);
        return S3Client.builder()
                .credentialsProvider(StaticCredentialsProvider.create(credentials))
                .endpointOverride(URI.create(endpoint))
                .region(Region.of(region))
                .build();
    }

}
```

### StorageService

```java

@Service
@RequiredArgsConstructor
public class StorageService {

    private final S3Client s3Client;

    @Value("${ncp.object-storage.bucket}")
    private String bucket;

    /**
     * S3에 업로드
     */
    public String uploadFile(String key, MultipartFile multipartFile) throws IOException {
        s3Client.putObject(
                PutObjectRequest.builder()
                        .bucket(bucket)
                        .key(key)
                        .contentType(multipartFile.getContentType())
                        .build(),
                RequestBody.fromInputStream(multipartFile.getInputStream(), multipartFile.getSize())
        );
        // return -> file URL
        return "https://kr.object.ncloudstorage.com/" + bucket + "/" + key;
    }

    /**
     * S3에서 파일 다운로드
     */
    public byte[] downloadFile(String key) throws IOException {
        return s3Client.getObject(
                GetObjectRequest.builder()
                        .bucket(bucket)
                        .key(key)
                        .build()
        ).readAllBytes();
    }

    /**
     * S3에서 파일 삭제
     */
    public void deleteFile(String key) {
        s3Client.deleteObject(
                DeleteObjectRequest.builder()
                        .bucket(bucket)
                        .key(key)
                        .build()
        );
    }
}
```

### StorageController

```java

@RestController
@RequestMapping("/test")
@RequiredArgsConstructor
public class StorageController {

    private final StorageService storageService;

    /**
     * 파일 업로드 API
     */
    @PostMapping("/storage/upload")
    public String uploadFile(@RequestPart("file") MultipartFile multipartFile) throws Exception {
        String filename = multipartFile.getOriginalFilename();
        storageService.uploadFile(filename, multipartFile);
        return "File uploaded successfully!";
    }

    /**
     * 파일 다운로드 API
     */
    @GetMapping("/storage/download/{filename}")
    public byte[] downloadFile(@PathVariable String filename) throws Exception {
        return storageService.downloadFile(filename);
    }

    /**
     * 파일 삭제 API
     */
    @DeleteMapping("/storage/delete/{filename}")
    public String deleteFile(@PathVariable String filename) {
        storageService.deleteFile(filename);
        return "File deleted successfully!";
    }
}

```
