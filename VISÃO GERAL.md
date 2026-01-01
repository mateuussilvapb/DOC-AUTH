# Documento Arquitetural — Auth Server
## Visão Arquitetural de Alto Nível

---

## 1. Visão Geral

O **Auth Server** é um serviço centralizado de autenticação e autorização responsável por prover identidade, controle de acesso por sistema e emissão de tokens de segurança para múltiplas aplicações clientes independentes.

A arquitetura foi projetada para:

- Centralizar autenticação e autorização
- Desacoplar completamente os sistemas clientes da lógica de identidade
- Permitir crescimento futuro com inclusão de novos sistemas sem refatoração estrutural
- Utilizar padrões consolidados de mercado (OAuth2 + OpenID Connect)

O Auth Server atua como **Identity Provider (IdP)** e **Authorization Server**, enquanto os sistemas clientes atuam como **OAuth2 Clients / Resource Servers**.

---

## 2. Visão Arquitetural em Alto Nível

### 2.1 Visão Lógica

```
┌────────────────────────────┐
│        Sistemas Clientes   │
│ (Web Apps / APIs externas) │
└─────────────┬──────────────┘
              │ OAuth2 / OIDC
              ▼
┌────────────────────────────┐
│         Auth Server        │
│ ─────────────────────────  │
│ • Autenticação             │
│ • Autorização por sistema  │
│ • Emissão de JWT           │
│ • Gestão de usuários       │
│ • Gestão de sistemas       │
│ • Gestão de perfis         │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│     Base de Dados Auth     │
│ (Usuários, Sistemas, etc.) │
└────────────────────────────┘
```

---

## 3. Componentes de Alto Nível

### 3.1 Sistemas Clientes

Responsabilidades:

- Redirecionar usuários para autenticação no Auth Server
- Informar o identificador do sistema solicitante (`client_id`)
- Validar tokens JWT recebidos
- Criar e manter a sessão local
- Mapear **Perfis genéricos (Auth Server)** para **Permissões locais**

Características:

- Aplicações independentes
- Não armazenam credenciais de usuários
- Não conhecem regras ou permissões de outros sistemas

---

### 3.2 Auth Server

Responsável por centralizar toda a lógica de identidade e acesso dos usuários.

Responsabilidades:

- Autenticação de usuários (usuário/e-mail e senha)
- Autorização de acesso por sistema cliente
- Emissão de tokens JWT
- Exposição de endpoints OAuth2 e OpenID Connect
- Gestão administrativa via frontend próprio
- Suporte a recuperação de acesso (ex.: redefinição de senha via e-mail)

Papéis arquiteturais:

- Authorization Server
- Identity Provider
- Single Source of Truth para identidade

Tecnologias:

- Backend: Spring Boot
- Segurança: OAuth2 + OpenID Connect
- Token: JWT
- Comunicação assíncrona externa: SMTP (via provedor de e-mail transacional)

---

### 3.3 Frontend Administrativo do Auth Server

Aplicação web dedicada à administração do Auth Server.

Responsabilidades:

- CRUD de usuários
- CRUD de sistemas clientes
- CRUD de perfis genéricos
- Vínculo usuário ↔ sistema ↔ perfil
- Acesso restrito a usuários com perfil MASTER

Observações:

- Não participa do fluxo de login dos sistemas clientes
- Consome APIs REST do Auth Server

Tecnologia:

- Angular

---

### 3.4 Base de Dados do Auth Server

Responsável por armazenar exclusivamente dados de identidade e controle de acesso.

Armazena:

- Usuários
- Sistemas clientes
- Perfis genéricos
- Relacionamentos de acesso

Princípios:

- Dados centralizados
- Modelo normalizado
- Nenhuma regra de negócio específica dos sistemas clientes

---

## 4. Visão de Integração

A comunicação entre os componentes ocorre exclusivamente por protocolos padronizados.

| Origem | Destino | Protocolo |
|------|--------|----------|
| Sistema Cliente | Auth Server | OAuth2 / OIDC |
| Auth Server | Sistema Cliente | JWT |
| Frontend Admin | Auth Server | REST (HTTP + JSON) |

Não existe comunicação direta entre sistemas clientes.

---

## 5. Princípios Arquiteturais

- **Separação de Responsabilidades**  
  O Auth Server cuida de identidade e acesso; os sistemas clientes cuidam de regras de negócio.

- **Baixo Acoplamento**  
  Sistemas clientes dependem apenas do token JWT, não da implementação interna do Auth Server.

- **Centralização de Identidade**  
  Usuários e acessos são geridos em um único ponto.

- **Extensibilidade**  
  Inclusão de novos sistemas sem impacto estrutural.

- **Aderência a Padrões de Mercado**  
  OAuth2, OpenID Connect e JWT.

---

## 6. Escopo Deliberadamente Fora do Auth Server

O Auth Server **não é responsável** por:

- Permissões funcionais (ex.: `CRIAR_PEDIDO`, `APROVAR_ESTOQUE`)
- Regras de negócio específicas de cada sistema
- Controle de acesso fino dentro dos sistemas clientes

Essas responsabilidades pertencem exclusivamente aos sistemas clientes.

---

## 7. Relacionamento com Próximas Seções

Esta visão arquitetural de alto nível será detalhada nas próximas seções do Documento Arquitetural:

1. Visão de Componentes (C4 – Container e Component)
2. Visão de Fluxos
   - Login
   - Emissão de Token
   - Validação de Token
3. Modelo Conceitual de Dados
4. Decisões Arquiteturais (ADRs)

---
