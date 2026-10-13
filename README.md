# 📘 Briefing Técnico e Contrato de API — VisitaJá (Frontend)

Olá! Bem-vindo ao projeto **VisitaJá**. 
Este documento contém o mapeamento das telas necessárias e o contrato da API REST para a construção do Frontend (React/Vite).

A API base já está rodando e com o CORS liberado para o `localhost`.

---

## 🖥️ 1. Telas Necessárias

O sistema é composto por 3 telas principais. Sugerimos o uso de **React + Vite** com a biblioteca de estilos da sua preferência (Tailwind, Material UI, etc.).

### A. Tela Pública (Arrematantes)
* **Objetivo:** Landing page para agendamento de visitas.
* **Componentes principais:**
  * Formulário em cascata (Selects encadeados):
    1. Escolhe o **Leilão** (carrega os ativos).
    2. Escolhe o **Pátio** vinculado àquele leilão.
    3. Escolhe o **Horário** disponível (mostrando a quantidade de vagas).
  * Campos de identificação: Nome, CPF/CNPJ, E-mail e Telefone.

### B. Tela de Login (Admin)
* **Objetivo:** Autenticação dos administradores do sistema.
* **Componentes principais:**
  * Formulário com E-mail e Senha.
  * A ação de submit deve bater na rota de Login, receber o Token JWT e salvá-lo no `localStorage` ou `sessionStorage`.

### C. Painel Admin (Privado)
* **Objetivo:** Gestão completa do sistema.
* **Layout:** Dashboard com menu lateral (Sidebar) e conteúdo dinâmico (Abas/Rotas).
* **Visões necessárias:**
  * **Dashboard/Overview:** Resumo rápido (total de agendamentos, vagas disponíveis, etc).
  * **Agendamentos:** Tabela com todos os agendamentos e botão para "Cancelar/Devolver Vaga".
  * **Leilões (Listagem):** Tabela de leilões com botões de Ação (Editar, Excluir).
  * **Mega Formulário de Leilões:** Uma tela inteligente onde o Admin cadastra os dados do Leilão, adiciona blocos dinâmicos de Pátios e define a janela de data/hora (Ex: Dias 01 a 05, das 08:00 às 17:00). O backend se vira para gerar as frações de horários de 1 em 1 hora.

---

## 🔌 2. Contrato da API (Endpoints)

*Base URL: `http://localhost:8080/api`*

### 🔓 Fluxo Público (Sem Autenticação)

**Listar Leilões Ativos**
* `GET /leiloes/ativos`
* *Retorno:* `[{ id, titulo, descricao, status }]`

**Listar Pátios do Leilão**
* `GET /patios/leilao/{leilaoId}`
* *Retorno:* `[{ id, nome, endereco, leilaoId }]`

**Listar Horários Disponíveis do Pátio**
* `GET /horarios/patio/{patioId}/disponiveis`
* *Retorno:* `[{ id, dataHora, vagasTotais, vagasDisponiveis, patioId, patioNome, leilaoTitulo }]`

**Efetuar Agendamento**
* `POST /agendamentos/publico/agendar`
* *Payload Esperado:*
```json
{
  "horarioId": 1,
  "visitante": {
    "nome": "João Silva",
    "cgc": "12345678909",
    "email": "joao@email.com",
    "telefone": "11999999999"
  }
}
```
* *Retorno esperado:* `201 Created`

---

### 🔐 Fluxo de Autenticação (Em breve no Backend)
*Nota: Você pode mockar esse fluxo temporariamente enquanto o backend finaliza a implementação do Spring Security.*

**Login do Admin**
* `POST /auth/login`
* *Payload Esperado:*
```json
{
  "email": "admin@visitaja.com.br",
  "senha": "123"
}
```
* *Retorno esperado:* `{ "token": "ey..." }`

---

### 🛡️ Fluxo Admin (Requer Cabeçalho: `Authorization: Bearer <token>`)

#### Agendamentos
* **Listar Todos:** `GET /agendamentos`
* **Cancelar:** `PUT /agendamentos/{id}/cancelar` (Retorna `204 No Content`)

#### Leilões (CRUD Completo)
* **Listar Todos (inclusive Inativos):** `GET /leiloes`
* **Buscar Leilão Específico (para tela de edição):** `GET /leiloes/{id}`
* **Criação em Lote (Mega Formulário):** `POST /leiloes/completo`
  * *Payload Esperado:*
```json
{
  "titulo": "Leilão Frota SP",
  "descricao": "Veículos leves",
  "status": "ATIVO",
  "patios": [
    {
      "nome": "Pátio Central",
      "endereco": "Rua X",
      "dataInicio": "2026-11-01",
      "dataFim": "2026-11-02",
      "horaInicio": "08:00:00",
      "horaFim": "12:00:00",
      "vagasPorHorario": 50
    }
  ]
}
```
* **Editar Leilão (Em breve):** `PUT /leiloes/{id}` (Receberá título, descrição, status)
* **Excluir Leilão (Em breve):** `DELETE /leiloes/{id}`

#### Dashboard Metrics (Em breve)
* **Resumo Geral:** `GET /dashboard/resumo`
* *Retorno esperado:*
```json
{
  "totalAgendamentos": 150,
  "leiloesAtivos": 3,
  "vagasDisponiveisTotais": 1200
}
```

---
### 🛠️ Dicas
* Para os endpoints marcados como **"Em breve"**, crie a interface normalmente e use dados *mockados* no frontend. Quando o backend for atualizado, bastará trocar a fonte dos dados.
* Qualquer erro 400 (Bad Request) geralmente indica que o formato do JSON enviado está diferente do mapeamento da API ou alguma validação de campo obrigatório falhou.
