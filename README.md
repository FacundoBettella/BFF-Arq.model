# 🏗️ Documentación de Arquitectura BFF

Este repositorio contiene la documentación completa de arquitectura para proyectos **Backend for Frontend (BFF)** utilizando Node.js y TypeScript.

## 📚 Documentos Disponibles

### 1. [ARQUITECTURA-BFF.md](./ARQUITECTURA-BFF.md)
**Guía Arquitectónica General - BFF Híbrido**

Documentación completa que cubre **dos enfoques arquitectónicos** para implementar un BFF:

- ✅ **Express. js con Framework Custom**:  Implementación manual de patrones (Mediator, Decoradores personalizados, TypeDI)
- ✅ **NestJS**:  Framework opinionado con arquitectura enterprise-grade integrada

---

### 2. [ARQUITECTURA-BFF-NESTJS.md](./ARQUITECTURA-BFF-NESTJS. md)
**Guía Arquitectónica Específica - BFF con NestJS**

Documentación **100% enfocada en NestJS** basada en un proyecto real en producción. 

**Incluye**:
- ✅ Arquitectura en capas (API, Core, Infrastructure)
- ✅ Dispatcher/Mediator Pattern con metadata
- ✅ CQRS (Commands & Queries)
- ✅ Request-scoped services
- ✅ Autenticación JWT con Guards
- ✅ Logging & Observabilidad (Correlation IDs)
- ✅ Configuración multi-ambiente (YAML)
- ✅ CI/CD Pipeline (GitLab CI + OpenShift)
- ✅ Testing (Unit, Integration)
- ✅ Swagger/OpenAPI documentation

---

## 🎯 ¿Cuál usar?

| Escenario | Documento Recomendado |
|-----------|----------------------|
| Estoy evaluando entre Express y NestJS | **ARQUITECTURA-BFF.md** (híbrido) |
| Mi proyecto usa NestJS | **ARQUITECTURA-BFF-NESTJS.md** |
| Necesito implementar patrones manualmente | **ARQUITECTURA-BFF. md** (sección Express) |
| Busco una solución enterprise lista para producción | **ARQUITECTURA-BFF-NESTJS.md** |
| Quiero entender las diferencias entre ambos | **Ambos documentos** |

---

## 📖 Estructura de los Documentos

Ambos documentos cubren: 

1. **Tipo de Arquitectura** - Patrones aplicados (Clean Architecture, DDD, CQRS, Mediator)
2. **Estructura de Capas** - Organización del código
3. **Stack Tecnológico** - Frameworks, librerías, herramientas
4. **Patrones de Diseño** - Creacionales, Estructurales, Comportamiento
5. **Estructura de Proyecto** - Árbol de carpetas completo
6. **Seguridad** - JWT, Guards, Validation, CORS
7. **Logging & Observabilidad** - Correlation IDs, Interceptors
8. **CI/CD** - Pipelines, Docker, Kubernetes/OpenShift
9. **Testing** - Unit, Integration, E2E
10. **Best Practices** - Código, Testing, Deployment

---

## 🚀 Quick Start

```bash
# Clonar el repositorio
git clone https://github.com/FacundoBettella/Microfront-Arq. git

# Ver documentación
cd Microfront-Arq
```

### Para proyectos NestJS:

```bash
# Leer documentación específica
cat ARQUITECTURA-BFF-NESTJS.md

# O abrir en tu editor preferido
code ARQUITECTURA-BFF-NESTJS.md
```

### Para comparar enfoques:

```bash
# Leer documentación híbrida
cat ARQUITECTURA-BFF.md
```

---

## 🏛️ Patrones Arquitectónicos Cubiertos

| Patrón | Descripción |
|--------|-------------|
| **Clean Architecture** | Separación en capas con dependencias hacia el dominio |
| **Hexagonal Architecture** | Ports & Adapters para infraestructura |
| **DDD** | Domain-Driven Design con features por dominio |
| **CQRS** | Separación de Commands (escritura) y Queries (lectura) |
| **Mediator/Dispatcher** | Desacoplamiento de controllers y handlers |
| **Repository Pattern** | Abstracción de acceso a datos |
| **Proxy Pattern** | Servicios proxy para funcionalidad adicional |
| **Circuit Breaker** | Resiliencia ante fallos externos |

---

## 🛠️ Stack Cubierto

### Framework
- **NestJS 11.x** (documento específico)
- **Express.js** (documento híbrido)

### Lenguaje
- **TypeScript 5.x** con strict mode

### Seguridad
- JWT Authentication
- Helmet. js
- class-validator
- Rate Limiting

### Testing
- Jest
- Supertest
- >80% coverage

### DevOps
- Docker
- Kubernetes/OpenShift
- GitLab CI / GitHub Actions
- SonarQube

---

## 📋 Casos de Uso

Esta arquitectura es ideal para: 

- 🏦 **Aplicaciones Financieras** (alta seguridad, trazabilidad)
- 🛒 **E-commerce** (escalabilidad, performance)
- 🏥 **Sistemas de Salud** (compliance, auditoría)
- 📱 **Mobile Backends** (BFF pattern)
- 🚀 **Cualquier sistema enterprise-grade** con requisitos de alta disponibilidad

---

## 👥 Contribuciones

Las contribuciones son bienvenidas.  Por favor: 

1. Fork el repositorio
2. Crea una rama para tu feature (`git checkout -b feature/amazing-feature`)
3. Commit tus cambios (`git commit -m 'Add amazing feature'`)
4. Push a la rama (`git push origin feature/amazing-feature`)
5. Abre un Pull Request

---

## 📝 Licencia

Este proyecto está bajo licencia de uso libre para propósitos educativos y comerciales.

---

## 📞 Contacto

**Autor**:  Facundo Bettella  
**Repositorio**: [FacundoBettella/Microfront-Arq](https://github.com/FacundoBettella/Microfront-Arq)

---

## 🔖 Versión

- **ARQUITECTURA-BFF.md**:  v2.0.0
- **ARQUITECTURA-BFF-NESTJS.md**: v1.0.0
- **Última actualización**: 2025-12-31

---

**⭐ Si te resulta útil, considera dar una estrella al repositorio!**
