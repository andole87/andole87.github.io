---
published: true
layout: single
title: "Spring Boot 파일 업로드"
category: TIL
comments: true
---

사용자가 업로드한 파일을 서버에 저장해봤다. 원래는 `S3`에 바로 넣으려고 했는데... `S3` AccessKey를 아직 못 받았다ㅠㅠ  
스프링이 기본 제공하는 api는 뭔가 부족해 보여 몇가지 방법을 동원했다.    
참고로 `common-fileupload`같은 라이브러리를 사용하지 않고, `ArgumentResolover`를 사용한다.

## MultipartFile
HTML `form` 태그는 클라이언트에서 서버로 데이터를 전송할 수 있게 한다.  
로그인처럼, 문자를 전송할 수도 있지만 파일도 전송할 수 있다.

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
    <input type="file" name="data">
    <input type="submit">
</form>
```
이 폼은 사용자가 선택한 파일을 `data`키에 넣어 **/upload**로 `POST` 요청을 보낸다.  

서블릿에서 이 파일을 받아볼 수 있다.  
`MultipartHttpServletRequest` 타입으로 정의되어 있는데, `HttpServletRequest`를 확장한다.  

스프링 부트는 기본으로 `MultipartFile`을 제어할 수 있게 해준다.  

간단한 예제 코드를 보자.
```java
@Controller
@RequestMapping("/upload")
public class FileUploadController {

    @GetMapping
    public String uploadPage() {
        return "upload";
    }

    @PostMapping
    public String uploadFile(@RequestParam("data") MultipartFile file) {
        file.transferTo(new File(...));
        return "redirect:/upload";
    }
}
```

`/upload`에 `Post`요청이 들어오면, 폼 데이터 안의 `data` 키의 값을 `MultipartFile`로 가져온다.  
이후 `MultipartFile`이 제공하는 몇가지 메서드를 사용할 수 있다.  

## 문제점
- hashing file name
`MultipartFile`을 `File`로 저장하는 과정에서 해시가 필요하다고 생각했다.   
사용자가 올리는 파일을 저장할 때 이름을 그대로 사용하면 충돌의 가능성이 매우 높아진다.   
사용자가 친절히 파일 이름을 `username-yyyy-mm-dd-checksum.jpg`로 바꾸어 올리진 않으니까.   
A도 `image.jpg`를 업로드하고 B도 `image.jpg`를 업로드하면 어떡할건데?  

그래서 `ArgumentResolver`를 사용해봤다.

### Object Mapping

위 문제점을 해결할, 파일을 관리하는 객체를 생각해보자.  
`MultipartFile`을 포장하고 몇가지 메서드를 제공하면 되겠다. 

```java
public class UploadFile {
    private MultipartFile file;
    private String extension;

    public UploadFile(MultipartFile file) {
        this.file = file;
        this.extension = getExtension(file.getOriginalFilename());
    }

    private String getExtension(Strubg originName) {
        int lastIndexOf = originName.lastIndexOf(".");
        if (lastIndexOf == -1) {
            logger.warn("확장자가 없는 파일이네여");
            return "";
        }
        return originName.substring(lastIndexOf);
    }

    public boolean save(String path) {
        try {
            file.transferTo(new File(path + "/" + hashName() + "." + extension));
            return true;
        } catch (IOException e) {
            logger.warn("IOException " + e.getMessage());
            return false;
        }
    }

    private String hashName() {
        String originName = file.getOriginalFilename();
        if (originName == null) {
            throw new IllegalArgumentException();
        }
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            md.update(originName.getBytes("UTF-8"), 0, fileName.length());
            return new BigInteger(1, md.digest()).toString(16);
        } catch (NoSuchAlgorithmException e) {
            logger.warn("MD5 알고리즘이 없다", e);
            return originName;
        } catch (UnsupportedEncodingException e) {
            logger.warn("인코딩 실패", e);
            return originName;
        }
    }
}
```

`MultipartFile`의 원래 파일 이름을 받아, 확장자를 뽑아내고 파일명을 해싱한다.

`ArgumentResolver`로 업로드한 `MultipartFile`을 `UploadFile`로 변경해서 컨트롤러에 전달하자.

### Custom Annotation

`ArgumentResolver`를 만들기 전에, 애너테이션으로 파라미터를 정해주자.  
어노테이션 프로세싱은 필요 없고 명시적으로 의미를 부여하기 위해서다.  
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface UploadedFile {
}
```
`@UploadedFile`이 붙은 파라미터에, 위에서 만든 `UploadFile` 객체를 매핑하게 해보자.  

### ArgumentResolver

```java
@Configuration
public class UploadFileResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(UploadedFile.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, 
                                    ModelAndViewContainer mavContainer, 
                                    NativeWebRequest webRequest, 
                                    WebDataBinderFactory binderFactory) throws Exception {

        MultipartHttpServletRequest request = (MultipartHttpServletRequest) webRequest.getNativeRequest();
        MultipartFile multipartFile = request.getFile("data");
        return new UploadFile(multipartFile);
    }
}
```
폼의 `data`키에 등록된 파일을 `UploadFile`객체로 매핑해준다.  

인제 함 써볼까!

### Controller
```java
@Controller
@RequestMapping("/upload")
public class FileUploadController {
    private static final String PATH = "파일을_저장할_경로";

    @GetMapping
    public String uploadPage() {
        return "upload";
    }

    @PostMapping
    public String uploadFile(@UploadedFile UploadFile file) {
        file.save(path);
        return "redirect:/upload";
    }
}
```

해싱된 파일이 경로에 저장된다.