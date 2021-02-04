---
title: Spring容器的基本实现
date: 2021-02-04 09:34:12
tags:
	- Spring Framework
	- Spring
categories:
    - Spring
---

### Spring的容器的基本实现

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
                         http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="testSpringBean" class="com.upuphub.spring.bean.TestSpringBean"/>
</beans>
```

```java
	public static void main(String[] args) {
		BeanFactory beanFactory =new DefaultListableBeanFactory();
		BeanDefinitionReader beanDefinitionReader =new XmlBeanDefinitionReader((BeanDefinitionRegistry) beanFactory);
		beanDefinitionReader.loadBeanDefinitions(new ClassPathResource("bean/beans-spring.xml"));
		TestSpringBean testSpringBean = (TestSpringBean) beanFactory.getBean("testSpringBean");
		System.out.println(testSpringBean.getName());
	}
```

<!-- more -->

