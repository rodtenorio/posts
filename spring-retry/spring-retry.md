# Spring Retry

## 1. Overview

Spring Retry provides the ability to automatically re-invoke a failed operation.

We’ll explore the various ways to utilize [Spring Retry](https://docs.spring.io/spring-framework/reference/core/resilience.html), using annotations.

## 2. Setting up Maven Dependencies

**Add the *spring-retry* dependency** into our *build.gradle* file:

```json
dependencies {
    implementation 'org.springframework.retry:spring-retry:2.0.13'
    implementation 'org.springframework.boot:spring-boot-starter-aspectj'
}
```

Further, the *spring-boot-starter-aspectj* dependency is crucial because:

- It automatically configures Spring Aspect-Oriented Programming (AOP).
- The *@ConcurrencyLimit* annotation relies on AOP proxies to intercept method calls and apply the throttling logic.

Check for the latest versions of the [spring-retry](https://central.sonatype.com/artifact/org.springframework.retry/spring-retry) and [spring-boot-starter-aspectj](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-aspectj).

## 3. Enabling it

To enable Spring Retry in an application, **we need to add the *@EnableRetry* annotation** to our *@Configuration* class:

```java
@Configuration
@EnableRetry
public class MyConfig { ... }
```

## 4. Using it

### 4.1. *@Retryable* Without Recovery

**We can use the *@Retryable* annotation to add retry functionality to methods**:

```java
@Service
public interface MyService {

    @Retryable a
    void retryService();
}
```

Since we haven’t specified any exceptions here, it attempts a retry for all the exceptions. Per *@Retryable*‘s default behavior, it **may retry up to two times, with a delay of one second between retries**. Further, the maximum number of attempts is *3*, which includes the initial failed call and two subsequent retries. If the maximum number of attempts is reached and there’s still an exception, it throws the *ExhaustedRetryException*.

### 4.2. *@Retryable* and** ***@Recover*

**Let’s now add a recovery method using the *@Recover* annotation**:

```java
@Service
public interface MyService {

    @Retryable(retryFor = SQLException.class)
    void retryServiceWithRecovery(String sql) throws SQLException;

    @Recover
    void recover(SQLException e, String sql);
}
```

Here, it attempts a retry when *SQLException* is thrown. **The *@Recover* annotation defines a separate recovery method when a *@Retryable* method fails with a specified exception**.

Consequently, if the *retryServiceWithRecovery* method keeps throwing *SqlException* after three attempts, it calls the *recover()* method.

The recovery handler should have the first parameter of type *Throwable* (optional) and the same return type. The remaining arguments are populated from the argument list of the failed method in the same order.

## 5. Using Listeners

Listeners provide additional callbacks upon retries. We can also utilize these for various cross-cutting concerns across different retry attempts.

### 6.1. Adding Callbacks

The *RetryListener* interface provides the callbacks:

```java
public class DefaultListenerSupport extends RetryListenerSupport {
    
    @Override
    public <T, E extends Throwable> void close(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onClose");
        //...
        super.close(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> void onError(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onError"); 
        //...
        super.onError(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> boolean open(RetryContext context,
      RetryCallback<T, E> callback) {
        logger.info("onOpen");
        //...
        return super.open(context, callback);
    }
}
```

The *open* and *close* callbacks come before and after the entire retry, while *onError* applies to the individual *RetryCallback* calls.

### 6.2. Registering the Listener

We registered our listener *DefaultListenerSupport* to our *RetryTemplate* bean using the *withListener(new DefaultListenerSupport())* method earlier.

## 7. **Conclusion**

We saw how to use Spring Retry using annotations and how it can make a code resilient to failures or unforseen outages.
