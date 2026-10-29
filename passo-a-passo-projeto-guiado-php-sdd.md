# Guia Mestre do Passo a Passo do Projeto Guiado (GitHub Spec Kit & SDD)
**Orquestrador Central do Ciclo de Vida em PHP 8.3, Laravel 11.x (MVC), MySQL via XAMPP e React 18 SPA**

---

## 1. Mapeamento e Articulação da Arquitetura Documental

No modelo **Spec-Driven Development (SDD)** operado via **GitHub Spec Kit**, o **Passo a Passo do Projeto Guiado** atua como o **documento mestre e orquestrador**. Ele alinha, consome e articula os 5 documentos de especificação especializada da aplicação, garantindo que cada um cumpra rigorosamente o seu papel no ciclo de vida de desenvolvimento:

```text
                               ┌───────────────────────────────────────────────────────────┐
                               │  PASSO A PASSO DO PROJETO GUIADO (Orquestrador Mestre)   │
                               └─────────────────────────────┬─────────────────────────────┘
                                                             │
        ┌─────────────────────────┬──────────────────────────┼──────────────────────────┬─────────────────────────┐
        ▼                         ▼                          ▼                          ▼                         ▼
┌───────────────┐        ┌─────────────────┐        ┌──────────────────┐       ┌──────────────────┐      ┌─────────────────┐
│ Documento de  │        │ Telas do Front- │        │ SDD de Alto      │       │ Tecnologia para  │      │ Tecnologia para │
│ Fornecedores  │        │ End (UX/UI)     │        │ Nível (REST)     │       │ Back-End         │      │ Front-End       │
├───────────────┤        ├─────────────────┤        ├──────────────────┤       ├──────────────────┤      ├─────────────────┤
│ • Regras RN01-│        │ • Rotas SPA     │        │ • Topologia      │       │ • PHP 8.3        │      │ • React 18 SPA  │
│   RN05        │        │ • Módulos UI    │        │ • Contratos REST │       │ • Laravel 11.x   │      │ • Vite + TS 5   │
│ • Requisitos  │        │ • Estados Visu- │        │ • Status HTTP    │       │ • Eloquent ORM   │      │ • Tailwind CSS  │
│   RF01-RF05   │        │   ais & Modais  │        │ • Transações     │       │ • Form Requests  │      │ • Axios Client  │
│ • Modelo 1:N  │        │ • Interação 1:N │        │   ACID           │       │ • MySQL (XAMPP)  │      │ • Context API   │
└───────┬───────┘        └────────┬────────┘        └────────┬─────────┘       └────────┬─────────┘      └────────┬────────┘
        │                         │                          │                          │                         │
        ▼                         ▼                          ▼                          ▼                         ▼
   /speckit.specify         /speckit.specify            /speckit.plan              /speckit.constitution     /speckit.constitution
                            /speckit.plan                                              /speckit.plan             /speckit.plan
                                                                                       /speckit.tasks            /speckit.tasks
```

### Matriz de Papéis dos Documentos Especializados

| Documento Especializado | Função Específica no Projeto | Fase do Spec Kit Onde É Consumido |
| :--- | :--- | :--- |
| **1. Documento de Fornecedores** | **SSOT de Requisitos e Domínio:** Contém a especificação funcional de negócios, definindo as regras `RN01` a `RN05`, os requisitos `RF01` a `RF05`, os `RNF01` a `RNF05` e o esquema relacional de 1:N entre Fornecedor e Produto. | Consumido na fase **/speckit.specify** e validado na fase **/speckit.converge**. |
| **2. SDD de Alto Nível** | **Design Arquitetural e Contratos REST:** Define a topologia Cliente-Servidor (Stateless API), as convenções de verbos HTTP, os padrões de erro, a estratégia de transações ACID e o isolamento de camadas. | Consumido na fase **/speckit.plan**. |
| **3. Tecnologia para Back-End** | **Especificação Técnica de Infraestrutura Backend:** Determina o uso de PHP 8.3+, Laravel 11.x, Eloquent ORM, Form Requests (`app/Http/Requests/`), API Resources (`app/Http/Resources/`) e persistência relacional no MySQL 8.0 via XAMPP (Porta 3306). | Consumido em **/speckit.constitution**, **/speckit.plan** e **/speckit.tasks**. |
| **4. Tecnologia para Front-End** | **Especificação Técnica de Infraestrutura Frontend:** Define a construção da SPA com React 18, Vite, TypeScript 5, Tailwind CSS, Axios com interceptadores e React Router DOM v6 totalmente desacoplados da API. | Consumido em **/speckit.constitution**, **/speckit.plan** e **/speckit.tasks**. |
| **5. Especificação de Telas** | **Design de UX/UI e Fluxos Visuais:** Especifica a engenharia dos 4 módulos visuais (`/login`, `/fornecedores`, `/produtos`, `/fornecedores/:id/produtos`), os estados visuais de carregamento e o mapeamento de respostas HTTP para alertas em tela. | Consumido em **/speckit.specify**, **/speckit.plan** e **/speckit.tasks**. |

---

## 2. Passo 1: `/speckit.constitution` (Leis Arquiteturais do Repositório)

O comando `/speckit.constitution` consolida as diretrizes inegociáveis que orientam os agentes de IA e desenvolvedores. Ele extrai os padrões definidos nos documentos de **Tecnologia para Back-End**, **Tecnologia para Front-End** e **SDD de Alto Nível**:

```markdown
# Project Constitution: Sistema de Gestão de Fornecedores e Produtos

## 1. Princípios Arquiteturais & Filosofia
- **Separation of Concerns (SoC):** Desacoplamento estrito entre Frontend (React 18 SPA) e Backend (Laravel 11.x RESTful Stateless API).
- **Domain & Data Integrity First:** Toda regra de negócio (RN01 a RN05) deve ser validada e garantida no nível de domínio e no banco de dados relacional MySQL.
- **Spec-Driven Consistency:** Nenhuma implementação de código ocorre sem especificação prévia aprovada e testada conceitualmente.

## 2. Padrões Técnicos e Stack Oficial
- **Backend:** PHP 8.3+, Laravel 11.x (MVC), Eloquent ORM, Form Requests (`app/Http/Requests/`), API Resources (`app/Http/Resources/`).
- **Banco de Dados Relacional:** MySQL 8.0 / MariaDB executado através do XAMPP Control Panel (Porta 3306) com engine InnoDB, transações ACID e suporte estrito a Foreign Keys.
- **Modelagem Relacional 1:N:** Um Fornecedor possui N Produtos vinculados pela chave estrangeira `fornecedor_id` na tabela `produtos`, com trava de integridade relacional `ON DELETE RESTRICT` (`RN01`, `RN04`).
- **Frontend SPA:** React 18+ com Vite, TypeScript 5.x, Tailwind CSS e Axios, consumindo a API RESTful do Laravel de forma totalmente desacoplada.
- **API Standards:** Padrão RESTful estrito com respostas semânticas e códigos HTTP canônicos (200, 201, 204, 401, 403, 404, 409, 422).

## 3. Diretrizes de Qualidade, Protocolo e Testes
- **TDD Inegociável:** Testes de integração antecedem o código de produção (`php artisan test` com a trait `RefreshDatabase` e `npm run build` para TypeScript).
- **Injeção de Dependência Idiomática:** Constructor Property Promotion nativa do PHP 8.3 e tipagem estrita (`declare(strict_types=1);`).
- **Instalação Compulsória de Dependências:** Instalação obrigatória via `composer install` no backend e `npm install` no frontend antes de qualquer teste.
```

---

## 3. Passo 2: `/speckit.specify` (Especificação Funcional e Regras de Negócio)

O comando `/speckit.specify` compila o escopo funcional e comportamental do produto consumindo as definições do **Documento de Fornecedores** e da **Especificação de Telas**:

```markdown
# Specification: Fornecedores e Produtos

## 1. Regras de Negócio (RN01 a RN05)
- [RN01] Vínculo 1:N Obrigatório: Todo produto cadastrado deve pertencer obrigatoriamente a um único fornecedor ativo existente no sistema (`fornecedor_id NOT NULL`).
- [RN02] Atributos de Aquisição Diretos: O preço de custo (`preco_custo` em R$) e o prazo de entrega (`prazo_entrega_dias` em dias) são armazenados diretamente na tabela `produtos`.
- [RN03] Unicidade de Identificadores: O CNPJ do fornecedor e o código SKU do produto devem ser únicos em toda a base de dados MySQL.
- [RN04] Proteção contra Exclusão de Fornecedor: Não é permitida a exclusão de um fornecedor que possua produtos associados (`ON DELETE RESTRICT`). A tentativa deve ser bloqueada retornando HTTP 409 Conflict.
- [RN05] Autenticação Obrigatória: Todas as operações de inclusão, alteração e exclusão exigem que o operador esteja previamente autenticado na sessão (`POST /api/auth/login`).

## 2. Requisitos Funcionais (RF01 a RF05)
- [RF01] Gestão de Fornecedores: Cadastro, listagem, edição e exclusão de fornecedores com razão social, CNPJ único, e-mail e telefone.
- [RF02] Gestão de Produtos: Cadastro de produtos com nome, SKU único, preço de custo, prazo de entrega e seleção obrigatória de fornecedor via campo dropdown.
- [RF03] Consulta Master-Detail: Tela de consulta para exibir os dados de um fornecedor acompanhado da lista completa e aninhada de seus produtos.
- [RF04] Gestão de Registros: Edição de dados e exclusão respeitando o bloqueio de integridade referencial (`RN04`).
- [RF05] Autenticação de Operador: Tela inicial de login (`/login`) validando e-mail e senha com gerenciamento de sessão/token Bearer.

## 3. Requisitos Não Funcionais (RNF01 a RNF05)
- [RNF01] Usabilidade: Interface SPA reativa, responsiva e intuitiva em React 18 + Tailwind CSS.
- [RNF02] Performance: Respostas da API e transições em menos de 1 segundo para operações locais.
- [RNF03] Confiabilidade e Integridade: Operações de banco executadas sob transações ACID no MySQL/InnoDB via XAMPP.
- [RNF04] Persistência Relacional: SGBD MySQL 8.0 no XAMPP (Porta 3306) com administração via phpMyAdmin.
- [RNF05] Padrão de API REST: Comunicação padronizada em JSON com códigos de status HTTP semânticos (200, 201, 204, 401, 403, 404, 409, 422).
```

---

## 4. Passo 3: `/speckit.plan` (Plano de Arquitetura, DDL e Contratos)

O comando `/speckit.plan` gera o plano de execução técnica consumindo o **SDD de Alto Nível**, a **Tecnologia BackEnd** e a **Tecnologia FrontEnd**:

### 1. Modelagem Relacional no MySQL (XAMPP Control Panel - Porta 3306)

* **Tabela `usuarios` (Autenticação):**
  * `id` (BIGINT AUTO_INCREMENT PK), `nome` (VARCHAR 150), `email` (VARCHAR 255 UNIQUE), `senha` (VARCHAR 255 - Bcrypt), `created_at`/`updated_at` (TIMESTAMP).
* **Tabela `fornecedores`:**
  * `id` (BIGINT AUTO_INCREMENT PK), `razao_social` (VARCHAR 255), `cnpj` (VARCHAR 18 UNIQUE), `email` (VARCHAR 255 UNIQUE), `telefone` (VARCHAR 20), `status_ativo` (BOOLEAN DEFAULT TRUE), `created_at`/`updated_at` (TIMESTAMP).
* **Tabela `produtos`:**
  * `id` (BIGINT AUTO_INCREMENT PK), `nome` (VARCHAR 255), `sku` (VARCHAR 50 UNIQUE), `descricao` (TEXT NULLABLE), `preco_custo` (DECIMAL 10,2), `prazo_entrega_dias` (INTEGER), `fornecedor_id` (BIGINT FK -> `fornecedores.id` com `ON DELETE RESTRICT`), `created_at`/`updated_at` (TIMESTAMP).

### 2. Configuração de Ambiente (`.env` Laravel ➔ XAMPP)
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gestao_fornecedores
DB_USERNAME=root
DB_PASSWORD=
```

### 3. Contratos dos Endpoints da API RESTful no Laravel

| Método HTTP | Endpoint REST | Request Payload / Parâmetros | Status HTTP Retornado | Regra Atendida |
| :---: | :--- | :--- | :---: | :---: |
| **`POST`** | `/api/auth/login` | `{ "email": "...", "senha": "..." }` | **200 OK** / **401 Unauthorized** | `RN05`, `RF05` |
| **`GET`** | `/api/fornecedores` | Query params: `?termo=...` | **200 OK** | `RF01` |
| **`POST`** | `/api/fornecedores` | `{ "razao_social": "...", "cnpj": "...", "email": "..." }` | **201 Created** / **422 Unprocessable** | `RN03`, `RF01` |
| **`DELETE`** | `/api/fornecedores/{id}` | URL Param: `id` | **204 No Content** / **409 Conflict** | `RN04`, `RF04` |
| **`GET`** | `/api/produtos` | Query params: `?termo=...` | **200 OK** | `RF02` |
| **`POST`** | `/api/produtos` | `{ "nome": "...", "sku": "...", "preco_custo": 150.00, "prazo_entrega_dias": 5, "fornecedor_id": 1 }` | **201 Created** / **422 Unprocessable** | `RN01`, `RN02`, `RN03` |
| **`GET`** | `/api/fornecedores/{id}/produtos` | URL Param: `id` | **200 OK** / **404 Not Found** | `RF03` |

---

## 5. Passos 4 e 5: `/speckit.tasks` e `/speckit.taskstoissues` (Decomposição Ágil)

Decomposição em 10 tarefas atômicas e 4 Issues agrupadas, com marcação `[P]` para tarefas elegíveis para desenvolvimento paralelo entre Backend e Frontend:

### Tarefas de Implementação (`tasks.md`)
- [ ] **Task 1 [P]:** Setup do projeto Laravel 11.x, criação do banco `gestao_fornecedores` no MySQL via XAMPP (Porta 3306) e ajuste do arquivo `.env`.
- [ ] **Task 2 [P]:** Setup do projeto React 18 SPA com Vite, TypeScript 5, Tailwind CSS e cliente Axios configurado em `src/services/api.ts`.
- [ ] **Task 3:** Criação das Laravel Migrations (`usuarios`, `fornecedores`, `produtos`) no MySQL com `restrictOnDelete()` e `unique()`.
- [ ] **Task 4:** Implementação dos Models Eloquent (`Usuario`, `Fornecedor`, `Produto`) com relacionamentos `hasMany` e `belongsTo`.
- [ ] **Task 5:** Implementação dos Form Requests (`StoreFornecedorRequest`, `StoreProdutoRequest`) e Exception Handler para capturar erro 409 do MySQL.
- [ ] **Task 6:** Implementação dos API Controllers (`AuthController`, `FornecedorController`, `ProdutoController`) e Eloquent API Resources (`JsonResource`).
- [ ] **Task 7 [P]:** Construção da Tela 1 de Login (`/login`) no React com Context API (`AuthContext`) e guarda de rotas.
- [ ] **Task 8 [P]:** Construção da Tela 2 de Fornecedores (`/fornecedores`) no React com tabela paginada, modal de cadastro e tratamento de alerta HTTP 409.
- [ ] **Task 9 [P]:** Construção da Tela 3 de Produtos (`/produtos`) no React com formulário e seletor dropdown dinâmico de fornecedores (`RN01`).
- [ ] **Task 10 [P]:** Construção da Tela 4 Master-Detail (`/fornecedores/:id/produtos`) e execução da suíte de testes (`php artisan test` e `npm run build`).

### Mapeamento de Issues do GitHub (`taskstoissues`)
* **Issue #1 [Infra]:** Setup de Ambiente Laravel, MySQL (XAMPP) e React SPA com Vite/TypeScript.
* **Issue #2 [Data]:** Migrations DDL no MySQL e Models Eloquent 1:N com integridade referencial.
* **Issue #3 [Backend]:** Endpoints RESTful, Form Requests, API Resources e tratamento de status HTTP (200, 201, 204, 401, 409, 422).
* **Issue #4 [Frontend]:** Componentização das 4 Telas em React SPA, rotas protegidas e integração Axios.

---

## 6. Passo 6: `/speckit.implement` (Execução de Código & Integridade)

O agente de IA gera os artefatos de código respeitando estritamente os contratos técnicos.

### 6.1 Backend: Migration da Tabela `produtos` (`database/migrations/`)
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('produtos', function (Blueprint $table) {
            $table->id();
            $table->string('nome');
            $table->string('sku', 50)->unique();
            $table->text('descricao')->nullable();
            $table->decimal('preco_custo', 10, 2);
            $table->integer('prazo_entrega_dias');
            $table->foreignId('fornecedor_id')
                  ->constrained('fornecedores')
                  ->restrictOnDelete(); // Restrição RN04 no MySQL/XAMPP
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('produtos');
    }
};
```

### 6.2 Backend: Model Eloquent `Produto.php` (`app/Models/`)
```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Produto extends Model
{
    protected $table = 'produtos';

    protected $fillable = [
        'nome',
        'sku',
        'descricao',
        'preco_custo',
        'prazo_entrega_dias',
        'fornecedor_id',
    ];

    protected $casts = [
        'preco_custo' => 'decimal:2',
        'prazo_entrega_dias' => 'integer',
    ];

    public function fornecedor(): BelongsTo
    {
        return $this->belongsTo(Fornecedor::class, 'fornecedor_id');
    }
}
```

### 6.3 Frontend: Cliente Axios e Interceptadores (`src/services/api.ts`)
```typescript
import axios from 'axios';

export const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('@app:token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('@app:token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

---

## 7. Passo 7: `/speckit.converge` (Convergência e Rastreabilidade)

A fase de convergência audita se a implementação em PHP/Laravel/MySQL (XAMPP) e React SPA atende 100% à especificação original. A inspeção direta do banco é realizada via **phpMyAdmin** (`http://localhost/phpmyadmin`) na porta 3306.

### Matriz Mestre de Rastreabilidade Bidirecional

| Requisito / Regra SDD | Documento de Origem | Componente Backend (Laravel 11) | Tabela MySQL (XAMPP) | Componente Frontend (React SPA) |
| :--- | :--- | :--- | :--- | :--- |
| **RF01 (CRUD Fornecedor)** | Documento de Fornecedores | `FornecedorController.php` | `fornecedores` | `FornecedoresPage.tsx` |
| **RF02 (CRUD Produto 1:N)** | Documento de Fornecedores | `ProdutoController.php` | `produtos` | `ProdutosPage.tsx` |
| **RF03 (Master-Detail)** | Documento de Fornecedores | `FornecedorController@produtos` | `fornecedores JOIN produtos` | `MasterDetailFornecedorPage.tsx` |
| **RN01 (FK Obrigatória)** | Documento de Fornecedores | `StoreProdutoRequest` (`exists`) | `fornecedor_id` FK `NOT NULL` | Seletor Dropdown de Fornecedores |
| **RN02 (Atributos Diretos)** | Documento de Fornecedores | Model `Produto` (`preco_custo`) | `produtos.preco_custo` | Inputs numéricos com máscara |
| **RN03 (Unicidade CNPJ/SKU)** | Documento de Fornecedores | Form Requests (`unique`) | Índices `UNIQUE` no MySQL | Alertas sob inputs (HTTP 422) |
| **RN04 (Bloqueio Exclusão)** | Documento de Fornecedores | Migration (`restrictOnDelete`) | Foreign Key Constraint | Modal de Alerta de Conflito (HTTP 409) |
| **RN05 (Autenticação)** | Documento de Fornecedores | `AuthController.php` (Sanctum) | `usuarios` | `LoginPage.tsx` & `AuthContext` |
| **Design RESTful** | SDD de Alto Nível | `routes/api.php` | SGBD MySQL (Porta 3306) | Client Axios (`src/services/api.ts`) |
| **Infraestrutura Backend** | Tecnologia para Back-End | Laravel 11.x (MVC) + Eloquent | Engine InnoDB (Porta 3306) | Consumo via API Resources |
| **Infraestrutura Frontend** | Tecnologia para Front-End | API RESTful Stateless | Servidor MySQL Local | React 18 + Vite + TypeScript 5 |
| **Módulos Visuais & UX** | Especificação de Telas | Eloquent API Resources | Banco `gestao_fornecedores` | 4 Telas SPA (`/login`, `/fornecedores`...) |
