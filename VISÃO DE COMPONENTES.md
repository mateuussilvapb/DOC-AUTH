# Documento Arquitetural - Visão de Componentes

## Auth Server

## 1. Objetivo da Visão de Componentes

Esta seção descreve a decomposição interna do Auth Server em componentes
arquiteturais, suas responsabilidades e relações, com o objetivo de:

-   Tornar explícita a separação de responsabilidades internas
-   Conectar os fluxos definidos aos componentes responsáveis
-   Preparar o terreno para implementação sem engessá-la
-   Facililitar manutenção, evolução e onboarding técnico

Esta visão corresponde ao nível de Componentes do modelo C4.

## 2. Visão Geral dos Componentes

O Auth Server é estruturado em componentes coesos, organizados por
responsabilidade funcional e técnica.

Em alto nível, os principais componentes são:

-   API de Autenticação (OAuth2 / OIDC)
-   Camada de Segurança
-   Gerenciamento de Usuários
-   Gerenciamento de Sistemas Clientes
-   Gerenciamento de Perfis e Vínculos
-   Emissão e Validação de Tokens
-   Recuperação de Acesso
-   API Administrativa
-   Camada de Persistência

## 3. Diagrama Conceitual de Componentes (Descrição Textual)

**Sistema Cliente**\
→ API de Autenticação\
→ Camada de Segurança\
→ Serviços de Domínio\
→ Camada de Persistência

**Frontend Administrativo**\
→ API Administrativa\
→ Serviços de Domínio\
→ Camada de Persistência

**Serviços de Domínio**\
→ Token Service\
→ Email Service (quando aplicável)

## 4. Componentes e Responsabilidades

### 4.1 API de Autenticação (OAuth2 / OIDC)

**Responsabilidade:**

-   Expor endpoints padrão de autenticação e autorização
-   Receber requisições de autorização dos sistemas clientes
-   Orquestrar o fluxo de login

**Principais interações:**

-   Camada de Segurança
-   Token Service
-   Serviços de Usuário e Sistema

**Observações:**

-   Atua como porta de entrada para o fluxo de login
-   Não contém regras de negócio profundas

### 4.2 Camada de Segurança

**Responsabilidade:**

-   Autenticar credenciais de usuários
-   Validar client_id e redirect_uri
-   Aplicar políticas de segurança
-   Garantir conformidade com OAuth2 e OIDC

**Principais interações:**

-   Gerenciamento de Usuários
-   Gerenciamento de Sistemas
-   Token Service

**Observações:**

-   Centraliza preocupações de segurança
-   Não conhece regras de negócio específicas

### 4.3 Gerenciamento de Usuários

**Responsabilidade:**

-   Manter dados de usuários
-   Validar status do usuário (ativo, bloqueado, etc.)
-   Gerenciar credenciais (hash de senha)
-   Identificar usuários especiais (ex.: Usuário Master)

**Principais interações:**

-   Camada de Persistência
-   Camada de Segurança
-   Recuperação de Acesso

### 4.4 Gerenciamento de Sistemas Clientes

**Responsabilidade:**

-   Manter cadastro de sistemas clientes
-   Validar existência e status do sistema solicitante
-   Fornecer metadados do sistema (ex.: redirect_uri)

**Principais interações:**

-   Camada de Persistência
-   Camada de Segurança

### 4.5 Gerenciamento de Perfis e Vínculos

**Responsabilidade:**

-   Gerenciar perfis genéricos
-   Gerenciar vínculo usuário ↔ sistema ↔ perfil
-   Validar autorização de acesso ao sistema

**Principais interações:**

-   Gerenciamento de Usuários
-   Gerenciamento de Sistemas
-   Token Service

**Observações:**

-   Não define permissões funcionais
-   Apenas valida acesso ao sistema

### 4.6 Token Service

**Responsabilidade:**

-   Gerar Authorization Code
-   Validar Authorization Code
-   Emitir tokens JWT
-   Definir e incluir claims no token
-   Controlar expiração e uso único do code

**Principais interações:**

-   Camada de Segurança
-   Gerenciamento de Perfis e Vínculos

**Observações:**

-   Componente crítico de segurança
-   Totalmente isolado de regras de negócio dos sistemas clientes

### 4.7 Recuperação de Acesso

**Responsabilidade:**

-   Gerar tokens de recuperação de senha
-   Controlar expiração e uso único do token
-   Orquestrar redefinição de senha
-   Disparar envio de e-mails de recuperação

**Principais interações:**

-   Gerenciamento de Usuários
-   Email Service
-   Camada de Persistência

### 4.8 Email Service

**Responsabilidade:**

-   Enviar e-mails transacionais
-   Integrar com provedor SMTP externo
-   Abstrair detalhes do serviço de e-mail

**Principais interações:**

-   Recuperação de Acesso

**Observações:**

-   Comunicação externa
-   Não contém lógica de domínio

### 4.9 API Administrativa

**Responsabilidade:**

-   Expor endpoints administrativos
-   Atender o frontend administrativo
-   Orquestrar operações de gestão

**Funcionalidades:**

-   CRUD de usuários
-   CRUD de sistemas
-   CRUD de perfis
-   Vínculo usuário ↔ sistema ↔ perfil

**Principais interações:**

-   Serviços de Domínio
-   Camada de Persistência

### 4.10 Camada de Persistência

**Responsabilidade:**

-   Acesso a dados
-   Persistência de entidades de identidade
-   Garantir integridade e consistência dos dados

**Armazena:**

-   Usuários
-   Sistemas clientes
-   Perfis
-   Vínculos
-   Tokens temporários (authorization code, reset de senha)

**Observações:**

-   Nenhuma lógica de negócio
-   Reutilizável por todos os serviços

## 5. Mapeamento Componentes × Fluxos

Login: API de Autenticação, Segurança, Usuários, Sistemas, Perfis, Token
Service\
Troca de Code por Token: API de Autenticação, Segurança, Token Service\
Recuperação de Senha: Recuperação de Acesso, Usuários, Email Service\
Administração: API Administrativa, Serviços de Domínio

## 6. Princípios Arquiteturais Aplicados

-   Separação clara de responsabilidades
-   Baixo acoplamento entre componentes
-   Alta coesão interna
-   Segurança centralizada
-   Extensibilidade para novos fluxos

## 7. Relacionamento com Outros Documentos

-   Visão Arquitetural de Alto Nível (Readme)
-   Documentação dos Fluxos (Login, Recuperação de Acesso, etc.)
-   Contrato do JWT
-   Modelo de Dados (ER)

## 8. Considerações Finais

A decomposição apresentada permite:

-   Evoluir o Auth Server sem impacto nos sistemas clientes
-   Introduzir novos fluxos com mínimo impacto estrutural
-   Manter segurança e governança centralizadas

Esta visão serve como base para decisões de implementação, testes e
evolução arquitetural.
