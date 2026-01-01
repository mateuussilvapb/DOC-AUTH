# Contrato do JWT

## Fluxo de Login (OAuth2 Authorization Code)

## 1. Objetivo

Este documento define o contrato do JSON Web Token (JWT) emitido pelo
Auth Server após a conclusão bem-sucedida do fluxo de login baseado em
OAuth2 Authorization Code.

O JWT é utilizado pelos Sistemas Clientes para:

-   Identificar o usuário autenticado
-   Validar autorização de acesso
-   Determinar permissões e vínculos com sistemas
-   Garantir integridade e autenticidade das requisições

Este contrato deve ser considerado estável e versionado, pois impacta
diretamente todos os consumidores do Auth Server.

## 2. Contexto no Fluxo de Login

O JWT é emitido no seguinte ponto do fluxo:

-   Usuário autentica-se no Auth Server
-   Auth valida credenciais, papel Master e vínculo com o sistema
-   Auth gera Authorization Code
-   Sistema Cliente troca o code pelo token no endpoint de token
-   Auth valida o code e emite o JWT
-   Sistema Cliente armazena o token e libera o acesso

## 3. Tipo de Token

-   Tipo: Access Token
-   Formato: JWT (JSON Web Token)
-   Assinatura: Assimétrica (RSA)
-   Padrão: RFC 7519

Opcionalmente, pode coexistir com:

-   Refresh Token (fora do escopo deste documento)

## 4. Estrutura Geral do JWT

O token é composto por três partes:

-   Header
-   Payload (Claims)
-   Signature

## 5. Header

O Header contém os metadados de assinatura do token.

Campos obrigatórios:

-   alg: RS256
-   typ: JWT
-   kid: Identificador da chave pública utilizada na assinatura

Exemplo conceitual:

alg: RS256\
typ: JWT\
kid: auth-key-01

## 6. Payload (Claims)

### 6.1 Claims Padrão (RFC)

Claims obrigatórias para conformidade e segurança.

-   iss (Issuer): Identificador do Auth Server que emitiu o token
-   sub (Subject): Identificador único do usuário no domínio de
    identidade
-   aud (Audience): Identificador do Sistema Cliente que consumirá o
    token
-   exp (Expiration Time): Data e hora de expiração do token
-   iat (Issued At): Data e hora de emissão do token
-   jti (JWT ID): Identificador único do token para rastreabilidade e
    revogação

### 6.2 Claims de Identidade do Usuário

Essas claims representam informações básicas do usuário autenticado.

-   user_id: Identificador interno do usuário
-   username: Login utilizado no processo de autenticação
-   email: Endereço de e-mail do usuário
-   name: Nome completo do usuário

### 6.3 Claims de Papel Global

Essas claims refletem decisões tomadas no ponto H do fluxo.

-   is_master: Indica se o usuário possui papel Master (equivalente ao
    Global Admin)

Valores possíveis:

-   true
-   false

### 6.4 Claims de Contexto do Sistema

Essas claims refletem decisões tomadas no ponto J do fluxo.

-   client_id: Identificador do Sistema Cliente para o qual o token foi
    emitido
-   system_roles: Lista de papéis do usuário dentro do sistema
    específico
-   system_permissions: Lista de permissões concedidas ao usuário no
    sistema

Essas informações permitem autorização sem nova consulta ao Auth Server.

### 6.5 Claims de Segurança e Controle

Claims adicionais para governança e auditoria.

-   auth_method: Método de autenticação utilizado (ex: password)
-   session_id: Identificador da sessão de autenticação
-   token_version: Versão do contrato do JWT

## 7. Exemplo Conceitual de Payload

Estrutura ilustrativa do payload do JWT:

iss: https://auth.seudominio.com\
sub: 12345\
aud: sistema-cliente-x\
exp: 1710000000\
iat: 1709996400\
jti: a1b2c3d4

user_id: 12345\
username: mateus.dias\
email: mateus@email.com\
name: Mateus Dias

is_master: false

client_id: sistema-cliente-x\
system_roles: ADMIN, USER\
system_permissions: READ, WRITE

auth_method: password\
session_id: sess-789\
token_version: 1

## 8. Regras de Emissão do Token

O Auth Server somente deve emitir o JWT se:

-   As credenciais forem válidas
-   O usuário for Master ou possuir vínculo com o sistema
-   O Authorization Code for válido, não expirado e não reutilizado

Caso contrário, o endpoint de token deve retornar erro OAuth2
apropriado.

## 9. Validação do Token pelos Sistemas Clientes

Os Sistemas Clientes devem:

-   Validar assinatura via chave pública do Auth Server
-   Validar expiração (exp)
-   Validar audience (aud)
-   Validar issuer (iss)
-   Validar presença das claims obrigatórias

Nenhuma chamada síncrona ao Auth Server é necessária para validação do
token.

## 10. Considerações Arquiteturais

-   O JWT é autocontido e stateless
-   Alterações no contrato exigem versionamento explícito
-   Claims não devem conter dados sensíveis
-   O tamanho do token deve ser monitorado

## 11. Próximos Documentos Relacionados

Este documento se conecta diretamente com:

-   Mapeamento de Endpoints (authorize, token)
-   Mapeamento de Responsabilidades Auth x Sistemas
-   Modelo de Dados de Usuário, Papel e Vínculo
-   Estratégia de Refresh Token e Logout
