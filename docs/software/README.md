# Реалізація інформаційного та програмного забезпечення


## SQL-скрипт для створення початкового наповнення бази даних

```sql
-- -----------------------------------------------------
-- Встановлення конфігурації
-- -----------------------------------------------------
SET client_min_messages TO WARNING;
-- Створення бази даних
DROP SCHEMA IF EXISTS mydb CASCADE;
CREATE SCHEMA mydb;
SET search_path TO mydb;
-- -----------------------------------------------------
-- Таблиця Role
-- -----------------------------------------------------
CREATE TABLE Role (
    id SERIAL PRIMARY KEY,
    roleName VARCHAR(45) NOT NULL,
    permission VARCHAR(45) NOT NULL
);
-- -----------------------------------------------------
-- Таблиця User
-- -----------------------------------------------------
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    firstName VARCHAR(45) NOT NULL,
    lastName VARCHAR(45) NOT NULL,
    email VARCHAR(45) NOT NULL UNIQUE,
    password VARCHAR(45) NOT NULL
);
-- -----------------------------------------------------
-- Таблиця Media
-- -----------------------------------------------------
CREATE TABLE Media (
    id SERIAL PRIMARY KEY,
    title VARCHAR(45) NOT NULL,
    keywords VARCHAR(45) NOT NULL,
    createdAt DATE NOT NULL,
    updatedAt DATE NOT NULL,
    userId INT NOT NULL REFERENCES Users(id) ON DELETE NO ACTION ON UPDATE NO ACTION
);
-- -----------------------------------------------------
-- Таблиця Admin
-- -----------------------------------------------------
CREATE TABLE Admin (
    id SERIAL PRIMARY KEY,
    name VARCHAR(45) NOT NULL
);
-- -----------------------------------------------------
-- Таблиця CommentModeration
-- -----------------------------------------------------
CREATE TABLE CommentModeration (
    commentId SERIAL PRIMARY KEY,
    userId INT NOT NULL REFERENCES Users(id) ON DELETE NO ACTION ON UPDATE NO ACTION,
    moderatorId INT NOT NULL REFERENCES Admin(id) ON DELETE NO ACTION ON UPDATE NO ACTION,
    moderationReason VARCHAR(45),
    moderationDate DATE NOT NULL,
    moderationStatus VARCHAR(45) NOT NULL
);
-- -----------------------------------------------------
-- Таблиця DeleteAccount
-- -----------------------------------------------------
CREATE TABLE DeleteAccount (
    id SERIAL PRIMARY KEY,
    userId INT NOT NULL REFERENCES Users(id) ON DELETE NO ACTION ON UPDATE NO ACTION,
    reason VARCHAR(45),
    date DATE NOT NULL,
    type VARCHAR(45) NOT NULL,
    description VARCHAR(45),
    adminId INT NOT NULL REFERENCES Admin(id) ON DELETE NO ACTION ON UPDATE NO ACTION
);
-- -----------------------------------------------------
-- Таблиця UserRole
-- -----------------------------------------------------
CREATE TABLE UserRole (
    userId INT NOT NULL REFERENCES Users(id) ON DELETE NO ACTION ON UPDATE NO ACTION,
    roleId INT NOT NULL REFERENCES Role(id) ON DELETE NO ACTION ON UPDATE NO ACTION,
    PRIMARY KEY (userId, roleId)
);
-- -----------------------------------------------------
-- Вставка даних у таблиці
-- -----------------------------------------------------
INSERT INTO Role (roleName, permission) VALUES
('Admin', 'Full Access'),
('Editor', 'Edit Content'),
('Viewer', 'View Content'),
('Moderator', 'Manage Comments'),
('Contributor', 'Submit Content');
INSERT INTO Users (firstName, lastName, email, password) VALUES
('John', 'Doe', 'john.doe@example.com', 'password123'),
('Jane', 'Smith', 'jane.smith@example.com', 'securepassword'),
('Alice', 'Johnson', 'alice.johnson@example.com', 'mypassword'),
('George', 'Joestar', 'george.joestar@example.com', 'bestpassword'),
('Nicole', 'Tesla', 'nicole.tesla@example.com', 'cringepassword');
INSERT INTO Media (title, keywords, createdAt, updatedAt, userId) VALUES
('test.png', 'image', '2018-07-12', '2020-08-12', 1),
('Metallica.mp3', 'rock,music,guitar', '2019-07-12', '2020-08-12', 2),
('message.txt', 'text', '2019-03-07', '2020-03-07', 3),
('recipe.mp4', 'video,cooking', '2019-01-01', '2020-01-01', 4),
('test.png', 'image', '2018-06-11', '2020-08-12', 5);
INSERT INTO Admin (name) VALUES
('Super Admin'),
('Moderator Andryi'),
('Moderator Boris'),
('Deleted Admin 1'),
('Deleted Admin 2');
INSERT INTO CommentModeration (userId, moderatorId, moderationReason, moderationDate, moderationStatus) VALUES
(1, 2, 'Inappropriate Language', '2020-08-12', 'Removed'),
(2, 1, 'Spam', '2024-11-13', 'Flagged'),
(3, 3, 'Off-topic', '2020-08-12', 'Removed'),
(4, 4, 'Hate Speech', '2019-01-01', 'Banned'),
(5, 5, 'Misleading Info', '2020-08-12', 'Under Review');
INSERT INTO DeleteAccount (userId, reason, date, type, description, adminId) VALUES
(3, 'Privacy Concerns', '2024-11-14', 'Permanent', 'User requested account deletion', 1),
(2, 'Inactive Account', '2024-11-13', 'Temporary', 'Account marked as inactive', 2),
(1, 'Too Many Emails', '2024-11-10', 'Temporary', 'User opted for temporary deactivation', 3),
(4, 'Security Issues', '2024-10-01', 'Permanent', 'Security concerns raised by user', 4),
(5, 'Other', '2024-09-15', 'Permanent', 'No specific reason provided', 5);
INSERT INTO UserRole (userId, roleId) VALUES
(1, 1),
(2, 2),
(3, 3),
(4, 4),
(5, 5);
COMMIT;
```

## RESTfull сервіс для управління даними

### Підключення до бази данних
```java
spring.application.name=demo
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/workdb
spring.datasource.username=postgres
spring.datasource.password=A-z228816
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

```
### Entity

### User
```java

package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;

@Data
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstname;

    private String lastname;

    private String email;

    private String password;
}
```
### Role
```java

package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;

@Data
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column (name = "rolename")
    private String roleName;

    private String permission;
}
```
### Media
```java

package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;

import java.time.LocalDate;

@Data
@Entity
@Table(name = "media")
public class Media {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String keywords;
    @Column(name = "createdat")
    private LocalDate createdAt;
    @Column(name = "updatedat")
    private LocalDate updatedAt;

    @ManyToOne
    @JoinColumn(name = "userid", nullable = false)
    private User user;
}
```
### Repository

### UserRepository
```java
package com.example.demo.repository;
import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
public interface UserRepository extends JpaRepository<User, Long> {
}
```
### RoleRepository
```java
package com.example.demo.repository;
import com.example.demo.entity.Role;
import org.springframework.data.jpa.repository.JpaRepository;
public interface RoleRepository extends JpaRepository<Role, Long> {
}
```
### MediaRepository
```java
package com.example.demo.repository;
import com.example.demo.entity.Media;
import org.springframework.data.jpa.repository.JpaRepository;
public interface MediaRepository extends JpaRepository<Media, Long> {
}
```
### Service 

### UserService
```java
package com.example.demo.service;
import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import java.util.List;
@Service
public class UserService {
    private final UserRepository userRepository;
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    public User createUser(User user) {
        return userRepository.save(user);
    }
    public User getUserById(Long id) {
        return userRepository.findById(id).orElseThrow(() -> new RuntimeException("User not found"));
    }
    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id);
        user.setFirstname(userDetails.getFirstname());
        user.setLastname(userDetails.getLastname());
        user.setEmail(userDetails.getEmail());
        user.setPassword(userDetails.getPassword());
        return userRepository.save(user);
    }
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```
### RoleService
```java
package com.example.demo.service;
import com.example.demo.entity.Role;
import com.example.demo.repository.RoleRepository;
import org.springframework.stereotype.Service;
import java.util.Optional;
@Service
public class RoleService {
    private final RoleRepository roleRepository;
    public RoleService(RoleRepository roleRepository) {
        this.roleRepository = roleRepository;
    }
    public Role createRole(Role role) {
        return roleRepository.save(role);
    }
    public Role getRoleById(Long id) {
        return roleRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Role not found with id: " + id));
    }
    public Role updateRole(Long id, Role roleDetails) {
        Role role = getRoleById(id);
        role.setRoleName(roleDetails.getRoleName());
        role.setPermission(roleDetails.getPermission());
        return roleRepository.save(role);
    }
    public void deleteRole(Long id) {
        roleRepository.deleteById(id);
    }
} 
```
### MediaService
```java
package com.example.demo.service;
import com.example.demo.entity.Media;
import com.example.demo.entity.User;
import com.example.demo.repository.MediaRepository;
import com.example.demo.repository.UserRepository;
import org.springframework.stereotype.Service;
import java.util.List;
@Service
public class MediaService {
    private final MediaRepository mediaRepository;
    private final UserRepository userRepository;
    public MediaService(MediaRepository mediaRepository, UserRepository userRepository) {
        this.mediaRepository = mediaRepository;
        this.userRepository = userRepository;
    }
    public List<Media> getAllMedia() {
        return mediaRepository.findAll();
    }
    public Media createMedia(Media media, Long userId) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));
        media.setUser(user);
        return mediaRepository.save(media);
    }
    public Media getMediaById(Long id) {
        return mediaRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Media not found"));
    }
    public Media updateMedia(Long id, Media mediaDetails, Long userId) {
        Media media = getMediaById(id);
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));
        media.setTitle(mediaDetails.getTitle());
        media.setKeywords(mediaDetails.getKeywords());
        media.setCreatedAt(mediaDetails.getCreatedAt());
        media.setUpdatedAt(mediaDetails.getUpdatedAt());
        media.setUser(user);
        return mediaRepository.save(media);
    }
    public void deleteMedia(Long id) {
        mediaRepository.deleteById(id);
    }
} 
```
### Controller
### UserController
```java
package com.example.demo.controller;
import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    }
    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User userDetails) {
        return ResponseEntity.ok(userService.updateUser(id, userDetails));
    }
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```
### RoleController
```java
package com.example.demo.controller;
import com.example.demo.entity.Role;
import com.example.demo.service.RoleService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
@RestController
@RequestMapping("/api/roles")
public class RoleController {
    private final RoleService roleService;
    public RoleController(RoleService roleService) {
        this.roleService = roleService;
    }
    @PostMapping
    public Role createRole(@RequestBody Role role) {
        return roleService.createRole(role);
    }
    @GetMapping("/{id}")
    public ResponseEntity<Role> getRoleById(@PathVariable Long id) {
        return ResponseEntity.ok(roleService.getRoleById(id));
    }
    @PutMapping("/{id}")
    public ResponseEntity<Role> updateRole(@PathVariable Long id, @RequestBody Role roleDetails) {
        return ResponseEntity.ok(roleService.updateRole(id, roleDetails));
    }
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteRole(@PathVariable Long id) {
        roleService.deleteRole(id);
        return ResponseEntity.noContent().build();
    }
}
```
### MediaController
```java
package com.example.demo.controller;
import com.example.demo.entity.Media;
import com.example.demo.service.MediaService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;
@RestController
@RequestMapping("/api/media")
public class MediaController {
    private final MediaService mediaService;
    public MediaController(MediaService mediaService) {
        this.mediaService = mediaService;
    }
    @GetMapping
    public List<Media> getAllMedia() {
        return mediaService.getAllMedia();
    }
    @PostMapping("/{userId}")
    public Media createMedia(@RequestBody Media media, @PathVariable Long userId) {
        return mediaService.createMedia(media, userId);
    }
    @GetMapping("/{id}")
    public ResponseEntity<Media> getMediaById(@PathVariable Long id) {
        return ResponseEntity.ok(mediaService.getMediaById(id));
    }
    @PutMapping("/{id}/{userId}")
    public ResponseEntity<Media> updateMedia(@PathVariable Long id, @RequestBody Media mediaDetails, @PathVariable Long userId) {
        return ResponseEntity.ok(mediaService.updateMedia(id, mediaDetails, userId));
    }
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteMedia(@PathVariable Long id) {
        mediaService.deleteMedia(id);
        return ResponseEntity.noContent().build();
    }
}
```
### Main Class for Spring Boot Application 
```java
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
