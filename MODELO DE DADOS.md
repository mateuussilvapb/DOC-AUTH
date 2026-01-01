# Modelo de Dados — Auth Server

## 1. Princípios Gerais

- O Auth Server é a fonte central de identidade
- Perfis pertencem a sistemas específicos
- Não existem permissões funcionais no Auth Server
- Tokens JWT não são persistidos

---

## 2. Entidades

### 2.1 Usuário (`user`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| username | String |
| email | String |
| password_hash | String |
| name | String |
| is_master | Boolean |
| status | Enum |
| created_at | Timestamp |
| updated_at | Timestamp |

---

### 2.2 Sistema Cliente (`client_system`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| client_id | String |
| client_secret | String |
| name | String |
| redirect_uri | String |
| status | Enum |
| created_at | Timestamp |
| updated_at | Timestamp |

---

### 2.3 Perfil do Sistema (`system_role`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| system_id | FK → client_system |
| code | String |
| description | String |
| created_at | Timestamp |

---

### 2.4 Vínculo Usuário ↔ Sistema (`user_system`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| user_id | FK → user |
| system_id | FK → client_system |
| status | Enum |
| created_at | Timestamp |

---

### 2.5 Perfis do Usuário no Sistema (`user_system_role`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| user_system_id | FK → user_system |
| system_role_id | FK → system_role |

---

### 2.6 Authorization Code (`authorization_code`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| code | String |
| user_id | FK → user |
| system_id | FK → client_system |
| expires_at | Timestamp |
| used | Boolean |
| created_at | Timestamp |

---

### 2.7 Token de Recuperação de Senha (`password_reset_token`)

| Campo | Tipo |
|-----|-----|
| id | UUID |
| token | String |
| user_id | FK → user |
| expires_at | Timestamp |
| used | Boolean |
| created_at | Timestamp |

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

---

## 4. Alinhamento com JWT

- `user_id`, `username`, `email`, `name` → Usuário
- `is_master` → Usuário
- `client_id` → Sistema cliente
- `system_roles` → Perfis do sistema solicitante
