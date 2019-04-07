---
layout: post
title: '스프링 프레임워크의 IoC 컨테이너'
subtitle: '빈을 등록하는 다섯가지 방법'
author: 'doorur'
date: 2019-04-07
header-img: img/in-post/spring.jpg
lang: ko
catalog: true
tags:
  - 스프링
---

### 1. IoC 컨테이너와 Bean
스프링에서는 의존성을 주입받아 사용할 수 있습니다. 한 객체에서 사용하는 의존을 객체 자체가 결정하는게 아니라 객체 바깥에서 결정되기 때문에 제어의 역전(Inversion of Control)이라고 불리고, IoC라는 용어로 사용되고 있습니다.  

여기서 말하는 의존이란 한 객체가 동작하기 위해 필요한 어떤 것입니다. 예를 들어 다음과 같은 코드가 있다면 BookService 객체는 BookRepository라는 객체가 있어야 동작할 수 있고, BookRepository에 변경이 일어난다면 BookService 객체의 동작에도 변경이 일어나겠죠. 이때 BookService가 BookeRepository에 의존성을 가지고있다고 말합니다.  

```java

public class BookService {
	private BookRepository bookRepository = new BookRepository();

	public Book saveBook(Book book) {
		// bookRepository.save()의 동작에 변경이 있다면 이 메소드의 동작도 변경됩니다.
		return bookRepository.save(book);
	}
}
```

스프링 프레임워크는 IoC 컨테이너를 제공해서 컨테이너 안에 Bean들을 담아두었다가 필요한 곳에 주입시켜줄 수 있습니다. 그래서 다음과 같은 코드가 가능해집니다.  

```java

@Service
public class BookService {
	@Autowired
	private BookRepository bookRepository;

	public Book saveBook(Book book) {
		return bookRepository.save(book);
	}
}
```

BookService 객체 안에서 new로 BookRepository 객체를 생성하지 않아도 BookService는 IoC컨테이너에 등록되어 있는 BookRepository 빈을 주입받아 사용할 수 있게 됩니다.  

### 2. ApplicationContext
스프링에서 제공하는 IoC 컨테이너의 구현체는 ApplicationContext 인터페이스를 구현하고 있습니다. 또 ApplicationContext 인터페이스는 BeanFactory 인터페이스를 상속받고 있기 때문에 BeanFactory 인터페이스의 메소드인 getBean 통해서 IoC 컨테이너에 등록된 빈을 꺼내와 사용할 수도 있습니다. 이때 파라미터로 넘겨주는 값은 빈의 ID입니다.  

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfig.class);
	
BookRepository bookRepository = (BookRepository)applicationContext.getBean("bookRepository");
```

### 3. Bean을 설정하는 다양한 방법

> 여기서 설명하는 다섯가지 방법은 [깃허브](https://github.com/doorur/spring-framework-core/commits/master)에서 각각 커밋으로 코드를 확인할 수 있습니다.  

스프링은 2003년도에 처음 등장하면서 많은 개발자들이 사용해왔고 사용자들의 요구에 따라 여러 부분을 개선했습니다. IoC에 Bean을 등록하는 방법도 여러가지가 있는데 어떻게 발전해 왔는지 다섯 가지 방법을 알아보겠습니다.  

#### 첫번째. xml config 파일에 빈 등록하기
xml config 파일에 프로젝트에서 사용할 모든 빈의 목록을 나열해주면 스프링 프레임워크가 이 xml 파일을 읽어서 IoC 컨테이너에 등록하는 방식입니다. 프로젝트가 커지면 커질 수록 등록해야할 빈도 많아져서 파일도 커지고 번거롭습니다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="bookService" class="me.doorur.springapplicationcontext.BookService">
        <property name="bookRepository" ref="bookRepository"/>
		  -- 여기서 name은 bookService에 있는 setter명
	  	  -- ref는 bookRepository의 beanId를 말한다
    </bean>

    <bean id="bookRepository" class="me.doorur.springapplicationcontext.BookRepository"/>
</beans>
```

#### 두번째. xml config 파일을 이용해 컴포넌트 스캔하기
xml config 파일에 빈을 일일히 선언해주지 않고 각 객체에 @Component 어노테이션을 달아두면 어플리케이션이 로딩될 때 어노테이션을 단 객체를 빈으로 등록해주는 방법입니다. @Service, @Repository, @Controller 어노테이션은 각각 @Component 어노테이션을 상속받은 어노테이션이기 때문에 상기 어노테이션을 써도 컴포넌트 스캔할 때 빈으로 등록됩니다.  

```java
@Service
public class bookService {
	...
}

... 
@Repository
public class bookRepository {
	...
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <context:component-scan base-package="com.doorur.springdemo"/>
	  -- base-package에 명시해준 패키지 밑에 있는 모든 파일을 훑어 @Component가 있는 객체를 빈으로 등록합니다

</beans>
```

이 방법은 빈을 일일히 등록해주지 않아도 되서 간편하지만 여전히 xml config 파일을 사용하는 방법입니다.  

#### 세번째. java config를 이용해서 빈 등록하기
xml config 파일을 사용하지 않고도 빈을 등록할 수 있습니다. @Configuration 어노테이션을 사용해서 자바 코드로 빈을 등록하는 방법입니다. 코드이기 때문에 IoC 컨테이너에 등록하기 전에 객체에 조작을 할 수 있어서 xml config 방식으로 빈을 등록하는 것보다 유연합니다.  

컴포넌트 스캔을 하지 않기 때문에 빈으로 등록할 객체에 @Component 어노테이션을 달아주지 않아도 됩니다.  

```java
@Configuration
public class ApplicationConfig {
	@Bean
	public BookRepository bookRepository() {
		return new BookRepository();
	}

	@Bean
	public BookService bookService() {
		BookService bookService = new BookService();
		// 이렇게 빈으로 등록하기 전 객체에 조작을 가할 수 있습니다.
		bookService.setBookRepository(bookRepository());

		return bookService;
}
```

객체를 빈으로 등록할 때 위의 BookService의 예처럼 다른 빈을 의존성으로 가지고 있는 경우가 있습니다. 그럴 때는 위와 같이 직접 의존성의 메소드를 호출해서 사용할 수도 있고, 파라미터로 넘겨받을 수도 있습니다. 다음과 같이 파라미터에 선언을 해주면 알아서 주입받아 옵니다.  

```java
@Bean
public BookService bookService(BookRepository bookRepository) {
	BookService bookService = new BookService();
	bookService.setBookRepository(bookRepository);

	return bookService;
}
```

이때 주의할 점은 IoC에 등록되어 있지 않은 객체는 파라미터로 주입받을 수 없다는 점입니다. 하지만 선언 순서는 중요하지 않으니 어디선가 빈으로 등록되어 있기만 하다면 사용할 수 있습니다. (bookService 등록 후에 bookRepository를 등록해도 동작합니다.)  

이 방법은 xml config 파일을 사용하지 않아도 되고 빈에 조작을 할 수 있어 유용하지만 여전히 빈을 일일히 등록해줘야 합니다. 번거롭죠.  

#### 네번째. java config를 이용해서 컴포넌트 스캔하기
자바 코드에서도 빈을 일일히 등록하지 않고 컴포넌트 스캔을 할 수 있습니다. @ComponentScan 어노테이션을 사용하면 @Component 어노테이션이 달린 객체들을 스캔해와서 빈으로 등록해 줍니다.  

```java
@Configuration
@ComponentScan(basePackages = "com.doorur.demo")
public class ApplicationConfig {
}
```

@Component 어노테이션을 달고 있는 클래스들을 스캔할 때는 @ComponentScan의 옵션인 basePackages에 정의 된 패키지를 기준으로 하위에 있는 클래스들을 탐색합니다. 컴포넌트를 선언한다고 해도 이 패키지 바깥에 있으면 빈으로 등록되지 않으니 이 부분을 주의해야 합니다.  

#### 다섯번째. @SpringBootApplication 어노테이션 사용하기
스프링 부트에서 제공하는 @SpringBootApplication은 @Configuration과 @ComponentScan을 포함하고 있습니다. 그래서 빈으로 등록하고자 하는 클래스에 @Component류의 어노테이션만 달아준다면 xml config 파일이나 java config 파일 없이도 IoC 컨테이너에 빈을 등록해서 사용할 수 있습니다.  

```java
@SpringBootApplication
public class DemoApplication {

}
```

### 왜 IoC 컨테이너에 빈을 등록해서 사용해야 할까?
그렇다면 객체들을 왜 자꾸 빈으로 만들어서 IoC 컨테이너에 등록해서 사용하는 걸까요? IoC 컨테이너에 등록하는 것에 여러가지 장점이 있습니다.  

가장 기본적인 이유는 변경에는 닫혀있고 확장에는 열려있는 구조를 위해서입니다. 객체지향의 5원칙(SOLID) 중에 하나인 OCP이죠. 의존성에 변경이 일어났을 때 그 객체를 사용하는 모든 부분을 수정하지 않고도 그 빈만 수정하면 됩니다. IoC로 관리하지 않았다면 객체를 new 해준 부분을 전부 수정해야합니다.  

이외에도 첫번째로 다른 빈에 주입하기 위해서입니다. 두번째로 다른 빈을 주입받기 위해서입니다. 세번째로 단위 테스트가 쉬운 코드를 만들기 위해서 입니다. 네번째로 객체를 여러개 생성할 필요 없이 한 인스턴스로만 사용하기 위해서 입니다. 다섯번째로 빈으로 등록되면 라이프사이클로 관리 되기 때문에 빈의 생성과 파괴 시점에 로직을 처리하기 용이하기 때문입니다. 기회가 된다면 이 이유들도 예제코드와 같이 블로깅 해보겠습니다.  