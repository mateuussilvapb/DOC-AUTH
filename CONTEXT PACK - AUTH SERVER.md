# Context Pack — Auth Server

## 1. Contexto Geral

Estou projetando um Servidor de Autenticação centralizado (Auth Server) responsável por autenticar usuários e autorizar o acesso a múltiplos sistemas clientes independentes.

Inicialmente haverá apenas um sistema cliente, mas a arquitetura deve suportar crescimento futuro sem refatorações estruturais.

---

## 2. Objetivo do Auth Server

O Auth Server deve:

- Autenticar usuários (login com usuário/e-mail e senha)
- Emitir tokens JWT para sistemas clientes
- Controlar quais usuários podem acessar quais sistemas
- Centralizar o cadastro e gestão de:
  - Usuários
  - Sistemas clientes
  - Perfis vinculados a sistemas específicos

---

## 3. Escopo do Auth Server

Inclui:

- Autenticação centralizada (OAuth2 + OIDC)
- Emissão e validação de tokens JWT
- Gestão de acesso usuário ↔ sistema
- Gestão de perfis por sistema
- Interface administrativa para usuários MASTER:
  - CRUD de usuários
  - CRUD de sistemas
  - CRUD de perfis
  - Vínculo usuário ↔ sistema ↔ perfil

---

## 4. Fora do Escopo

O Auth Server não é responsável por:

- Regras de negócio específicas de cada sistema
- Permissões funcionais
- Controle fino de acesso dentro dos sistemas clientes

---

## 5. Modelo Conceitual de Autorização

- Um usuário pode estar vinculado a um ou mais sistemas
- Para cada vínculo usuário ↔ sistema:
  - O usuário possui um ou mais perfis
- Perfis existem exclusivamente no contexto de um sistema
- Um perfil não pode existir sem um sistema associado
- Sistemas clientes mapeiam:
  - Perfil (Auth Server) → Permissões locais

---

## 6. Usuário MASTER

- Usuários MASTER possuem acesso a todos os sistemas
- Não possuem perfis obrigatórios
- A informação é propagada no token JWT
- As regras de acesso total são aplicadas nos sistemas clientes

---

## 7. Fluxo de Login

- Usuário acessa um sistema cliente
- Sistema redireciona para o Auth Server
- Usuário informa credenciais
- Auth Server valida credenciais
- Auth Server valida acesso ao sistema
- Auth Server emite Authorization Code
- Sistema troca o code por JWT
- Sistema valida o token e cria sessão local

---

## 8. Tecnologias Alvo

- Backend: Spring Boot
- Autenticação: OAuth2 + OpenID Connect
- Token: JWT
- Frontend Administrativo: Angular
- Sistemas clientes: aplicações independentes
