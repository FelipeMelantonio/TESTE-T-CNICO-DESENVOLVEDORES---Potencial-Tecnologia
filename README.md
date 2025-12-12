# 🏦 Mini Conta Digital API

**Spring Boot + JWT + Docker**

API REST para cadastro de usuários, contas digitais e transações internas/externas, com autenticação JWT, histórico completo e auditoria.

---

## 📌 Instruções

(Linha 174 começa realmente os testes em JSON)

---

## 🔧 Pré-requisitos

- Docker
- Docker Compose

---

## 🚀 1. Executar a Aplicação

### Subir a aplicação

```bash
docker compose up -d --build
```

A aplicação será iniciada automaticamente.

**API disponível em:**
👉 http://localhost:8080

---

## 📊 2. Visualizar Logs e Auditoria

Para acompanhar os logs da aplicação e os registros de auditoria:

```bash
docker logs -f miniconta_api
```

---

## 📌 Regras Gerais + Operações

### 🔐 Token

Token sempre será enviado assim:

```
Authorization: Bearer <TOKEN>
```

### 👤 Usuários

Existem dois perfis:

- **USER**
- **ADMIN**

#### Regras:
- Lembre que o ID 1 é do admin
- Obrigatório registrar e depois fazer login
- A cada endpoint mostra no console os detalhes da requisição
- Somente ADMIN pode criar outro ADMIN
- Existe um ADMIN pré-criado automaticamente ao iniciar a aplicação
- Valor > 0 / não pode deixar saldo negativo

#### Admin Padrão (Seed):

```
email: admin@admin.com
senha: admin123
id: 1
```

Todos os outros usuários começam a partir do id = 2.

---

## 🔑 1️⃣ Login como ADMIN (obrigatório primeiro)

**Endpoint:**
```
POST /auth/login
```

**Payload:**
```json
{
  "email": "admin@admin.com",
  "senha": "admin123"
}
```

---

## 👑 2️⃣ Criar Administrador (ADMIN)

⚠️ Somente com token de ADMIN.

**Endpoint:**
```
POST /auth/registrar
```

**Payload:**
```json
{
  "nome": "Administrador",
  "email": "admin2@test.com",
  "senha": "123456",
  "cpf": "22222222222",
  "role": "ADMIN"
}
```

---

## 👤 3️⃣ Registrar Usuário (USER)

**Endpoint:**
```
POST /auth/registrar
```

**Payload:**
```json
{
  "nome": "João Silva",
  "email": "joao@example.com",
  "senha": "senha123",
  "cpf": "12345678900",
  "role": "USER"
}
```

---

## 🔐 4️⃣ Login como Usuário (USER)

**Endpoint:**
```
POST /auth/login
```

**Payload:**
```json
{
  "email": "joao@example.com",
  "senha": "senha123"
}
```

---

## 📌 Controle por Token (JWT)

Todas as operações financeiras exigem autenticação via JWT (Transferir, Sacar, Depositar...)

O usuário autenticado (token) é sempre considerado o responsável pela operação.

O sistema não confia apenas em IDs enviados no corpo.

Os dados são validados contra o usuário autenticado no token.

---

## 💰 Operações Financeiras

---

## 🏦 Conta Digital

### 5️⃣ Criar Conta para Usuário (1 Conta por Usuário)

🔒 Precisa de token (do usuário autenticado)

**Endpoint:**
```
POST /contas/usuario/{usuarioId}
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

---

## 🧾 Histórico de Transações

🔒 Precisa de token

**Endpoint:**
```
GET /transacoes/conta/{contaId}
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

**Retorna:**

- tipo
- valor
- conta origem/destino
- timestamp
- saldo após operação

---

## 🔎 Consultas de Usuários e Contas

### Buscar Usuário por ID

🔒 Precisa de token

**Endpoint:**
```
GET /api/usuarios/{id}
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

#### Regras:

- exige JWT válido
- usuário só pode consultar seus próprios dados

---

### Buscar Conta do Usuário Autenticado

🔒 Precisa de token

**Endpoint:**
```
GET /contas/{id}
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

#### Regras:

- não permite acessar contas de outros usuários
- o token define qual conta pode ser visualizada

---

## 💸 Transações

### 6️⃣ Depósito

🔒 Precisa de token (do dono da conta)

**Endpoint:**
```
POST /transacoes/deposito
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

**Payload:**
```json
{
  "contaId": 1,
  "valor": 100.00
}
```

#### Regra de Negócio:

O depósito só pode ser realizado pelo dono da conta.
O `contaId` informado deve pertencer ao usuário autenticado no token.

#### ✅ Exemplo Válido

```json
{
  "contaId": 1,
  "valor": 100.00
}
```

#### ❌ Exemplo Inválido

```json
{
  "contaId": 2,
  "valor": 100.00
}
```

❌ Operação negada: o usuário autenticado não é o dono da conta.

---

### 7️⃣ Saque

🔒 Precisa de token (do dono da conta)

**Endpoint:**
```
POST /transacoes/saque
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

**Payload:**
```json
{
  "contaId": 1,
  "valor": 50.00
}
```

#### Regras:

- valor > 0
- não pode deixar saldo negativo

---

### 8️⃣ Transferência Interna

🔒 Precisa de token (do dono da conta de origem)

**Endpoint:**
```
POST /transacoes/transferencia-interna
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

**Payload:**
```json
{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 10.00
}
```

#### Regra de Negócio:

Somente o dono da conta de origem pode executar a transferência.
O token deve pertencer ao usuário da `contaOrigemId`.
valor > 0

#### ✅ Exemplo Válido

```json
{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 5.00
}
```

✔️ Transferência permitida.

#### ❌ Exemplo Inválido

```json
{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 5.00
}
```

❌ Operação negada: apenas o dono da conta de origem pode transferir.(Token errado: usuário que não é dono da contaOrigemId) 

---

### 9️⃣ Transferência Externa

🔒 Precisa de token (do dono da conta de origem)

**Primeiro:** listar bancos (BrasilAPI)

```
GET /bancos
GET /bancos/{codigo}
```

**Endpoint:**
```
POST /transacoes/transferencia-externa
```

**Header:**
```
Authorization: Bearer <TOKEN>
```

**Payload:**
```json
{
  "contaOrigemId": 1,
  "valor": 30.00,
  "banco": 1,
  "agencia": "1234",
  "conta": "56789-0",
  "cpfDestino": "99988877766"
}
```

#### Regra de Negócio:

Apenas o dono da conta de origem autenticado pode executar.
Não existe conta destino interna.
O sistema valida saldo antes da operação.

#### ✅ Exemplo Válido

```json
{
  "contaOrigemId": 1,
  "valor": 50.00,
  "banco": 341,
  "agencia": "1234",
  "conta": "56789-0",
  "cpfDestino": "98765432100"
}
```

✔️ Transferência externa realizada com sucesso.

#### ❌ Exemplo Inválido

```json
{
  "contaOrigemId": 1,
  "valor": 50.00,
  "banco": 341,
  "agencia": "1234",
  "conta": "56789-0",
  "cpfDestino": "98765432100"
}
```

❌ Operação negada por violação de segurança.

---

## 🕵️ Auditoria

Após cada operação, o console mostra:

```
[AUDIT] ts=2025-12-11T22:52:17 user=nicolas@example.com endpoint=POST /transacoes/deposito payload=...
```

Inclui:

- usuário
- endpoint
- data/hora
- payload
- emails de origem/destino

---

## ✅ Passo a passo dos testes
(somente JSON)

👉 Recomendo usar Thunder Client / Postman / Insomnia.

### Sequência de Testes:

1. **Login como ADMIN** (`POST /auth/login`) - usar email e senha do admin padrão
2. **Criar outro ADMIN** (se necessário so registrar e criar) - usar token do admin 1
3. **Registrar um USER** (`POST /auth/registrar`)
4. **Login como USER** (`POST /auth/login`) - usar email e senha criados
5. **Criar Conta** (`POST /contas/usuario/{usuarioId}`) - com token do USER
6. **Depósito** (`POST /transacoes/deposito`) - com token do USER
7. **Saque** (`POST /transacoes/saque`) - com token do USER
8. **Transferência Interna** (`POST /transacoes/transferencia-interna`) - com token do usuário de origem
9. **Transferência Externa** (`POST /transacoes/transferencia-externa`) - com token do usuário de origem
10. **Visualizar Histórico** (`GET /transacoes/conta/{contaId}`) - com token
11. **Consultar Auditoria** (via console)

---

## 📝 Tecnologias Utilizadas

- **Spring Boot 3.x**
- **JWT (JSON Web Tokens)**
- **Spring Security**
- **JPA/Hibernate**
- **Docker & Docker Compose**
- **Maven**

---

## 📄 Licença

Este projeto é de uso educacional e técnico.

---

## 👨‍💻 Autor

Desenvolvido por Felipe Ferreira Melantonio como teste técnico para **Potencial Tecnologia**.

