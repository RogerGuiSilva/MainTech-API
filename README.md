# MainTech

Sistema de gestao de manutencao industrial para acompanhar maquinas, equipamentos, falhas operacionais e indicadores em painel.

O projeto e dividido em duas partes:

- `Back`: API REST em Flask com banco SQLite.
- `front`: interface React com Vite, React Router e graficos com Recharts.

## Funcionalidades

- Cadastro, listagem, busca e exclusao de maquinas.
- Cadastro, listagem, busca e exclusao de equipamentos.
- Registro de falhas vinculadas a uma maquina ou a um equipamento.
- Edicao de falhas com descricao, tipo, gravidade, data e status.
- Historico de falhas resolvidas ou canceladas.
- Painel de controle com totais, status de maquinas/equipamentos e tendencia mensal de falhas.
- Atualizacao automatica de status para `PARADA` quando existe falha ativa em `ANALISE` ou `MANUTENCAO`.

## Tecnologias

### Backend

- Python
- Flask
- Flask-CORS
- SQLite

### Frontend

- React
- Vite
- React Router DOM
- Recharts
- ESLint

## Estrutura

```text
MainTech-API/
|-- Back/
|   |-- main.py
|   |-- requirements.txt
|   `-- app/
|       |-- routes.py
|       |-- maquinas.py
|       |-- equipamentos.py
|       |-- falhas.py
|       |-- seed_maquinas.py
|       |-- seed_equipamentos.py
|       `-- dataBase/
|           |-- db.py
|           `-- app.db
`-- front/
    |-- package.json
    |-- vite.config.js
    |-- src/
    |   |-- App.jsx
    |   |-- main.jsx
    |   `-- pages/
    |-- css/
    |-- public/
    `-- prints/
```

## Requisitos

- Python 3.11 ou superior
- Node.js 20 ou superior
- npm

## Como Rodar

### 1. Backend

Entre na pasta do backend:

```bash
cd Back
```

Crie e ative um ambiente virtual:

```bash
python -m venv .venv
```

No Windows:

```bash
.\.venv\Scripts\activate
```

No Linux/macOS:

```bash
source .venv/bin/activate
```

Instale as dependencias:

```bash
pip install -r requirements.txt
```

Inicialize o banco de dados:

```bash
python -m app.dataBase.db
```

Opcionalmente, carregue dados iniciais:

```bash
python -m app.seed_maquinas
python -m app.seed_equipamentos
```

Inicie a API:

```bash
python main.py
```

A API ficara disponivel em:

```text
http://localhost:5000
```

### 2. Frontend

Em outro terminal, entre na pasta do frontend:

```bash
cd front
```

Instale as dependencias:

```bash
npm install
```

Inicie o servidor de desenvolvimento:

```bash
npm run dev
```

Abra a URL exibida pelo Vite, geralmente:

```text
http://localhost:5173
```

## Rotas da Interface

| Rota | Descricao |
| --- | --- |
| `/` | Pagina inicial |
| `/controle` | Painel de controle e indicadores |
| `/maquinas` | Lista de maquinas |
| `/maquinas/nova` | Cadastro de maquina |
| `/equipamentos` | Lista de equipamentos |
| `/equipamentos/novo` | Cadastro de equipamento |
| `/falhas` | Lista de falhas ativas |
| `/falhas/nova` | Cadastro de falha |
| `/falhas/editar/:id` | Edicao de falha |
| `/falhas/historico` | Historico de falhas resolvidas/canceladas |

## Endpoints da API

### Maquinas

| Metodo | Endpoint | Descricao |
| --- | --- | --- |
| `GET` | `/maquinas` | Lista todas as maquinas |
| `POST` | `/maquinas` | Cadastra uma maquina |
| `DELETE` | `/maquinas/<id>` | Exclui uma maquina |

Exemplo de cadastro:

```json
{
  "nome": "Injetora Plastica 120T",
  "setor": "Injecao",
  "tipo": "Maquina",
  "status": "DISPONIVEL"
}
```

### Equipamentos

| Metodo | Endpoint | Descricao |
| --- | --- | --- |
| `GET` | `/equipamentos` | Lista todos os equipamentos |
| `POST` | `/equipamentos` | Cadastra um equipamento |
| `DELETE` | `/equipamentos/<id>` | Exclui um equipamento |

Exemplo de cadastro:

```json
{
  "nome": "Multimetro Digital",
  "setor": "Eletrica",
  "tipo": "Instrumento de Medicao",
  "status": "DISPONIVEL"
}
```

### Falhas

| Metodo | Endpoint | Descricao |
| --- | --- | --- |
| `GET` | `/falhas` | Lista falhas ativas |
| `GET` | `/falhas/historico` | Lista falhas resolvidas ou canceladas |
| `GET` | `/falhas/<id>` | Busca uma falha pelo ID |
| `POST` | `/falhas` | Cadastra uma falha |
| `PUT` | `/falhas/<id>` | Atualiza dados de uma falha |
| `PATCH` | `/falhas/<id>/status` | Atualiza apenas o status da falha |
| `DELETE` | `/falhas/<id>` | Remove uma falha |

Exemplo de cadastro vinculado a uma maquina:

```json
{
  "descricao": "Motor com aquecimento acima do normal",
  "tipo": "Mecanica",
  "gravidade": "Alta",
  "data_ocorrencia": "2026-05-31",
  "status": "ANALISE",
  "maquina_id": 1,
  "equipamento_id": null
}
```

Exemplo de cadastro vinculado a um equipamento:

```json
{
  "descricao": "Equipamento sem leitura no display",
  "tipo": "Eletrica",
  "gravidade": "Media",
  "data_ocorrencia": "2026-05-31",
  "status": "ANALISE",
  "maquina_id": null,
  "equipamento_id": 1
}
```

## Regras de Status

### Falhas

Status aceitos:

- `ANALISE`
- `MANUTENCAO`
- `RESOLVIDA`
- `CANCELADA`

Transicoes controladas pela rota `PATCH /falhas/<id>/status`:

- `ANALISE` pode ir para `MANUTENCAO`, `RESOLVIDA` ou `CANCELADA`.
- `MANUTENCAO` pode ir para `RESOLVIDA` ou `CANCELADA`.
- `RESOLVIDA` e `CANCELADA` sao estados finais.

### Maquinas

Status aceitos:

- `DISPONIVEL`
- `EM_USO`
- `PARADA`

Quando uma maquina possui falha ativa em `ANALISE` ou `MANUTENCAO`, o status dela e atualizado para `PARADA`. Quando nao existem falhas ativas, o status volta para `DISPONIVEL`.

### Equipamentos

Os equipamentos tambem ficam `PARADA` quando possuem falha ativa em `ANALISE` ou `MANUTENCAO`. Sem falhas ativas, voltam para `DISPONIVEL`.

## Banco de Dados

O banco usado pelo projeto e SQLite. As tabelas principais sao:

- `maquinas`
- `equipamentos`
- `falhas`

O arquivo do banco fica em:

```text
Back/app/dataBase/app.db
```

O script `Back/app/dataBase/db.py` cria as tabelas quando necessario.

## Observacoes

- O frontend chama a API diretamente em `http://localhost:5000`.
- Para usar outra porta ou outro host no backend, ajuste as chamadas `fetch` dentro de `front/src/pages`.
- Os scripts de seed ignoram tabelas que ja possuem dados.
- Nao ha testes automatizados configurados neste repositorio no momento.

## Scripts Uteis

No frontend:

```bash
npm run dev
npm run build
npm run lint
npm run preview
```

No backend:

```bash
python main.py
python -m app.dataBase.db
python -m app.seed_maquinas
python -m app.seed_equipamentos
```
