# Integrating `@Component` and `@Cacheable` Annotations into the Delegate Class

This guide explains how to introduce the `@Component` and `@Cacheable` annotations in a delegate class for improved Spring Boot functionality.

## Adding `@Component` Annotation

To register the delegate class as a Spring-managed component, follow these steps:

### 1. Import `@Component` at apiDelegate.mustache
Add the import statement to include the required annotation:
```bash
...Other Config ignored
import org.springframework.stereotype.Component;

{{#useBeanValidation}}
import {{javaxPackage}}.validation.constraints.*;
import {{javaxPackage}}.validation.Valid;
{{/useBeanValidation}}
...Other Config ignored
```
### 2. Annotate the Delegate Interface at apiDelegate.mustache
   Place the @Component annotation just before defining the delegate interface
```bash
...Other Config ignored

/**
 * A delegate to be called by the {@link {{classname}}Controller}}.
 * Implement this interface with a {@link org.springframework.stereotype.Service} annotated class.
 */
{{>generatedAnnotation}}
@Component
public interface {{classname}}Delegate {
{{#jdk8-default-interface}}

...Other Config ignored
```
## Implementing @Cacheable Annotation
To enable caching for specific operations, follow the steps below.
### 1. Update employeeInfo.yml
Define caching configurations by updating the employeeInfo.yml file:
```yml
   x-spring-cacheable:
   name: fetchEmployeeInfoCache
```
now employeeInfo.yml look like
```yaml
---
fetch-employee-info:
  get:
    summary: Get Employee Information
    operationId: fetchEmployeeInfo
    x-spring-cacheable:
      name: fetchEmployeeInfoCache
```
### 2. Modify apiDelegate.mustache
Edit the Mustache template to integrate caching logic.

1. Import @Cacheable
Ensure that the @Cacheable annotation is included conditionally:
    ```mustache
    {{^vendorExtensions.x-spring-cacheable}}
    import org.springframework.cache.annotation.Cacheable;
    {{/vendorExtensions.x-spring-cacheable}}
    ```
2. Apply @Cacheable to the Method
Configure caching parameters dynamically within the method declaration:
    ```mustache
    {{#vendorExtensions.x-spring-cacheable}}
    @Cacheable(
    cacheNames={{#name}}"{{.}}"{{/name}}{{^name}}"default"{{/name}},
    unless={{#unless}}"{{.}}"{{/unless}}{{^unless}}"#result == null || #result.size() <= 0"{{/unless}},
    keyGenerator={{#keyGenerator}}"{{.}}Generator"{{/keyGenerator}}{{^keyGenerator}}"default"{{/keyGenerator}}
    )
    {{/vendorExtensions.x-spring-cacheable}}
    ```

## Final Delegate Implementation Example
Below is the expected final structure of the delegate class after applying both annotations:
```java

@Component
public interface EmployeeInfoControllerApiDelegate {

    /**
     * GET /api/v1/employee/info/{sid} : Get Employee Information
     *
     * @param sid Employee SID (required)
     * @return Employee information response
     * @see EmployeeInfoControllerApi#fetchEmployeeInfo
     */
    @Cacheable(
        cacheNames = "fetchEmployeeInfoCache",
        unless = "#result == null || #result.size() <= 0",
        keyGenerator = "default"
    )
    ResponseEntity<Void> fetchEmployeeInfo(String sid);
}
```

By following these steps, you ensure that the delegate class is properly managed by Spring and efficiently caches API results for optimized performance.


