# Modelo de Dados — Auth Server

## 1. Princípios Gerais

- O Auth Server é a fonte central de identidade
- Perfis pertencem a sistemas específicos
- Não existem permissões funcionais no Auth Server
- Tokens JWT não são persistidos

---

## 2. Entidades

### 2.1 Usuário (`user`)

Representa a identidade do usuário no domínio do Auth Server.

| Campo         | Tipo        | Observações                  |
| ------------- | ----------- | ---------------------------- |
| id            | UUID / Long | Identificador interno        |
| username      | String      | Único                        |
| email         | String      | Único                        |
| password_hash | String      | Hash seguro (BCrypt, Argon2) |
| name          | String      | Nome completo                |
| is_master     | Boolean     | Usuário global               |
| status        | Enum        | ACTIVE, BLOCKED, DISABLED    |
| created_at    | Timestamp   | Auditoria                    |
| updated_at    | Timestamp   | Auditoria                    |
| created_by    | String      | Auditoria                    |

**Observações arquiteturais:**

- `is_master` impacta diretamente o fluxo de login
- Usuário Master ignora vínculos
- Nenhuma informação de sistema ou perfil fica aqui

---

### 2.2 Sistema Cliente (`client_system`)

Representa um sistema que consome autenticação.

| Campo         | Tipo        | Observações             |
| ------------- | ----------- | ----------------------- |
| id            | UUID / Long | Identificador interno   |
| client_id     | String      | Público (OAuth2)        |
| client_secret | String      | Apenas server-to-server |
| name          | String      | Nome do sistema         |
| redirect_uri  | String      | Validado no authorize   |
| status        | Enum        | ACTIVE, INACTIVE        |
| created_at    | Timestamp   | Auditoria               |
| updated_at    | Timestamp   | Auditoria               |
| created_by    | String      | Auditoria               |

**Observações:**

- `client_id` aparece no JWT (`aud`, `client_id`)
- Pode suportar múltiplos `redirect_uris` futuramente

---

### 2.3 Perfil do Sistema (`system_role`)

Perfis de sistemas atribuíveis a usuários.

| Campo       | Tipo                  | Observações                 |
| ----------- | --------------------- | --------------------------- |
| id          | UUID / Long           | Identificador interno       |
| system_id   | FK → client_system    | Roles pertencem a sistemas  |
| code        | String                | Ex: ADMIN, USER             |
| description | String                | Texto explicativo           |
| created_at  | Timestamp             | Auditoria                   |
| created_by  | String                | Auditoria                   |

**Observações:**

- Entidade fraca (depende da existência de um sistema)
- Não conhece permissões funcionais
- Mapeamento para permissões ocorre nos sistemas clientes

---

### 2.4 Vínculo Usuário ↔ Sistema (`user_system`)

Representa o acesso de um usuário a um sistema.

| Campo      | Tipo               | Observações     |
| ---------- | ------------------ | --------------- |
| id         | UUID / Long        | Identificador   |
| user_id    | FK → user          |                 |
| system_id  | FK → client_system |                 |
| status     | Enum               | ACTIVE, REVOKED |
| created_at | Timestamp          | Auditoria       |
| created_by | String             | Auditoria       |

**Regras:**

- Um usuário pode ter vários sistemas
- Um sistema pode ter vários usuários
- Usuário Master não precisa de registro aqui

---

### 2.5 Perfis do Usuário no Sistema (`user_system_role`)

Associação N:N entre vínculo e perfis.

| Campo          | Tipo             | Observações |
| -------------- | ---------------- | ----------- |
| id             | UUID / Long      |             |
| user_system_id | FK → user_system |             |
| system_role_id | FK → role        |             |

**Resultado prático:**

- Um usuário pode ter múltiplos perfis em um mesmo sistema
- O JWT inclui essa informação (system_roles)

---

### 2.6 Authorization Code (`authorization_code`)

Utilizado no OAuth2 Authorization Code Flow.

| Campo      | Tipo               | Observações             |
| ---------- | ------------------ | ----------------------- |
| id         | UUID / Long        |                         |
| code       | String             | Valor enviado ao client |
| user_id    | FK → user          |                         |
| system_id  | FK → client_system |                         |
| expires_at | Timestamp          | Curto (ex: 5 min)       |
| used       | Boolean            | Uso único               |
| created_at | Timestamp          |                         |

**Observações:**

- Nunca reutilizado
- Apagável por job de limpeza
- Não substitui token

---

### 2.7 Token de Recuperação de Senha (`password_reset_token`)

Fluxo fora do login principal, mas necessário.

| Campo      | Tipo        | Observações |
| ---------- | ----------- | ----------- |
| id         | UUID / Long |             |
| token      | String      | Uso único   |
| user_id    | FK → user   |             |
| expires_at | Timestamp   |             |
| used       | Boolean     |             |
| created_at | Timestamp   |             |


---

## 3. Relacionamentos

```
USER
 └──< USER_SYSTEM >── CLIENT_SYSTEM
           |
           └──< USER_SYSTEM_ROLE >── SYSTEM_ROLE
                                        |
                                        └── CLIENT_SYSTEM
```

```
USER
 └── AUTHORIZATION_CODE
```

```
USER
 └── PASSWORD_RESET_TOKEN
```
---

## 4. Alinhamento com JWT

| Claim JWT     | Origem no Modelo        |
| ------------- | ----------------------- |
| sub / user_id | user.id                 |
| username      | user.username           |
| email         | user.email              |
| name          | user.name               |
| is_master     | user.is_master          |
| client_id     | client_system.client_id |
| system_roles  | role.code (via vínculo) |

