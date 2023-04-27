ApplicationContext 인터페이스 내부 중 BeanFactory를 살펴본다.

````java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
}
````
![캡처123456](https://user-images.githubusercontent.com/42162127/234851530-e1e38743-f96e-487b-b9ef-fd7f5c5efa09.PNG)
UML을 보면 AnnotationConfigApplicationContext 클래스를 시작으로 보면 BeanFactory를 구현하는 곳이 
GenericApplicationContext 클래스에서 한다는걸 알 수 있습니다.
Adapter 패턴을 통해 구현합니다.

BeanFactory VS ApplicationContext

BeanFactory 는 lazy-loading ApplicationContext 은 eager-loading/pre-loading 이라 한다.

[이전 편](https://github.com/zzangoobrother/study-organization/blob/main/spring/core/1.%20IOC%20-%20Annotation%20%EA%B8%B0%EB%B0%98%20bean%20%EB%93%B1%EB%A1%9D%EC%9B%90%EB%A6%AC2.md)

이전편에서는 BeanDefinition을 가지고 있다가 getBean() 메소드를 호출하면 bean이 생성되는 것을 확인했었다.
ApplicationContext을 보자

````java
BeanFactory ctx = new AnnotationConfigApplicationContext(BeanConfiguration.class);
log.debug("before getBean()");
TestBean testBean = ctx.getBean(TestBean.class);

@Configuration
public class BeanConfiguration {
    @Bean
    public TestBean testBean() {
        return new TestBean();
    }
}

public class TestBean {
    public TestBean() {
        log.debug("TestBean init");
    }
}
````
어떤 결과나 나올까?
````java
TestBean init
before getBean()
````
bean이 생성된 후 다음 동작이 이어진다. 하나씩 보자

````java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
	this();
	register(componentClasses);
	refresh();
}
````
refresh() 메소드에 답이 있다.

````java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		....
		// Instantiate all remaining (non-lazy-init) singletons.
		finishBeanFactoryInitialization(beanFactory);
	}
}

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
	....
	
	// Instantiate all remaining (non-lazy-init) singletons.
	beanFactory.preInstantiateSingletons();
}
````

````java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
	@Override
	public void preInstantiateSingletons() throws BeansException {

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof SmartFactoryBean<?> smartFactoryBean && smartFactoryBean.isEagerInit()) {
						getBean(beanName);
					}
				}
				else {
					getBean(beanName);
				}
			}
		}
	}
}
````
refresh() -> finishBeanFactoryInitialization() -> preInstantiateSingletons()
getBean()을 호출합니다.
따라서 먼저 초기화가 출력 후 다음 메시지가 출력이 된것 입니다.
만약 finishBeanFactoryInitialization() 메소드를 주석을 하고 실행한다면

````java
before getBean()
TestBean init
````
위와 같이 나올 것이다.

결론적으로 loading 시점 차이는 존재하지 않지만 구현에 따라 달라질 뿐이다.

