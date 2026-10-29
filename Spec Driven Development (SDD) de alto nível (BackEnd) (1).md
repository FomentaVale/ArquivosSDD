# **Spec-Driven Development (SDD) — BackEnd**

**Especificação Arquitetural, Contratos de API RESTful e Persistência Relacional com Laravel (MVC), Eloquent ORM e MySQL via XAMPP**  
**Metodologia:** Spec-Driven Development (SDD) & GitHub Spec Kit  
**Arquitetura Backend:** RESTful Stateless API (PHP 8.3 / Laravel 11.x / 12.x MVC)  
**Ambiente de Banco de Dados:** MySQL 8.0 / MariaDB via XAMPP Control Panel (Porta 3306 / phpMyAdmin)  
**ORM & Persistência:** Eloquent ORM \+ Migrations Versionadas  
**Governança de IA:** Laravel Boost (.ai/guidelines/\*) \+ Constituição do Repositório (memory/constitution.md)

## 1\. Visão Geral e Condições de Conclusão (BackEnd)

### Diretriz de Engenharia SDD

Este **Software Design Document (SDD) de Alto Nível — BackEnd** estabelece a especificação arquitetural da API RESTful responsável pelo processamento das regras de negócio, validação de segurança e persistência do sistema de Gestão de Fornecedores e Produtos. Sob a disciplina do Spec-Driven Development (SDD), esta documentação atua como a **única fonte primária da verdade** (Single Source of Truth). Nenhuma rota, regra de negócio ou migration é implementada no Laravel sem um contrato prévio formalizado e aprovado.

### Princípios da Camada de Serviços e Infraestrutura

* **Ambiente de Banco de Dados XAMPP:** A camada de dados relacional é padronizada no **MySQL (MariaDB)** inicializado localmente através do **XAMPP Control Panel** (serviço MySQL rodando na porta 3306 com administração via phpMyAdmin).  
* **Arquitetura Stateless:** O backend opera como uma API RESTful *stateless*, comunicando-se com o frontend por meio do intercâmbio exclusivo de payloads JSON via protocolo HTTP.  
* **Desacoplamento por Camadas (SoC):** As rotas redirecionam para **Form Requests** de validação, que alimentam os **API Controllers**, que por sua vez delegam as regras ao **Eloquent ORM** e formatam as respostas com **API Resources**.  
* **Integridade Garantida no SGBD:** A consistência dos dados (unicidade de CNPJ/SKU e bloqueio de exclusão orfanada) é forçada no nível do banco de dados relacional MySQL com restrições UNIQUE e ON DELETE RESTRICT.

## 2\. Arquitetura Geral do BackEnd Laravel (MVC)

A camada de backend adota a arquitetura de 4 subcamadas idiomática do Laravel conectada ao banco MySQL do XAMPP:

```
HTTP Request (Payload JSON)
 │
 ▼
routes/api.php (Mapeamento de Rotas REST)
 │
 ▼
app/Http/Requests/ (Form Request: Validação & Sanitização)
 │
 ▼
app/Http/Controllers/Api/ (API Controller Fino)
 │
 ▼
app/Models/ (Eloquent Model: Regras de Domínio & Relacionamentos)
 │
 ▼
MySQL Database (Serviço ativo via XAMPP Control Panel - Porta 3306)
 │
 ▼
app/Http/Resources/ (Eloquent API Resource: Contrato JSON)
 │
 ▼
HTTP Response (Status 200 / 201 / 204 / 409 / 422)
```

### Detalhamento das Responsabilidades

* **Roteamento (routes/api.php):** Registra as URIs da API RESTful seguindo as convenções de verbos HTTP (GET, POST, PUT, DELETE).  
* **Validação de Entrada (app/Http/Requests/):** Classes dedicadas que interceptam a requisição antes do Controller, aplicando as regras no método rules(). Em caso de falha, disparam automaticamente o código HTTP 422 Unprocessable Entity.  
* **Controllers (app/Http/Controllers/Api/):** Mantidos como "controllers finos" (thin controllers), responsáveis por orquestrar o fluxo HTTP e invocar as operações Eloquent sob transações quando necessário.  
* **Persistência (app/Models/ & database/migrations/):** Models Eloquent gerenciando os relacionamentos (1:N e N:N) e Migrations garantindo a criação das tabelas no MySQL do XAMPP.  
* **Transformação de Saída (app/Http/Resources/):** Transforma os objetos Eloquent em estruturas JSON limpas, impedindo o vazamento de chaves internas ou dados sensíveis.

## 3\. Stack Tecnológica de BackEnd e Configuração XAMPP

| Camada / Componente | Tecnologia Selecionada | Justificativa Arquitetural SDD & Configuração |
| :---- | :---- | :---- |
| **Linguagem Base** | PHP 8.3+ | Tipagem estrita (declare(strict\_types=1);), Constructor Property Promotion e extensão zip ativada no php.ini do XAMPP. |
| **Framework Web** | Laravel 11.x / 12.x | Estruturação MVC expressiva, gerenciamento de dependências via Container de Serviços. |
| **SGBD Relacional** | MySQL / MariaDB (XAMPP) | Serviço inicializado via XAMPP Control Panel (Porta 3306). Banco gestao\_fornecedores gerido via phpMyAdmin. |
| **Gerenciador de Pacotes** | Composer (composer.json) | Gestão de dependências e autoloader PSR-4. |
| **ORM & Mapeamento** | Eloquent ORM | Suporte nativo a Migrations versionadas, Eager Loading (with()) e gerenciamento de transações. |
| **Testes Automatizados** | Pest PHP / PHPUnit | Execução de testes de integração com a trait RefreshDatabase conectada ao MySQL/XAMPP. |
| **Assistência & Governança IA** | Laravel Boost \+ GitHub Spec Kit | Contexto vetorizado de documentação, execução local de testes (php artisan test) e validação constitucional (memory/constitution.md). |

### Configuração de Conexão no Arquivo .env (Laravel \-\> XAMPP)

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gestao_fornecedores
DB_USERNAME=root
DB_PASSWORD=
```

## 4\. Especificação do Modelo de Dados Relacional (1:N) & Migrations DDL (MySQL/XAMPP)

O esquema relacional é estruturado no modelo 1:N (Um Fornecedor possui N Produtos), gravado no banco MySQL do XAMPP onde a chave estrangeira fornecedor\_id é armazenada na tabela produtos:

### 4.1 Tabela usuarios (Autenticação de Operadores)

* **Migration:** create\_usuarios\_table  
* **Campos:**  
  * id: bigIncrements('id') — Chave Primária (PK)  
  * nome: string('nome', 150\) — Obrigatório  
  * email: string('email')-\>unique() — Único e Obrigatório  
  * senha: string('senha') — Hash Bcrypt Obrigatório  
  * created\_at / updated\_at: timestamps()

### 4.2 Tabela fornecedores

* **Migration:** create\_fornecedores\_table  
* **Campos:**  
  * id: bigIncrements('id') — Chave Primária (PK)  
  * razao\_social: string('razao\_social') — Obrigatório  
  * cnpj: string('cnpj', 18)-\>unique() — CNPJ Único e Obrigatório (RN03)  
  * email: string('email')-\>unique() — E-mail Comercial Único  
  * telefone: string('telefone', 20)-\>nullable()  
  * status\_ativo: boolean('status\_ativo')-\>default(true)  
  * created\_at / updated\_at: timestamps()

### 4.3 Tabela produtos

* **Migration:** create\_produtos\_table  
* **Campos:**  
  * id: bigIncrements('id') — Chave Primária (PK)  
  * nome: string('nome') — Obrigatório  
  * sku: string('sku', 50)-\>unique() — SKU Único e Obrigatório (RN03)  
  * descricao: text('descricao')-\>nullable()  
  * preco\_custo: decimal('preco\_custo', 10, 2\) — Atributo Direto (RN02)  
  * prazo\_entrega\_dias: integer('prazo\_entrega\_dias') — Atributo Direto (RN02)  
  * fornecedor\_id: foreignId('fornecedor\_id')-\>constrained('fornecedores')-\>restrictOnDelete() — FK 1:N Obrigatória (RN01, RN04)

created\_at / updated\_at: timestamps()

### Fornecedores

## 5\. Especificação de Contratos e Interfaces (API RESTful)

As rotas da API expostas em routes/api.php obedecem rigorosamente às convenções RESTful:

| Endpoint REST | Verbo HTTP | Request Payload / Params | Resposta JSON & Código HTTP | Regra Vinculada |
| :---- | :---- | :---- | :---- | :---- |
| /api/auth/login | POST | { "email": "...", "senha": "..." } | **200 OK:** { "token": "...", "usuario": {...} }**401 Unauthorized:** Credenciais inválidas. | RN05 / RF05 |
| /api/fornecedores | GET | Query: ?termo=... | **200 OK:** Lista paginada de fornecedores com contagem de produtos. | RF01 |
| /api/fornecedores | POST | { "razao\_social": "...", "cnpj": "...", "email": "...", "telefone": "..." } | **201 Created:** Fornecedor criado.**422 Unprocessable:** Erro de validação de CNPJ ou e-mail. | RN03 / RF01 |
| /api/fornecedores/{id} | DELETE | URL Param: id | **204 No Content:** Sucesso.**409 Conflict:** Bloqueado por existir produtos vinculados no MySQL. | RN04 / RF04 |
| /api/produtos | GET | Query: ?termo=... | **200 OK:** Lista de produtos com Eager Loading (with('fornecedor')). | RF02 |
| /api/produtos | POST | { "nome": "...", "sku": "...", "preco\_custo": 150.00, "prazo\_entrega\_dias": 5, "fornecedor\_id": 1 } | **201 Created:** Produto criado.**422 Unprocessable:** SKU duplicado ou fornecedor\_id inexistente. | RN01, RN02, RN03 |
| /api/fornecedores/{id}/produtos | GET | URL Param: id | **200 OK:** Dados do fornecedor com array aninhado de produtos (Master-Detail).**404 Not Found:** Fornecedor não existe. | RF03 |

## 6\. Tratamento de Erros, Segurança e Transações ACID no MySQL/XAMPP

1. **Validação de Payloads em Form Requests:** As regras são centralizadas em StoreFornecedorRequest e StoreProdutoRequest. Validações como unique:fornecedores,cnpj e exists:fornecedores,id garantem que dados inválidos sejam rejeitados antes de chegarem ao MySQL do XAMPP.  
2. **Restrição de Exclusão Referencial no MySQL (ON DELETE RESTRICT):** A tentativa de deletar um fornecedor que possua produtos cadastrados dispara uma exceção Illuminate\\Database\\QueryException pelo MySQL do XAMPP. O Handler de Exceções (bootstrap/app.php) intercepta essa violação de chave estrangeira e retorna a resposta JSON semántica:  
   {  
     "message": "Não é possível excluir o fornecedor pois existem produtos vinculados a ele no banco de dados.",  
     "error\_code": "RESOURCE\_HAS\_DEPENDENCIES"  
   }  
   (Retornando o código HTTP 409 Conflict).  
3. **Transações ACID no InnoDB/MySQL (DB::transaction):** Operações que envolvem mutações em múltiplos registros são encapsuladas em blocos transacionais para garantir atomicidade no engine InnoDB do MySQL:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($payload) {
    // Executa mutações no MySQL com rollback automático em caso de falha
});
```

4. **Segurança de Senhas:** Todas as senhas de operadores são armazenadas no MySQL com hash Bcrypt via Hash::make(\$senha).

## 7\. Matriz de Conformidade e Rastreabilidade SDD (BackEnd & XAMPP)

A matriz abaixo comprova a rastreabilidade total entre as regras de negócio e os artefatos no Laravel com MySQL/XAMPP:

| Regra / Requisito | Regra de Negócio (RN) | Artefato Backend Laravel & MySQL (XAMPP) |
| :---- | :---- | :---- |
| **Vínculo 1:N Obrigatório** | RN01 | Model Produto com belongsTo(Fornecedor::class, 'fornecedor\_id') e FK NOT NULL no MySQL. |
| **Atributos de Aquisição** | RN02 | Colunas preco\_custo e prazo\_entrega\_dias na Migration produtos e no \$fillable da Model. |
| **Unicidade CNPJ / SKU** | RN03 | Índices UNIQUE no MySQL via Migration e regras de validação Form Request. |
| **Integridade na Exclusão** | RN04 | Constraint restrictOnDelete() no MySQL e captura de exceção no Handler retornando HTTP 409\. |
| **Autenticação de Operador** | RN05 | Middleware de autenticação (Sanctum ou Session) consultando a tabela usuarios no MySQL/XAMPP. |

Pesquisado e desenvolvido por Gabriel Ferreira da Silva 