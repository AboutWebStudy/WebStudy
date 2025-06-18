서로 다른 계정 간 S3 To S3 복사작업 하는 방법 with php laravel

다른 언어도 동일하게 copyObject API를 사용하면 가능하다.

## 사전 데이터

```jsx
aws-sdk-php
A 계정으로 만든 버킷
B 계정으로 만든 버킷
A,B 버킷에 접근할 iam 계정
```

## 사전 작업

AWS 콘솔에서 iam 계정 번호를 준비해, 사용할 버킷 A, B에 아래와 같이 your_iam_ID 위치에 권한을, your_bucket에 버킷 이름을 넣는다.

- ListBucket : 디렉토리 복사코드를 준비하기 위한 리스트 조회 권한
- GetObject : 파일을 다운로드할 수 있는 조회 권한
- PutObject : 파일을 업로드 할 수 있는 권한

해당 위치 접근 방법은 **Amazon S3 > 버킷 > 권한 > 버킷 정책**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAccountyour_iam_IDAccess",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::your_iam_ID:root"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::your_bucket",
                "arn:aws:s3:::your_bucket*"
            ]
        }
    ]
}
```

## CopyObject API 구현

CopyObject API는 파일 단위로 A에서 B로 옮기는 작업이다.
따라서 단일 건에 대해서는 권한만 잘 오픈해준다면, 어렵지 않은 작업이다.

### 단일 파일 복사 작업

```php
use Aws\S3\S3Client;

class S3ToS3Transfer{

    public function fileCopy(){
        $targetS3Client = new S3Client([
            'credentials' => [
                'key' => iam_key,
                'secret' => iam_secret_Key,
            ],
            'region' => 'ap-northeast-2',
            'version' => 'latest',
        ]);

        //source 위치에 있는 파일을 key 위치로 복사
        $targetS3Client->copyObject([
            'Bucket'     => 'target_bucket',
            'Key'        => 'target_path',
            'CopySource' => urlencode('source_bucket/' . $sourcePath),
        ]);

    }
```

CopySource의 버킷을 명시해줘야 정상 동작 가능

### 디렉토리 복사 작업

디렉토리 복사는 파일의 목록을 읽어오는 ListBucket API를 활용해야 한다.

흐름은 대략 이렇다

1. Source S3 스토리지 리스트 불러오기(ListObjectsV2 API)
2. 불러온 리스트에서 size가 0인 디렉토리 제외하고 모두 배열에 저장(1000개 이상일 경우 페이지네이션 처리)
3. foreach로 각 object를 돌며 단일 copyObject API 요청

```php
    public function directoryCopy(array $data){
        $sourceS3Client = new S3Client([
            'credentials' => [
                'key' => iam_key,
                'secret' => iam_secret_key,
            ],
            'region' => 'ap-northeast-2',
            'version' => 'latest',
        ]);

        $targetS3Client = new S3Client([
            'credentials' => [
                'key' => iam_key,
                'secret' => iam_secret_key,
            ],
            'region' => 'ap-northeast-2',
            'version' => 'latest',
        ]);

        // source S3 디렉토리 탐색 시작(1000개 단위 페이징 되어있음)
        $requestParams = [
                'Bucket' => $sourceStorageInfo->virtual_path,
                'Prefix' => $sourcePath, // 조회할 디렉토리 지정
            ];

        do{
            $objects = $sourceS3Client->listObjectsV2($requestParams);

            // 다음 페이지 존재확인 IsTruncated : 1 or 0
            if ($objects['IsTruncated']){
                $requestParams = [
                    'Bucket' => $sourceStorageInfo->virtual_path,
                    'Prefix' => $sourcePath, // 조회할 디렉토리 지정
                    'ContinuationToken' => $objects['NextContinuationToken'] // 다음 페이지 토큰
                ];

                $objectsNextPageFlag = true;

            } else {
                $objectsNextPageFlag = false;
            }

            //오브젝트 리스트 파싱
            if (isset($objects['Contents'])) {
                foreach ($objects['Contents'] as $object) {
                    // 디렉토리 오브젝트 제외
                    if ($object['Size'] == 0) {
                        continue;
                    }

                    $notIncludeDirectoryObjects[] = $object;
                }
            }
            
        } while($objectsNextPageFlag); // 페이징 끝날 때까지 반복

        //배열 전체 탐색해 target S3에 파일 복사
        foreach ($notIncludeDirectoryObjects as $object) {
            $objectKey = $object['Key']; // 버킷 제외 파일 경로
            $fileNameWithExt = basename($objectKey); // 파일명 추출

            //source 위치에 있는 파일을 key 위치로 복사
            $targetS3Client->copyObject([
                'Bucket'     => 'target_bucket',
                'Key'        => 'target_path',
                'CopySource' => urlencode('source_bucket/' . $sourcePath), // 버킷 + 소스 위치
            ]);
        }
    }
```

리스트 조회한 데이터를 그대로 복사요청 진행하면 되는 로직
