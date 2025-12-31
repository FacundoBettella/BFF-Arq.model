# 📋 GUÍA ARQUITECTÓNICA COMPLETA - BFF (Backend for Frontend)

## 1. TIPO DE ARQUITECTURA

Este tipo de proyecto implementa una arquitectura híbrida que combina varios patrones oficiales:  

### 🏗️ Clean Architecture / Hexagonal Architecture (Ports & Adapters)

- **Core/Domain Layer**:  Lógica de negocio pura sin dependencias externas
- **Application Layer**: Casos de uso y orquestación
- **Infrastructure Layer**: Implementaciones concretas y adaptadores externos
- **Entrypoints**: Controladores HTTP que actúan como adaptadores de entrada

### 📐 CQRS (Command Query Responsibility Segregation)

- **Commands**: Operaciones que modifican estado (`createOrderCommand`, `updateUserCommand`)
- **Queries**: Operaciones de solo lectura (`getUserQuery`, `getProductsQuery`)
- Separación clara entre lecturas y escrituras

### 🎯 Mediator Pattern

- Desacoplamiento total entre controladores y handlers
- Uso de Mediator para enrutar Commands/Queries a sus respectivos handlers
- Implementación basada en contenedores de IoC

### 🔷 Domain-Driven Design (DDD)

- Organización por features/dominios (users, orders, products, payments)
- Aggregates y Entities en la capa de dominio
- Domain Services para lógica de negocio compleja
- Repositories para abstracción de persistencia

---

## 2. ESTRUCTURA DE CAPAS

```
┌─────────────────────────────────────────────────────┐
│           ENTRYPOINTS (Controllers)                 │
│         - REST API Controllers                      │
│         - Decorators (@Controller, @Get, @Post)     │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│            APPLICATION LAYER                        │
│         - Commands & Queries (CQRS)                 │
│         - Handlers (MediatorRequestHandler)         │
│         - DTOs (Data Transfer Objects)              │
│         - Application Services                      │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│              DOMAIN LAYER                           │
│         - Entities & Aggregates                     │
│         - Domain Services                           │
│         - Domain Interfaces (IServices)             │
│         - Domain Errors                             │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│          INFRASTRUCTURE LAYER                       │
│         - API Clients (External Services)           │
│         - Repositories (Data Access)                │
│         - Mappers                                   │
│         - Framework (HttpClient, Cache, etc.)       │
└─────────────────────────────────────────────────────┘
```

---

## 3. TECNOLOGÍAS Y HERRAMIENTAS PRINCIPALES

### 🚀 Backend Framework

- **Node.js** (LTS version recomendada)
- **Express. js** - Framework web minimalista y extensible
- **NestJS** - Framework opinionado con arquitectura modular
- **TypeScript** - Lenguaje tipado para mejor mantenibilidad

### 📦 Dependency Injection

- **TypeDI / InversifyJS / TSyringe** - Contenedor IoC
- **NestJS DI Container** - Sistema nativo de inyección de dependencias
- Decorador `@Service()` / `@Injectable()` para registro de servicios
- Inyección mediante `@Inject()`
- Soporte para scopes:  Singleton, Request, Transient

### 🔌 HTTP Client & Resilience

- **Axios** - Cliente HTTP con interceptors
- **axios-retry** - Reintentos automáticos con backoff exponencial
- **Opossum / Brakes** - Circuit Breaker pattern
- Decorador `@WithBreaker()` para resiliencia

### 🧪 Testing

- **Jest / Vitest** - Framework de testing
- **ts-jest** - Soporte TypeScript
- **Supertest** - Testing de APIs HTTP
- **jest-when / sinon** - Mocking avanzado
- Coverage configurado con thresholds

### 📊 Logging & Monitoring

- **Winston / Pino / log4js** - Sistema de logging estructurado
- **cls-hooked / AsyncLocalStorage** - Context propagation (Correlation IDs)
- Trazabilidad con `X-Correlation-ID` o `X-Request-ID`
- Logging interceptors para request/response tracking

### 💾 Cache

- **ioredis** - Cliente Redis enterprise-ready
- **node-cache** - Cache en memoria simple
- Sistema de cache con TTL y buckets configurables

### 🔐 Seguridad

- **jsonwebtoken / jwt-decode** - Manejo de tokens JWT
- **@nestjs/jwt** - Módulo JWT para NestJS
- **helmet** - Seguridad HTTP headers
- **express-rate-limit** - Rate limiting
- **class-validator** - Validación de DTOs
- **class-transformer** - Transformación de datos
- Middlewares de autenticación y autorización
- CORS configurado según necesidades

### 📝 Documentación API

- **swagger-jsdoc** - Generación de specs OpenAPI
- **swagger-ui-express** - UI interactiva
- **@nestjs/swagger** - Integración Swagger para NestJS

### 🏗️ Build & Deploy

- **Docker** - Containerización
- **Kubernetes / OpenShift** - Orquestación
- **GitHub Actions / GitLab CI / Jenkins** - CI/CD
- **Docker Hub / AWS ECR / Quay Registry** - Registro de imágenes

---

## 4. PATRONES DE DISEÑO IMPLEMENTADOS

### 🎨 Creacionales

- **Factory Pattern**:  Creación de instancias de HTTP clients, loggers, etc.
- **Singleton**:  Container de IoC, configuraciones
- **Builder Pattern**: Construcción de objetos complejos (DTOs, Requests)

### 🔧 Estructurales

- **Adapter Pattern**: API Clients adaptando servicios externos
- **Decorator Pattern**:  Decoradores custom (@Controller, @Get, @Service, etc.)
- **Proxy Pattern**: Middlewares como proxies de request, proxy services
- **Facade Pattern**: HTTP Service Facade para abstraer comunicaciones

### ⚙️ Comportamiento

- **Mediator/Dispatcher Pattern**: Desacoplamiento de comandos/queries
- **Chain of Responsibility**: Middlewares en Express/NestJS
- **Strategy Pattern**: Lógicas variables según contexto
- **Repository Pattern**: Abstracción de acceso a datos
- **Circuit Breaker Pattern**: Protección contra fallos en cascada
- **Observer Pattern**: Event-driven architecture (opcional)

---

## 5. FEATURES CLAVE DEL FRAMEWORK

### 🎯 Sistema de Decoradores

```typescript
// Express/Custom Framework
@Controller('path')          // Define controladores
@Get({ query: QueryClass })  // Métodos HTTP
@Post({ body: CommandClass })
@Middleware(AuthMiddleware)  // Middlewares
@Service()                   // Inyección de dependencias
@Handler(QueryClass)         // Handlers del Mediator
@WithBreaker()               // Circuit Breakers
@Cached({ ttl: 300 })        // Cache decorator

// NestJS
@Controller('path')
@Get(':id')
@Post()
@UseGuards(AuthGuard)
@UseInterceptors(LoggingInterceptor)
@Injectable()
@RequestHandler(QueryClass)
@Public()                    // Rutas públicas sin auth
```

### 📡 Sistema de Mediator/Dispatcher

- Auto-discovery de handlers mediante reflection
- Routing automático basado en tipos
- Manejo de errores centralizado
- Pipeline de behaviors (validación, logging, etc.)
- Request-scoped para contexto aislado por petición

```typescript
// Dispatcher Pattern
Controller → Dispatcher → RequestHandler → Service → External API

// Flujo de ejecución: 
// 1. Controller recibe request HTTP
// 2. Crea Query/Command object
// 3. Dispatcher resuelve el handler apropiado vía metadata
// 4. Handler ejecuta lógica de negocio
// 5. Retorna resultado al controller
```

### 🌐 HTTP Client Framework

- Configuración centralizada por servicio
- Interceptors de request/response
- Logging automático con correlation IDs
- Reintentos y circuit breakers
- Timeout management
- Request/Response transformation

### 🧱 Application Framework

- Sistema de middleware con safety wrapper
- Auto-registro de rutas y controladores
- Validación automática de DTOs con `class-validator`
- Error handling centralizado
- Transform pipes para conversión automática

---

## 6. ORGANIZACIÓN POR FEATURES (Vertical Slicing)

### Opción 1: Express/Custom Framework

```
src/
├── core/
│   ├── application/
│   │   └── features/
│   │       └── users/
│   │           ├── commands/              # Operaciones de escritura
│   │           │   ├── createUser/
│   │           │   │   ├── createUserCommand.ts
│   │           │   │   └── createUserCommandHandler.ts
│   │           │   └── updateUser/
│   │           ├── queries/               # Operaciones de lectura
│   │           │   ├── getUser/
│   │           │   │   ├── getUserQuery.ts
│   │           │   │   └── getUserQueryHandler.ts
│   │           │   └── listUsers/
│   │           ├── dtos/                  # Data Transfer Objects
│   │           └── validators/            # Validaciones
│   └── domain/
│       └── features/
│           └── users/
│               ├── entities/              # Entidades de dominio
│               ├── aggregates/            # Agregados
│               ├── services/              # Servicios de dominio
│               └── interfaces/            # Contratos
├── infrastructure/
│   └── features/
│       └── users/
│           ├── repositories/              # Implementación de repos
│           ├── mappers/                   # Data mappers
│           └── services/                  # Servicios de infraestructura
└── entrypoints/
    └── controllers/
        └── users/
            └── userController.ts          # Controlador HTTP
```

### Opción 2: NestJS

```
src/
├── api/                                   # CAPA DE PRESENTACIÓN
│   ├── main.ts                            # Entry point
│   └── controllers/                       # Controladores REST por dominio
│       ├── assignations/
│       ├── combo/
│       ├── consistency/
│       └── health/
│
├── core/                                  # CAPA DE DOMINIO
│   ├── constants/
│   ├── features/                          # Features por dominio
│   │   ├── health/
│   │   └── business-domain/
│   │       ├── queries/                   # Queries (lectura)
│   │       ├── commands/                  # Commands (escritura)
│   │       ├── dtos/                      # Data Transfer Objects
│   │       └── services/                  # Servicios de dominio
│   ├── seed-work/                         # Building blocks reutilizables
│   │   ├── dispatcher. ts
│   │   ├── request-handler-decorator.ts
│   │   ├── cqrs/
│   │   ├── current-user-accessor/
│   │   ├── calculators/
│   │   ├── groupers/
│   │   └── interfaces/
│   └── utils/
│
└── infrastructure/                        # CAPA DE INFRAESTRUCTURA
    ├── bootstrap/                         # Configuración y arranque
    │   ├── bootstrap.ts
    │   ├── config/
    │   │   ├── app/                       # Configuración de la app
    │   │   │   ├── swagger-config.ts
    │   │   │   ├── cors-config.ts
    │   │   │   ├── pipes-config.ts
    │   │   │   └── body-parser-config.ts
    │   │   └── providers/                 # Providers personalizados
    │   ├── middlewares/
    │   │   ├── auth-guard.ts
    │   │   └── correlation-id-middleware.ts
    │   ├── interceptors/
    │   │   └── logging-interceptor.ts
    │   ├── modules/                       # Módulos NestJS
    │   │   ├── app.module.ts
    │   │   └── features/
    │   │       ├── business. module.ts
    │   │       ├── health.module.ts
    │   │       ├── logger.module.ts
    │   │       └── currentUser.module.ts
    │   ├── http-clients/                  # Clientes HTTP configurados
    │   └── seed-work/                     # Utilidades de infraestructura
    │       └── logger/
    └── features/                          # Implementaciones de servicios
        ├── business-domain/
        │   ├── service. ts
        │   └── proxy-service.ts
        └── consistency/
```

---

## 7. CONFIGURACIÓN Y AMBIENTES

### Gestión de Configuración

```
config/
├── default.yaml          // Configuración base
├── development.yaml      // Desarrollo local
├── ci.yaml              // Continuous Integration
├── sandbox.yaml         // QA/Testing
├── staging.yaml         // UAT/Pre-producción
├── production.yaml      // Producción
└── test.yaml           // Testing
```

### Estructura de Configuración YAML

```yaml
# development.yaml
NODE_ENV: development
APP_NAME: bff-service
PORT: 3000

CORS: 
  ORIGIN: ["http://localhost:3000", "http://localhost:4200"]
  METHODS: ["GET", "POST", "PUT", "PATCH", "DELETE"]
  CREDENTIALS: true

EXTERNAL_APIS:
  SERVICE_A: 
    BASE_URL: https://api-dev.example.com
    TIMEOUT: 5000
  SERVICE_B: 
    BASE_URL: https://api-dev.example.com/v2
    TIMEOUT: 3000

JWT:
  SECRET: ${JWT_SECRET}
  EXPIRES_IN: 3600

HEALTH_CHECK:
  LIVENESS_PATH: /health/liveness
  READINESS_PATH: /health/readiness

DEPLOYMENT:
  REPLICAS: 2
  CPU_LIMIT: 500m
  MEMORY_LIMIT: 512Mi
```

### Variables de Entorno

- **Secrets**: Credenciales, API keys, tokens (JWT_SECRET, DB_PASSWORD)
- **ConfigMaps**: Configuración no sensible
- **Environment-specific**: URLs, timeouts, feature flags

### Health Checks

```
GET /health              // Basic health
GET /health/liveness     // Kubernetes/OpenShift liveness probe
GET /health/readiness    // Kubernetes/OpenShift readiness probe
GET /metrics             // Prometheus metrics (opcional)
```

---

## 8. CI/CD PIPELINE

### Pipeline Stages

```yaml
stages:
  - install     # Instalar dependencias
  - lint        # Linting y formateo
  - test        # Tests unitarios e integración
  - quality     # Code quality analysis
  - build       # Build de aplicación
  - docker      # Build de imagen Docker
  - deploy      # Deployment a ambientes
```

### Ejemplo de Pipeline (GitLab CI)

```yaml
# .gitlab-ci.yml

variables:
  DOCKER_REGISTRY: quay.io
  IMAGE_NAME: $CI_PROJECT_NAME

stages:
  - package
  - test
  - quality
  - build-image
  - deploy

package: 
  stage: package
  script: 
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

test: 
  stage: test
  script:
    - npm run test: cov
  coverage: '/All files[^|]*\|[^|]*\s+([\d\. ]+)/'
  artifacts:
    reports: 
      coverage_report:
        coverage_format: cobertura
        path:  coverage/cobertura-coverage. xml

quality:
  stage: quality
  script:
    - sonar-scanner
  only:
    - main
    - staging
    - development

build-image:
  stage:  build-image
  script: 
    - docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
    - docker push $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA

deploy-dev:
  stage: deploy
  script:
    - oc apply -f k8s/deployment-dev.yaml
  only:
    - development
  when: on_success

deploy-staging:
  stage: deploy
  script:
    - oc apply -f k8s/deployment-staging.yaml
  only:
    - staging
  when: manual

deploy-prod:
  stage: deploy
  script:
    - oc apply -f k8s/deployment-prod.yaml
  only:
    - main
  when: manual
```

### Detalle de Etapas

1. **Install/Package**:  `npm ci` (reproducible installs)
2. **Lint**: eslint + prettier
3. **Test**:
   - Unit tests con coverage mínimo (>80%)
   - Integration tests
   - E2E tests (opcional)
4. **Quality**: SonarQube analysis (code smells, vulnerabilities, coverage)
5. **Build**: TypeScript compilation
6. **Docker Build**: Multi-stage Docker build → Registry (Quay/ECR)
7. **Deploy**:
   - Dev → automático
   - Staging → automático con aprobación
   - Production → manual con aprobación

### GitFlow Strategy

```
main (protected)     → Producción
  ↑
staging (protected)  → UAT/Pre-producción
  ↑
development          → Integración continua
  ↑
feature/*           → Desarrollo de features
hotfix/*            → Parches urgentes
release/*           → Preparación de releases
```

**Merge Requests**:
- Requieren aprobación
- Pasan quality gates
- Tests exitosos

---

## 9. PRINCIPIOS ARQUITECTÓNICOS APLICADOS

### ✅ SOLID Principles

- **S** - Single Responsibility:  Cada clase/módulo una responsabilidad
- **O** - Open/Closed: Extensible mediante decoradores y plugins
- **L** - Liskov Substitution: Interfaces respetadas
- **I** - Interface Segregation: Interfaces específicas por feature
- **D** - Dependency Inversion: Dependencias hacia abstracciones

### ✅ Separation of Concerns

Capas bien definidas con responsabilidades claras: 
- **API Layer**: Manejo de HTTP, validación de entrada
- **Core Layer**: Lógica de negocio pura
- **Infrastructure Layer**:  Implementaciones técnicas

### ✅ Dependency Rule

Las dependencias apuntan hacia adentro (hacia el dominio):
```
Infrastructure → Application → Domain
     ↓               ↓            ↓
  (adapters)    (use cases)  (entities)
```

### ✅ DRY (Don't Repeat Yourself)

Framework reutilizable, seed-work compartido, utilities

### ✅ YAGNI (You Aren't Gonna Need It)

No sobre-ingeniería, solo lo necesario

### ✅ Request-Scoped Services

Context aislado por petición HTTP para thread-safety

### ✅ Metadata-Driven Architecture

Decorators para configuración declarativa

---

## 10. ESTRUCTURA DE ARCHIVOS COMPLETA

### Express/Custom Framework

```
project-root/
├── src/
│   ├── core/
│   │   ├── application/
│   │   │   ├── behaviour/           # CQRS base classes
│   │   │   │   ├── command.ts
│   │   │   │   ├── query. ts
│   │   │   │   └── dto.ts
│   │   │   └── features/            # Features por dominio
│   │   └── domain/
│   │       ├── entity.ts            # Base entity
│   │       ├── domainError.ts       # Domain errors
│   │       └── features/            # Domain models
│   ├── infrastructure/
│   │   ├── apis/                    # External API clients
│   │   ├── features/                # Infrastructure implementations
│   │   ├── bootstraping/            # App initialization
│   │   │   ├── environments/        # Config loaders
│   │   │   ├── loggers/             # Logger setup
│   │   │   └── middlewares/         # Global middlewares
│   │   ├── seedWork/                # Infrastructure utilities
│   │   └── utils/                   # Helper functions
│   ├── entrypoints/
│   │   ├── www. ts                   # App entry point
│   │   └── controllers/             # HTTP controllers
│   └── libs/                        # Shared libraries
├── utils/
│   └── framework/                   # Custom framework
│       ├── app.ts                   # App class
│       ├── appController.ts         # Controller base
│       ├── decorators/              # Custom decorators
│       ├── mediator/                # Mediator implementation
│       ├── httpClient/              # HTTP client framework
│       ├── cache/                   # Cache system
│       └── interfaces/              # Framework interfaces
├── test/                            # Tests mirror src structure
├── mocks/                           # Test mocks
├── config/                          # Environment configs
├── coverage/                        # Test coverage reports
├── build/                           # Compiled output
├── Dockerfile                       # Container definition
├── docker-compose.yml               # Local development
├── jest.config.js                   # Test configuration
├── tsconfig.json                    # TypeScript config
├── package.json                     # Dependencies
└── README.md                        # Documentation
```

### NestJS

```
project-root/
├── src/
│   ├── api/                         # CAPA DE PRESENTACIÓN
│   │   ├── main. ts                  # Entry point
│   │   └── controllers/             # Controladores REST
│   │       ├── assignations/
│   │       ├── combo/
│   │       ├── consistency/
│   │       └── health/
│   ├── core/                        # CAPA DE DOMINIO
│   │   ├── constants/
│   │   ├── features/                # Features por dominio
│   │   │   ├── health/
│   │   │   └── business-domain/
│   │   │       ├── queries/
│   │   │       ├── commands/
│   │   │       ├── dtos/
│   │   │       └── services/
│   │   ├── seed-work/               # Building blocks
│   │   │   ├── dispatcher.ts
│   │   │   ├── request-handler-decorator. ts
│   │   │   ├── cqrs/
│   │   │   ├── current-user-accessor/
│   │   │   └── interfaces/
│   │   └── utils/
│   └── infrastructure/              # CAPA DE INFRAESTRUCTURA
│       ├── bootstrap/
│       │   ├── bootstrap.ts
│       │   ├── config/
│       │   │   ├── app/
│       │   │   │   ├── swagger-config.ts
│       │   │   │   ├── cors-config.ts
│       │   │   │   ├── pipes-config.ts
│       │   │   │   └── body-parser-config.ts
│       │   │   └── providers/
│       │   ├── middlewares/
│       │   │   ├── auth-guard.ts
│       │   │   └── correlation-id-middleware.ts
│       │   ├── interceptors/
│       │   │   └── logging-interceptor.ts
│       │   ├── modules/
│       │   │   ├── app.module.ts
│       │   │   └── features/
│       │   │       ├── business.module.ts
│       │   │       ├── health.module. ts
│       │   │       ├── logger.module.ts
│       │   │       └── currentUser.module.ts
│       │   ├── http-clients/
│       │   └── seed-work/
│       │       └── logger/
│       └── features/
│           └── business-domain/
│               ├── service.ts
│               └── proxy-service.ts
├── config/                          # Configuración por ambiente
│   ├── default. yaml
│   ├── development.yaml
│   ├── ci. yaml
│   ├── sandbox.yaml
│   ├── staging.yaml
│   └── production.yaml
├── test/                            # Tests
├── k8s/                             # Kubernetes/OpenShift manifests
├── Dockerfile
├── docker-compose.yml
├── nest-cli.json
├── tsconfig.json
├── jest.config.js
├── . gitlab-ci.yml / .github/workflows/
└── README.md
```

---

## 11. SEGURIDAD

### 🔒 Autenticación JWT

#### Guard de Autenticación

```typescript
// auth-guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector
  ) {}

  canActivate(context: ExecutionContext): boolean {
    // Verificar si la ruta es pública
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler()
    );
    if (isPublic) return true;

    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) {
      throw new UnauthorizedException('Token no proporcionado');
    }

    try {
      // Validar y decodificar token
      const payload = this.jwtService.verify(token);
      
      // Inyectar payload en el request
      request. user = payload;
      
      return true;
    } catch (error) {
      throw new UnauthorizedException('Token inválido o expirado');
    }
  }

  private extractToken(request: Request): string | null {
    const authorization = request.headers.authorization;
    if (!authorization) return null;

    // Soportar múltiples esquemas (Bearer, etc.)
    const [scheme, token] = authorization.split(' ');
    return scheme === 'Bearer' ? token :  null;
  }
}
```

**Características**:
- Validación de token JWT del header `Authorization`
- Soporte para múltiples esquemas de autenticación
- Decodifica payload y lo inyecta en el request
- Decorator `@Public()` para endpoints públicos
- Lanza `UnauthorizedException` si falla

```typescript
// Uso en controladores
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {
  
  @Get()
  findAll() {
    // Ruta protegida
  }

  @Public()
  @Post('login')
  login() {
    // Ruta pública
  }
}
```

### 🔑 Current User Accessor

Request-scoped service que extrae información del usuario: 

```typescript
@Injectable({ scope: Scope.REQUEST })
export class CurrentUserAccessor {
  constructor(@Inject(REQUEST) private request: Request) {}

  get user(): User {
    return this.request.user;
  }

  get userId(): string {
    return this.request.user?. sub || this.request.user?.userId;
  }

  get roles(): string[] {
    return this.request.user?.roles || [];
  }

  hasRole(role: string): boolean {
    return this.roles.includes(role);
  }
}
```

**Uso en servicios**:
```typescript
@Injectable()
export class OrdersService {
  constructor(private currentUser: CurrentUserAccessor) {}

  async createOrder(dto: CreateOrderDto) {
    const userId = this.currentUser.userId;
    // Lógica de negocio con userId
  }
}
```

### 📊 Correlation ID

Middleware de trazabilidad: 

```typescript
// correlation-id-middleware.ts
@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Extraer o generar Correlation ID
    let correlationId = req.headers['x-correlation-id'] as string;
    
    if (! correlationId) {
      correlationId = `AUTO-${uuidv4()}`;
    }

    // Inyectar en headers de request y response
    req. headers['x-correlation-id'] = correlationId;
    res.setHeader('X-Correlation-ID', correlationId);

    next();
  }
}
```

**Características**:
- Header:  `X-Correlation-ID`
- Genera UUID v4 si no existe
- Prefijo `AUTO-` para IDs autogenerados
- Facilita trazabilidad entre microservicios
- Se propaga a servicios externos

### 🛡️ Helmet. js

Protección contra vulnerabilidades web comunes:

```typescript
// main.ts
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Helmet security headers
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc:  ["'self'"],
      },
    },
    hsts: {
      maxAge:  31536000,
      includeSubDomains: true,
    },
  }));

  await app.listen(3000);
}
```

### 🔐 CORS Configurable

```typescript
// cors-config.ts
export const corsConfig = (config: ConfigService) => ({
  origin: config.get<string[]>('CORS. ORIGIN'),
  methods: config.get<string[]>('CORS.METHODS'),
  credentials: config.get<boolean>('CORS.CREDENTIALS'),
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Correlation-ID'],
  exposedHeaders: ['X-Correlation-ID'],
});

// main.ts
app.enableCors(corsConfig(configService));
```

```yaml
# config/development.yaml
CORS:
  ORIGIN: 
    - http://localhost:3000
    - http://localhost:4200
  METHODS:
    - GET
    - POST
    - PUT
    - PATCH
    - DELETE
  CREDENTIALS: true
```

### ✅ Validación de Datos

```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,              // Auto-transformar a tipos
    whitelist: true,              // Remover props no declaradas
    forbidNonWhitelisted: true,   // Error si hay props extras
    transformOptions: {
      enableImplicitConversion: true,
    },
  })
);

// DTO con validaciones
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(18)
  @Max(120)
  age: number;

  @IsArray()
  @IsString({ each: true })
  @IsOptional()
  roles?: string[];
}
```

---

## 12. LOGGING & OBSERVABILIDAD

### 📡 Logging Interceptor

```typescript
// logging-interceptor.ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, headers, body } = request;
    const correlationId = headers['x-correlation-id'];

    // Log de REQUEST
    this.logger.log({
      message: 'HTTP REQUEST',
      method,
      url,
      correlationId,
      headers:  this.sanitizeHeaders(headers),
      body: this.sanitizeBody(body),
    });

    const now = Date.now();

    return next.handle().pipe(
      tap((data) => {
        const response = context.switchToHttp().getResponse();
        const { statusCode } = response;
        const responseTime = Date.now() - now;

        // Log de RESPONSE
        this.logger.log({
          message: 'HTTP RESPONSE',
          method,
          url,
          statusCode,
          correlationId,
          responseTime:  `${responseTime}ms`,
          body: this.sanitizeBody(data),
        });
      }),
      catchError((error) => {
        const responseTime = Date.now() - now;

        // Log de ERROR
        this.logger.error({
          message: 'HTTP ERROR',
          method,
          url,
          correlationId,
          responseTime: `${responseTime}ms`,
          error: error.message,
          stack: error.stack,
        });

        throw error;
      })
    );
  }

  private sanitizeHeaders(headers: any): any {
    const sanitized = { ...headers };
    // Ocultar información sensible
    if (sanitized.authorization) {
      sanitized.authorization = '***REDACTED***';
    }
    return sanitized;
  }

  private sanitizeBody(body: any): any {
    if (!body) return body;
    
    const sensitiveFields = ['password', 'token', 'secret', 'apiKey'];
    const sanitized = { ...body };

    sensitiveFields.forEach(field => {
      if (sanitized[field]) {
        sanitized[field] = '***REDACTED***';
      }
    });

    return sanitized;
  }
}
```

**Se registra**:
- ✅ HTTP REQUEST:  URL, método, headers, body
- ✅ HTTP RESPONSE: status code, response time, body
- ✅ Errores y excepciones con stack trace
- ✅ Correlation ID para trazabilidad
- ✅ Sanitización de información sensible

**Aplicación global**:
```typescript
// main.ts
app.useGlobalInterceptors(
  new LoggingInterceptor(app.get(Logger))
);
```

### 📊 Structured Logging

```typescript
// logger.service.ts
@Injectable()
export class Logger {
  private winston: winston.Logger;

  constructor() {
    this.winston = winston.createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format:  winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
      ),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
      ],
    });
  }

  log(message: any, context?: string) {
    this.winston.info({ ... message, context });
  }

  error(message: any, trace?: string, context?: string) {
    this.winston.error({ ...message, trace, context });
  }

  warn(message: any, context?: string) {
    this.winston. warn({ ...message, context });
  }

  debug(message: any, context?: string) {
    this.winston.debug({ ...message, context });
  }
}
```

---

## 13. IMPLEMENTACIÓN PASO A PASO

### Fase 1: Setup Inicial (Semana 1)

- ✅ Inicializar proyecto Node.js + TypeScript
- ✅ Configurar ESLint + Prettier
- ✅ Setup Jest para testing
- ✅ Configurar estructura de carpetas
- ✅ Implementar configuración por ambientes (YAML)
- ✅ Setup Docker y docker-compose

### Fase 2: Framework Base (Semana 2-3)

- ✅ Implementar sistema de decoradores
- ✅ Crear base classes (Command, Query, Entity)
- ✅ Implementar Dispatcher/Mediator pattern
- ✅ Setup Dependency Injection (NestJS/TypeDI)
- ✅ Crear HttpClient con interceptors
- ✅ Configurar módulos principales

### Fase 3: Infrastructure (Semana 4)

- ✅ Implementar logging con correlation IDs
- ✅ Configurar Circuit Breaker
- ✅ Implementar cache system
- ✅ Setup error handling global
- ✅ Crear middlewares (auth, validation)
- ✅ Configurar guards e interceptors
- ✅ Setup Swagger/OpenAPI

### Fase 4: Primera Feature (Semana 5)

- ✅ Implementar una feature completa (ej: Users)
- ✅ Crear controlador con decoradores
- ✅ Implementar Commands y Queries
- ✅ Crear handlers y servicios
- ✅ Implementar proxy services
- ✅ Crear tests unitarios e integración
- ✅ Documentar con Swagger

### Fase 5: DevOps (Semana 6)

- ✅ Crear Dockerfile multi-stage
- ✅ Configurar docker-compose para local
- ✅ Setup CI/CD pipeline (GitLab/GitHub)
- ✅ Configurar health checks y probes
- ✅ Crear manifests de Kubernetes/OpenShift
- ✅ Deploy a ambiente de desarrollo
- ✅ Configurar SonarQube

---

## 14. CHECKLIST DE CALIDAD

### 📋 Code Quality

- [ ] Cobertura de tests > 80%
- [ ] Linting sin errores
- [ ] Type safety sin `any` explícitos
- [ ] Documentación de código (JSDoc/TSDoc)
- [ ] Code review aprobado
- [ ] SonarQube quality gate passed

### 🔒 Seguridad

- [ ] Dependencias sin vulnerabilidades críticas (`npm audit`)
- [ ] Secrets no en código (usar variables de entorno)
- [ ] HTTPS enforced en producción
- [ ] Rate limiting implementado
- [ ] Input validation en todos los endpoints
- [ ] JWT con expiración configurada
- [ ] Helmet.js configurado
- [ ] CORS restrictivo

### ⚡ Performance

- [ ] Response time < 200ms (endpoints simples)
- [ ] Cache implementado donde corresponde
- [ ] Connection pooling configurado
- [ ] Timeouts apropiados en HTTP clients
- [ ] Payload size limits configurados

### 📊 Observabilidad

- [ ] Logging estructurado (JSON)
- [ ] Correlation IDs en todas las requests
- [ ] Health checks funcionando (liveness/readiness)
- [ ] Error tracking configurado
- [ ] Request/Response logging

### 🏗️ Deployment

- [ ] Dockerfile optimizado (multi-stage)
- [ ] Image size < 500MB
- [ ] CI/CD pipeline funcionando
- [ ] Kubernetes/OpenShift manifests validados
- [ ] Probes configuradas correctamente
- [ ] Resource limits definidos
- [ ] ConfigMaps y Secrets externalizados

---

## 15. TECNOLOGÍAS ALTERNATIVAS

### Framework Web

- **Express.js** - Minimalista, flexible, gran ecosistema
- **NestJS** ← **Recomendado para BFF** - Opinionado, modular, TypeScript-first
- **Fastify** - Más rápido, moderno, schema-based
- **Koa** - Minimalista, del team de Express

### Dependency Injection

- **NestJS DI Container** ← **Recomendado si usas NestJS**
- **TypeDI** - Decorators, similar a NestJS
- **InversifyJS** - Más maduro, SOLID-compliant
- **TSyringe** - Microsoft, lightweight
- **Awilix** - Simple y efectivo

---

## 16. MEJORES PRÁCTICAS

### 🎯 Código

- ✅ Usar TypeScript **strict mode**
- ✅ Evitar `any`, usar tipos específicos o `unknown`
- ✅ Preferir **composición sobre herencia**
- ✅ Funciones puras donde sea posible
- ✅ Inmutabilidad de datos (usar `readonly`, spread operators)
- ✅ Separar lógica de negocio de infraestructura
- ✅ Usar interfaces para contratos

### 🔐 Seguridad

- ✅ **Nunca** commitear secrets
- ✅ Usar **variables de entorno** para configuración sensible
- ✅ **Validar toda entrada** de usuario
- ✅ **Sanitizar outputs** (prevenir XSS)
- ✅ Mantener **dependencias actualizadas** (`npm audit`)
- ✅ Usar **Helmet. js** para headers de seguridad
- ✅ Implementar **rate limiting**
- ✅ **JWT con expiración** corta
- ✅ **HTTPS** en producción

### ⚡ Performance

- ✅ **Lazy loading** donde aplique
- ✅ **Cache estratégico** (Redis, in-memory)
- ✅ **Pagination** en listas grandes
- ✅ **Compression** habilitado (gzip/brotli)
- ✅ **Connection pooling** para DB y HTTP
- ✅ **Timeouts** apropiados
- ✅ **Async/await** para I/O operations
- ✅ Evitar **N+1 queries**

### 🏗️ Deployment

- ✅ **Multi-stage Dockerfile** para reducir tamaño
- ✅ **Health checks** (liveness/readiness)
- ✅ **Graceful shutdown**
- ✅ **Resource limits** en Kubernetes
- ✅ **Horizontal Pod Autoscaling**
- ✅ **Rolling updates** con zero downtime

```dockerfile
# Multi-stage Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . . 
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package. json ./

ENV NODE_ENV=production
EXPOSE 3000

CMD ["node", "dist/main.js"]
```

---

## 17. RESUMEN EJECUTIVO

Esta arquitectura representa un **BFF (Backend for Frontend) enterprise-grade** que implementa:

### ✅ Arquitectura

- **Clean Architecture + DDD + CQRS + Dispatcher/Mediator**
- **Layered Architecture** con separación clara de responsabilidades
- **Request-scoped services** para contexto aislado
- **Metadata-driven** con decorators

### ✅ Framework & Stack

- **NestJS** (opcionalmente Express) con **TypeScript 5+**
- **Dependency Injection** completa
- **Request/Response Pipeline** con interceptors
- **Custom decorators** para extensibilidad

### ✅ Resiliencia

- **Circuit Breakers** para protección contra fallos
- **Retry strategies** con backoff exponencial
- **Timeouts** configurables
- **Graceful degradation**

### ✅ Observabilidad

- **Structured logging** (JSON)
- **Correlation IDs** end-to-end
- **Request/Response tracking**
- **Health checks** (liveness/readiness)
- **Metrics** (opcional:  Prometheus)

### ✅ Seguridad

- **JWT Authentication** con guards
- **Input validation** automática
- **Rate limiting**
- **Helmet.js** security headers
- **CORS** configurable
- **Secrets management** externalized

### ✅ Testing

- **Unit tests** con Jest (>80% coverage)
- **Integration tests** para APIs
- **E2E tests** (opcional)
- **Mocking** de dependencias

### ✅ CI/CD

- **Automated pipeline** (GitLab CI / GitHub Actions)
- **Quality gates** (SonarQube)
- **Docker** containerization
- **Kubernetes/OpenShift** deployment
- **GitFlow** strategy

### ✅ Configuración

- **Multi-environment** (YAML configs)
- **Environment variables** para secrets
- **Feature flags** support

---

## 18. CASOS DE USO IDEALES

### 🏦 Aplicaciones Financieras

- Alta seguridad requerida (JWT, encryption)
- Trazabilidad completa (correlation IDs)
- Resiliencia crítica (circuit breakers)

### 🛒 E-commerce de Alto Tráfico

- Escalabilidad horizontal
- Cache strategies
- Performance optimization

### 🏥 Sistemas de Salud

- Compliance y auditoría
- Logging exhaustivo
- Validación estricta

### 📱 Mobile Backends

- BFF pattern ideal
- Optimización de payloads
- Versionado de APIs

### 🎮 Gaming Platforms

- Low latency
- Alta concurrencia
- Real-time capabilities

### 🚀 Cualquier Sistema que Requiera

- ✅ Alta disponibilidad (99.9%+)
- ✅ Escalabilidad horizontal
- ✅ Observabilidad completa
- ✅ Seguridad por diseño
- ✅ Mantenibilidad a largo plazo

---

## 19. PRÓXIMOS PASOS

### Para Empezar

1. **Clonar template** o inicializar proyecto
2. **Configurar ambientes** (development, staging, production)
3. **Implementar primera feature** completa
4. **Setup CI/CD** pipeline
5. **Deploy a desarrollo**

### Para Escalar

1. **Añadir más features** siguiendo la estructura
2. **Implementar cache** (Redis)
3. **Añadir métricas** (Prometheus)
4. **Configurar APM** (Application Performance Monitoring)
5. **Implementar rate limiting** avanzado

### Para Optimizar

1. **Analizar performance** con profiling
2. **Optimizar queries** y reducir N+1
3. **Implementar CDN** para assets estáticos
4. **Configurar load balancing**
5. **Horizontal Pod Autoscaling**

---

## 📚 Referencias

- [Clean Architecture (Robert C. Martin)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture. html)
- [Domain-Driven Design (Eric Evans)](https://www.domainlanguage.com/ddd/)
- [CQRS Pattern (Martin Fowler)](https://martinfowler.com/bliki/CQRS.html)
- [Microservices Patterns (Chris Richardson)](https://microservices.io/patterns/index.html)
- [The Twelve-Factor App](https://12factor.net/)
- [NestJS Documentation](https://docs.nestjs.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

**Versión**:  2.0.0  
**Última actualización**: 2025-12-31  
**Autor**: Facundo Bettella  
**Repositorio**: [FacundoBettella/Microfront-Arq](https://github.com/FacundoBettella/Microfront-Arq)

---

## 📄 Licencia

Este documento es de uso libre para propósitos educativos y comerciales. 
