# 📋 GUÍA ARQUITECTÓNICA COMPLETA - BFF con NestJS

## 1. TIPO DE ARQUITECTURA

Este proyecto implementa un **Backend for Frontend (BFF)** construido con **NestJS** y **TypeScript**, combinando varios patrones arquitectónicos empresariales:

### 🏗️ Layered Architecture (Arquitectura en Capas)

- **API Layer (Presentación)**: Controllers que exponen endpoints REST
- **Core Layer (Dominio)**: Lógica de negocio pura sin dependencias externas
- **Infrastructure Layer**:  Implementaciones técnicas, HTTP clients, configuración

### 📐 CQRS (Command Query Responsibility Segregation)

- **Commands**: Operaciones que modifican estado (`UpdateOrderCommand`, `CreateUserCommand`)
- **Queries**: Operaciones de solo lectura (`GetUserQuery`, `SearchProductsQuery`)
- Separación clara entre lecturas y escrituras mediante handlers específicos

### 🎯 Dispatcher/Mediator Pattern

- Desacoplamiento total entre controllers y handlers
- Dispatcher resuelve el handler apropiado mediante metadata (decoradores)
- Request-scoped para contexto aislado por petición HTTP
- Pipeline de behaviors (validación, logging, transformación)

### 🔷 Domain-Driven Design (DDD)

- Organización por **features/dominios** de negocio
- **Seed-work**:  Building blocks reutilizables (base classes, interfaces, utilities)
- **Domain Services**: Lógica de negocio compleja
- **DTOs**: Data Transfer Objects para comunicación entre capas

---

## 2. ESTRUCTURA DE CAPAS

```
┌─────────────────────────────────────────────────────┐
│        API LAYER (Controllers)                      │
│     - REST API Endpoints                            │
│     - DTOs & Validation                             │
│     - Swagger Documentation                         │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│         DISPATCHER (Mediator)                       │
│     - Request Routing                               │
│     - Handler Resolution (Metadata)                 │
│     - Request-Scoped Context                        │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│     CORE LAYER (Domain + Application)               │
│                                                      │
│   Features:                                          │
│     - Commands & Queries (CQRS)                     │
│     - Handlers (Business Logic)                     │
│     - Domain Services                               │
│     - DTOs & Interfaces                             │
│                                                      │
│   Seed-Work:                                        │
│     - Base Classes                                  │
│     - Type Guards & Validators                      │
│     - Current User Accessor                         │
│     - Calculators & Groupers                        │
└────────────────┬────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────┐
│     INFRASTRUCTURE LAYER                            │
│     - HTTP Clients (Axios)                          │
│     - Services & Proxy Services                     │
│     - Middlewares & Guards                          │
│     - Interceptors (Logging)                        │
│     - Bootstrap & Configuration                     │
│     - NestJS Modules                                │
└─────────────────────────────────────────────────────┘
```

---

## 3. STACK TECNOLÓGICO

### 🚀 Framework Core

- **NestJS 11.x** - Framework TypeScript-first con arquitectura modular
- **Node.js 18+** (LTS) - Runtime JavaScript
- **TypeScript 5.x** - Lenguaje tipado con strict mode

### 📦 Dependency Injection

- **NestJS DI Container** - Sistema nativo de inyección de dependencias
- **Scopes soportados**:
  - `Scope.DEFAULT` (Singleton) - Instancia única
  - `Scope.REQUEST` - Nueva instancia por petición HTTP
  - `Scope.TRANSIENT` - Nueva instancia en cada inyección

### 🔌 HTTP Client & Resilience

- **Axios** - Cliente HTTP con interceptors
- **axios-retry** - Reintentos automáticos con backoff exponencial
- **Custom HTTP Facade** - Abstracción sobre Axios
- Circuit Breaker pattern (opcional con Opossum)

### 🧪 Testing

- **Jest** - Framework de testing all-in-one
- **@nestjs/testing** - Utilities para testing de NestJS
- **Supertest** - Testing de APIs HTTP
- **Coverage** configurado con thresholds (>80%)

### 📊 Logging & Monitoring

- **Winston / Pino** - Logging estructurado en JSON
- **Custom Logger Module** - Integración con NestJS
- **Correlation IDs** - Trazabilidad end-to-end con `X-Correlation-ID`
- **Logging Interceptor** - Log automático de requests/responses

### 💾 Cache

- **@nestjs/cache-manager** - Módulo de cache integrado
- **cache-manager** - Cache manager flexible
- **ioredis** (opcional) - Cliente Redis para cache distribuido

### 🔐 Seguridad

- **@nestjs/jwt** - Manejo de tokens JWT
- **@nestjs/passport** (opcional) - Estrategias de autenticación
- **helmet** - Security headers HTTP
- **class-validator** - Validación de DTOs con decoradores
- **class-transformer** - Transformación automática de datos
- **@nestjs/throttler** - Rate limiting integrado

### 📝 Documentación API

- **@nestjs/swagger** - Generación automática de OpenAPI specs
- **swagger-ui-express** - UI interactiva para documentación

### 🏗️ Configuration & Environment

- **@nestjs/config** - Gestión de configuración por ambiente
- **js-yaml** - Parser de archivos YAML
- **Joi** (opcional) - Validación de schemas de configuración

### 🏗️ Build & Deploy

- **Docker** - Containerización multi-stage
- **Kubernetes / OpenShift** - Orquestación de contenedores
- **GitLab CI / GitHub Actions** - CI/CD pipelines
- **Quay / ECR / Docker Hub** - Container registries

---

## 4. PATRONES DE DISEÑO IMPLEMENTADOS

### 🎨 Creacionales

- **Factory Pattern**:  HTTP Client Factory, Logger Factory
- **Singleton**:  Módulos de NestJS (providers por defecto)
- **Builder Pattern**: DTOs complejos, request builders

### 🔧 Estructurales

- **Adapter Pattern**: HTTP Clients como adaptadores de servicios externos
- **Decorator Pattern**: Sistema de decoradores de NestJS (@Injectable, @Controller, etc.)
- **Proxy Pattern**: Proxy Services que añaden funcionalidad (logging, caching, retry)
- **Facade Pattern**: HTTP Service Facade para abstraer comunicación HTTP

### ⚙️ Comportamiento

- **Dispatcher/Mediator Pattern**: Desacoplamiento de controllers y handlers
- **Chain of Responsibility**: Middlewares, Guards, Interceptors, Pipes
- **Strategy Pattern**: Diferentes estrategias de autenticación, cache, etc.
- **Observer Pattern**: Sistema de eventos de NestJS (EventEmitter)
- **Template Method**:  Lifecycle hooks de NestJS

---

## 5. SISTEMA DE DISPATCHER (MEDIATOR)

### 🎯 Flujo de Ejecución

```typescript
// 1. Controller recibe HTTP Request
@Controller('orders')
export class OrdersController {
  constructor(private dispatcher: Dispatcher) {}

  @Post('search')
  async search(@Body() query: SearchOrdersQuery) {
    // 2. Delega al Dispatcher
    return this.dispatcher.dispatch(query);
  }
}

// 3. Dispatcher resuelve el Handler mediante metadata
@Injectable({ scope: Scope.REQUEST })
export class Dispatcher {
  private handlers = new Map();

  async dispatch<T>(request: IRequest): Promise<T> {
    const handlerClass = this.resolveHandler(request);
    const handler = this.moduleRef.get(handlerClass, { strict: false });
    return handler.handle(request);
  }

  private resolveHandler(request: IRequest): Type<IRequestHandler> {
    // Usa Reflect.getMetadata para obtener el handler asociado
    return Reflect.getMetadata(REQUEST_HANDLER_KEY, request.constructor);
  }
}

// 4. Handler ejecuta lógica de negocio
@RequestHandler(SearchOrdersQuery)
@Injectable()
export class SearchOrdersQueryHandler implements IRequestHandler<SearchOrdersQuery, OrderDto[]> {
  constructor(private ordersService: OrdersService) {}

  async handle(query: SearchOrdersQuery): Promise<OrderDto[]> {
    return this.ordersService.search(query);
  }
}

// 5. Service realiza la operación
@Injectable()
export class OrdersService {
  constructor(private httpClient: HttpClient) {}

  async search(query: SearchOrdersQuery): Promise<OrderDto[]> {
    return this.httpClient.post('/api/orders/search', query);
  }
}
```

### 📡 Decorador @RequestHandler

```typescript
// request-handler-decorator.ts
export const REQUEST_HANDLER_KEY = 'REQUEST_HANDLER';

export function RequestHandler(requestClass: Type<any>): ClassDecorator {
  return (target: any) => {
    // Almacena metadata:  Query/Command → Handler
    Reflect.defineMetadata(REQUEST_HANDLER_KEY, target, requestClass);
    
    // Marca la clase como Injectable
    Injectable()(target);
  };
}
```

### 🔄 Request-Scoped Dispatcher

```typescript
@Injectable({ scope: Scope.REQUEST })
export class Dispatcher {
  constructor(
    private moduleRef: ModuleRef,
    private currentUser: CurrentUserAccessor, // Request-scoped
  ) {}

  async dispatch<T>(request: IRequest): Promise<T> {
    // Contexto aislado por petición HTTP
    const handler = this.resolveHandler(request);
    return handler.handle(request);
  }
}
```

---

## 6. ESTRUCTURA DE PROYECTO NESTJS

```
project-root/
├── src/
│   ├── api/                                   # 📡 CAPA DE PRESENTACIÓN
│   │   ├── main.ts                            # Entry point de la aplicación
│   │   └── controllers/                       # Controladores REST por dominio
│   │       ├── assignations/
│   │       │   └── assignations.controller.ts
│   │       ├── combo/
│   │       │   └── combo.controller.ts
│   │       ├── consistency/
│   │       │   └── consistency.controller.ts
│   │       └── health/
│   │           └── health.controller.ts
│   │
│   ├── core/                                  # 🎯 CAPA DE DOMINIO
│   │   ├── constants/                         # Constantes del dominio
│   │   │   └── index.ts
│   │   │
│   │   ├── features/                          # Features organizadas por dominio
│   │   │   ├── health/
│   │   │   │   ├── queries/
│   │   │   │   │   ├── health-check.query.ts
│   │   │   │   │   └── health-check.handler.ts
│   │   │   │   └── dtos/
│   │   │   │       └── health-status.dto.ts
│   │   │   │
│   │   │   └── business-domain/               # Dominio de negocio principal
│   │   │       ├── queries/                   # 📖 Queries (lectura)
│   │   │       │   ├── search-items/
│   │   │       │   │   ├── search-items.query. ts
│   │   │       │   │   └── search-items.handler.ts
│   │   │       │   └── get-details/
│   │   │       │       ├── get-details.query. ts
│   │   │       │       └── get-details. handler.ts
│   │   │       │
│   │   │       ├── commands/                  # ✍️ Commands (escritura)
│   │   │       │   ├── create-item/
│   │   │       │   │   ├── create-item.command.ts
│   │   │       │   │   └── create-item.handler.ts
│   │   │       │   └── update-item/
│   │   │       │       ├── update-item.command.ts
│   │   │       │       └── update-item.handler.ts
│   │   │       │
│   │   │       ├── dtos/                      # Data Transfer Objects
│   │   │       │   ├── item. dto.ts
│   │   │       │   ├── create-item.dto.ts
│   │   │       │   └── update-item.dto.ts
│   │   │       │
│   │   │       └── services/                  # Servicios de dominio
│   │   │           └── business. service.ts
│   │   │
│   │   ├── seed-work/                         # 🧰 Building blocks reutilizables
│   │   │   ├── dispatcher. ts                  # Dispatcher/Mediator
│   │   │   ├── request-handler-decorator.ts   # Decorador @RequestHandler
│   │   │   │
│   │   │   ├── cqrs/                          # Base classes CQRS
│   │   │   │   ├── request. interface.ts
│   │   │   │   ├── request-handler.interface.ts
│   │   │   │   ├── query.base.ts
│   │   │   │   └── command.base.ts
│   │   │   │
│   │   │   ├── current-user-accessor/         # Acceso al usuario actual
│   │   │   │   ├── current-user-accessor.ts
│   │   │   │   └── user. interface.ts
│   │   │   │
│   │   │   ├── calculators/                   # Utilidades de cálculo
│   │   │   │   └── business-calculator.ts
│   │   │   │
│   │   │   ├── groupers/                      # Utilidades de agrupación
│   │   │   │   └── data-grouper.ts
│   │   │   │
│   │   │   └── interfaces/                    # Interfaces compartidas
│   │   │       ├── paginated-response.interface.ts
│   │   │       └── base-entity.interface.ts
│   │   │
│   │   └── utils/                             # Utilidades generales
│   │       ├── date. utils.ts
│   │       └── string.utils.ts
│   │
│   └── infrastructure/                        # 🏗️ CAPA DE INFRAESTRUCTURA
│       ├── bootstrap/                         # Configuración y arranque
│       │   ├── bootstrap.ts                   # Función de bootstrap principal
│       │   │
│       │   ├── config/                        # Configuración de la aplicación
│       │   │   ├── app/
│       │   │   │   ├── swagger-config.ts      # Configuración Swagger
│       │   │   │   ├── cors-config.ts         # Configuración CORS
│       │   │   │   ├── pipes-config.ts        # ValidationPipe config
│       │   │   │   └── body-parser-config.ts  # Body parser limits
│       │   │   │
│       │   │   └── providers/                 # Providers personalizados
│       │   │       └── custom. provider.ts
│       │   │
│       │   ├── middlewares/                   # Middlewares globales
│       │   │   ├── auth-guard.ts              # Guard de autenticación JWT
│       │   │   └── correlation-id.middleware.ts # Middleware Correlation ID
│       │   │
│       │   ├── interceptors/                  # Interceptors globales
│       │   │   └── logging.interceptor.ts     # Log de requests/responses
│       │   │
│       │   ├── modules/                       # Módulos NestJS
│       │   │   ├── app.module.ts              # Módulo raíz
│       │   │   │
│       │   │   └── features/                  # Módulos por feature
│       │   │       ├── business. module.ts
│       │   │       ├── health.module.ts
│       │   │       ├── logger.module.ts
│       │   │       └── current-user.module.ts
│       │   │
│       │   ├── http-clients/                  # Clientes HTTP configurados
│       │   │   ├── http-client.facade.ts      # Facade sobre Axios
│       │   │   ├── http-client.config.ts
│       │   │   └── interceptors/
│       │   │       ├── correlation-id.interceptor.ts
│       │   │       └── logging.interceptor.ts
│       │   │
│       │   └── seed-work/                     # Infraestructura compartida
│       │       └── logger/
│       │           ├── logger.service.ts
│       │           └── logger. module.ts
│       │
│       └── features/                          # Implementaciones por feature
│           ├── business-domain/
│           │   ├── business. service.ts        # Servicio principal
│           │   └── business-proxy. service.ts  # Proxy service
│           │
│           └── consistency/
│               └── consistency.service.ts
│
├── config/                                    # ⚙️ CONFIGURACIÓN POR AMBIENTE
│   ├── default.yaml                           # Config base
│   ├── development.yaml                       # Desarrollo local
│   ├── ci.yaml                                # Continuous Integration
│   ├── sandbox.yaml                           # QA/Testing
│   ├── staging.yaml                           # UAT/Pre-producción
│   └── production. yaml                        # Producción
│
├── test/                                      # 🧪 TESTS (espejo de src/)
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── k8s/                                       # ☸️ Kubernetes/OpenShift manifests
│   ├── deployment-dev.yaml
│   ├── deployment-staging.yaml
│   ├── deployment-prod.yaml
│   ├── service. yaml
│   └── configmap.yaml
│
├── Dockerfile                                 # 🐳 Container definition
├── docker-compose.yml                         # Local development
├── nest-cli.json                              # NestJS CLI config
├── tsconfig.json                              # TypeScript config
├── tsconfig.build.json                        # Build-specific config
├── jest.config.js                             # Jest config
├── . eslintrc.js                               # ESLint config
├── .prettierrc                                # Prettier config
├── .gitlab-ci.yml / .github/workflows/        # CI/CD
├── package.json
└── README.md
```

---

## 7. CONFIGURACIÓN Y AMBIENTES

### 📁 Estructura de Configuración

```
config/
├── default.yaml          # Configuración base (común a todos los ambientes)
├── development.yaml      # Desarrollo local
├── ci.yaml              # Continuous Integration
├── sandbox.yaml         # QA/Testing
├── staging.yaml         # UAT/Pre-producción
├── production.yaml      # Producción
└── test. yaml           # Testing unitario/integración
```

### 📄 Ejemplo de Configuración (development.yaml)

```yaml
# development.yaml
NODE_ENV: development
APP_NAME: bff-service
PORT: 3000

# CORS Configuration
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

# External APIs
EXTERNAL_APIS:
  SERVICE_A:
    BASE_URL: https://api-dev.example.com
    TIMEOUT: 5000
    RETRY_ATTEMPTS: 3
    RETRY_DELAY: 1000
  
  SERVICE_B:
    BASE_URL: https://api-dev.example.com/v2
    TIMEOUT: 3000
    RETRY_ATTEMPTS: 2

# JWT Configuration
JWT:
  SECRET: ${JWT_SECRET}  # Variable de entorno
  EXPIRES_IN: 3600
  ISSUER: bff-service

# Logging
LOGGING:
  LEVEL:  debug
  FORMAT: json
  CONSOLE: true
  FILE: false

# Health Checks
HEALTH_CHECK: 
  LIVENESS_PATH: /health/liveness
  READINESS_PATH: /health/readiness
  ENABLED: true

# Deployment (Kubernetes/OpenShift)
DEPLOYMENT:
  REPLICAS: 2
  CPU_REQUEST: 100m
  CPU_LIMIT: 500m
  MEMORY_REQUEST: 256Mi
  MEMORY_LIMIT:  512Mi

# Rate Limiting
RATE_LIMIT:
  TTL: 60
  LIMIT: 100

# Swagger
SWAGGER:
  ENABLED: true
  PATH: /api/docs
  TITLE: BFF Service API
  DESCRIPTION: Backend for Frontend API Documentation
  VERSION: 1.0.0
```

### 🔧 Carga de Configuración en NestJS

```typescript
// app.module.ts
import { ConfigModule } from '@nestjs/config';
import { load } from 'js-yaml';
import { readFileSync } from 'fs';
import { join } from 'path';

const loadYamlConfig = () => {
  const env = process.env.NODE_ENV || 'development';
  const configPath = join(__dirname, '..', '..', 'config', `${env}.yaml`);
  return load(readFileSync(configPath, 'utf8'));
};

@Module({
  imports: [
    ConfigModule. forRoot({
      isGlobal: true,
      load: [loadYamlConfig],
      cache: true,
    }),
    // ...  otros módulos
  ],
})
export class AppModule {}
```

### 🔐 Variables de Entorno

```bash
# . env.development
NODE_ENV=development
PORT=3000

# Secrets (NUNCA commitear estos valores)
JWT_SECRET=your-super-secret-jwt-key-change-in-production
DATABASE_PASSWORD=db-password-here
API_KEY_SERVICE_A=api-key-here

# Feature Flags
FEATURE_NEW_PAYMENT_FLOW=true
FEATURE_ADVANCED_SEARCH=false
```

### ❤️ Health Checks

```typescript
// health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) {}

  @Get('liveness')
  @HealthCheck()
  liveness() {
    // Verifica que la aplicación está viva
    return this.health.check([]);
  }

  @Get('readiness')
  @HealthCheck()
  readiness() {
    // Verifica que la aplicación está lista para recibir tráfico
    return this.health.check([
      () => this.http.pingCheck('service-a', 'https://api.example.com/health'),
    ]);
  }
}
```

---

## 8. SEGURIDAD

### 🔒 Autenticación JWT

#### Auth Guard

```typescript
// auth-guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { JwtService } from '@nestjs/jwt';
import { Request } from 'express';

export const IS_PUBLIC_KEY = 'isPublic';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector:  Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Verificar si la ruta es pública
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    const request = context.switchToHttp().getRequest<Request>();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException('Token no proporcionado');
    }

    try {
      // Validar y decodificar el token
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env. JWT_SECRET,
      });

      // Inyectar payload del usuario en el request
      request['user'] = payload;
    } catch (error) {
      throw new UnauthorizedException('Token inválido o expirado');
    }

    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const authorization = request.headers.authorization;
    if (!authorization) return undefined;

    const [type, token] = authorization.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

#### Public Decorator

```typescript
// public. decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

#### Uso en Controllers

```typescript
// users.controller.ts
@Controller('users')
@UseGuards(AuthGuard)  // Aplicar guard a todo el controller
export class UsersController {
  
  @Get()
  findAll() {
    // Ruta PROTEGIDA - requiere JWT válido
    return this.usersService.findAll();
  }

  @Public()  // Marcar como pública
  @Post('login')
  login(@Body() loginDto: LoginDto) {
    // Ruta PÚBLICA - no requiere JWT
    return this.authService.login(loginDto);
  }

  @Get('profile')
  getProfile(@Request() req) {
    // Acceder al usuario desde el request
    return req. user;
  }
}
```

### 🔑 Current User Accessor

```typescript
// current-user-accessor.ts
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

export interface User {
  sub: string;
  userId: string;
  email: string;
  roles: string[];
}

@Injectable({ scope: Scope.REQUEST })
export class CurrentUserAccessor {
  constructor(@Inject(REQUEST) private readonly request: Request) {}

  get user(): User {
    return this.request['user'];
  }

  get userId(): string {
    return this.user?. sub || this.user?.userId;
  }

  get email(): string {
    return this.user?.email;
  }

  get roles(): string[] {
    return this. user?.roles || [];
  }

  hasRole(role: string): boolean {
    return this.roles.includes(role);
  }

  hasAnyRole(... roles: string[]): boolean {
    return roles.some(role => this.roles.includes(role));
  }

  hasAllRoles(...roles: string[]): boolean {
    return roles.every(role => this. roles.includes(role));
  }
}
```

#### Uso en Services

```typescript
// orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    private currentUser: CurrentUserAccessor,
    private ordersRepository: OrdersRepository,
  ) {}

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    const userId = this.currentUser.userId;
    
    return this.ordersRepository.create({
      ... dto,
      userId,
      createdBy: userId,
    });
  }

  async getMyOrders(): Promise<Order[]> {
    const userId = this. currentUser.userId;
    return this.ordersRepository.findByUserId(userId);
  }
}
```

### 📊 Correlation ID Middleware

```typescript
// correlation-id.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req:  Request, res: Response, next:  NextFunction) {
    // Extraer Correlation ID del header o generar uno nuevo
    let correlationId = req.headers['x-correlation-id'] as string;

    if (!correlationId) {
      correlationId = `AUTO-${uuidv4()}`;
    }

    // Inyectar en request y response
    req. headers['x-correlation-id'] = correlationId;
    res.setHeader('X-Correlation-ID', correlationId);

    // Almacenar en el contexto (opcional, para usarlo en loggers)
    req['correlationId'] = correlationId;

    next();
  }
}
```

#### Aplicar Middleware Globalmente

```typescript
// app.module.ts
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      . apply(CorrelationIdMiddleware)
      .forRoutes('*');  // Aplicar a todas las rutas
  }
}
```

### 🛡️ Helmet. js - Security Headers

```typescript
// main.ts
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Helmet - Security headers
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc:  ["'self'"],
        imgSrc: ["'self'", 'data:', 'https:'],
      },
    },
    hsts: {
      maxAge:  31536000,
      includeSubDomains: true,
      preload: true,
    },
    frameguard: {
      action:  'deny',
    },
  }));

  await app.listen(3000);
}
```

### 🔐 CORS Configuration

```typescript
// cors-config.ts
import { CorsOptions } from '@nestjs/common/interfaces/external/cors-options. interface';
import { ConfigService } from '@nestjs/config';

export const getCorsConfig = (configService: ConfigService): CorsOptions => ({
  origin: configService.get<string[]>('CORS.ORIGIN'),
  methods: configService.get<string[]>('CORS. METHODS'),
  credentials: configService.get<boolean>('CORS.CREDENTIALS'),
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Correlation-ID',
    'X-Request-ID',
  ],
  exposedHeaders: [
    'X-Correlation-ID',
    'X-Request-ID',
  ],
  maxAge: 3600,
});

// main.ts
app.enableCors(getCorsConfig(configService));
```

### ✅ Validation Pipe

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    transform: true,              // Auto-transformar a instancias de clase
    whitelist: true,              // Remover propiedades no declaradas
    forbidNonWhitelisted: true,   // Lanzar error si hay props extras
    transformOptions: {
      enableImplicitConversion: true,  // Conversión automática de tipos
    },
    disableErrorMessages: false,  // Mensajes de error detallados
  }),
);
```

#### DTOs con Validación

```typescript
// create-user.dto.ts
import { IsString, IsEmail, IsInt, Min, Max, IsArray, IsOptional, MinLength, MaxLength } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ example:  'John Doe', description:  'Full name of the user' })
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @ApiProperty({ example: 'john@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 25, minimum: 18, maximum: 120 })
  @IsInt()
  @Min(18)
  @Max(120)
  age: number;

  @ApiProperty({ example: ['user', 'admin'], required: false })
  @IsArray()
  @IsString({ each: true })
  @IsOptional()
  roles?: string[];
}
```

### 🚦 Rate Limiting

```typescript
// app.module.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60,      // Time window en segundos
      limit: 100,   // Máximo de requests en el time window
    }),
  ],
})
export class AppModule {}

// Aplicar globalmente
// main.ts
import { ThrottlerGuard } from '@nestjs/throttler';

app.useGlobalGuards(new ThrottlerGuard());

// O aplicar a controllers específicos
@Controller('users')
@UseGuards(ThrottlerGuard)
export class UsersController {}
```

---

## 9. LOGGING & OBSERVABILIDAD

### 📡 Logging Interceptor

```typescript
// logging.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, Logger } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap, catchError } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

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
      headers: this.sanitizeHeaders(headers),
      body: this.sanitizeBody(body),
    });

    const startTime = Date.now();

    return next.handle().pipe(
      tap((data) => {
        const response = context.switchToHttp().getResponse();
        const { statusCode } = response;
        const responseTime = Date.now() - startTime;

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
        const responseTime = Date.now() - startTime;

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
      }),
    );
  }

  private sanitizeHeaders(headers: any): any {
    const sanitized = { ...headers };
    const sensitiveHeaders = ['authorization', 'cookie', 'x-api-key'];
    
    sensitiveHeaders.forEach(header => {
      if (sanitized[header]) {
        sanitized[header] = '***REDACTED***';
      }
    });

    return sanitized;
  }

  private sanitizeBody(body: any): any {
    if (!body) return body;
    if (typeof body !== 'object') return body;

    const sensitiveFields = ['password', 'token', 'secret', 'apiKey', 'creditCard'];
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

### 📊 Custom Logger Service

```typescript
// logger.service.ts
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { createLogger, format, transports, Logger as WinstonLogger } from 'winston';

@Injectable()
export class LoggerService implements NestLoggerService {
  private logger: WinstonLogger;

  constructor() {
    this.logger = createLogger({
      level:  process.env.LOG_LEVEL || 'info',
      format: format.combine(
        format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
        format.errors({ stack: true }),
        format.splat(),
        format.json(),
      ),
      defaultMeta: { service: process.env.APP_NAME || 'bff-service' },
      transports:  [
        new transports.Console({
          format: format.combine(
            format.colorize(),
            format.printf(({ timestamp, level, message, ... meta }) => {
              return `${timestamp} [${level}]:  ${message} ${Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ''}`;
            }),
          ),
        }),
        new transports. File({ filename: 'logs/error.log', level: 'error' }),
        new transports.File({ filename: 'logs/combined.log' }),
      ],
    });
  }

  log(message: any, context?: string) {
    this.logger.info(message, { context });
  }

  error(message: any, trace?: string, context?: string) {
    this.logger.error(message, { trace, context });
  }

  warn(message: any, context?: string) {
    this.logger.warn(message, { context });
  }

  debug(message: any, context?: string) {
    this.logger.debug(message, { context });
  }

  verbose(message: any, context?: string) {
    this.logger.verbose(message, { context });
  }
}

// logger.module.ts
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}
```

### 🔍 HTTP Client con Logging

```typescript
// http-client.facade.ts
import { Injectable, Logger } from '@nestjs/common';
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import axiosRetry from 'axios-retry';

@Injectable()
export class HttpClientFacade {
  private readonly logger = new Logger(HttpClientFacade.name);
  private axiosInstance: AxiosInstance;

  constructor(private configService: ConfigService) {
    this.axiosInstance = axios. create({
      timeout: 5000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    // Configurar reintentos
    axiosRetry(this.axiosInstance, {
      retries: 3,
      retryDelay: axiosRetry.exponentialDelay,
      retryCondition: (error) => {
        return axiosRetry.isNetworkOrIdempotentRequestError(error) || error.response?.status === 429;
      },
    });

    // Request Interceptor
    this.axiosInstance.interceptors.request.use(
      (config) => {
        const correlationId = config.headers['x-correlation-id'];
        
        this.logger.log({
          message: 'External HTTP REQUEST',
          method: config.method?. toUpperCase(),
          url: config.url,
          correlationId,
        });

        return config;
      },
      (error) => {
        this.logger.error('HTTP Request Error', error);
        return Promise.reject(error);
      },
    );

    // Response Interceptor
    this. axiosInstance.interceptors.response.use(
      (response) => {
        const correlationId = response.config.headers['x-correlation-id'];
        
        this.logger.log({
          message: 'External HTTP RESPONSE',
          method:  response.config.method?.toUpperCase(),
          url: response. config.url,
          status: response.status,
          correlationId,
        });

        return response;
      },
      (error) => {
        const correlationId = error.config?.headers['x-correlation-id'];
        
        this.logger.error({
          message: 'External HTTP ERROR',
          method: error.config?.method?.toUpperCase(),
          url: error.config?.url,
          status: error.response?.status,
          correlationId,
          error: error.message,
        });

        return Promise.reject(error);
      },
    );
  }

  async get<T = any>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance. get<T>(url, config);
    return response.data;
  }

  async post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.post<T>(url, data, config);
    return response. data;
  }

  async put<T = any>(url:  string, data?: any, config?:  AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.put<T>(url, data, config);
    return response.data;
  }

  async patch<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.patch<T>(url, data, config);
    return response.data;
  }

  async delete<T = any>(url: string, config?:  AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.delete<T>(url, config);
    return response.data;
  }
}
```

---

## 10. CI/CD PIPELINE

### 📋 GitLab CI

```yaml
# .gitlab-ci.yml

variables:
  DOCKER_REGISTRY: quay.io
  IMAGE_NAME: $CI_PROJECT_NAME
  SONAR_HOST_URL: https://sonarqube.example.com

stages:
  - package
  - test
  - quality
  - build-image
  - deploy

# ========== STAGE 1: PACKAGE ==========
package:
  stage: package
  image: node:18-alpine
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  only:
    - main
    - staging
    - development

# ========== STAGE 2: TEST ==========
test: unit:
  stage: test
  image: node:18-alpine
  dependencies:
    - package
  script:
    - npm run test:cov
  coverage:  '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports: 
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week
  only:
    - main
    - staging
    - development

test:integration:
  stage: test
  image: node:18-alpine
  dependencies:
    - package
  script:
    - npm run test:integration
  only:
    - main
    - staging
    - development

# ========== STAGE 3: QUALITY ==========
lint:
  stage: quality
  image: node:18-alpine
  dependencies:
    - package
  script:
    - npm run lint
  only:
    - main
    - staging
    - development

sonarqube:
  stage: quality
  image: sonarsource/sonar-scanner-cli:latest
  dependencies:
    - test: unit
  script:
    - sonar-scanner
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.sources=src
      -Dsonar. tests=test
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
      -Dsonar.typescript.lcov.reportPaths=coverage/lcov.info
  only:
    - main
    - staging
    - development

# ========== STAGE 4: BUILD IMAGE ==========
build-image: 
  stage: build-image
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $DOCKER_REGISTRY_USER -p $DOCKER_REGISTRY_PASSWORD $DOCKER_REGISTRY
  script:
    - docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
    - docker tag $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA $DOCKER_REGISTRY/$IMAGE_NAME:latest
    - docker push $DOCKER_REGISTRY/$IMAGE_NAME: $CI_COMMIT_SHORT_SHA
    - docker push $DOCKER_REGISTRY/$IMAGE_NAME:latest
  only:
    - main
    - staging
    - development

# ========== STAGE 5: DEPLOY ==========
deploy:dev:
  stage: deploy
  image: openshift/origin-cli:latest
  script: 
    - oc login $OPENSHIFT_SERVER --token=$OPENSHIFT_TOKEN
    - oc project $OPENSHIFT_PROJECT_DEV
    - oc set image deployment/$APP_NAME $APP_NAME=$DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - oc rollout status deployment/$APP_NAME
  environment:
    name: development
    url: https://bff-dev.example.com
  only:
    - development
  when: on_success

deploy:staging: 
  stage: deploy
  image:  openshift/origin-cli:latest
  script:
    - oc login $OPENSHIFT_SERVER --token=$OPENSHIFT_TOKEN
    - oc project $OPENSHIFT_PROJECT_STAGING
    - oc set image deployment/$APP_NAME $APP_NAME=$DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - oc rollout status deployment/$APP_NAME
  environment:
    name:  staging
    url: https://bff-staging.example.com
  only:
    - staging
  when: manual

deploy:prod:
  stage: deploy
  image: openshift/origin-cli:latest
  script:
    - oc login $OPENSHIFT_SERVER --token=$OPENSHIFT_TOKEN
    - oc project $OPENSHIFT_PROJECT_PROD
    - oc set image deployment/$APP_NAME $APP_NAME=$DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - oc rollout status deployment/$APP_NAME
  environment:
    name: production
    url: https://bff.example.com
  only:
    - main
  when: manual
```

### 🐳 Dockerfile Multi-Stage

```dockerfile
# Dockerfile

# ========== STAGE 1: BUILD ==========
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./
COPY tsconfig*.json ./
COPY nest-cli.json ./

# Instalar dependencias (incluyendo devDependencies para build)
RUN npm ci

# Copiar código fuente
COPY src ./src
COPY config ./config

# Build de la aplicación
RUN npm run build

# Remover devDependencies
RUN npm prune --production

# ========== STAGE 2: PRODUCTION ==========
FROM node:18-alpine AS production

WORKDIR /app

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && adduser -S nestjs -u 1001

# Copiar archivos necesarios desde builder
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs: nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/package.json ./
COPY --chown=nestjs:nodejs config ./config

# Variables de entorno
ENV NODE_ENV=production
ENV PORT=3000

# Exponer puerto
EXPOSE 3000

# Cambiar a usuario no-root
USER nestjs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:3000/health/liveness', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Comando de inicio
CMD ["node", "dist/main.js"]
```

### ☸️ Kubernetes Deployment

```yaml
# k8s/deployment-prod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bff-service
  namespace: production
  labels:
    app: bff-service
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bff-service
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: bff-service
        version: v1
    spec: 
      containers:
      - name:  bff-service
        image:  quay.io/myorg/bff-service:latest
        ports:
        - containerPort: 3000
          protocol: TCP
        env: 
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        - name: JWT_SECRET
          valueFrom: 
            secretKeyRef:
              name: bff-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/liveness
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/readiness
            port:  3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
      imagePullSecrets:
      - name: quay-registry-secret

---
apiVersion: v1
kind: Service
metadata:
  name: bff-service
  namespace: production
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
  selector:
    app: bff-service

---
apiVersion: autoscaling/v2
kind:  HorizontalPodAutoscaler
metadata:
  name: bff-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: bff-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type:  Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type:  Utilization
        averageUtilization: 80
```

---

## 11. SWAGGER/OPENAPI DOCUMENTATION

### 📝 Configuración de Swagger

```typescript
// swagger-config.ts
import { INestApplication } from '@nestjs/common';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { ConfigService } from '@nestjs/config';

export function setupSwagger(app: INestApplication, configService: ConfigService): void {
  const swaggerEnabled = configService.get<boolean>('SWAGGER. ENABLED', true);

  if (! swaggerEnabled) {
    return;
  }

  const config = new DocumentBuilder()
    .setTitle(configService.get<string>('SWAGGER.TITLE', 'BFF Service API'))
    .setDescription(configService.get<string>('SWAGGER.DESCRIPTION', 'Backend for Frontend API Documentation'))
    .setVersion(configService.get<string>('SWAGGER.VERSION', '1.0.0'))
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        name: 'JWT',
        description: 'Enter JWT token',
        in: 'header',
      },
      'JWT-auth',
    )
    .addTag('Health', 'Health check endpoints')
    .addTag('Users', 'User management endpoints')
    .addTag('Orders', 'Order management endpoints')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  const swaggerPath = configService.get<string>('SWAGGER.PATH', '/api/docs');
  
  SwaggerModule.setup(swaggerPath, app, document, {
    swaggerOptions: {
      persistAuthorization: true,
      tagsSorter: 'alpha',
      operationsSorter: 'alpha',
    },
    customSiteTitle: 'BFF Service API Docs',
  });
}
```

### 📖 Decoradores en Controllers

```typescript
// users. controller.ts
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth, ApiBody } from '@nestjs/swagger';

@ApiTags('Users')
@Controller('users')
@ApiBearerAuth('JWT-auth')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private readonly dispatcher: Dispatcher) {}

  @Get()
  @ApiOperation({ summary: 'Get all users', description: 'Returns a list of all users' })
  @ApiResponse({ status: 200, description: 'Users retrieved successfully', type: [UserDto] })
  @ApiResponse({ status: 401, description:  'Unauthorized' })
  async findAll(): Promise<UserDto[]> {
    return this.dispatcher. dispatch(new GetAllUsersQuery());
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async findOne(@Param('id') id: string): Promise<UserDto> {
    return this.dispatcher.dispatch(new GetUserByIdQuery(id));
  }

  @Post()
  @ApiOperation({ summary: 'Create new user' })
  @ApiBody({ type: CreateUserDto })
  @ApiResponse({ status: 201, description:  'User created successfully', type:  UserDto })
  @ApiResponse({ status: 400, description:  'Bad request' })
  async create(@Body() dto: CreateUserDto): Promise<UserDto> {
    return this. dispatcher.dispatch(new CreateUserCommand(dto));
  }
}
```

### 📦 DTOs con Swagger

```typescript
// user. dto.ts
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { IsString, IsEmail, IsInt, Min, Max, IsArray, IsOptional } from 'class-validator';

export class UserDto {
  @ApiProperty({ example: '123e4567-e89b-12d3-a456-426614174000', description: 'User ID' })
  id: string;

  @ApiProperty({ example: 'John Doe', description: 'Full name' })
  name: string;

  @ApiProperty({ example: 'john@example.com', description: 'Email address' })
  email: string;

  @ApiProperty({ example:  25, description: 'User age' })
  age: number;

  @ApiPropertyOptional({ example: ['user', 'admin'], description: 'User roles' })
  roles?: string[];
}

export class CreateUserDto {
  @ApiProperty({ example:  'John Doe', description: 'Full name', minLength: 3, maxLength: 50 })
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @ApiProperty({ example: 'john@example. com', description: 'Email address' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 25, description: 'User age', minimum: 18, maximum: 120 })
  @IsInt()
  @Min(18)
  @Max(120)
  age: number;

  @ApiPropertyOptional({ example: ['user'], description: 'User roles', type: [String] })
  @IsArray()
  @IsString({ each: true })
  @IsOptional()
  roles?: string[];
}
```

---

## 12. TESTING

### 🧪 Unit Tests

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { HttpClientFacade } from '../http-clients/http-client.facade';

describe('UsersService', () => {
  let service: UsersService;
  let httpClient: jest.Mocked<HttpClientFacade>;

  beforeEach(async () => {
    const mockHttpClient = {
      get: jest.fn(),
      post: jest.fn(),
      put: jest.fn(),
      delete: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: HttpClientFacade,
          useValue: mockHttpClient,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    httpClient = module.get(HttpClientFacade);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('findAll', () => {
    it('should return an array of users', async () => {
      // Arrange
      const mockUsers = [
        { id: '1', name: 'John', email: 'john@example. com', age: 25 },
        { id: '2', name:  'Jane', email: 'jane@example.com', age: 30 },
      ];
      httpClient.get.mockResolvedValue(mockUsers);

      // Act
      const result = await service.findAll();

      // Assert
      expect(result).toEqual(mockUsers);
      expect(httpClient.get).toHaveBeenCalledWith('/users');
    });

    it('should handle errors gracefully', async () => {
      // Arrange
      httpClient.get.mockRejectedValue(new Error('Network error'));

      // Act & Assert
      await expect(service.findAll()).rejects.toThrow('Network error');
    });
  });

  describe('create', () => {
    it('should create a user successfully', async () => {
      // Arrange
      const createDto = { name: 'John', email: 'john@example. com', age: 25 };
      const mockCreatedUser = { id: '1', ...createDto };
      httpClient.post.mockResolvedValue(mockCreatedUser);

      // Act
      const result = await service.create(createDto);

      // Assert
      expect(result).toEqual(mockCreatedUser);
      expect(httpClient.post).toHaveBeenCalledWith('/users', createDto);
    });
  });
});
```
