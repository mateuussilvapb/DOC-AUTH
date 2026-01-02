# Documento Arquitetural — Visão de Implementação

## Auth Server

---

## 1. Objetivo da Visão de Implementação

Este documento descreve a estrutura de implementação do Auth Server, definindo:

- Arquitetura de camadas e responsabilidades técnicas
- Organização de pacotes e módulos
- Padrões de código e convenções
- Mapeamento entre componentes arquiteturais e estrutura de código
- Decisões técnicas de implementação

Esta visão complementa a Visão de Componentes, traduzindo conceitos arquiteturais em estrutura concreta de código.

---

## 2. Arquitetura de Implementação

### 2.1 Padrão Arquitetural

O Auth Server adota **Arquitetura em Camadas com DDD (Domain-Driven Design) Tático**.

**Justificativa:**

- Separação clara de responsabilidades
- Facilita testes em cada camada
- Isola regras de negócio de detalhes técnicos
- Permite evolução independente de camadas
- Alinha-se com princípios SOLID

### 2.2 Camadas Principais

```
┌─────────────────────────────────────────┐
│      Presentation Layer                 │  Controllers, DTOs, Validações de Entrada
├─────────────────────────────────────────┤
│      Application Layer                  │  Services, Use Cases, Orquestração
├─────────────────────────────────────────┤
│      Domain Layer                       │  Entidades, Value Objects, Regras de Negócio
├─────────────────────────────────────────┤
│      Infrastructure Layer               │  JPA, Email, JWT, Implementações Técnicas
└─────────────────────────────────────────┘
```

**Regras de Dependência:**

- Presentation → Application → Domain
- Infrastructure → Domain (implementa interfaces do domínio)
- Domain não depende de nenhuma outra camada

---

## 3. Estrutura de Pacotes

### 3.1 Organização Raiz

```
com.seudominio.authserver/
├── config/                    # Configurações do Spring
├── presentation/              # Camada de Apresentação
├── application/               # Camada de Aplicação
├── domain/                    # Camada de Domínio
└── infrastructure/            # Camada de Infraestrutura
```

### 3.2 Detalhamento por Camada

#### **Presentation Layer** (`presentation/`)

Responsável por expor APIs e receber requisições.

```
presentation/
├── controller/
│   ├── oauth/                 # Endpoints OAuth2/OIDC
│   │   ├── AuthorizationController.java
│   │   └── TokenController.java
│   ├── admin/                 # API Administrativa
│   │   ├── UserAdminController.java
│   │   ├── SystemAdminController.java
│   │   ├── RoleAdminController.java
│   │   └── UserSystemBindingController.java
│   └── recovery/              # Recuperação de senha
│       └── PasswordRecoveryController.java
├── dto/
│   ├── request/               # DTOs de entrada
│   └── response/              # DTOs de saída
└── mapper/                    # Conversão DTO ↔ Domain
```

**Responsabilidades:**

- Receber requisições HTTP
- Validar entrada (Bean Validation)
- Converter DTOs em objetos de domínio
- Delegar processamento para Application Layer
- Formatar respostas

**Não deve:**

- Conter regras de negócio
- Acessar diretamente repositórios
- Implementar lógica de segurança complexa

---

#### **Application Layer** (`application/`)

Orquestra casos de uso e coordena serviços.

```
application/
├── service/
│   ├── authentication/        # Autenticação e OAuth2
│   │   ├── AuthenticationService.java
│   │   └── OAuth2FlowService.java
│   ├── user/                  # Gestão de usuários
│   │   ├── UserService.java
│   │   └── UserValidationService.java
│   ├── system/                # Gestão de sistemas
│   │   └── ClientSystemService.java
│   ├── role/                  # Gestão de perfis
│   │   └── SystemRoleService.java
│   ├── binding/               # Vínculos usuário-sistema
│   │   └── UserSystemBindingService.java
│   ├── token/                 # Emissão de tokens
│   │   ├── TokenService.java
│   │   ├── JwtTokenService.java
│   │   └── AuthorizationCodeService.java
│   └── recovery/              # Recuperação de acesso
│       ├── PasswordRecoveryService.java
│       └── EmailService.java
└── usecase/                   # Use Cases (opcional)
```

**Responsabilidades:**

- Implementar casos de uso do sistema
- Coordenar operações entre serviços de domínio
- Gerenciar transações
- Aplicar políticas de segurança de alto nível
- Transformar exceções de domínio em respostas adequadas

**Não deve:**

- Acessar diretamente entidades JPA
- Implementar lógica de persistência
- Conter regras de negócio complexas (essas ficam no Domain)

---

#### **Domain Layer** (`domain/`)

Núcleo da aplicação, contém as regras de negócio.

```
domain/
├── model/
│   ├── user/
│   │   ├── User.java              # Entidade de domínio
│   │   ├── UserStatus.java        # Enum
│   │   └── Password.java          # Value Object
│   ├── system/
│   │   ├── ClientSystem.java
│   │   └── SystemStatus.java
│   ├── role/
│   │   └── SystemRole.java
│   ├── binding/
│   │   ├── UserSystem.java
│   │   ├── UserSystemRole.java
│   │   └── BindingStatus.java
│   └── token/
│       ├── AuthorizationCode.java
│       ├── PasswordResetToken.java
│       └── JwtPayload.java        # Value Object
├── repository/                    # Interfaces
│   ├── UserRepository.java
│   ├── ClientSystemRepository.java
│   └── ...
├── service/                       # Domain Services
│   ├── AccessValidator.java
│   └── MasterUserPolicy.java
└── exception/                     # Exceções de domínio
    ├── UserNotFoundException.java
    ├── InvalidCredentialsException.java
    └── SystemAccessDeniedException.java
```

**Responsabilidades:**

- Modelar entidades e agregados
- Implementar regras de negócio
- Definir contratos de repositórios (interfaces)
- Validar invariantes de domínio
- Expor comportamentos através de métodos de entidade

**Características importantes:**

- Independente de frameworks
- Não conhece JPA, Spring, etc.
- Repositórios são interfaces (implementadas em Infrastructure)
- Value Objects garantem integridade (ex: Password com hash BCrypt)

**Exemplo de Entidade de Domínio:**

```java
public class User {
    private Long id;
    private String username;
    private String email;
    private Password password;
    private String name;
    private boolean isMaster;
    private UserStatus status;
    
    // Comportamentos de negócio
    public boolean canAccessSystem(ClientSystem system) {
        if (isMaster) return true;
        // lógica de validação
    }
    
    public void blockAccess() {
        this.status = UserStatus.BLOCKED;
    }
}
```

---

#### **Infrastructure Layer** (`infrastructure/`)

Implementa detalhes técnicos e integrações externas.

```
infrastructure/
├── persistence/
│   ├── entity/                    # Entidades JPA
│   │   ├── UserEntity.java
│   │   ├── ClientSystemEntity.java
│   │   └── ...
│   ├── jpa/                       # Repositórios JPA
│   │   ├── UserJpaRepository.java
│   │   └── ...
│   ├── mapper/                    # Entity ↔ Domain
│   │   ├── UserEntityMapper.java
│   │   └── ...
│   └── repository/                # Implementações
│       ├── UserRepositoryImpl.java
│       └── ...
├── security/
│   ├── jwt/
│   │   ├── JwtProvider.java
│   │   └── JwtValidator.java
│   └── oauth2/
│       └── CustomAuthenticationProvider.java
├── email/
│   ├── EmailSender.java           # Interface
│   └── SmtpEmailSender.java       # Implementação
└── exception/
    └── GlobalExceptionHandler.java
```

**Responsabilidades:**

- Implementar interfaces de repositório definidas no Domain
- Gerenciar persistência (JPA/Hibernate)
- Implementar integrações externas (SMTP, etc.)
- Prover implementações de segurança (JWT, OAuth2)
- Tratar exceções técnicas

**Padrão de Implementação de Repositório:**

```java
// Domain
public interface UserRepository {
    Optional<User> findByUsername(String username);
    User save(User user);
}

// Infrastructure
@Repository
public class UserRepositoryImpl implements UserRepository {
    
    @Autowired
    private UserJpaRepository jpaRepository;
    
    @Autowired
    private UserEntityMapper mapper;
    
    @Override
    public Optional<User> findByUsername(String username) {
        return jpaRepository.findByUsername(username)
            .map(mapper::toDomain);
    }
    
    @Override
    public User save(User user) {
        UserEntity entity = mapper.toEntity(user);
        UserEntity saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }
}
```

---

## 4. Mapeamento: Componentes Arquiteturais → Implementação

| Componente Arquitetural | Camada | Pacote Principal |
|------------------------|--------|------------------|
| API de Autenticação | Presentation | `presentation.controller.oauth` |
| API Administrativa | Presentation | `presentation.controller.admin` |
| Camada de Segurança | Application + Infrastructure | `application.service.authentication` + `infrastructure.security` |
| Token Service | Application + Infrastructure | `application.service.token` + `infrastructure.security.jwt` |
| Gerenciamento de Usuários | Application + Domain | `application.service.user` + `domain.model.user` |
| Gerenciamento de Sistemas | Application + Domain | `application.service.system` + `domain.model.system` |
| Gerenciamento de Perfis | Application + Domain | `application.service.role` + `domain.model.role` |
| Recuperação de Acesso | Application + Infrastructure | `application.service.recovery` + `infrastructure.email` |
| Camada de Persistência | Infrastructure | `infrastructure.persistence` |

---

## 5. Convenções de Código

### 5.1 Nomenclatura

**Classes:**

- Entidades de Domínio: `User`, `ClientSystem`
- Entidades JPA: `UserEntity`, `ClientSystemEntity`
- Services: `UserService`, `TokenService`
- Controllers: `UserAdminController`, `AuthorizationController`
- DTOs: `CreateUserRequest`, `UserResponse`
- Repositories: `UserRepository` (interface), `UserRepositoryImpl` (implementação)
- Mappers: `UserMapper`, `UserEntityMapper`

**Métodos:**

- Queries: `findByUsername()`, `existsByEmail()`
- Commands: `createUser()`, `blockUser()`
- Validações: `validateCredentials()`, `canAccessSystem()`

### 5.2 Packages por Feature vs Layer

**Escolha adotada: Hybrid (Layer First, Feature Second)**

- Primeiro nível: camadas (`application`, `domain`, `infrastructure`)
- Segundo nível: features (`user`, `system`, `token`)

**Justificativa:**

- Mantém separação clara de responsabilidades
- Facilita navegação por desenvolvedores novos
- Permite identificar rapidamente violações de camada

---

## 6. Padrões de Design Aplicados

### 6.1 Repository Pattern

- Interfaces no Domain
- Implementações no Infrastructure
- Abstrai detalhes de persistência

### 6.2 Service Pattern

- Application Services: orquestração
- Domain Services: lógica de negócio complexa que não pertence a uma entidade

### 6.3 DTO Pattern

- Separação entre modelo de domínio e modelo de API
- Mappers dedicados para conversão

### 6.4 Value Objects

- Objetos imutáveis que representam conceitos
- Exemplos: `Password`, `Email`, `JwtPayload`

### 6.5 Strategy Pattern

- Diferentes estratégias de autenticação
- Diferentes provedores de e-mail

---

## 7. Tecnologias e Frameworks

### 7.1 Core

- **Spring Boot 3.x**: Framework principal
- **Spring Security**: Autenticação e autorização
- **Spring OAuth2 Authorization Server**: Implementação OAuth2/OIDC
- **Java 17+**: Linguagem base

### 7.2 Persistência

- **Spring Data JPA**: Acesso a dados
- **Hibernate**: ORM
- **PostgreSQL**: Banco de dados (recomendado)
- **Flyway/Liquibase**: Migrations

### 7.3 Segurança

- **jjwt (Java JWT)**: Manipulação de tokens
- **BCrypt**: Hash de senhas

### 7.4 Comunicação

- **Spring Mail**: Envio de e-mails
- **REST**: Protocolo de comunicação

### 7.5 Testes

- **JUnit 5**: Framework de testes
- **Mockito**: Mocks
- **Spring Boot Test**: Testes de integração
- **TestContainers**: Testes com banco de dados

### 7.6 Documentação

- **SpringDoc OpenAPI**: Documentação de API

---

## 8. Configuração e Profiles

### 8.1 Arquivos de Configuração

```
resources/
├── application.yml              # Configurações comuns
├── application-dev.yml          # Desenvolvimento
├── application-test.yml         # Testes
└── application-prod.yml         # Produção
```

### 8.2 Propriedades Principais

```yaml
# application.yml
spring:
  application:
    name: auth-server
    
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    
auth:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 3600000  # 1 hora
    issuer: https://auth.seudominio.com
    
  oauth2:
    authorization-code-ttl: 300  # 5 minutos
    
  email:
    smtp:
      host: ${SMTP_HOST}
      port: ${SMTP_PORT}
      username: ${SMTP_USERNAME}
      password: ${SMTP_PASSWORD}
```

---

## 9. Estrutura de Testes

```
test/
└── java/
    └── com/seudominio/authserver/
        ├── unit/
        │   ├── domain/              # Testes unitários de entidades
        │   │   └── UserTest.java
        │   └── application/         # Testes de services (mockando repos)
        │       └── UserServiceTest.java
        │
        ├── integration/
        │   └── infrastructure/      # Testes de repositórios (TestContainers)
        │       └── UserRepositoryImplTest.java
        │
        └── e2e/
            └── presentation/        # Testes de API (MockMvc)
                └── AuthorizationControllerTest.java
```

---

## 10. Migrations de Banco de Dados

### 10.1 Estratégia

- **Flyway** para controle de versão do schema
- Migrations versionadas sequencialmente
- Nunca alterar migrations aplicadas em produção

### 10.2 Estrutura

```
resources/
└── db/
    └── migration/
        ├── V1__create_user_table.sql
        ├── V2__create_client_system_table.sql
        ├── V3__create_system_role_table.sql
        ├── V4__create_user_system_table.sql
        ├── V5__create_user_system_role_table.sql
        ├── V6__create_authorization_code_table.sql
        └── V7__create_password_reset_token_table.sql
```

---

## 11. Segurança na Implementação

### 11.1 Armazenamento de Senhas

- **BCrypt** com strength 12+
- Nunca armazenar senhas em texto plano
- Implementado no Value Object `Password`

### 11.2 Validação de Tokens

- Validação de assinatura JWT
- Validação de claims (`exp`, `aud`, `iss`)
- Implementado em `JwtValidator`

### 11.3 Protection contra CSRF

- Aplicável apenas em endpoints que modificam estado
- OAuth2 endpoints protegidos por `state` parameter

---

## 12. Logs e Auditoria

### 12.1 Níveis de Log

- **ERROR**: Falhas críticas
- **WARN**: Tentativas de acesso negado
- **INFO**: Operações administrativas (CRUD)
- **DEBUG**: Fluxo de autenticação detalhado

### 12.2 Campos de Auditoria

Presentes em todas as entidades:

- `created_at`
- `updated_at`
- `created_by`

---