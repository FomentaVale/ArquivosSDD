  
**Fornecedores de Produtos**

Este documento apresenta a especificação e modelagem de dados simplificada baseada no relacionamento de **Um-para-Muitos (1:N)**. Um fornecedor pode ter múltiplos produtos vinculados, mas cada produto pertence obrigatoriamente a um único fornecedor responsável.

Abaixo, estruturei um **Documento de Requisitos** direto e focado, dividido entre Regras de Negócio, Requisitos Funcionais e Requisitos Não Funcionais.

## 1\. Regras de Negócio (RN)

As regras de negócio definem as premissas e restrições lógicas do sistema antes mesmo de pensarmos nas telas.

| ID | Regra | Descrição |
| :---- | :---- | :---- |
| **RN01** | Vínculo Obrigatório 1:N | Todo produto cadastrado deve estar obrigatoriamente vinculado a um fornecedor ativo existente no sistema. |
| **RN02** | Atributos de Aquisição | O preço de custo (R\$) e o prazo de entrega (em dias) são atributos diretos e obrigatórios do produto no momento do seu cadastro. |
| **RN03** | Unicidade de Dados | O CNPJ do fornecedor e o SKU do produto devem ser únicos e exclusivos em toda a base de dados. |
| **RN04** | Integridade Referencial na Exclusão | Não é permitido excluir um fornecedor que possua produtos vinculados no sistema. O operador deve desvincular/excluir os produtos antes de remover o fornecedor. |
| **RN05** | Controle de Acesso | Nenhuma operação de inclusão, alteração ou exclusão de dados pode ser realizada sem que o operador esteja autenticado no sistema. |

## 2\. Requisitos Funcionais (RF)

Os requisitos funcionais descrevem **o que** o sistema deve fazer (as ações que o usuário poderá executar).

| ID | Nome | Descrição | Prioridade |
| :---- | :---- | :---- | :---- |
| **RF01** | Cadastrar e Listar Fornecedores | O sistema deve permitir o cadastro de fornecedores com validação de CNPJ e e-mail, bem como listar todos os fornecedores ativos com a contagem de produtos associados. | Alta |
| **RF02** | Cadastrar e Listar Produtos | O sistema deve permitir o cadastro de produtos exigindo Nome, SKU, Descrição, Preço de Custo, Prazo de Entrega e a seleção obrigatória do Fornecedor por meio de um campo seletor (dropdown). A listagem deve exibir o nome do fornecedor responsável ao lado de cada produto. | Alta |
| **RF03** | Consulta de Produtos por Fornecedor (Master-Detail) | O sistema deve disponibilizar uma visualização filtrada onde, ao selecionar um fornecedor, são exibidos todos os produtos fornecidos por aquela empresa. | Alta |
| **RF04** | Gestão de Registros (Edição e Exclusão) | O sistema deve permitir editar os dados de fornecedores e produtos, e permitir exclusões respeitando o bloqueio por integridade referencial (RN04). | Alta |
| **RF05** | Autenticação Simples de Operador | O sistema deve fornecer uma tela inicial de login (e-mail e senha). O acesso aos módulos só deve ser liberado mediante validação com sucesso das credenciais. | Alta |

## 3\. Requisitos Não Funcionais (RNF)

Os requisitos não funcionais descrevem **como** o sistema deve se comportar (qualidade, segurança, desempenho).

| ID | Categoria | Descrição |
| :---- | :---- | :---- |
| **RNF01** | Usabilidade e Interface | Interface limpa, responsiva e intuitiva, com formulários orientativos e mensagens claras de validação e erro. |
| **RNF02** | Performance | Respostas da API e transições de tela em menos de 1 segundo para operações locais. |
| **RNF03** | Confiabilidade e Integridade | Operações de banco executadas sob transações ACID para garantir consistência entre chaves primárias e estrangeiras. |
| **RNF04** | Banco de Dados XAMPP | O sistema deve utilizar o banco relacional MySQL/MariaDB disponibilizado via XAMPP ou ambiente local com phpMyAdmin habilitado para gestão visual, garantindo a persistência dos dados entre reinicializações e eliminando a necessidade de SGBDs externos adicionais. |
| **\]RNF05** | Padrão de API REST | Comunicação entre frontend e backend padronizada em JSON via HTTP com códigos de status canônicos (200, 201, 400, 401, 404, 409). |

Para refletir a relação **Um-para-Muitos (1:N)**, a estrutura do banco de dados opera estritamente com **duas entidades/tabelas**, armazenando a chave estrangeira fornecedor\_id diretamente na tabela de Produtos.

### 1\. Tabela Fornecedores

Guarda apenas os dados que pertencem exclusivamente ao fornecedor.

* **id** (INT, **Chave Primária \- PK**): Identificador único do fornecedor.  
* **razao\_social** (VARCHAR): Nome da empresa.  
* **cnpj** (VARCHAR): Documento (único).  
* **email** (VARCHAR): E-mail comercial do fornecedor.  
* **telefone** (VARCHAR): Telefone de contato principal.  
* **data\_cadastro** (DATETIME): Registro de quando foi adicionado ao sistema.

### 2\. Tabela Produtos

Guarda apenas os dados inerentes ao produto em si, independentemente de quem o vende.

* **id** (INT, **Chave Primária \- PK**): Identificador único do produto.  
* **nome** (VARCHAR): Nome comercial do produto.  
* **sku** (VARCHAR): Código interno/SKU do produto (único).  
* **descricao** (TEXT): Detalhes e especificações do produto.  
* **preco\_custo** (DECIMAL): Valor de aquisição do produto.  
* **prazo\_entrega\_dias** (INT): Prazo estimado de entrega fornecido pelo fabricante.

**fornecedor\_id** (INT, **Chave Estrangeira \- FK**): Referência obrigatória ao ID do fornecedor responsável.

Pesquisado e desenvolvido por Gabriel Ferreira da Silva 