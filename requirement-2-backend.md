## CONTEXTO DEL PROYECTO

Implementar el backend completo para el sistema S-FMDRS (Simplified Functional Movement Disorders Rating Scale), una aplicación médica de evaluación clínica de Trastornos del Movimiento Funcional. El backend debe soportar autenticación, gestión de usuarios (3 roles), tratamientos, cuestionarios, análisis con IA, y toda la lógica de negocio necesaria.

---

## ESPECIFICACIONES TÉCNICAS

### Stack Tecnológico Backend
- **Framework**: Node.js con NestJS
- **Base de Datos**: PostgreSQL 15+
- **ORM**: Sequelize
- **Autenticación**: JWT (JSON Web Tokens)
- **Validación**: class-validator + class-transformer
- **Contenedorización**: Docker + Docker Compose
- **Variables de Entorno**: dotenv

### Arquitectura
```
├── Backend API (NestJS)
│   ├── Puerto: 3000
│   ├── Autenticación JWT
│   └── RESTful API
├── PostgreSQL Database
│   ├── Puerto: 5432
│   └── Volumen persistente
└── (Opcional) Nginx Reverse Proxy
    └── Puerto: 80
```

---

## ESTRUCTURA DE BASE DE DATOS

### Diagrama ER (Entidad-Relación)

```sql
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│    usuarios      │       │   tratamientos   │       │  cuestionarios   │
├──────────────────┤       ├──────────────────┤       ├──────────────────┤
│ id (PK)          │       │ id (PK)          │       │ id (PK)          │
│ nombres          │       │ numero           │       │ numero           │
│ apellidos        │◄──────┤ paciente_id (FK) │◄──────┤ tratamiento_id   │
│ dni (UNIQUE)     │       │ medico_id (FK)   │       │ fecha            │
│ email (UNIQUE)   │       │ estado           │       │ tipo             │
│ password         │       │ nombre           │       │ puntaje_total    │
│ rol              │       │ objetivo         │       │ tiempo_protocolo │
│ edad             │       │ fecha_inicio     │       │ created_at       │
│ sexo             │       │ fecha_fin        │       │ updated_at       │
│ especialidad     │       │ meta_objetivo    │       └──────────────────┘
│ activo           │       │ meta_descripcion │                │
│ created_at       │       │ created_at       │                │
│ updated_at       │       │ updated_at       │                │
└──────────────────┘       └──────────────────┘                │
                                                                │
        ┌───────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│ respuestas_region│       │  analisis_ia     │       │  comentarios     │
├──────────────────┤       ├──────────────────┤       ├──────────────────┤
│ id (PK)          │       │ id (PK)          │       │ id (PK)          │
│ cuestionario_id  │       │ cuestionario_id  │       │ cuestionario_id  │
│ region_id        │       │ modelo           │       │ medico_id (FK)   │
│ nombre_region    │       │ contenido (TEXT) │       │ contenido (TEXT) │
│ severidad        │       │ timestamp        │       │ timestamp        │
│ duracion         │       │ created_at       │       │ created_at       │
│ total            │       └──────────────────┘       └──────────────────┘
│ notas (TEXT)     │
└──────────────────┘

┌──────────────────┐
│ password_resets  │
├──────────────────┤
│ id (PK)          │
│ usuario_id (FK)  │
│ token            │
│ expires_at       │
│ usado            │
│ created_at       │
└──────────────────┘
```

### Esquema SQL Completo

```sql
-- Extensiones
CREATE EXTENSION IF NOT EXISTS "int-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Tabla: usuarios
CREATE TABLE usuarios (
  id SERIAL PRIMARY KEY NOT NULL,
  nombres VARCHAR(100) NOT NULL,
  apellidos VARCHAR(100) NOT NULL,
  dni VARCHAR(8) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL, -- BCrypt hash
  rol VARCHAR(20) NOT NULL CHECK (rol IN ('ADMINISTRADOR', 'MEDICO', 'PACIENTE')),
  edad INTEGER,
  sexo CHAR(1) CHECK (sexo IN ('M', 'F')),
  especialidad VARCHAR(100), -- Solo para médicos
  activo BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_usuarios_email ON usuarios(email);
CREATE INDEX idx_usuarios_dni ON usuarios(dni);
CREATE INDEX idx_usuarios_rol ON usuarios(rol);

-- Tabla: tratamientos
CREATE TABLE tratamientos (
  id SERIAL PRIMARY KEY NOT NULL,
  numero INTEGER NOT NULL,
  paciente_id int NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  medico_id int NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  estado VARCHAR(20) NOT NULL DEFAULT 'ACTIVO' CHECK (estado IN ('ACTIVO', 'TERMINADO', 'CANCELADO')),
  nombre VARCHAR(255),
  objetivo TEXT,
  fecha_inicio TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  fecha_fin TIMESTAMP,
  meta_objetivo INTEGER, -- Puntaje objetivo
  meta_descripcion TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_paciente_numero UNIQUE (paciente_id, numero)
);

CREATE INDEX idx_tratamientos_paciente ON tratamientos(paciente_id);
CREATE INDEX idx_tratamientos_medico ON tratamientos(medico_id);
CREATE INDEX idx_tratamientos_estado ON tratamientos(estado);

-- Tabla: cuestionarios
CREATE TABLE cuestionarios (
  id SERIAL PRIMARY KEY NOT NULL,
  tratamiento_id int NOT NULL REFERENCES tratamientos(id) ON DELETE CASCADE,
  numero INTEGER NOT NULL,
  fecha TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  tipo VARCHAR(20) NOT NULL CHECK (tipo IN ('INICIAL', 'SEGUIMIENTO', 'FINAL')),
  puntaje_total INTEGER NOT NULL CHECK (puntaje_total >= 0 AND puntaje_total <= 54),
  tiempo_protocolo INTEGER, -- Segundos que tomó el protocolo
  observaciones_generales TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_tratamiento_numero UNIQUE (tratamiento_id, numero)
);

CREATE INDEX idx_cuestionarios_tratamiento ON cuestionarios(tratamiento_id);
CREATE INDEX idx_cuestionarios_fecha ON cuestionarios(fecha);

-- Tabla: respuestas_region
CREATE TABLE respuestas_region (
  id SERIAL PRIMARY KEY NOT NULL,
  cuestionario_id int NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
  region_id VARCHAR(50) NOT NULL,
  nombre_region VARCHAR(100) NOT NULL,
  severidad INTEGER NOT NULL CHECK (severidad >= 0 AND severidad <= 3),
  duracion INTEGER NOT NULL CHECK (duracion >= 0 AND duracion <= 3),
  total INTEGER GENERATED ALWAYS AS (severidad + duracion) STORED,
  notas TEXT,
  
  CONSTRAINT unique_cuestionario_region UNIQUE (cuestionario_id, region_id)
);

CREATE INDEX idx_respuestas_cuestionario ON respuestas_region(cuestionario_id);

-- Tabla: analisis_ia
CREATE TABLE analisis_ia (
  id SERIAL PRIMARY KEY NOT NULL,
  cuestionario_id int NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
  modelo VARCHAR(20) NOT NULL CHECK (modelo IN ('claude', 'gemini')),
  contenido TEXT NOT NULL,
  timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_cuestionario_analisis UNIQUE (cuestionario_id)
);

-- Tabla: comentarios
CREATE TABLE comentarios (
  id SERIAL PRIMARY KEY NOT NULL,
  cuestionario_id int NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
  medico_id int NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  contenido TEXT NOT NULL,
  timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_cuestionario_comentario UNIQUE (cuestionario_id)
);

-- Tabla: password_resets (para recuperación de contraseña)
CREATE TABLE password_resets (
  id SERIAL PRIMARY KEY NOT NULL,
  usuario_id int NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  token VARCHAR(255) NOT NULL UNIQUE,
  expires_at TIMESTAMP NOT NULL,
  usado BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_password_resets_token ON password_resets(token);
CREATE INDEX idx_password_resets_usuario ON password_resets(usuario_id);

-- Trigger para actualizar updated_at automáticamente
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_usuarios_updated_at BEFORE UPDATE ON usuarios
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_tratamientos_updated_at BEFORE UPDATE ON tratamientos
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_cuestionarios_updated_at BEFORE UPDATE ON cuestionarios
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Datos de Prueba (Seed)

```sql
-- Insertar administrador
INSERT INTO usuarios (id, nombres, apellidos, dni, email, password, rol)
VALUES (
  'Carlos Alberto',
  'Mendoza Silva',
  '12345678',
  'admin@sfmdrs.com',
  '$2b$10$6TnPq7QNXB5n1LFx4F8Zt.Qr4vwYvXnxG5Z2KqRG3FqF8Zt.Qr4vw', -- Admin123
  'ADMINISTRADOR'
);

-- Insertar médicos
INSERT INTO usuarios (id, nombres, apellidos, dni, email, password, rol, especialidad)
VALUES 
(
  'María Elena',
  'García López',
  '23456789',
  'maria.garcia@hospital.com',
  '$2b$10$xK9mP2vLN8qR3tY5wZ7aB.uC4dF6eG8hI0jK2lM4nO6pQ8rS0tU2v', -- xK9mP2vL
  'MEDICO',
  'Neurología'
),
(
  'Roberto Carlos',
  'Fernández Torres',
  '34567890',
  'roberto.fernandez@hospital.com',
  '$2b$10$aB7nQ4sMO9pR2tU6vX8yA.zC1dE3fG5hI7jK9lM1nO3pQ5rS7tU9v', -- aB7nQ4sM
  'MEDICO',
  'Neurología'
);

-- Insertar pacientes
INSERT INTO usuarios (id, nombres, apellidos, dni, email, password, rol, edad, sexo)
VALUES 
(
  'Juan José',
  'Pérez Rodríguez',
  '45678901',
  'juan.perez@email.com',
  '$2b$10$pL3kR8xNM7oQ1tS5uW9yB.zA2cD4eF6gH8iJ0kL2mN4oP6qR8sT0u', -- pL3kR8xN
  'PACIENTE',
  45,
  'M'
),
(
  'Ana María',
  'González Sánchez',
  '56789012',
  'ana.gonzalez@email.com',
  '$2b$10$qW2mT9bVO8pR3sU7vX0yC.zA3dE5fG7hI9jK1lM3nO5pQ7rS9tU1v', -- qW2mT9bV
  'PACIENTE',
  38,
  'F'
),
(
  'Luis Alberto',
  'Martínez Ruiz',
  '67890123',
  'luis.martinez@email.com',
  '$2b$10$zX4vC7nMO0pR5sU9vY2zA.BC4dE6fG8hI1jK3lM5nO7pQ9rS1tU3v', -- zX4vC7nM
  'PACIENTE',
  52,
  'M'
),
(
  'Carmen Rosa',
  'López Díaz',
  '78901234',
  'carmen.lopez@email.com',
  '$2b$10$hG6tR2kLM9oP8rU3vX6yB.zA1cD5eF7gH0iJ2kL4mN6oP8qR0sT2u', -- hG6tR2kL
  'PACIENTE',
  41,
  'F'
);

-- NO insertar tratamientos ni cuestionarios en seed
-- Estos serán creados por los médicos a través de la API
```

---

## ESTRUCTURA DEL PROYECTO NESTJS

```
backend/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── common/
│   │   ├── decorators/
│   │   │   ├── roles.decorator.ts
│   │   │   └── current-user.decorator.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   └── roles.guard.ts
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts
│   │   ├── interceptors/
│   │   │   └── transform.interceptor.ts
│   │   └── pipes/
│   │       └── validation.pipe.ts
│   |___modules/ (aca van todos los modulos, ya los tengo creados, solo necesito que los revises)


├── test/
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── .env.example
├── .dockerignore
├── .gitignore
├── nest-cli.json
├── package.json
├── tsconfig.json
└── README.md
```

---

## ENDPOINTS DE LA API

### 1. Autenticación (`/api/auth`)

La autenticacion sera con una cookie creada en el backend con el modulo de cookie de express.
Ejemplo: res.cookie(token, {
  httpOnly: true
}) Es un ejemplo de uso y creacion, necesito que lo mejores.

```typescript
POST   /api/auth/login
Body: { email: string, password: string }
(Despues de enviar la cookie al frontend se enviara la información del usuario)
Response: { user: UserDto, expires_in: number }

POST   /api/auth/logout
Headers: Cookie: token
Response: { message: string }

GET    /api/auth/me
Headers: Cookie: token
Response: UserDto
```

### 2. Usuarios (`/api/usuarios`) - Solo Admin

```typescript
GET    /api/usuarios
Query: ?rol=MEDICO&activo=true
Roles: ADMINISTRADOR
Response: Usuario[]

GET    /api/usuarios/:id
Roles: ADMINISTRADOR, MEDICO (solo sus pacientes)
Response: Usuario

POST   /api/usuarios
Body: CreateUsuarioDto
Roles: ADMINISTRADOR
Response: { usuario: Usuario, password: string }

PATCH  /api/usuarios/:id
Body: UpdateUsuarioDto
Roles: ADMINISTRADOR
Response: Usuario

DELETE /api/usuarios/:id
Roles: ADMINISTRADOR
Response: { message: string }

POST   /api/usuarios/:id/desactivar
Roles: ADMINISTRADOR
Response: Usuario
```

### 3. Tratamientos (`/api/tratamientos`)

```typescript
GET    /api/tratamientos
Query: ?pacienteId={int}&medicoId={int}&estado=ACTIVO
Roles: MEDICO, PACIENTE (filtrado automático según rol)
Response: Tratamiento[]

GET    /api/tratamientos/:id
Roles: MEDICO (creador), PACIENTE (si es suyo)
Response: Tratamiento (con cuestionarios anidados)

POST   /api/tratamientos
Body: CreateTratamientoDto {
  pacienteId: string,
  nombre?: string,
  objetivo?: string,
  metaObjetivo?: number,
  metaDescripcion?: string
}
Roles: MEDICO
Response: Tratamiento

PATCH  /api/tratamientos/:id
Body: UpdateTratamientoDto
Roles: MEDICO (creador)
Response: Tratamiento

PATCH  /api/tratamientos/:id/estado
Body: { estado: 'TERMINADO' | 'CANCELADO' }
Roles: MEDICO (creador)
Response: Tratamiento

GET    /api/tratamientos/:id/progreso
Roles: MEDICO (creador), PACIENTE (si es suyo)
Response: {
  evaluacionInicial: Cuestionario,
  evaluacionActual: Cuestionario,
  mejoriaVsAnterior: number,
  mejoriaVsInicial: number,
  top3AreasMejoradas: Region[],
  areaRequiereAtencion: Region,
  diasProximaEvaluacion: number,
  progresoMeta: number
}
```

### 4. Cuestionarios (`/api/cuestionarios`)

```typescript
GET    /api/cuestionarios
Query: ?tratamientoId={int}
Roles: MEDICO, PACIENTE (con filtrado)
Response: Cuestionario[]

GET    /api/cuestionarios/:id
Roles: MEDICO, PACIENTE (con filtrado)
Response: Cuestionario (con respuestas, análisis IA y comentarios)

POST   /api/cuestionarios
Body: CreateCuestionarioDto {
  tratamientoId: string,
  tipo: 'INICIAL' | 'SEGUIMIENTO' | 'FINAL',
  tiempoProtocolo: number,
  respuestas: RespuestaRegionDto[],
  observacionesGenerales?: string
}
Roles: MEDICO
Response: Cuestionario

POST   /api/cuestionarios/:id/comentario
Body: { contenido: string }
Roles: MEDICO
Response: Comentario

GET    /api/cuestionarios/:id/comparacion
Query: ?comparadoCon={cuestionarioId}
Roles: MEDICO, PACIENTE
Response: {
  anterior: Cuestionario,
  actual: Cuestionario,
  diferencias: RegionComparacion[],
  mejoriaTotal: number
}
```

### 5. Analytics (`/api/analytics`) - Solo Médicos

```typescript
GET    /api/analytics/dashboard
 
Roles: MEDICO
Response: {
  puntajePromedio: number,
  totalPacientes: number,
  distribucionSeveridad: {
    leve: number,
    moderado: number,
    severo: number
  },
  regionMasAfectada: string,
  tendenciaMensual: number
}

GET    /api/analytics/paciente/:pacienteId
 
Roles: MEDICO
Response: {
  historialCompleto: Cuestionario[],
  evolucionTemporal: DataPoint[],
  regionesMasAfectadas: Region[],
  tasaMejoria: number
}
```

---

## CONFIGURACIÓN DOCKER

### Dockerfile

```dockerfile
# backend/docker/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar package files
COPY package*.json ./
COPY tsconfig*.json ./
COPY nest-cli.json ./

# Instalar dependencias
RUN npm ci

# Copiar código fuente
COPY src ./src

# Build
RUN npm run build

# Imagen de producción
FROM node:18-alpine AS production

WORKDIR /app

# Copiar package files y instalar solo dependencias de producción
COPY package*.json ./
RUN npm ci --only=production

# Copiar build desde builder
COPY --from=builder /app/dist ./dist

# Variables de entorno por defecto
ENV NODE_ENV=production
ENV PORT=4000

EXPOSE 4000

# Comando de inicio
CMD ["node", "dist/main"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Base de datos PostgreSQL
  postgres:
    image: postgres:15-alpine
    container_name: sfmdrs_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME:-sfmdrs}
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_INITDB_ARGS: "-E UTF8 --locale=C"
    ports:
      - "${DB_PORT:-5433}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/01-init.sql
      - ./database/seed.sql:/docker-entrypoint-initdb.d/02-seed.sql
    networks:
      - sfmdrs-network

  # Backend API (NestJS)
  backend:
    build:
      context: .
      dockerfile: docker/Dockerfile
    container_name: sfmdrs_backend
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      NODE_ENV: ${NODE_ENV:-production}
      PORT: ${PORT:-3000}
      DB_HOST: postgres
      DB_PORT: 4000
      DB_NAME: ${DB_NAME:-sfmdrs}
      DB_USER: ${DB_USER:-postgres}
      DB_PASSWORD: ${DB_PASSWORD:-postgres}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRATION: ${JWT_EXPIRATION:-24h}
      CLAUDE_API_KEY: ${CLAUDE_API_KEY}
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      SMTP_HOST: ${SMTP_HOST}
      SMTP_PORT: ${SMTP_PORT:-587}
      SMTP_USER: ${SMTP_USER}
      SMTP_PASS: ${SMTP_PASS}
      CORS_ORIGIN: ${CORS_ORIGIN:-*}
    ports:
      - "${PORT:-4000}:4000"
    networks:
      - sfmdrs-network
    volumes:
      - ./logs:/app/logs

  # (Opcional) Nginx como Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: sfmdrs_nginx
    restart: unless-stopped
    depends_on:
      - backend
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    networks:
      - sfmdrs-network

volumes:
  postgres_data:
    driver: local

networks:
  sfmdrs-network:
    driver: bridge
```

### .env.example

```bash
# .env.example
# Copiar a .env y configurar valores reales

# Application
NODE_ENV=development
PORT=4000
API_PREFIX=api

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=sfmdrs
DB_USER=postgres
DB_PASSWORD=your_secure_password_here

# JWT
JWT_SECRET=your_super_secret_jwt_key_change_in_production
JWT_EXPIRATION=24h
JWT_REFRESH_SECRET=your_refresh_token_secret
JWT_REFRESH_EXPIRATION=7d

# IA APIs
CLAUDE_API_KEY=sk-ant-api03-xxxxxxxxxxxxxxxxxxxxx
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxx

# Email (SMTP)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@sfmdrs.com
SMTP_PASS=your_smtp_password
SMTP_FROM=S-FMDRS System <noreply@sfmdrs.com>

# CORS
CORS_ORIGIN=*

# Logging
LOG_LEVEL=debug

```

---

## IMPLEMENTACIÓN DE ENTIDADES CON ORM SEQUELIZE

## IMPLEMENTAR SERVICIOS CON CONTROLADORES; MODULOS para cada submodulo creado recientemente

## DEJAR LISTO TODO PARA EXPOSICION DE ENDPOINTS

```

## PRIORIDADES DE IMPLEMENTACIÓN

### FASE 1 (Crítica)
1. ✅ Setup Docker + PostgreSQL + NestJS
2. ✅ Esquema de base de datos completo
3. ✅ Sistema de autenticación JWT
4. ✅ Módulo de usuarios (CRUD)

### FASE 2 (Esencial)
6. ✅ Módulo de tratamientos
7. ✅ Módulo de cuestionarios (completo con 9 regiones)
9. ✅ Sistema de permisos por rol (Guards)
10. ✅ Servicio de email

### FASE 3 (Importante)
11. ✅ Módulo de analytics (KPIs médicos)
12. ✅ Endpoints de progreso del paciente
13. ✅ Sistema de comentarios médico
14. ✅ Validaciones

### FASE 4 (Complementaria)
17. Logs estructurados

---

## OBJETIVO FINAL

Backend profesional, dockerizado, seguro y escalable que:
- ✅ Gestiona 3 roles de usuario con permisos diferenciados
- ✅ Persiste todos los datos en PostgreSQL
- ✅ Envía emails con credenciales a nuevos usuarios
- ✅ Soporta toda la lógica de negocio del sistema S-FMDRS

**IMPORTANTE**: Este backend debe reemplazar completamente los datos mock de Cursor y proporcionar toda la funcionalidad real del sistema.