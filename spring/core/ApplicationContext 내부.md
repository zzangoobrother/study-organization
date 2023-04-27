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

