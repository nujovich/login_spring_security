# Garaje Code Pills: Login paso a paso con Spring Security 6

Esta aplicación es un ejemplo de cómo configurar una aplicación web utilizando Spring Web y Spring Security. La aplicación cuenta con dos controladores: uno para rutas públicas y otro para rutas privadas. La seguridad está configurada para permitir el acceso a ciertas rutas de forma pública, mientras que otras requieren autenticación y/o autorización específica.

## Requisitos

- JDK 17 o superior
- Maven 3.6 o superior
- Una base de datos configurada (H2, MySQL, PostgreSQL, etc.)
- Spring Web
- Spring Security
- Spring Data JPA
- Lombok

## Instalación

1. Clonar el repositorio:
    ```sh
    git clone https://github.com/nujovich/login_spring_security.git
    cd tu_repositorio
    ```

2. Configurar la base de datos en el archivo `application.properties` o `application.yml`:
    ```properties
    spring.datasource.url=jdbc:mysql://localhost:3306/tu_base_de_datos
    spring.datasource.username=tu_usuario
    spring.datasource.password=tu_contraseña
    spring.jpa.hibernate.ddl-auto=update
    ```

3. Ejecutar la aplicación:
    ```sh
    mvn spring-boot:run
    ```

## Estructura del Proyecto

- **src/main/java/com/tuempresa/tuapp**: Contiene el código fuente principal.
  - **controller**: Contiene los controladores para las rutas públicas y privadas.
    - `PublicController.java`
    - `HomeController.java`
  - **config**: Contiene la configuración de seguridad.
    - `SecurityConfig.java`
  - **entity**: Contiene las entidades JPA.
    - `User.java`
  - **repository**: Contiene los repositorios JPA.
    - `UserRepository.java`
  - **service.impl**: Contiene los servicios de negocio.
    - `UserDetailsServiceImpl.java`
  - **JpaLoginApplication.java**: Clase principal para ejecutar la aplicación.

## Configuración de Seguridad

La configuración de seguridad se encuentra en la clase `SecurityConfig.java`. Aquí está el código de configuración utilizado:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.requestMatchers("/public/**").permitAll()
                        .requestMatchers("/v1/home").authenticated()
                        .requestMatchers("/v1/admin").hasAuthority("ADMIN").anyRequest().authenticated()
                )
                .formLogin(Customizer.withDefaults())
                .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Descripción de la Configuración
- csrf(AbstractHttpConfigurer::disable): 
Deshabilita la protección CSRF.

- authorizeHttpRequests: 
Configura las autorizaciones de las rutas. Las rutas que coinciden con /public/** son permitidas para todos. Las rutas que coinciden con /v1/home requieren autenticación. Las rutas que coinciden con /v1/admin requieren que el usuario tenga la autoridad ADMIN.

- formLogin(Customizer.withDefaults()): Habilita el formulario de inicio de sesión por defecto.
- passwordEncoder(): Define un codificador de contraseñas utilizando BCrypt.

### Controladores
- PublicController: Maneja las rutas públicas que no requieren autenticación.

```java
@RestController
@RequestMapping("/public")
public class PublicController {

    @GetMapping("/home")
    public String home() {
        return "Public Home";
    }
}
PrivateController
Maneja las rutas privadas que requieren autenticación y/o autorización.

```java
@RestController
@RequestMapping("/v1")
@RequiredArgsConstructor
public class HomeController {

    @GetMapping("/home")
    public String home() {
        return "Private Home";
    }

    @GetMapping("/admin")
    @PreAuthorize("hasAuthority('ADMIN')")
    public String admin() {
        return "Admin";
    }
}
```

### Autenticación con JPA
La autenticación se realiza mediante JPA, utilizando una tabla user en la base de datos. La entidad User y el repositorio UserRepository son los responsables de manejar los datos del usuario.

#### User Entity

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role;

}
```

#### UserRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```
