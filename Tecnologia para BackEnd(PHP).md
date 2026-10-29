# **Tecnologia para BackEnd (PHP 8.3 / Laravel 11.x)**

Condições de conclusão  
Para adaptar o **Spec Driven Development (SDD)** especificamente para o ecossistema **PHP 8.3 com Laravel 11.x (MVC)**, precisamos definir a arquitetura padrão do framework e resolver um detalhe técnico importante: como o Eloquent ORM lida com tabelas associativas e relacionamentos 1:N e N:N que possuem colunas extras (como o preco\_custo).

No modelo simplificado **1:N (Um-para-Muitos)**, o mapeamento é realizado através dos métodos hasMany e belongsTo nos Models (app/Models/), sem a necessidade de chaves compostas complexas.

Abaixo estão as especificações técnicas para o desenvolvimento em Laravel 11.x.

* ## 1\. Stack e Dependências Principais (composer.json)

* **Laravel Controllers & Routing:** Controllers nativos em app/Http/Controllers/ registrados em routes/api.php ou routes/web.php.  
* **Eloquent ORM & Migrations:** Para mapeamento objeto-relacional (Models em app/Models/) e controle de versão de banco (Migrations em database/migrations/).  
* **Database (SQLite / MySQL):** SQLite (database/database.sqlite) para desenvolvimento rápido ou MySQL (XAMPP / Porta 3306).  
* **Recursos Nativos do PHP 8.3:** Constructor Property Promotion e strict\_types=1 para escrita limpa e segura sem boilerplate.  
* **Form Requests & API Resources:** Form Requests dedicados em app/Http/Requests/ para validação com rules() e API Resources em app/Http/Resources/ para transformação e padronização JSON.  
* **Laravel Sanctum / Breeze:** Para autenticação de APIs, tokens e controle de acesso seguro.

## 2\. Configuração de Conexão (.env)

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=appdb
DB_USERNAME=root
DB_PASSWORD=
```

## 3\. Mapeamento de Banco de Dados, Relacionamentos e Governança

No Eloquent ORM, relacionamentos 1:N são mapeados via hasMany() e belongsTo(). Em relacionamentos N:N com atributos extras na pivot (como preco\_custo), utiliza-se belongsToMany()-\>withPivot('preco\_custo') ou um Model associativo dedicado. Nas migrations, a integridade relacional deve impor trava via onDelete('restrict').

A governança do projeto segue rigorosamente o Spec-Driven Development (SDD), com suporte às diretrizes de IA via Laravel Boost (.ai/guidelines/\*) e a constituição técnica central do projeto mantida em memory/constitution.md (GitHub Spec Kit).

Pesquisado e desenvolvido por Gabriel Ferreira da Silva 