# lobo-post1-u11

**Programación Web — Unidad 11: Buenas Prácticas y Patrones de Diseño**  
**Post-Contenido 1 — Refactorización con SOLID, DAO/DTO y @ControllerAdvice**  
Estudiante: Farid Lobo | Código: 1152338

---

## Descripción

Aplicación Spring Boot que refactoriza un catálogo de productos aplicando:

- **SRP** (Single Responsibility Principle): cada clase tiene una única responsabilidad.
- **DIP** (Dependency Inversion Principle): el controlador depende de la interfaz `ProductoService`, no de la implementación concreta.
- **Patrón DAO**: `ProductoRepository` extiende `JpaRepository`, centralizando el acceso a datos.
- **Patrón DTO**: `ProductoRequestDTO` (entrada con validaciones) y `ProductoResponseDTO` (salida sin campos sensibles).
- **Factory**: `ProductoFactory` centraliza la conversión entre entidad y DTOs.
- **@RestControllerAdvice**: `GlobalExceptionHandler` centraliza el manejo de excepciones con respuestas `ApiError` estandarizadas.

---

## Arquitectura en Capas

```
                    ┌─────────────────────────┐
                    │   ProductoController    │  ← Capa Web (REST)
                    │  @RestController        │
                    └────────────┬────────────┘
                                 │ depende de interfaz (DIP)
                    ┌────────────▼────────────┐
                    │   ProductoService       │  ← Interfaz de Servicio
                    │   (interfaz)            │
                    └────────────┬────────────┘
                                 │ implementada por
                    ┌────────────▼────────────┐
                    │  ProductoServiceImpl    │  ← Lógica de negocio (SRP)
                    │  @Service               │
                    └──────┬──────────┬───────┘
                           │          │
          ┌────────────────▼──┐  ┌────▼──────────────────┐
          │ ProductoRepository│  │   ProductoFactory     │
          │ (DAO - JPA)       │  │   @Component          │
          └────────────────┬──┘  └────┬──────────────────┘
                           │          │
          ┌────────────────▼──┐  ┌────▼──────────────────┐
          │  Producto         │  │  ProductoRequestDTO   │
          │  @Entity          │  │  ProductoResponseDTO  │
          └───────────────────┘  └───────────────────────┘

          ┌─────────────────────────────────────────────┐
          │         GlobalExceptionHandler              │
          │         @RestControllerAdvice               │
          │  - RecursoNoEncontradoException  → 404      │
          │  - MethodArgumentNotValid        → 400      │
          │  - Exception genérica            → 500      │
          └─────────────────────────────────────────────┘
```

---

## Estructura de Paquetes

```
src/main/java/com/empresa/catalogo/
├── CatalogoApplication.java
├── controller/
│   └── ProductoController.java
├── service/
│   ├── ProductoService.java          (interfaz)
│   └── ProductoServiceImpl.java      (implementación)
├── repository/
│   └── ProductoRepository.java       (DAO)
├── dto/
│   ├── ProductoRequestDTO.java
│   └── ProductoResponseDTO.java
├── entity/
│   └── Producto.java
├── factory/
│   └── ProductoFactory.java
└── exception/
    ├── ApiError.java
    ├── GlobalExceptionHandler.java
    └── RecursoNoEncontradoException.java
```

---

## Requisitos

- Java 17+
- Maven 3.9.x

---

## Ejecución

```bash
# Clonar el repositorio
git clone https://github.com/faridl28/lobo-post1-u11.git
cd lobo-post1-u11

# Compilar
mvn compile

# Ejecutar
mvn spring-boot:run
```

La aplicación iniciará en **http://localhost:8081**

---
## Capturas de pantalla

<img width="921" height="477" alt="image" src="https://github.com/user-attachments/assets/5f07c9dd-5295-4553-80d3-1a66fb652bb7" />
<img width="921" height="498" alt="image" src="https://github.com/user-attachments/assets/944d7996-7470-496a-8b1c-4fada3517973" />
<img width="921" height="492" alt="image" src="https://github.com/user-attachments/assets/e5b3b8e0-ac3f-4cfc-8109-deb4c2157ffe" />
<img width="921" height="497" alt="image" src="https://github.com/user-attachments/assets/0de5e314-a0e1-432d-b611-a3e14acd3105" />


## Endpoints disponibles

| Método | URL | Descripción |
|--------|-----|-------------|
| POST | `/api/productos` | Crear producto |
| GET | `/api/productos` | Listar productos activos |
| GET | `/api/productos/{id}` | Buscar por ID |
| DELETE | `/api/productos/{id}` | Eliminar producto |

---

## Ejemplos de uso (curl)

**Checkpoint 2 — POST exitoso:**
```bash
curl -s -X POST http://localhost:8081/api/productos \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Laptop","precio":3500000,"categoria":"ELECTRONICA"}' | jq
```

**Checkpoint 3 — Error 404:**
```bash
curl -s http://localhost:8081/api/productos/999 | jq
# {"status":404,"error":"Not Found","mensaje":"Producto con id 999 no encontrado.",...}
```

**Checkpoint 3 — Error 400:**
```bash
curl -s -X POST http://localhost:8081/api/productos \
  -H "Content-Type: application/json" \
  -d '{}' | jq
# {"status":400,"error":"Bad Request","mensaje":"nombre: El nombre es obligatorio; precio: El precio debe ser mayor a cero",...}
```

---

## Consola H2

Disponible en: **http://localhost:8081/h2-console**  
JDBC URL: `jdbc:h2:mem:catalogodb`  
Usuario: `sa` | Sin contraseña
