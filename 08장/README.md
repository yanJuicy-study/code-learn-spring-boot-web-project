# chapter08

File Upload

spring boot의 파일 업로드 설정은 크게 두 가지가 있다.

1. 별도의 파일 업로드 라이브러리 사용(commons fileupload)
2. Servlet 3버전 부터 추가된 자체 라이브러리 사용

스프링부트 내장 톰캣으로 실행한다면 [application.properties](http://application.properties) 설정만으로 가능하다.

```java
spring.servlet.multipart.enable=true
spring.servlet.multipart.location=C:\\upload
spring.servlet.multipart.max-request-size=30MB
spring.servlet.multipart.max-file-size=10MB
```

spring.servlet.multipart.enable : 파일 업로드 가능 여부
spring.servlet.multipart.location : 업로드된 파일 임시 저장 경로
spring.servlet.multipart.max-request-size : 한 번에 업로드 될 최대 용량
spring.servlet.multipart.max-file-size : 파일 하나당 최대 크기

```java
public void uploadFile(MultipartFile[] uploadFiles) {
	for(MultipartFile uploadFile : uploadFiles) {
		String originName = uploadFile.getOriginFilename();
		String fileName = originName.substring(originName.lastIndexOf("\\") + 1);
		// 파일의 절대경로에서 파일 이름만 가져옴. IE, Edge는 전체 경로가 들어옴.
		// 경로없이 파일명만 받는다면
		// String fileName = originName.substring(originName.lastIndexOf("."));
	}
}
```

업로드된 파일을 저장한다.

application.properties에 설정값을 추가하고

```java
org.zerock.upload.path=C:\\upload
```

```java
@Value("${org.zerock.upload.path}")
private String uploadPath;
```

설정한 경로를 사용한다.

파일을 저장할 때 고려해야 하는 문제가 있다.

1 . 동일한 이름의 파일

이름이 동일한 파일이 업로드되면 기존 파일은 제거된다.

이를 방지하기 위해 고유값을 사용하는데,

일반적으로 시간값을 추가하거나 UUID를 사용한다.

2 . 한 폴더에 많은 파일

한 개의 폴더에는 저장할 수 있는 파일의 양이 제한되어 있다.

(FAT32 포맷 형식의 폴더는 65543개)

일반적으로 년/월/일 폴더를 생성하여 분배한다.

3 . 확장자 체크

첨부파일 중 ‘쉘 스크립트'형식의 파일을 업로드하여 공격하는 기법이 있기 때문에

확장자명을 검사하는 과정이 있어야 한다.

대부분의 웹 사이트들은 기본적으로 썸네일을 가지고 있다.

썸네일 이미지의 처리는 다음 과정으로 처리된다.

1. 업로드된 파일을 저장하고 썸네일 파일을 생성한다.
2. 썸네일 파일은 s_등의 플래그로 일반 파일과 구분한다.

build.gradle에 Thumbnailator를 추가한다.

(http://github.com/coobird/thumbnailator)

```java
compile group: 'net.coobird', name: 'thumbnailator', version: '0.4.12'
```

업로드된 파일을 삭제할 때에는 파일의 URL을 사용한다.

URL자체가 년/월/일/uuid_파일명 의 형식이므로 이를 이용하여 파일의 위치를 찾을 수 있다.

단, 썸네일을 생성했다면 썸네일 파일도 같이 삭제해야 한다.
