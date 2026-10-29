# **Arquitetura de Front-End SPA e Integração com API RESTful (React 18 / TypeScript / Vite / Laravel 11\)**

# **Condições de Conclusão**

Para alinhar a especificação do sistema ao ambiente de desenvolvimento desacoplado/SPA baseado em **React 18, Vite, TypeScript e Tailwind CSS** integrados à API RESTful em **PHP 8.3 e Laravel 11.x** (executada sobre servidor Apache/MySQL via XAMPP), este documento descreve detalhadamente a stack do front-end, a arquitetura de comunicação, as interfaces TypeScript, a cliente Axios com interceptores de sessão/CSRF, a matriz de tratamento de respostas HTTP e a rastreabilidade com o SDD.

A aplicação adota a abordagem Single Page Application (SPA), onde o front-end em React lida dinamicamente com a renderização de rotas e componentes de interface, consumindo a API RESTful do Laravel via requisições assíncronas (JSON). Toda a persistência e validação de regras de negócio são gerenciadas no backend pelo Eloquent ORM no MySQL local (XAMPP).

# **1\. Visão Geral e Arquitetura de Comunicação (SPA React \+ API RESTful)**

A comunicação entre a SPA (React 18 \+ Vite) e o backend (Laravel 11\) ocorre exclusivamente através de chamadas HTTP/HTTPS RESTful formatadas em JSON, utilizando autenticação por sessão/cookies protegida contra CSRF (Laravel Breeze / Sanctum SPA Authentication):

* **Desacoplamento Front/Back:** O front-end React opera de forma totalmente independente da camada de apresentação tradicional do Laravel (Blade), agindo como cliente consomidor de dados.  
* **Fluxo de Dados Assíncrono:** Todas as operações CRUD e consultas para relatórios são executadas em segundo plano via Axios, mantendo a interface fluida e sem recarregamento de página.  
* **Segurança e Proteção CSRF:** Antes de cada requisição mutativa (POST, PUT, DELETE), a SPA obtém o cookie de proteção via GET /sanctum/csrf-cookie e o injeta automaticamente nos cabeçalhos HTTP.

# **2\. Stack Tecnológica do Front-End (React 18 / TypeScript)**

A stack tecnológica do front-end foi selecionada para garantir tipagem estática rigorosa, rápida compilação e estilização moderna:

* **React 18:** Biblioteca principal para construção de componentes funcionais declarativos e gerenciamento reativo do estado da interface.  
* **TypeScript 5.x:** Linguagem base para tipagem forte de payloads, respostas de API e propriedades de componentes, prevenindo erros em tempo de desenvolvimento.  
* **Vite:** Ferramenta de build de altíssimo desempenho para desenvolvimento local e empacotamento otimizado de produção.  
* **Tailwind CSS:** Framework CSS utilitário para design moderno, responsivo e adaptável a desktops e dispositivos móveis.  
* **React Router DOM 6:** Gerenciamento de rotas no cliente com suporte a rotas protegidas (autenticadas) e navegação sem reload.  
* **Axios:** Cliente HTTP baseado em Promessas com interceptores para injeção automática de tokens CSRF e tratamento global de erros HTTP.  
* **Lucide React / Heroicons:** Conjunto de ícones vetoriais leves para enriquecimento visual dos menus, botões e alertas da interface.

# **3\. Arquitetura de Camadas e Organização de Pastas (SPA React)**

O código-fonte do front-end está organizado na estrutura de diretórios do projeto React/Vite sob a pasta src/:

| Diretório Front-End (src/) | Responsabilidade no Front-End | Correspondência Backend (Laravel API) |
| :---- | :---- | :---- |
| `src/types/` | Definição de interfaces e tipos TypeScript (Fornecedor, Produto, Associacao, User). | Models Eloquent / DTOs / Resources |
| `src/services/api.ts` | Instância Axios configurada com baseURL, credentials e interceptores de erro/CSRF. | routes/api.php e Sanctum Middleware |
| `src/pages/auth/Login.tsx` | Componente de Login da SPA com validação de credenciais e captura de erros HTTP 401\. | POST /api/login (AuthenticatedSessionController) |
| `src/pages/fornecedores/` | Componentes de listagem, formulário de cadastro e modal de edição de Fornecedores. | API Resource routes /api/fornecedores |
| `src/pages/produtos/` | Componentes de listagem, busca dinâmica e formulário CRUD de Produtos. | API Resource routes /api/produtos |
| `src/pages/associacao/` | Tela de vínculo N:N entre Produto e Fornecedor informando Preço de Custo e Prazo. | POST /api/produtos/{id}/fornecedores |
| `src/pages/relatorios/` | Painel de relatórios expansíveis Master-Detail (RF05/RF06) em tabela dinâmicas React. | GET /api/relatorios/fornecedores-produtos |

# **4\. Interfaces TypeScript do Sistema**

As interfaces TypeScript garantem o contrato de dados entre as respostas JSON da API Laravel e os componentes React da SPA:

```ts
// src/types/index.ts

export interface Fornecedor {
  id_fornecedor: number;
  razao_social: string;
  cnpj: string;
  telefone: string;
  email: string;
  created_at?: string;
  updated_at?: string;
  produtos?: ProdutoAssociado[];
}

export interface Produto {
  id_produto: number;
  sku_codigo: string;
  nome: string;
  descricao: string;
  categoria: string;
  created_at?: string;
  updated_at?: string;
  fornecedores?: FornecedorAssociado[];
}

export interface ProdutoAssociado extends Produto {
  pivot: {
    preco_custo: number;
    prazo_entrega_dias: number;
  };
}

export interface FornecedorAssociado extends Fornecedor {
  pivot: {
    preco_custo: number;
    prazo_entrega_dias: number;
  };
}

export interface AssociacaoPayload {
  id_produto: number;
  id_fornecedor: number;
  preco_custo: number;
  prazo_entrega_dias: number;
}

export interface ApiValidationError {
  message: string;
  errors: Record<string, string[]>;
}
```

# **5\. Configuração do Cliente Axios e Interceptores HTTP**

A comunicação HTTP da SPA é centralizada em um módulo Axios configurado para lidar automaticamente com cookies de sessão, CSRF e capturar exceções globais:

```ts
// src/services/api.ts
import axios from 'axios';

export const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  withCredentials: true,
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

// Obtém o cookie CSRF do Sanctum antes de requisições de modificação
export const getCsrfToken = async () => {
  await axios.get('http://localhost:8000/sanctum/csrf-cookie', { withCredentials: true });
};

// Interceptor de resposta global para tratamento de erros e redirecionamento de sessão
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = '/login?expired=true';
    }
    return Promise.reject(error);
  }
);
```

# **6\. Matriz de Tratamento de Respostas HTTP na SPA**

A SPA trata rigorosamente todos os retornos e códigos de status HTTP enviados pela API RESTful do Laravel:

| Código HTTP / Resposta | Significado na API RESTful Laravel 11 | Comportamento Visual na SPA (React 18\) |
| :---- | :---- | :---- |
| 200 OK | Requisição ou carregamento executado com sucesso. | Renderização imediata dos dados nos componentes React ou atualização do estado local. |
| 201 Created | Registro criado com sucesso no banco de dados MySQL. | Exibe notificação Toast de sucesso, fecha o modal/formulário e atualiza a listagem dinamicamente. |
| 401 Unauthenticated | Sessão expirada ou credenciais inválidas na tela de login. | Interceptor redireciona para a tela /login exibindo mensagem de sessão expirada. |
| 409 Conflict | Tentativa de duplicar vínculo de produto e fornecedor (RN02). | Exibe modal de confirmação para o usuário perguntando se deseja atualizar os valores do vínculo existente. |
| 422 Unprocessable Entity | Falha nas regras de validação enviadas pelo Form Request (CNPJ inválido, SKU duplicado, preço \<= 0). | Mapeia a propriedade errors da resposta nos estados dos inputs, destacando bordas em vermelho e mensagens de erro. |

# **7\. Matriz de Rastreabilidade com a Especificação do SDD**

Esta matriz garante a correspondência direta entre as visões da SPA, os componentes React, os endpoints da API Laravel e os requisitos do SDD:

| Requisito / Tela (SDD) | Componente SPA (React 18\) | Endpoint API (Laravel 11\) | Regra de Negócio / Estado |
| :---- | :---- | :---- | :---- |
| RF01 \- Autenticação | `<Login />` | `POST /api/login` | Validação de e-mail/senha e emissão do cookie de sessão. |
| RF02 \- Fornecedores | `<FornecedoresList />` | `GET/POST /api/fornecedores` | Validação de CNPJ único e formato de telefone/e-mail. |
| RF03 \- Produtos | `<ProdutosList />` | `GET/POST /api/produtos` | Validação de código SKU único e campos obrigatórios. |
| RF04 \- Associação N:N | `<AssociarProduto />` | `POST /api/associacoes` | RN02 (Verificação de duplicidade) e preços/prazos \> 0\. |
| RF05 / RF06 \- Relatórios | `<RelatorioMasterDetail />` | `GET /api/relatorios/n-n` | Carregamento Eager Loading no Eloquent com busca expansível no front-end. |

# **8\. Governança e Execução do Ambiente SPA \+ API**

Para executar a solução localmente com o Apache/MySQL no XAMPP e os servidores de desenvolvimento:

* `php artisan migrate --seed`: Executa as migrações no banco MySQL e inicializa os dados de teste.  
* `php artisan serve`: Inicia a API RESTful do Laravel na porta 8000\.  
* `npm run dev`: Inicia o servidor de desenvolvimento do Vite/React 18 na porta 5173\.

