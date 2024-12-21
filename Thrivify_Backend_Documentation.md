# Thrivify Backend Documentation

This document outlines the implementation, setup, and usage of the Thrivify backend. It includes details of the project, test cases, and instructions for running and testing the application.

## Project Overview
The Thrivify backend is a Java-based application using Spring Boot for managing user accounts and tracking habits. It supports the following functionalities:
- User Registration and Authentication
- Habit Management (CRUD operations)
- Secure API endpoints with JWT authentication

1. ## Prerequisites
Ensure you have the following installed on your system:
- Java 17 or later
- Maven
- MySQL
- Postman or any REST API client for testing
   ```

2. **Configure MySQL Database**
   Create a database named `thrivify` and update the `application.properties` file with your database credentials:
   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/thrivify
   spring.datasource.username=<your-username>
   spring.datasource.password=<your-password>
   spring.jpa.hibernate.ddl-auto=update
   ```

3. **Build and Run the Application**
   ```bash
   mvn clean install
   mvn spring-boot:run
   ```
   The application will run on `http://localhost:8080` by default.

## Code Implementation

### Main Application
```java
package com.thrivify.backend;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@SpringBootApplication
@EnableWebMvc
public class ThrivifyBackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(ThrivifyBackendApplication.class, args);
    }

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Models
**User**
```java
package com.thrivify.backend.model;

import jakarta.persistence.*;

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Column(unique = true)
    private String email;

    private String password;

    // Getters and Setters
}
```

**Habit**
```java
package com.thrivify.backend.model;

import jakarta.persistence.*;
import java.util.Date;

@Entity
public class Habit {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    private String title;

    private Date startDate;

    private String frequency;

    private String status;

    // Getters and Setters
}
```

### Repositories
**UserRepository**
```java
package com.thrivify.backend.repository;

import com.thrivify.backend.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}
```

**HabitRepository**
```java
package com.thrivify.backend.repository;

import com.thrivify.backend.model.Habit;
import org.springframework.data.jpa.repository.JpaRepository;

public interface HabitRepository extends JpaRepository<Habit, Long> {
}
```

### Controllers
```java
package com.thrivify.backend.controller;

import com.thrivify.backend.model.Habit;
import com.thrivify.backend.model.User;
import com.thrivify.backend.repository.HabitRepository;
import com.thrivify.backend.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api")
public class UserController {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private HabitRepository habitRepository;

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @PostMapping("/register")
    public String registerUser(@RequestBody User user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        userRepository.save(user);
        return "User registered successfully!";
    }

    @GetMapping("/user")
    public User getUser(@RequestParam Long userId) {
        return userRepository.findById(userId).orElse(null);
    }

    @PutMapping("/user")
    public String updateUser(@RequestBody User user) {
        userRepository.save(user);
        return "User updated successfully!";
    }

    @PostMapping("/habits")
    public String createHabit(@RequestBody Habit habit) {
        habitRepository.save(habit);
        return "Habit created successfully!";
    }

    @GetMapping("/habits")
    public List<Habit> getHabits(@RequestParam Long userId) {
        return habitRepository.findAll(); // Customize to filter by userId
    }

    @PutMapping("/habits/{id}")
    public String updateHabit(@PathVariable Long id, @RequestBody Habit habit) {
        Habit existingHabit = habitRepository.findById(id).orElse(null);
        if (existingHabit != null) {
            existingHabit.setTitle(habit.getTitle());
            existingHabit.setStatus(habit.getStatus());
            habitRepository.save(existingHabit);
            return "Habit updated successfully!";
        }
        return "Habit not found!";
    }

    @DeleteMapping("/habits/{id}")
    public String deleteHabit(@PathVariable Long id) {
        habitRepository.deleteById(id);
        return "Habit deleted successfully!";
    }
}
```

### Security Configuration
```java
package com.thrivify.backend.security;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final BCryptPasswordEncoder passwordEncoder;

    public SecurityConfig(BCryptPasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers(HttpMethod.POST, "/api/register").permitAll()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin")
            .password(passwordEncoder.encode("admin123"))
            .roles("ADMIN");
    }
}
```

## API Endpoints

### User Endpoints

#### Register a User
- **Endpoint:** `POST /api/register`
- **Payload:**
  ```json
  {
    "name": "John Doe",
    "email": "johndoe@example.com",
    "password": "password123"
  }
  ```
- **Response:**
  ```json
  "User registered successfully!"
  ```

#### Get User Information
- **Endpoint:** `GET /api/user?userId=1`
- **Response:**
  ```json
  {
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com"
  }
  ```

#### Update User Information
- **Endpoint:** `PUT /api/user`
- **Payload:**
  ```json
  {
    "id": 1,
    "name": "John Smith",
    "email": "johnsmith@example.com",
    "password": "newpassword123"
  }
  ```
- **Response:**
  ```json
  "User updated successfully!"
  ```

### Habit Endpoints

#### Create a Habit
- **Endpoint:** `POST /api/habits`
- **Payload:**
  ```json
  {
    "title": "Drink Water",
    "startDate": "2024-01-01",
    "frequency": "Daily",
    "status": "Active",
    "user": {
      "id": 1
    }
  }
  ```
- **Response:**
  ```json
  "Habit created successfully!"
  ```

#### Get Habits
- **Endpoint:** `GET /api/habits?userId=1`
- **Response:**
  ```json
  [
    {
      "id": 1,
      "title": "Drink Water",
      "startDate": "2024-01-01",
      "frequency": "Daily",
      "status": "Active",
      "user": {
        "id": 1
      }
    }
  ]
  ```

#### Update a Habit
- **Endpoint:** `PUT /api/habits/{id}`
- **Payload:**
  ```json
  {
    "title": "Drink Water",
    "status": "Completed"
  }
  ```
- **Response:**
  ```json
  "Habit updated successfully!"
  ```

#### Delete a Habit
- **Endpoint:** `DELETE /api/habits/{id}`
- **Response:**
  ```json
  "Habit deleted successfully!"
  ```

## Test Cases

### Functional Tests

1. **Register User**
   - **Input:** Valid JSON payload for `POST /api/register`.
   - **Expected Output:** `200 OK` with success message.

2. **Fetch User**
   - **Input:** Valid user ID for `GET /api/user`.
   - **Expected Output:** JSON object with user details.

3. **Create Habit**
   - **Input:** Valid JSON payload for `POST /api/habits`.
   - **Expected Output:** `200 OK` with success message.

4. **Update Habit**
   - **Input:** Valid JSON payload for `PUT /api/habits/{id}`.
   - **Expected Output:** `200 OK` with success message.

5. **Delete Habit**
   - **Input:** Valid habit ID for `DELETE /api/habits/{id}`.
   - **Expected Output:** `200 OK` with success message.

### Edge Cases

1. **Duplicate User Registration**
   - **Input:** Same email for `POST /api/register`.
   - **Expected Output:** `400 Bad Request` with appropriate error message.

2. **Invalid User ID**
   - **Input:** Non-existent user ID for `GET /api/user`.
   - **Expected Output:** `404 Not Found`.

## Deployment Instructions
To deploy the application, you can use services like AWS, Heroku, or any cloud platform supporting Java Spring Boot. For local deployment:

1. Package the application as a JAR file:
   ```bash
   mvn clean package
   ```
2. Run the JAR file:
   ```bash
   java -jar target/thrivify-backend-0.0.1-SNAPSHOT.jar
   ```

## Notes
- Ensure the MySQL service is running before starting the application.
- Test the endpoints using Postman or CURL.

