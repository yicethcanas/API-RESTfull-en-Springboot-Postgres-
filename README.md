# Examen 2 — Microservicios Integrados

**Programación Avanzada — Institución Universitaria Pascual Bravo**
**Autores:** Victor Bracamonte · Yiceth Cañas
**Fecha:** Marzo 2026

Solución basada en microservicios para calcular festivos colombianos y generar
calendarios anuales clasificados. 



##  Estructura de archivos

```
PrgramaciónAva-Taller2/
├── README.md                         
├── Examen_2_Microservicios_integrados.md
└── calendario-api/                    ← API 2 (Spring Boot + Postgres)
    ├── api/                           ← proyecto Maven multi-módulo
    │   ├── pom.xml                    (parent, packaging=pom)
    │   ├── .gitignore
    │   ├── dominio/
    │   │   ├── pom.xml
    │   │   └── src/main/java/festivos/api/dominio/
    │   │       ├── entidades/   → Tipo, Calendario
    │   │       └── modelos/     → Festivo, Dia, GrupoDias
    │   ├── core/
    │   │   ├── pom.xml
    │   │   └── src/main/java/festivos/api/core/servicios/
    │   │       → IFestivoServicio, ICalendarioServicio, ITipoServicio
    │   ├── infraestructura/
    │   │   ├── pom.xml
    │   │   └── src/main/java/festivos/api/infraestructura/repositorios/
    │   │       → ITipoRepositorio, ICalendarioRepositorio
    │   ├── aplicacion/
    │   │   ├── pom.xml
    │   │   └── src/main/java/festivos/api/aplicacion/
    │   │       ├── servicios/       → FestivoServicio, CalendarioServicio, TipoServicio
    │   │       ├── clientes/        → FestivoCliente (REST hacia API 1)
    │   │       └── configuracion/   → RestTemplateConfiguracion
    │   └── presentacion/
    │       ├── pom.xml
    │       └── src/main/
    │           ├── java/festivos/api/
    │           │   ├── ApiApplication.java
    │           │   └── presentacion/controladores/
    │           │       → FestivoControlador, CalendarioControlador, TipoControlador
    │           └── resources/application.properties
    └── bd/                            ← scripts SQL 
        ├── DDL_Festivos.sql           (CREATE DATABASE + tablas + índices)
        ├── DDL_Festivos_Secuencias.sql(CREATE SEQUENCE para generadores JPA)
        └── DML_Festivos.sql           (INSERT tipos: Laboral, Fin de Semana, Festivo)

GitHub/APIRESTfullenExpressJS-MongoDB/  ← API 1 (Express + MongoDB)
```

---



**Endpoints**

| Método | URL | Respuesta |
|---|---|---|
| GET | `/api/festivos/verificar/:year/:month/:day` | `{ mensaje, festivo? }` |
| GET | `/api/festivos/obtener/:year` | `[{ festivo, fecha: "YYYY-MM-DD" }, ...]` |

**Tipos de festivo** (`id` del documento Mongo):
- `1` Fijo
- `2` Fijo + Ley de Puente (traslado al siguiente lunes)
- `3` Basado en Domingo de Pascua
- `4` Pascua + Ley de Puente

###  API 2 

**Endpoints**

| Método | URL | Descripción |
|---|---|---|
| GET | `/api/festivos/obtener/{anio}` | Reenvía la lista de festivos desde API 1 |
| POST | `/api/calendario/generar/{anio}` | Genera y persiste el calendario anual → `true` si OK |
| GET | `/api/calendario/obtener/{anio}` | Calendario agrupado: Laboral / Fin de Semana / Festivo |
| GET | `/api/tipos/` | Lista los tipos de clasificación |
| GET | `/api/tipos/obtener/{id}` | Obtiene un tipo por id |
| POST | `/api/tipos/` | Agrega un tipo |


Secuencias generadas por JPA (deben existir en BD antes de arrancar):
`secuencia_tipo`, `secuencia_calendario`.

Datos semilla en tabla `tipo`: `1 = Laboral`, `2 = Fin de Semana`, `3 = Festivo`.

---

##  Dependencias

###  API 1 (Express + MongoDB)

**Runtime (Node.js 18+)**
- `express` 5.x
- `mongoose` 9.x
- `mongodb` 7.x
- `cors` 2.x
- `dotenv` 17.x

**Desarrollo**
- `nodemon` 3.x

### API 2 (Spring Boot + PostgreSQL)

**Parent:** `spring-boot-starter-parent:4.0.4` (Java 25)


## 5. Requisitos previos

- **Node.js** 18+ y **npm** 9+
- **Java** 25 (o compatible con Spring Boot 4.0.4)
- **Maven** 3.9+
- **PostgreSQL** 14+ corriendo en `localhost:5432`
- Cuenta de **MongoDB Atlas** (o MongoDB local)

---

## 6. Configuración

### API 1 — `.env`
```env
MONGODB_URI=mongodb+srv://<usuario>:<password>@<cluster>.mongodb.net/festivos?retryWrites=true&w=majority
PORT=8080
```

### API 2 — `api/presentacion/src/main/resources/application.properties`
```properties
spring.application.name=api
server.port=8081

spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/festivos
spring.datasource.username=postgres
spring.datasource.password=sa

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

festivos.api.base-url=http://localhost:8080
```

---

##  Ejecución paso a paso

### Paso 1 — Preparar PostgreSQL

```bash
# 1) Crear base y esquema
psql -U postgres -f "C:/Users/Usuario/Desktop/PrgramaciónAva-Taller2/calendario-api/bd/DDL_Festivos.sql"

# 2) Cargar datos semilla de tipos
psql -U postgres -d festivos -f "C:/Users/Usuario/Desktop/PrgramaciónAva-Taller2/calendario-api/bd/DML_Festivos.sql"

# 3) Crear secuencias que usan los @SequenceGenerator
psql -U postgres -d festivos -f "C:/Users/Usuario/Desktop/PrgramaciónAva-Taller2/calendario-api/bd/DDL_Festivos_Secuencias.sql"
```

### Paso 2 — Levantar API 1 (Express + MongoDB)

```bash

npm install
npm run seed     # primera vez: poblar Mongo con los 18 festivos
npm start        # o: npm run dev (con nodemon)
```

Mensaje esperado:
```
MongoDB Atlas conectado
Servidor corriendo en puerto 8080
```

### Paso 3 — Levantar API 2 (Spring Boot + Postgres)


# compilar todo el multi-módulo
mvn clean install

# arrancar la app desde el módulo presentacion
mvn -pl presentacion spring-boot:run

# alternativa: jar ejecutable
mvn -pl presentacion spring-boot:repackage
java -jar presentacion/target/presentacion-0.0.1-SNAPSHOT.jar
```

Queda escuchando en `http://localhost:8081`.



### API 2 — Swagger UI
```
http://localhost:8081/swagger-ui.html
```
