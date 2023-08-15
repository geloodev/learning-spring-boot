# Learning Spring Boot 🍃

This project was created to learning the basics of the Spring Boot framework while following Amigoscode’s Spring Boot for Beginners course.

### Tools
[![My Skills](https://skillicons.dev/icons?i=java,spring,docker,postgres,postman,vscode)](https://skillicons.dev)

## Java Files
The Java files follow a simple MVC-like architecture:

- **Model:** Customer.java
- **Controller:** CustomerController.java
- **Repository:** CustomerRepository.java

### Customer.java
Annotations: ```@Entity``` defines the Entity Table, ```@Id``` designates the Primary Key, ```@SequenceGenerator``` creates the PK in a sequential manner, and ```@GeneratedValue``` specifies how the PK will be generated.

```java
package com.geloodev;

import java.util.Objects;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.SequenceGenerator;

@Entity
public class Customer {

    @Id
    @SequenceGenerator(
        name = "customer_id_sequence",
        sequenceName = "customer_id_sequence",
        allocationSize = 1
    )
    @GeneratedValue(
        strategy = GenerationType.SEQUENCE,
        generator = "customer_id_sequence"
    )
    private Integer id;

    private String name;
    private String email;
    private Integer age;

    public Customer(Integer id, String name, String email, Integer age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.age = age;
    }

    public Customer() {}

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Customer customer = (Customer) o;
        return Objects.equals(id, customer.id)  &&
               Objects.equals(name, customer.name)  &&
               Objects.equals(email, customer.email)  &&
               Objects.equals(age, customer.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, email, age);
    }

    @Override
    public String toString() {
        return "Customer{" +
               "id=" + id +
               ", name='" + name + '\'' +
               ", email='" + email + '\'' +
               ", age=" + age +
               '}';

    }
}
  ```

### CustomerController.java
Annotations: ```@RestController``` designates the class as a REST Controller, ```@RequestMapping``` is specifies the path where functions will make HTTP Requests, ```@GetMapping``` sets the function to do GET Requests, and the same are to ```@PostMapping``` for POST Requests, ```@DeleteMapping``` for DELETE Requests and ```@PutMapping``` for PUT Requests. Additionally, ```@RequestBody``` maps the HTTP Request Body to a object, and ```@PathVariable``` handle template variables in the request URI and set then as method parameters.

```java
package com.geloodev;

import java.util.List;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.*;


@RestController
@RequestMapping("api/v1/customers")
public class CustomerController {
    
    private final CustomerRepository customerRepository;

    public CustomerController(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    @GetMapping
    public List<Customer> getCustomers() {
        return customerRepository.findAll();
    }

    record CustomerRequest(
        String name,
        String email,
        Integer age
    ) {}

    @PostMapping
    public void addCustomer(@RequestBody CustomerRequest request) {
        Customer customer = new Customer();
        customer.setName(request.name());
        customer.setEmail(request.email());
        customer.setAge(request.age());
        customerRepository.save(customer);
    }

    @DeleteMapping("/{customerId}")
    public void deleteCustomer(@PathVariable("customerId") Integer id) {
        customerRepository.deleteById(id);
    }

    @PutMapping("/{id}")
    public void updateCustomer(
        @PathVariable("id") Integer id,
        @RequestBody CustomerRequest request
    ) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow();
        
        customer.setName(request.name());
        customer.setEmail(request.email());
        customer.setAge(request.age());

        customerRepository.save(customer);
    }
}
```

### Customer.java
```JpaRepository``` extends the ```CrudRepository``` and receives the domain class and the ID type that it should handle.]

```java
package com.geloodev;

import org.springframework.data.jpa.repository.JpaRepository;

public interface CustomerRepository extends JpaRepository<Customer, Integer> {
    
}
```

## Other Files
### pom.xml
The Maven configuration for dependencies, plugins, etc.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.geloodev</groupId>
	<artifactId>learning-spring-boot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>learning-spring-boot</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>17</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

### docker-compose.yml
The Docker configuration to manage the PostgreSQL container.

```yaml
services:
  db:
    container_name: first-spring-boot
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      PGDATA: /data/postgres
    volumes:
      - db:/data/postgres
    ports:
      - "5433:5432"
    networks:
      - db
    restart: unless-stopped

networks:
  db:
    driver: bridge

volumes:
  db:
```
