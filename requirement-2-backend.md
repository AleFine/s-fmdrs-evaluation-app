## CONTEXTO DEL PROYECTO

Implementar el backend completo para el sistema S-FMDRS (Simplified Functional Movement Disorders Rating Scale), una aplicación médica de evaluación clínica de Trastornos del Movimiento Funcional. El backend debe soportar autenticación, gestión de usuarios (3 roles), tratamientos, cuestionarios, análisis con IA, y toda la lógica de negocio necesaria.

---

## ESPECIFICACIONES TÉCNICAS

### Stack Tecnológico Backend
- **Framework**: Node.js con NestJS
- **Base de Datos**: PostgreSQL 15+
- **ORM**: TypeORM o Prisma
- **Autenticación**: JWT (JSON Web Tokens)
- **Validación**: class-validator + class-transformer
- **Documentación API**: Swagger/OpenAPI
- **Contenedorización**: Docker + Docker Compose
- **Variables de Entorno**: dotenv
- **Integración IA**: 
  - Anthropic API (Claude)
  - Google Generative AI (Gemini)

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
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Tabla: usuarios
CREATE TABLE usuarios (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
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
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  numero INTEGER NOT NULL,
  paciente_id UUID NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  medico_id UUID NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
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
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tratamiento_id UUID NOT NULL REFERENCES tratamientos(id) ON DELETE CASCADE,
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
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cuestionario_id UUID NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
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
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cuestionario_id UUID NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
  modelo VARCHAR(20) NOT NULL CHECK (modelo IN ('claude', 'gemini')),
  contenido TEXT NOT NULL,
  timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_cuestionario_analisis UNIQUE (cuestionario_id)
);

-- Tabla: comentarios
CREATE TABLE comentarios (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cuestionario_id UUID NOT NULL REFERENCES cuestionarios(id) ON DELETE CASCADE,
  medico_id UUID NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
  contenido TEXT NOT NULL,
  timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_cuestionario_comentario UNIQUE (cuestionario_id)
);

-- Tabla: password_resets (para recuperación de contraseña)
CREATE TABLE password_resets (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  usuario_id UUID NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
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
  '00000000-0000-0000-0000-000000000001',
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
  '10000000-0000-0000-0000-000000000001',
  'María Elena',
  'García López',
  '23456789',
  'maria.garcia@hospital.com',
  '$2b$10$xK9mP2vLN8qR3tY5wZ7aB.uC4dF6eG8hI0jK2lM4nO6pQ8rS0tU2v', -- xK9mP2vL
  'MEDICO',
  'Neurología'
),
(
  '10000000-0000-0000-0000-000000000002',
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
  '20000000-0000-0000-0000-000000000001',
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
  '20000000-0000-0000-0000-000000000002',
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
  '20000000-0000-0000-0000-000000000003',
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
  '20000000-0000-0000-0000-000000000004',
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
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── strategies/
│   │   │   └── jwt.strategy.ts
│   │   └── dto/
│   │       ├── login.dto.ts
│   │       └── change-password.dto.ts
│   ├── usuarios/
│   │   ├── usuarios.module.ts
│   │   ├── usuarios.controller.ts
│   │   ├── usuarios.service.ts
│   │   ├── entities/
│   │   │   └── usuario.entity.ts
│   │   └── dto/
│   │       ├── create-usuario.dto.ts
│   │       └── update-usuario.dto.ts
│   ├── tratamientos/
│   │   ├── tratamientos.module.ts
│   │   ├── tratamientos.controller.ts
│   │   ├── tratamientos.service.ts
│   │   ├── entities/
│   │   │   └── tratamiento.entity.ts
│   │   └── dto/
│   │       ├── create-tratamiento.dto.ts
│   │       └── update-tratamiento.dto.ts
│   ├── cuestionarios/
│   │   ├── cuestionarios.module.ts
│   │   ├── cuestionarios.controller.ts
│   │   ├── cuestionarios.service.ts
│   │   ├── entities/
│   │   │   ├── cuestionario.entity.ts
│   │   │   ├── respuesta-region.entity.ts
│   │   │   ├── analisis-ia.entity.ts
│   │   │   └── comentario.entity.ts
│   │   └── dto/
│   │       ├── create-cuestionario.dto.ts
│   │       ├── create-respuesta.dto.ts
│   │       ├── generate-analisis.dto.ts
│   │       └── create-comentario.dto.ts
│   ├── ia/
│   │   ├── ia.module.ts
│   │   ├── ia.service.ts
│   │   ├── providers/
│   │   │   ├── claude.provider.ts
│   │   │   └── gemini.provider.ts
│   │   └── templates/
│   │       └── analisis-clinico.template.ts
│   ├── analytics/
│   │   ├── analytics.module.ts
│   │   ├── analytics.controller.ts
│   │   └── analytics.service.ts
│   ├── email/
│   │   ├── email.module.ts
│   │   └── email.service.ts
│   └── config/
│       ├── database.config.ts
│       └── jwt.config.ts
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

```typescript
POST   /api/auth/login
Body: { email: string, password: string }
Response: { access_token: string, user: UserDto, expires_in: number }

POST   /api/auth/logout
Headers: Authorization: Bearer {token}
Response: { message: string }

POST   /api/auth/refresh
Body: { refresh_token: string }
Response: { access_token: string, expires_in: number }

GET    /api/auth/me
Headers: Authorization: Bearer {token}
Response: UserDto
```

### 2. Usuarios (`/api/usuarios`) - Solo Admin

```typescript
GET    /api/usuarios
Query: ?rol=MEDICO&activo=true
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR
Response: Usuario[]

GET    /api/usuarios/:id
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR, MEDICO (solo sus pacientes)
Response: Usuario

POST   /api/usuarios
Body: CreateUsuarioDto
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR
Response: { usuario: Usuario, password: string }

PATCH  /api/usuarios/:id
Body: UpdateUsuarioDto
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR
Response: Usuario

DELETE /api/usuarios/:id
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR
Response: { message: string }

POST   /api/usuarios/:id/desactivar
Headers: Authorization: Bearer {token}
Roles: ADMINISTRADOR
Response: Usuario
```

### 3. Tratamientos (`/api/tratamientos`)

```typescript
GET    /api/tratamientos
Query: ?pacienteId={uuid}&medicoId={uuid}&estado=ACTIVO
Headers: Authorization: Bearer {token}
Roles: MEDICO, PACIENTE (filtrado automático según rol)
Response: Tratamiento[]

GET    /api/tratamientos/:id
Headers: Authorization: Bearer {token}
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
Headers: Authorization: Bearer {token}
Roles: MEDICO
Response: Tratamiento

PATCH  /api/tratamientos/:id
Body: UpdateTratamientoDto
Headers: Authorization: Bearer {token}
Roles: MEDICO (creador)
Response: Tratamiento

PATCH  /api/tratamientos/:id/estado
Body: { estado: 'TERMINADO' | 'CANCELADO' }
Headers: Authorization: Bearer {token}
Roles: MEDICO (creador)
Response: Tratamiento

GET    /api/tratamientos/:id/progreso
Headers: Authorization: Bearer {token}
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
Query: ?tratamientoId={uuid}
Headers: Authorization: Bearer {token}
Roles: MEDICO, PACIENTE (con filtrado)
Response: Cuestionario[]

GET    /api/cuestionarios/:id
Headers: Authorization: Bearer {token}
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
Headers: Authorization: Bearer {token}
Roles: MEDICO
Response: Cuestionario

POST   /api/cuestionarios/:id/analisis-ia
Body: { modelo: 'claude' | 'gemini' }
Headers: Authorization: Bearer {token}
Roles: MEDICO
Response: AnalisisIA

POST   /api/cuestionarios/:id/comentario
Body: { contenido: string }
Headers: Authorization: Bearer {token}
Roles: MEDICO
Response: Comentario

GET    /api/cuestionarios/:id/comparacion
Query: ?comparadoCon={cuestionarioId}
Headers: Authorization: Bearer {token}
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
Headers: Authorization: Bearer {token}
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
Headers: Authorization: Bearer {token}
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
ENV PORT=3000

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

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
      - "${DB_PORT:-5432}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/01-init.sql
      - ./database/seed.sql:/docker-entrypoint-initdb.d/02-seed.sql
    networks:
      - sfmdrs-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres} -d ${DB_NAME:-sfmdrs}"]
      interval: 10s
      timeout: 5s
      retries: 5

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
      DB_PORT: 5432
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
      - "${PORT:-3000}:3000"
    networks:
      - sfmdrs-network
    volumes:
      - ./logs:/app/logs
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

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
PORT=3000
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

# Rate Limiting
RATE_LIMIT_TTL=60
RATE_LIMIT_MAX=100
```

---

## IMPLEMENTACIÓN DE ENTIDADES (TypeORM)

### Usuario Entity

```typescript
// src/usuarios/entities/usuario.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, OneToMany } from 'typeorm';
import { Exclude } from 'class-transformer';
import { Tratamiento } from '../../tratamientos/entities/tratamiento.entity';

export enum RolUsuario {
  ADMINISTRADOR = 'ADMINISTRADOR',
  MEDICO = 'MEDICO',
  PACIENTE = 'PACIENTE'
}

@Entity('usuarios')
export class Usuario {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 100 })
  nombres: string;

  @Column({ length: 100 })
  apellidos: string;

  @Column({ length: 8, unique: true })
  dni: string;

  @Column({ length: 255, unique: true })
  email: string;

  @Exclude()
  @Column({ length: 255 })
  password: string;

  @Column({
    type: 'enum',
    enum: RolUsuario
  })
  rol: RolUsuario;

  @Column({ type: 'int', nullable: true })
  edad?: number;

  @Column({ type: 'char', length: 1, nullable: true })
  sexo?: 'M' | 'F';

  @Column({ length: 100, nullable: true })
  especialidad?: string;

  @Column({ default: true })
  activo: boolean;

  @OneToMany(() => Tratamiento, tratamiento => tratamiento.paciente)
  tratamientosComoPaciente: Tratamiento[];

  @OneToMany(() => Tratamiento, tratamiento => tratamiento.medico)
  tratamientosComoMedico: Tratamiento[];

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Helper methods
  get nombreCompleto(): string {
    return `${this.nombres} ${this.apellidos}`;
  }

  esMedico(): boolean {
    return this.rol === RolUsuario.MEDICO;
  }

  esPaciente(): boolean {
    return this.rol === RolUsuario.PACIENTE;
  }

  esAdministrador(): boolean {
    return this.rol === RolUsuario.ADMINISTRADOR;
  }
}
```

### Tratamiento Entity

```typescript
// src/tratamientos/entities/tratamiento.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany, CreateDateColumn, UpdateDateColumn, JoinColumn } from 'typeorm';
import { Usuario } from '../../usuarios/entities/usuario.entity';
import { Cuestionario } from '../../cuestionarios/entities/cuestionario.entity';

export enum EstadoTratamiento {
  ACTIVO = 'ACTIVO',
  TERMINADO = 'TERMINADO',
  CANCELADO = 'CANCELADO'
}

@Entity('tratamientos')
export class Tratamiento {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'int' })
  numero: number;

  @ManyToOne(() => Usuario, usuario => usuario.tratamientosComoPaciente, { eager: true })
  @JoinColumn({ name: 'paciente_id' })
  paciente: Usuario;

  @Column({ name: 'paciente_id' })
  pacienteId: string;

  @ManyToOne(() => Usuario, usuario => usuario.tratamientosComoMedico, { eager: true })
  @JoinColumn({ name: 'medico_id' })
  medico: Usuario;

  @Column({ name: 'medico_id' })
  medicoId: string;

  @Column({
    type: 'enum',
    enum: EstadoTratamiento,
    default: EstadoTratamiento.ACTIVO
  })
  estado: EstadoTratamiento;

  @Column({ length: 255, nullable: true })
  nombre?: string;

  @Column({ type: 'text', nullable: true })
  objetivo?: string;

  @Column({ name: 'fecha_inicio', type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  fechaInicio: Date;

  @Column({ name: 'fecha_fin', type: 'timestamp', nullable: true })
  fechaFin?: Date;

  @Column({ name: 'meta_objetivo', type: 'int', nullable: true })
  metaObjetivo?: number;

  @Column({ name: 'meta_descripcion', type: 'text', nullable: true })
  metaDescripcion?: string;

  @OneToMany(() => Cuestionario, cuestionario => cuestionario.tratamiento)
  cuestionarios: Cuestionario[];

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Helper methods
  estaActivo(): boolean {
    return this.estado === EstadoTratamiento.ACTIVO;
  }

  puedeSerModificado(): boolean {
    return this.estado !== EstadoTratamiento.CANCELADO;
  }
}
```

### Cuestionario Entity

```typescript
// src/cuestionarios/entities/cuestionario.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany, OneToOne, CreateDateColumn, UpdateDateColumn, JoinColumn } from 'typeorm';
import { Tratamiento } from '../../tratamientos/entities/tratamiento.entity';
import { RespuestaRegion } from './respuesta-region.entity';
import { AnalisisIA } from './analisis-ia.entity';
import { Comentario } from './comentario.entity';

export enum TipoCuestionario {
  INICIAL = 'INICIAL',
  SEGUIMIENTO = 'SEGUIMIENTO',
  FINAL = 'FINAL'
}

@Entity('cuestionarios')
export class Cuestionario {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Tratamiento, tratamiento => tratamiento.cuestionarios)
  @JoinColumn({ name: 'tratamiento_id' })
  tratamiento: Tratamiento;

  @Column({ name: 'tratamiento_id' })
  tratamientoId: string;

  @Column({ type: 'int' })
  numero: number;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  fecha: Date;

  @Column({
    type: 'enum',
    enum: TipoCuestionario
  })
  tipo: TipoCuestionario;

  @Column({ name: 'puntaje_total', type: 'int' })
  puntajeTotal: number;

  @Column({ name: 'tiempo_protocolo', type: 'int', nullable: true })
  tiempoProtocolo?: number;

  @Column({ name: 'observaciones_generales', type: 'text', nullable: true })
  observacionesGenerales?: string;

  @OneToMany(() => RespuestaRegion, respuesta => respuesta.cuestionario, { cascade: true })
  respuestas: RespuestaRegion[];

  @OneToOne(() => AnalisisIA, analisis => analisis.cuestionario, { cascade: true })
  analisisIA?: AnalisisIA;

  @OneToOne(() => Comentario, comentario => comentario.cuestionario, { cascade: true })
  comentario?: Comentario;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Helper methods
  calcularPuntajeTotal(): number {
    return this.respuestas.reduce((sum, r) => sum + r.total, 0);
  }

  getSeveridadCategoria(): string {
    const porcentaje = (this.puntajeTotal / 54) * 100;
    if (porcentaje <= 33) return 'LEVE';
    if (porcentaje <= 66) return 'MODERADO';
    return 'SEVERO';
  }
}
```

---

## SERVICIO DE INTEGRACIÓN CON IA

### Claude Provider

```typescript
// src/ia/providers/claude.provider.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import Anthropic from '@anthropic-ai/sdk';
import { Cuestionario } from '../../cuestionarios/entities/cuestionario.entity';
import { generarPromptAnalisisContinuar0:39nico } from '../templates/analisis-clinico.template';

@Injectable()
export class ClaudeProvider {
  private client: Anthropic;

  constructor(private configService: ConfigService) {
    this.client = new Anthropic({
      apiKey: this.configService.get<string>('CLAUDE_API_KEY'),
    });
  }

  async generarAnalisis(
    cuestionario: Cuestionario,
    cuestionarioAnterior?: Cuestionario
  ): Promise<string> {
    const prompt = generarPromptAnalisisClinico(cuestionario, cuestionarioAnterior);

    const message = await this.client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      temperature: 0.7,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
}
```

### Gemini Provider

```typescript
// src/ia/providers/gemini.provider.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { GoogleGenerativeAI } from '@google/generative-ai';
import { Cuestionario } from '../../cuestionarios/entities/cuestionario.entity';
import { generarPromptAnalisisClinico } from '../templates/analisis-clinico.template';

@Injectable()
export class GeminiProvider {
  private genAI: GoogleGenerativeAI;

  constructor(private configService: ConfigService) {
    this.genAI = new GoogleGenerativeAI(
      this.configService.get<string>('GEMINI_API_KEY')
    );
  }

  async generarAnalisis(
    cuestionario: Cuestionario,
    cuestionarioAnterior?: Cuestionario
  ): Promise<string> {
    const model = this.genAI.getGenerativeModel({ model: 'gemini-1.5-pro' });
    const prompt = generarPromptAnalisisClinico(cuestionario, cuestionarioAnterior);

    const result = await model.generateContent(prompt);
    const response = await result.response;
    return response.text();
  }
}
```

### Template de Análisis Clínico

```typescript
// src/ia/templates/analisis-clinico.template.ts
import { Cuestionario } from '../../cuestionarios/entities/cuestionario.entity';

export function generarPromptAnalisisClinico(
  cuestionario: Cuestionario,
  cuestionarioAnterior?: Cuestionario
): string {
  const datosActuales = {
    puntajeTotal: cuestionario.puntajeTotal,
    tipo: cuestionario.tipo,
    fecha: cuestionario.fecha,
    respuestas: cuestionario.respuestas.map(r => ({
      region: r.nombreRegion,
      severidad: r.severidad,
      duracion: r.duracion,
      total: r.total,
      notas: r.notas
    }))
  };

  const datosAnteriores = cuestionarioAnterior ? {
    puntajeTotal: cuestionarioAnterior.puntajeTotal,
    fecha: cuestionarioAnterior.fecha,
    respuestas: cuestionarioAnterior.respuestas.map(r => ({
      region: r.nombreRegion,
      total: r.total
    }))
  } : null;

  return `
Eres un neurólogo especialista en Trastornos del Movimiento Funcional (TNF). 
Genera un análisis clínico profesional basado en la siguiente evaluación S-FMDRS:

## EVALUACIÓN ACTUAL
- Fecha: ${new Date(datosActuales.fecha).toLocaleDateString()}
- Tipo: ${datosActuales.tipo}
- Puntaje Total: ${datosActuales.puntajeTotal}/54 puntos
- Categoría de Severidad: ${getSeveridadCategoria(datosActuales.puntajeTotal)}

### Desglose por Regiones:
${datosActuales.respuestas.map(r => `
- **${r.region}**: ${r.total} puntos (Severidad: ${r.severidad}, Duración: ${r.duracion})
  ${r.notas ? `Notas: ${r.notas}` : ''}
`).join('\n')}

${datosAnteriores ? `
## EVALUACIÓN ANTERIOR (Comparación)
- Fecha: ${new Date(datosAnteriores.fecha).toLocaleDateString()}
- Puntaje Total: ${datosAnteriores.puntajeTotal}/54 puntos
- Diferencia: ${datosActuales.puntajeTotal - datosAnteriores.puntajeTotal} puntos (${calcularPorcentajeCambio(datosAnteriores.puntajeTotal, datosActuales.puntajeTotal)}%)

### Cambios por Región:
${datosActuales.respuestas.map((r, i) => {
  const anterior = datosAnteriores.respuestas.find(ra => ra.region === r.region);
  if (!anterior) return '';
  const cambio = r.total - anterior.total;
  const simbolo = cambio > 0 ? '⬆️' : cambio < 0 ? '⬇️' : '➡️';
  return `- ${r.region}: ${anterior.total} → ${r.total} pts ${simbolo} (${cambio >= 0 ? '+' : ''}${cambio})`;
}).join('\n')}
` : ''}

## INSTRUCCIONES
Genera un análisis clínico completo que incluya:

1. **Resumen Ejecutivo**: Breve overview del estado del paciente
2. **Hallazgos Principales**: Top 3 regiones más afectadas con interpretación clínica
3. **Patrones Observados**: Distribución de síntomas, lateralidad, fenómenos especiales
4. **${datosAnteriores ? 'Evolución' : 'Baseline'}**: ${datosAnteriores ? 'Análisis de cambios y progreso' : 'Establecimiento de línea base'}
5. **Recomendaciones**: 3-4 recomendaciones terapéuticas específicas
6. **Próximos Pasos**: Seguimiento sugerido y consideraciones

Usa formato Markdown con headers (##, ###), listas, y énfasis donde apropiado.
Mantén un tono profesional, clínico y constructivo.
Evita diagnósticos médicos absolutos, usa lenguaje probabilístico cuando sea apropiado.

NOTA IMPORTANTE: Finaliza con un disclaimer: "*Este análisis es generado por IA y debe ser revisado y validado por el médico tratante.*"
`;
}

function getSeveridadCategoria(puntaje: number): string {
  const porcentaje = (puntaje / 54) * 100;
  if (porcentaje <= 33) return 'LEVE (0-18 puntos)';
  if (porcentaje <= 66) return 'MODERADO (19-36 puntos)';
  return 'SEVERO (37-54 puntos)';
}

function calcularPorcentajeCambio(anterior: number, actual: number): string {
  const cambio = ((actual - anterior) / anterior) * 100;
  return cambio >= 0 ? `+${cambio.toFixed(1)}` : cambio.toFixed(1);
}
```

---

## SERVICIO DE EMAIL

```typescript
// src/email/email.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as nodemailer from 'nodemailer';

@Injectable()
export class EmailService {
  private transporter: nodemailer.Transporter;

  constructor(private configService: ConfigService) {
    this.transporter = nodemailer.createTransporter({
      host: this.configService.get<string>('SMTP_HOST'),
      port: this.configService.get<number>('SMTP_PORT'),
      secure: false,
      auth: {
        user: this.configService.get<string>('SMTP_USER'),
        pass: this.configService.get<string>('SMTP_PASS'),
      },
    });
  }

  async enviarCredencialesNuevoUsuario(
    email: string,
    nombre: string,
    password: string,
    rol: string
  ): Promise<void> {
    const mailOptions = {
      from: this.configService.get<string>('SMTP_FROM'),
      to: email,
      subject: `Bienvenido al Sistema S-FMDRS - Credenciales de Acceso`,
      html: `
        <!DOCTYPE html>
        <html>
        <head>
          <style>
            body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
            .container { max-width: 600px; margin: 0 auto; padding: 20px; }
            .header { background: #0066CC; color: white; padding: 20px; text-align: center; }
            .content { background: #f9f9f9; padding: 30px; border-radius: 8px; margin-top: 20px; }
            .credentials { background: white; padding: 20px; border-left: 4px solid #0066CC; margin: 20px 0; }
            .credential-item { margin: 10px 0; }
            .credential-label { font-weight: bold; color: #0066CC; }
            .credential-value { font-family: monospace; font-size: 16px; padding: 5px 10px; background: #f0f0f0; border-radius: 4px; }
            .footer { text-align: center; margin-top: 30px; font-size: 12px; color: #666; }
            .warning { background: #fff3cd; border-left: 4px solid #ffc107; padding: 15px; margin: 20px 0; }
          </style>
        </head>
        <body>
          <div class="container">
            <div class="header">
              <h1>🏥 Sistema S-FMDRS</h1>
              <p>Evaluación de Trastornos del Movimiento Funcional</p>
            </div>
            
            <div class="content">
              <h2>¡Bienvenido, ${nombre}!</h2>
              <p>Se ha creado una cuenta para usted en el Sistema S-FMDRS con el rol de <strong>${rol}</strong>.</p>
              
              <div class="credentials">
                <h3>Sus Credenciales de Acceso</h3>
                <div class="credential-item">
                  <span class="credential-label">Email:</span><br>
                  <span class="credential-value">${email}</span>
                </div>
                <div class="credential-item">
                  <span class="credential-label">Contraseña Temporal:</span><br>
                  <span class="credential-value">${password}</span>
                </div>
              </div>
              
              <div class="warning">
                <strong>⚠️ Importante:</strong>
                <ul>
                  <li>Esta contraseña es temporal</li>
                  <li>Se recomienda cambiarla después del primer inicio de sesión</li>
                  <li>No comparta esta información con nadie</li>
                </ul>
              </div>
              
              <p>Para acceder al sistema, use las credenciales proporcionadas en la aplicación móvil S-FMDRS.</p>
              
              <p>Si tiene alguna pregunta o problema, contacte al administrador del sistema.</p>
            </div>
            
            <div class="footer">
              <p>Este es un correo automático, por favor no responda.</p>
              <p>&copy; 2024 Sistema S-FMDRS. Todos los derechos reservados.</p>
            </div>
          </div>
        </body>
        </html>
      `
    };

    await this.transporter.sendMail(mailOptions);
  }

  async enviarNotificacionNuevoTratamiento(
    emailPaciente: string,
    nombrePaciente: string,
    nombreMedico: string,
    nombreTratamiento: string
  ): Promise<void> {
    // Similar estructura HTML para notificar al paciente
    // ... implementar según necesidad
  }
}
```

---

## COMANDOS DOCKER

### Construir y levantar servicios
```bash
# Construir imágenes
docker-compose build

# Levantar todos los servicios
docker-compose up -d

# Ver logs
docker-compose logs -f backend

# Ver logs de base de datos
docker-compose logs -f postgres

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes (⚠️ elimina datos)
docker-compose down -v
```

### Comandos útiles de desarrollo
```bash
# Ejecutar migraciones
docker-compose exec backend npm run migration:run

# Rollback de migraciones
docker-compose exec backend npm run migration:revert

# Seed de datos
docker-compose exec backend npm run seed

# Acceder a PostgreSQL
docker-compose exec postgres psql -U postgres -d sfmdrs

# Backup de base de datos
docker-compose exec postgres pg_dump -U postgres sfmdrs > backup.sql

# Restaurar backup
docker-compose exec -T postgres psql -U postgres sfmdrs < backup.sql

# Reiniciar solo backend
docker-compose restart backend

# Ver estado de servicios
docker-compose ps
```

---

## PRUEBAS Y VALIDACIÓN

### Health Check Endpoint

```typescript
// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { ApiTags, ApiOperation } from '@nestjs/swagger';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@ApiTags('Health')
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  @ApiOperation({ summary: 'Check API health status' })
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}
```

### Endpoints de Prueba

```bash
# Login como administrador
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@sfmdrs.com","password":"Admin123"}'

# Crear un médico
curl -X POST http://localhost:3000/api/usuarios \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "nombres": "Test",
    "apellidos": "Doctor",
    "dni": "99999999",
    "email": "test.doctor@hospital.com",
    "rol": "MEDICO",
    "especialidad": "Neurología"
  }'

# Obtener todos los pacientes
curl -X GET "http://localhost:3000/api/usuarios?rol=PACIENTE" \
  -H "Authorization: Bearer {token}"
```

---

## INTEGRACIÓN CON REACT NATIVE

### Ejemplo de Servicio API en React Native

```typescript
// services/api.service.ts (para React Native)
import AsyncStorage from '@react-native-async-storage/async-storage';

const API_BASE_URL = 'http://localhost:3000/api';

class ApiService {
  private async getAuthHeader() {
    const token = await AsyncStorage.getItem('@sfmdrs:token');
    return token ? { Authorization: `Bearer ${token}` } : {};
  }

  async login(email: string, password: string) {
    const response = await fetch(`${API_BASE_URL}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    if (!response.ok) throw new Error('Login failed');
    
    const data = await response.json();
    await AsyncStorage.setItem('@sfmdrs:token', data.access_token);
    return data;
  }

  async crearMedico(datos: CreateMedicoDto) {
    const response = await fetch(`${API_BASE_URL}/usuarios`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(await this.getAuthHeader())
      },
      body: JSON.stringify({ ...datos, rol: 'MEDICO' })
    });
    
    if (!response.ok) throw new Error('Error creating doctor');
    return response.json();
  }

  async crearTratamiento(datos: CreateTratamientoDto) {
    const response = await fetch(`${API_BASE_URL}/tratamientos`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(await this.getAuthHeader())
      },
      body: JSON.stringify(datos)
    });
    
    if (!response.ok) throw new Error('Error creating treatment');
    return response.json();
  }

  async guardarCuestionario(datos: CreateCuestionarioDto) {
    const response = await fetch(`${API_BASE_URL}/cuestionarios`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(await this.getAuthHeader())
      },
      body: JSON.stringify(datos)
    });
    
    if (!response.ok) throw new Error('Error saving questionnaire');
    return response.json();
  }

  async generarAnalisisIA(cuestionarioId: string, modelo: 'claude' | 'gemini') {
    const response = await fetch(
      `${API_BASE_URL}/cuestionarios/${cuestionarioId}/analisis-ia`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          ...(await this.getAuthHeader())
        },
        body: JSON.stringify({ modelo })
      }
    );
    
    if (!response.ok) throw new Error('Error generating AI analysis');
    return response.json();
  }
}

export default new ApiService();
```

---

## SEGURIDAD

### Configuraciones de Seguridad

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import helmet from 'helmet';
import * as compression from 'compression';
import rateLimit from 'express-rate-limit';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Security headers
  app.use(helmet());
  
  // CORS
  app.enableCors({
    origin: process.env.CORS_ORIGIN?.split(',') || '*',
    credentials: true
  });
  
  // Compression
  app.use(compression());
  
  // Rate limiting
  app.use(
    rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 100 // limit each IP to 100 requests per windowMs
    })
  );
  
  // Global validation pipe
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true
  }));
  
  // API prefix
  app.setGlobalPrefix('api');
  
  // Swagger documentation
  const config = new DocumentBuilder()
    .setTitle('S-FMDRS API')
    .setDescription('API para Sistema de Evaluación S-FMDRS')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document);
  
  await app.listen(process.env.PORT || 3000);
}

bootstrap();
```

---

## PRIORIDADES DE IMPLEMENTACIÓN

### FASE 1 (Crítica)
1. ✅ Setup Docker + PostgreSQL + NestJS
2. ✅ Esquema de base de datos completo
3. ✅ Sistema de autenticación JWT
4. ✅ Módulo de usuarios (CRUD)
5. ✅ Seeds con datos de prueba (admin + médicos + pacientes)

### FASE 2 (Esencial)
6. ✅ Módulo de tratamientos
7. ✅ Módulo de cuestionarios (completo con 9 regiones)
8. ✅ Integración con APIs de IA (Claude + Gemini)
9. ✅ Sistema de permisos por rol (Guards)
10. ✅ Servicio de email

### FASE 3 (Importante)
11. ✅ Módulo de analytics (KPIs médicos)
12. ✅ Endpoints de progreso del paciente
13. ✅ Sistema de comentarios médico
14. ✅ Validaciones y error handling robusto
15. ✅ Documentación Swagger completa

### FASE 4 (Complementaria)
16. Testing (unit + integration)
17. Logs estructurados
18. Optimización de queries
19. Caché con Redis (opcional)
20. CI/CD pipeline

---

## OBJETIVO FINAL

Backend profesional, dockerizado, seguro y escalable que:
- ✅ Gestiona 3 roles de usuario con permisos diferenciados
- ✅ Persiste todos los datos en PostgreSQL
- ✅ Integra Claude y Gemini para análisis clínico automatizado
- ✅ Envía emails con credenciales a nuevos usuarios
- ✅ Proporciona APIs RESTful documentadas con Swagger
- ✅ Soporta toda la lógica de negocio del sistema S-FMDRS
- ✅ Se conecta seamlessly con la aplicación React Native
- ✅ Incluye datos de prueba para demostración inmediata

**IMPORTANTE**: Este backend debe reemplazar completamente los datos mock de RapidNative y proporcionar toda la funcionalidad real del sistema.