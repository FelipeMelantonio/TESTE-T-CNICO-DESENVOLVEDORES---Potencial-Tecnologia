# 🏦 Mini Conta Digital API  
**Spring Boot + JWT + Docker**

API REST para cadastro de usuários, contas digitais e transações internas/externas, com autenticação JWT, histórico completo e auditoria.

---

## 📌 Instruções(Linha 172 começa realmente os testes em json)

### 🔧 Pré-requisitos
- **Docker**
- **Docker Compose**

---

### 🚀 1. Executar a aplicação

#### Subir a aplicação

```bash
docker compose up -d --build
A aplicação será iniciada automaticamente.
API disponível em:
👉 http://localhost:8080

📊 2. Visualizar logs e auditoria
Para acompanhar os logs da aplicação e os registros de auditoria:
docker logs -f miniconta_api

📌 Regras Gerais
🔐 Token
Token sempre vai ser assim:


Authorization: Bearer <TOKEN>
👤 Usuários
Existem dois perfis:

USER

ADMIN

Regras:

Obrigatório registrar e depois fazer login.

Somente ADMIN pode criar outro ADMIN.

Existe um ADMIN pré-criado automaticamente ao iniciar a aplicação.

Admin padrão (seed):

email: admin@admin.com
senha: admin123
id: 1
Todos os outros usuários começam a partir do id = 2.

🔑 1️⃣ Login como ADMIN (obrigatório primeiro)
POST /auth/login


{
  "email": "admin@admin.com",
  "senha": "admin123"
}
👑 2️⃣ Criar Administrador (ADMIN)
⚠️ Somente com token de ADMIN.


{
  "nome": "Administrador",
  "email": "admin2@test.com",
  "senha": "123456",
  "cpf": "22222222222",
  "role": "ADMIN"
}
📌 Controle por Token (JWT)
Todas as operações financeiras exigem autenticação via JWT
(Transferir, Sacar, Depositar...)

O usuário autenticado (token) é sempre considerado o responsável pela operação.

O sistema não confia apenas em IDs enviados no corpo.

Os dados são validados contra o usuário autenticado no token.

💰 Depósito — Regra de Negócio
O depósito só pode ser realizado pelo dono da conta.

O contaId informado deve pertencer ao usuário autenticado no token.

✅ Exemplo válido
Token → Usuário dono da conta ID 1:


{
  "contaId": 1,
  "valor": 100.00
}
❌ Exemplo inválido
Token → Usuário não dono da conta:

{
  "contaId": 2,
  "valor": 100.00
}
❌ Operação negada: o usuário autenticado não é o dono da conta.

🔁 Transferência Interna (entre contas do sistema)
📌 Regra de Negócio
Somente o dono da conta de origem pode executar a transferência.

O token deve pertencer ao usuário da contaOrigemId.

✅ Exemplo válido
Token → Usuário dono da conta 1:

{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 5.00
}
✔️ Transferência permitida.

❌ Exemplo inválido
Token → Usuário dono da conta 2:


{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 5.00
}
❌ Operação negada: apenas o dono da conta de origem pode transferir.

🌍 Transferência Externa
📌 Regra de Negócio
Apenas o dono da conta de origem autenticado pode executar.

Não existe conta destino interna.

O sistema valida saldo antes da operação.

✅ Exemplo válido
Token → Usuário dono da conta 1:


{
  "contaOrigemId": 1,
  "valor": 50.00,
  "banco": "341",
  "agencia": "1234",
  "conta": "56789-0",
  "cpf": "98765432100"
}
✔️ Transferência externa realizada com sucesso.

❌ Exemplo inválido
Token → Usuário que não é dono da conta de origem:


{
  "contaOrigemId": 1,
  "valor": 50.00,
  "banco": "341",
  "agencia": "1234",
  "conta": "56789-0",
  "cpf": "98765432100"
}
❌ Operação negada por violação de segurança.

✅ AQUI COMEÇA OS PASSOS A PASSOS DOS TESTES
(somente JSON)

👉 Recomendo usar Thunder Client / Postman / Insomnia.

✅ Endpoints
1️⃣ Cadastro de usuário
POST /auth/registrar


{
  "nome": "Nicolas",
  "email": "nicolas@example.com",
  "senha": "123456",
  "cpf": "12345678901"
}
2️⃣ Login (gera JWT)
POST /auth/login


{
  "email": "nicolas@example.com",
  "senha": "123456"
}
🏦 Conta Digital
3️⃣ Criar conta para usuário
(1 conta por usuário)

🔒 Precisa token
Header:


Authorization: Bearer <TOKEN>
POST /contas/usuario/{usuarioId}
Exemplo:


POST /contas/usuario/1
Resposta:

{
  "id": 1,
  "numeroConta": "549312",
  "saldo": 0,
  "dataCriacao": "2025-12-11T18:19:17.8572075",
  "status": "ATIVA",
  "usuario": {
    "id": 1,
    "nome": "Nicolas",
    "email": "nicolas@example.com",
    "cpf": "12345678901",
    "role": "USER"
  }
}
Regras:

saldo inicial = 0

erro esperado ao criar outra conta:


"Usuário já possui conta."
💸 Transações
4️⃣ Depósito
POST /transacoes/deposito

{
  "contaId": 1,
  "valor": 100.00
}
5️⃣ Saque
POST /transacoes/saque


{
  "contaId": 1,
  "valor": 50.00
}
Regras:

valor > 0

não pode deixar saldo negativo

6️⃣ Transferência interna
POST /transacoes/transferencia-interna


{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 10.00
}
7️⃣ Transferência externa
Antes: listar bancos (BrasilAPI)


GET /bancos
GET /bancos/{codigo}

{
  "contaOrigemId": 1,
  "valor": 30.00,
  "banco": 1,
  "agencia": "1234",
  "conta": "56789-0",
  "cpfDestino": "99988877766"
}
🧾 Histórico de transações
GET /transacoes/conta/{contaId}

Retorna:

tipo

valor

conta origem/destino

timestamp

saldo após operação

🕵️ Auditoria
Após cada operação, o console mostra:


[AUDIT] ts=2025-12-11T22:52:17 user=nicolas@example.com endpoint=POST /transacoes/deposito payload=...
Inclui:

usuário

endpoint

data/hora

payload

emails de origem/destino

🔎 Buscar usuário por ID
GET /api/usuarios/{id}

Regra:

exige JWT válido

usuário só pode consultar seus próprios dados

🔎 Buscar conta do usuário autenticado
GET /contas/{id}

Regra:

não permite acessar contas de outros usuários

o token define qual conta pode ser visualizada

