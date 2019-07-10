---
published: true
layout: single
title: "[Java] 어노테이션 프로세싱 - 어노테이션은 어떻게 동작하나?"
category: TIL
comments: true
---

미디엄에 어노테이션 프로세싱에 대해 좋은 글을 보았다. [해당 글을](https://medium.com/@jintin/annotation-processing-in-java-3621cb05343a)참고해서 구현해봤다. 우리말로 옮겨 적는다.  

# Annotation Processing in Java

자바는 다른 모던 프로그래밍 랭귀지에 비해 긴 코드를 쓰게 하는것으로 종종 평가된다. 무엇이 진실인지보다 중요한 것은, 자바의 정수를 기능적으로 향상시킬 몇 가지 방법이 있다는 것이다. 이 중에서도 어노테이션 프로세싱은 분명히 당신 마음에 들 것이다.


## What is Annotation Processing?

어노테이션은 클래스, 메서드, 파라미터에 메타 정보를 라벨링하는 태그 메커니즘이다.  
어노테이션 프로세싱은 컴파일 타임에 어노테이션을 분석한다. 그리고 나서 붙여진 어노테이션에 따라 클래스를 만들어 낸다. Here is how it works.

1. Create Annotation Classes.
2. Create Annotation Parser classes.
3. Add Annotations in your project.
4. Compile, and Annotation Parsers will handle the Annotations.
5. Auto-generated classes will added in build folder.  

> 1. 어노테이션 클래스를 생성한다.
> 2. 어노테이션 파서 클래스를 생성한다.
> 3. 어노테이션을 사용한다.
> 4. 컴파일하면, 어노테이션 파서가 어노테이션을 처리한다.
> 5. 자동 생성된 클래스가 빌드 폴더에 추가된다.

Annotation Parser classes는 **오직 프로젝트를 컴파일 할 때만** 필요하다. 빌드가 끝나면 쓰이지 않는다.

## Example

어노테이션 프로세싱을 사용해서 간단한 팩토리 패턴을 구현해 보자.  
팩토리 클래스가 개와 고양이를 만들어 준다고 가정하자.

```java
public interface Animal {
    void bark();
}

public class Dog implements Animal {
    @Override
    public void bark() { System.out.println("woof"); }
}

public class Cat implements Animal {
    @Override
    public void bark() { System.out.println("meow"); }
}

public class AnimalFactory {
    public static Animal CreateAnimal() {
        switch(tag) {
            case "dog":
                return new Dog();
            case "cat":
                return new Cat();
            default:
                throw new RuntimeException("not support animal");
        }
    }
}

public static void main(String[] args) {
    Animal dog = AnimalFactory.createAnimal("dog");
    dog.bark(); // woof
    Animal cat = AnimalFactory.createAnimal("cat");
    cat.bark(); //meow
}
```

AnimalFactory가 생산한 인스턴스가 Dog인지 Cat인지 알 필요 없다. 그러나 AnimalFactory는 너무 지저분하다. 어노테이션 프로세싱이 이것들을 자동 생산하게 해 보자.


## Create Annotation Class

```java
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface AutoFactory {
    
}

@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @class AutoElement {
    String tag();
}
```

**팩토리**와 **Real Class** 를 위한 두 개의 어노테이션을 정의했다.  

`@Retention`은 어노테이션의 라이프사이클을 정의하는 곳이다. 관련 설정으로 `SOURCE`(**컴파일에만** 사용), `CLASS`(.class 파일로 변환될), `RUNTIME`(**런타임때** 사용)이 있다. 여기서는 컴파일에만 사용할 것이므로, `SOURCE`로 설정했다. 

`@Target`은 메타 정보를 추가할 타겟의 타입의 지시자다. (@Target is a indicator of what is our target type to annotated) `TYPE`은 클래스와 인터페이스에 사용한다.

> 주) Retention은 용도, 사용처로 이해할 수 있겠고  
> Target은 어노테이션의 타겟(클래스, 메서드, 필드, 파라미터 ...)로 이해할 수 있다.   
> 자세한 사항은 package java.lang.annotation 에서 찾아볼 수 있다.  
> [미주](#source-code-docs)로 간략한 javadoc을 남겼다.  

## Create Annotation Parser

쉬운 설정을 위해 다음 라이브러리를 사용한다.

```java
compile 'com.google.auto.service:auto-service:1.0-rc4'
compile 'com.squareup:javapoet:1.10.0'
```

커스텀 어노테이션 프로세서의 기본 형식이다.

```java
@AutoService(Processor.class)
public class AutoFactoryProcesser extends AbstractProcessor {
  @Override
  public synchronized void init(ProcessingEnvironment env) { ... }

  @Override
  public Set<String> getSupportedAnnotationTypes() { ... }

  @Override
  public SourceVersion getSupportedSourceVersion() { ... }

  @Override
  public boolean process(Set<? extends TypeElement> set,          
                         RoundEnvironment env) { ... }
}
```

`AbstractProcessor`는 컴파일 파이프라인에 추가할 수 있는 플러그인이다. 소스코드를 분석하고 코드를 생산하기도 한다. `@AutoService`는 어노테이션 프로세서를 컴파일러에 바인딩하게 도와주는 구글 라이브러리다. 메서드를 하나씩 살펴보자.

```java
private Filer filer;
private Messager messager;

@Override
public synchronized void init(ProcessingEnvironment processingEnv) {
    super.init(processingEnv);
    filer = processingEnv.getFiler();
    messager = processingEnv.getMessager();
}
```

`init` 함수로 `ProcessingEnvironment`를 얻을 수 있다. `ProcessingEnvironment`는 `Filer`(파일 생성), `Messager`(로그를 위한)을 포함하고 있다. `init`함수를 추가해 주지 않으면, `proccessingEnv`변수는 부모 클래스에 보관된다.

```java
@Override
public Set<String> getSupportedAnnotationTypes() {
    Set<String> annotations = new LinkedHashSet<>();
    annotations.add(AutoFactory.class.getCanonicalName());
    annotations.add(AutoElement.class.getCanonicalName());
    return annotations;
}
```

`getSupportedAnnotationTypes`에 어떤 어노테이션이 들어갈 것인지 컴파일러에게 정의해줘야 한다.(for performance) 이 안에 위에서 만든 두개의 어노테이션(AutoFactory, AutoElement)을 추가했다.

```java	
@Override
public SourceVersion getSupportedSourceVersion() {
    return SourceVersion.latestSupported();
}
```

`getSupportedSourceVersion`은 자바 버전을 가리킨다. 특정 자바 버전을 반드시 지켜야 하는게 아니라면, 보통은 비워둔다.

```java
@Override
public boolean process(Set<? extends TypeElement> set,
                       RoundEnvironment roundEnvironment) {
  Map<ClassName, List<ElementInfo>> result = new HashMap<>();
    
    // AutoFactory 어노테이션이 붙은 모든 요소에 대해서 인터페이스가 아니면 에러
  for (Element annotatedElement : roundEnvironment.getElementsAnnotatedWith(AutoFactory.class)) {
    if (annotatedElement.getKind() != ElementKind.INTERFACE) {
      error("Only interface can be annotated with AutoFactory", annotatedElement);
      return false;
    }
      
      
      // 어노테이션 대상의 클래스네임을 HashMap에 put한다. value는 ArrayList
    TypeElement typeElement = (TypeElement) annotatedElement;
    ClassName className = ClassName.get(typeElement);
    if (!result.containsKey(className)) {
      result.put(className, new ArrayList<>());
    }
  }
    
    // AutoElement 어노테이션이 붙은 모든 요소에 대해서 클래스가 아니면 에러
  for (Element annotatedElement : roundEnvironment.getElementsAnnotatedWith(AutoElement.class)) {
    if (annotatedElement.getKind() != ElementKind.CLASS) {
      error("Only class can be annotated with AutoElement", annotatedElement);
      return false;
    }
      
      //result HashMap의 value였던 List에 클래스 이름 추가.
    AutoElement autoElement = annotatedElement.getAnnotation(AutoElement.class);
    TypeElement typeElement = (TypeElement) annotatedElement;
    ClassName className = ClassName.get(typeElement);
    List<? extends TypeMirror> list = typeElement.getInterfaces(); // ㅋㅋ 제네릭 PECS
    for (TypeMirror typeMirror : list) {
      ClassName typeName = getName(typeMirror);
        if (result.containsKey(typeName)) {
          result.get(typeName).add(new ElementInfo(autoElement.tag(), className));
          break;
        }
      }
  }
    
    // init으로 만든 filer와 위에서 만든 result로 팩토리를 만듬
  try {
    new FactoryBuilder(filer, result).generate(); 
  } catch (IOException e) {
    error(e.getMessage());
  }
  return false;
}
```

`process` 에서 어노테이션을 조작한다. 여기서는 오직 타입에 대해서만 다루었는데(리플렉션), 보통의 자바 어플리케이션에서는 이렇게 하지 않는다.

**유심히 보아야 할 것은 어노테이션 정보(이름, 패키지, 태그....)를 추출해서 `FactoryBuilder`에게 전달했다는 것이다.**

```java
class FactoryBuilder {
    ...

    public void generate() throws IOException {
        // 넘겨받은 HashMap은 <인터페이스 이름 AutoFactory, ArrayList<요소이름 AutoElement>>
      for (ClassName key : input.keySet()) {

          // 메서드를 만들어 준다. 이름은 create + 키 네임. ex) createElement
        MethodSpec.Builder methodBuilder = MethodSpec.methodBuilder("create" + key.simpleName())

            // 접근제어자는 public static
            .addModifiers(Modifier.PUBLIC, Modifier.STATIC)

            // 파라미터는 String
            .addParameter(String.class, "type")

            // 내부 로직은 switch(매개변수)
            .beginControlFlow("switch(type)");

          // switch statment 상세
        for (ElementInfo elementInfo : input.get(key)) {

            // AutoElement의 이름과 같으면 해당 <T> AutoElement를 생성
          methodBuilder
              .addStatement("case $S: return new $T()", elementInfo.tag, elementInfo.className);
        }

        // switch statment에 해당하는게 없을 경우 RuntimeException
        methodBuilder
            .endControlFlow()
            .addStatement("throw new RuntimeException(\"not support type\")")
            .returns(key);

          // 위에서 구성한 메서드를 갖는 클래스를 생성. 클래스 이름은 key + "Factory"
        MethodSpec methodSpec = methodBuilder.build();
        TypeSpec helloWorld = TypeSpec.classBuilder(key.simpleName() + "Factory")
            .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
            .addMethod(methodSpec)
            .build();
        JavaFile javaFile = JavaFile.builder(key.packageName(), helloWorld)
            .build();

        javaFile.writeTo(filer);
      }
    }
}
```

`FactoryBuilder` 내부의 `generate`메서드는 자바 코드를 생성하고, `filer`는 빌드 폴더에 `.class`파일을 생성한다.



자, 다시 메인 프로젝트로 가 보자.

```java
@AutoFactory
public interface Animal {
  void bark();
}
@AutoElement(tag = "dog")
public class Dog implement Animal {
  @Override
  public void bark() { System.out.println("woof"); }
}
@AutoElement(tag = "cat")
public class Cat implement Animal {
  @Override
  public void bark() { System.out.println("meow"); }
}
```

컴파일을 끝내고 빌드가  완료되면, 다음과 같은 코드가 만들어진다.

```java
public final class AnimalFactory {
  public static Animal createAnimal(String type) {
    switch(type) {
      case "cat": return new Cat();
      case "dog": return new Dog();
    }
    throw new RuntimeException("not support type");
  }
}
```

적절한 어노테이션 프로세싱으로, 수많은 팩토리 클래스와 Element 클래스를 유지하는 노력을 줄일 수 있다.




## Source Code docs

### Retention
```java
/**
 * Annotations are to be discarded by the compiler.
 */
SOURCE,

/**
 * Annotations are to be recorded in the class file by the compiler
 * but need not be retained by the VM at run time.  This is the default
 * behavior.
 */
CLASS,

/**
 * Annotations are to be recorded in the class file by the compiler and
 * retained by the VM at run time, so they may be read reflectively.
 *
 * @see java.lang.reflect.AnnotatedElement
 */
RUNTIME
```

### Target
```java

/** Class, interface (including annotation type), or enum declaration */
TYPE,

/** Field declaration (includes enum constants) */
FIELD,

/** Method declaration */
METHOD,

/** Formal parameter declaration */
PARAMETER,

/** Constructor declaration */
CONSTRUCTOR,

/** Local variable declaration */
LOCAL_VARIABLE,

/** Annotation type declaration */
ANNOTATION_TYPE,

/** Package declaration */
PACKAGE,

/**
 * Type parameter declaration
 *
 * @since 1.8
 */
TYPE_PARAMETER,

/**
 * Use of a type
 *
 * @since 1.8
 */
TYPE_USE
```