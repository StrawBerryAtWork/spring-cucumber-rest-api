# Test your REST API with Spring, Cucumber and Gherkin !

<div align="center">
  <a name="logo" href="https://www.redfroggy.fr"><img src="assets/logo.png" alt="RedFroggy"></a>
  <h4 align="center">A RedFroggy project</h4>
</div>
<br/>
<div align="center">
  <a href="https://forthebadge.com"><img src="https://forthebadge.com/images/badges/fuck-it-ship-it.svg"/></a>
  <a href="https://forthebadge.com"><img src="https://forthebadge.com/images/badges/built-with-love.svg"/></a>
<a href="https://forthebadge.com"><img src="https://forthebadge.com/images/badges/made-with-java.svg"/></a>
</div>
<div align="center">
  <a href="https://circleci.com/gh/RedFroggy/spring-cucumber-rest-api"><img src="https://circleci.com/gh/RedFroggy/spring-cucumber-rest-api.svg?style=svg"/></a>
</div>
<br/>
<br/>

Made with Spring, [Cucumber](https://cucumber.io/) and [Gherkin](https://cucumber.io/docs/gherkin/) !
Inspired from the awesome [apickli project](https://github.com/apickli/apickli) project.

## Stack
- Spring Boot
- Cucumber
- Jayway JsonPath
- Gherkin

## Installation
- Run `npm install` to add commitlint and husky libs

## Installation
- Run `npm install` to add commitlint and husky libs

## Description
- Predefined steps
- Handle RESTFUL http requests
- Possibility to set request headers or parameters
- Possibility to test response headers
- Possibility to test response status code
- Possibility to test the body response using a json path

## Demo & Example

![Spring Cucumber Gherkin Demo](assets/demo.gif)

You can look at the [users.feature](src/test/resources/features/users.feature) file for a more detailed example.

## Share data between steps
- You can use the following step to store data from a json response body to a shared context:
```gherkin
And I store the value of body path $.id as idUser in scenario scope
```
- You can use the following step to store data from a response header to a shared context:
```gherkin
And I store the value of response header Authorization as authHeader in scenario scope
```
- The result of the JsonPath `$.id` will be stored in the `idUser` variable.
- To reuse this variable in another step, you can do:
```gherkin
When I DELETE /users/`$idUser`
And I set Authorization header to `$authHeader`
```


## How to use it in my existing project ?

### Add a CucumberTest  file

```java
@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = {"pretty"},
        features = "src/test/resources/features",
        glue = {"fr.redfroggy.bdd.glue"})
public class CucumberTest {

}
````
- Set the glue property to  `fr.redfroggy.bdd.glue"` and add your package glue.
- Set your `features` folder property
- Add your `.feature` files under your `features` folder
- In your `.feature` files you should have access to all the steps defined in the [DefaultRestApiBddStepDefinition](src/main/java/fr/redfroggy/bdd/glue/DefaultRestApiBddStepDefinition.java) file.


### Add default step definition file
It is mandatory to have a class annotated with `@CucumberContextConfiguration` to be able to run the tests.
This class must be in the same `glue` package that you've specified in the `CucumberTest` class.

```java
@CucumberContextConfiguration
@SpringBootTest(classes = Application.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class DefaultStepDefinition {

}
````

### Specify an authentication mode
- You can authenticate using the step: `I authenticate with login/password (.*)/(.*)` but the authentication mode must be implemented by you.
- You need to create a class that implements the `BddRestTemplateAuthentication` interface. 
- You can inject a `TestRestTemplate` instance in your code, so you can do pretty much anything you want.
- For example, for a JWT authentication you can do :
```java
@Component
public class JwtRestTemplateAuthentication implements BddRestTemplateAuthentication {

    @Autowired(required = false)
    private TestRestTemplate restTemplate;

    @Override
    public TestRestTemplate authenticate(String login, String password) {
        String token = generateJwt();
        restTemplate.getRestTemplate().getInterceptors().add(
            (outReq, bytes, clientHttpReqExec) -> {
                outReq.getHeaders().set(
                    HttpHeaders.AUTHORIZATION, token
                );
                return clientHttpReqExec.execute(outReq, bytes);
            });

        return restTemplate;
    }
}
```
- For a basic authentication, you can do :
```java
@Component
public class BasicAuthAuthentication implements BddRestTemplateAuthentication {

    final TestRestTemplate template;

    public BasicAuthAuthentication(TestRestTemplate template) {
        this.template = template;
    }

    @Override
    public TestRestTemplate authenticate(String login, String password) {
        return this.template.withBasicAuth(login, password);
    }
}
```

## Run local unit tests

````bash
$ mvn test
````
