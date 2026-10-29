**Tecnologia para Front-End (SPA React 18 \+ Vite \+ TypeScript)**

*Integração da Interface do Usuário com API RESTful em PHP 8.3 / Laravel 11.x via Eloquent API Resources e Modelo SDD*

| MetodologiaSpec-Driven Dev (SDD) | Front-EndReact 18 / Vite / TS | EstilizaçãoTailwind CSS | Integração APIAxios (Laravel REST) |
| :---: | :---: | :---: | :---: |

| Diretriz de Engenharia e Desacoplamento Front-End no SDD:A arquitetura Front-End opera como uma Single Page Application (SPA) totalmente desacoplada do Back-End. Toda a comunicação com o servidor PHP 8.3 / Laravel 11.x ocorre via chamadas assíncronas HTTP/REST com payloads JSON mapeados rigidamente através de Eloquent API Resources. Esta especificação detalha a arquitetura de comunicação Axios com interceptores para tratamento global de erros (401, 403, 409, 422, 500), contratos TypeScript de requisição e resposta, e a matriz de rastreabilidade completa com o Software Design Document (SDD) e regras de negócio (RN01 a RN05). |
| :---- |

# **1\. Stack Tecnológica do Front-End (Ecossistema React)**

A escolha do ecossistema React 18+ impulsionado pelo Vite e TypeScript proporciona um ambiente de desenvolvimento ágil, com hot module replacement (HMR) instantâneo e verificação estática de tipos. A tipagem estrita garante que as interfaces do Front-End espelhem fielmente os DTOs e recursos retornados pelos endpoints da API Laravel.

| Tecnologia / Pacote | Versão / Ferramenta | Papel na Arquitetura Front-End |
| ----- | ----- | ----- |
| **React 18+** | react (^18.3) | Biblioteca principal para construção de interfaces reativas baseadas em componentes. |
| **Vite** | vite (^5.x) | Ferramenta de build de alta performance para inicialização e empacotamento da SPA. |
| **TypeScript** | typescript (^5.x) | Tipagem estática estrita para interfaces de dados, DTOs e contratos de API. |
| **Tailwind CSS** | tailwindcss (^3.x) | Framework CSS utilitário para estilização ágil, responsiva e consistente. |
| **React Router DOM** | react-router-dom (^6.x) | Gerenciamento de rotas do lado do cliente e controle de navegação protegida. |
| **Axios** | axios (^1.x) | Cliente HTTP baseado em Promises para consumo de endpoints RESTful da API Laravel. |

# **2\. Arquitetura Modular de Pastas e Componentes**

O projeto React adota uma organização por responsabilidades claras. A segregação entre serviços de API, definições de tipos, contextos de autenticação, componentes reutilizáveis e páginas garante um código sustentável e fácil de auditar durante a implementação do SDD.

src/  
├── assets/              \# Imagens, logos e estilos globais  
├── components/          \# Componentes de UI genéricos (Navbar, Modal, Alert, FormInput)  
├── context/             \# Estado global da aplicação (AuthContext.tsx)  
├── pages/               \# Páginas da aplicação (LoginPage, FornecedoresPage, ProdutosPage)  
├── services/            \# Cliente HTTP Axios configurado (api.ts)  
├── types/               \# Interfaces TypeScript dos modelos (fornecedor.ts, produto.ts, auth.ts)  
├── App.tsx              \# Definição de Rotas e Provedores Globais  
└── main.tsx             \# Ponto de entrada da aplicação React

# **3\. Tipagem Estática dos Contratos da API (TypeScript Interfaces)**

Para garantir que o consumo da API em PHP/Laravel ocorra sem incompatividades de tipos, as interfaces TypeScript definem estritamente a estrutura das entidades e payloads de requisição.

// src/types/api.ts  
export interface ApiResponse\<T\> {  
    data: T;  
    message?: string;  
}

export interface PaginatedResponse\<T\> {  
    data: T\[\];  
    links: {  
        first: string;  
        last: string;  
        prev: string \| null;  
        next: string \| null;  
    };  
    meta: {  
        current\_page: number;  
        from: number;  
        last\_page: number;  
        per\_page: number;  
        to: number;  
        total: number;  
    };  
}

export interface ValidationErrorResponse {  
    message: string;  
    errors: Record\<string, string\[\]\>;  
}

// src/types/fornecedor.ts  
export interface FornecedorResource {  
    id: number;  
    razao\_social: string;  
    cnpj: string;  
    email: string;  
    telefone: string \| null;  
    created\_at: string;  
    updated\_at: string;  
}

export interface CreateFornecedorDTO {  
    razao\_social: string;  
    cnpj: string;  
    email: string;  
    telefone?: string;  
}

export interface UpdateFornecedorDTO extends Partial\<CreateFornecedorDTO\> {}

// src/types/produto.ts  
export interface ProdutoResource {  
    id: number;  
    nome: string;  
    sku: string;  
    descricao: string \| null;  
    preco\_custo: number;  
    prazo\_entrega\_dias: number;  
    fornecedor\_id: number;  
    fornecedor?: FornecedorResource; // Laravel Eloquent API Resource Relation  
    created\_at: string;  
    updated\_at: string;  
}

export interface CreateProdutoDTO {  
    nome: string;  
    sku: string;  
    descricao?: string;  
    preco\_custo: number;  
    prazo\_entrega\_dias: number;  
    fornecedor\_id: number;  
}

export interface UpdateProdutoDTO extends Partial\<CreateProdutoDTO\> {}

# **4\. Configuração do Cliente HTTP (Axios Integration)**

A comunicação com o backend em PHP 8.3 / Laravel 11.x é centralizada na instância do Axios (\`src/services/api.ts\`). O cliente conta com interceptores de requisição para injeção automática do Token Bearer Sanctum e interceptores de resposta para tratamento padronizado e centralizado de status HTTP (401, 403, 409, 422, 500).

// src/services/api.ts  
import axios, { AxiosError } from 'axios';  
import { ValidationErrorResponse } from '../types/api';

export const api \= axios.create({  
    baseURL: import.meta.env.VITE\_API\_BASE\_URL \|\| 'http\://localhost:8000/api',  
    headers: {  
        'Content-Type': 'application/json',  
        'Accept': 'application/json',  
    },  
    withCredentials: true,  
});

// Interceptor de Requisição: Injeção do Token Bearer  
api.interceptors.request.use((config) \=\> {  
    const token \= localStorage.getItem('@App:token');  
    if (token && config.headers) {  
        config.headers.Authorization \= \`Bearer \${token}\`;  
    }  
    return config;  
});

// Interceptor de Resposta: Tratamento Global de Erros HTTP  
api.interceptors.response.use(  
    (response) \=\> response,  
    (error: AxiosError) \=\> {  
        if (\!error.response) {  
            console.error('Erro de Conexão/Rede:', error.message);  
            return Promise.reject({ message: 'Falha na comunicação com o servidor.' });  
        }

        const { status, data } \= error.response;

        switch (status) {  
            case 401: // Unauthorized  
                localStorage.removeItem('@App:token');  
                window.location.href \= '/login';  
                break;  
            case 403: // Forbidden  
                console.warn('Acesso negado para o recurso solicitado.');  
                break;  
            case 409: // Conflict (ex: Violação de integridade / RN04)  
                console.warn('Conflito de regra de negócio:', data);  
                break;  
            case 422: // Unprocessable Entity (Erros de validação de FormRequest)  
                const validationErrors \= data as ValidationErrorResponse;  
                return Promise.reject(validationErrors);  
            case 500: // Internal Server Error  
                console.error('Erro interno no servidor Laravel.');  
                break;  
            default:  
                console.error(\`Erro HTTP não tratado: \${status}\`);  
        }

        return Promise.reject(data \|\| { message: 'Ocorreu um erro inesperado.' });  
    }  
);

# **5\. Especificação das Telas da SPA React e Rotas Protegidas**

A aplicação é composta por 4 visões principais, operando sob um layout comum com barra de navegação responsiva e controle rigoroso de autenticação via \`ProtectedRoute\`.

| Rota Front-End | Componente / Tela | Funcionalidades e Regras de Negócio |
| ----- | ----- | ----- |
| **/login** | LoginPage | Formulário de acesso. Autentica via POST /api/auth/login (Laravel Sanctum). Armazena token e trata falhas 401/422. |
| **/fornecedores** | FornecedoresPage | Listagem paginada (\`PaginatedResponse\<FornecedorResource\>\`) e cadastro via modal com máscaras e tratamento de conflito (RF01). |
| **/produtos** | ProdutosPage | Listagem e criação de produtos. Exige vínculo obrigatório com fornecedor ativo via dropdown dinâmico (RN01, RF02). |
| **/fornecedores/:id/produtos** | MasterDetailFornecedorPage | Visão Master-Detail. Carrega dados do fornecedor e seus produtos associados via endpoint otimizado com Eloquent API Resources (RF03). |

# **6\. Matriz de Conformidade do Front-End com o SDD**

Esta matriz comprova que a implementação da SPA React cumpre 100% dos requisitos funcionais e regras de negócio estipulados no Software Design Document do sistema de Gestão de Fornecedores e Produtos.

| Requisito / Regra | Componente / Tela | Contrato / Resource API | Tratamento de Erro & Validação |
| ----- | ----- | ----- | ----- |
| **RF01 (CRUD Fornecedores)** | FornecedoresPage.tsx | \`FornecedorResource\`\`CreateFornecedorDTO\` | Máscaras de CNPJ, validação de e-mail e tratamento de HTTP 422 (\`ValidationErrorResponse\`). |
| **RF02 / RN01 (CRUD Produtos & Vínculo)** | ProdutosPage.tsx | \`ProdutoResource\`\`CreateProdutoDTO\` | Exigência de \`fornecedor\_id\` ativo no dropdown. Validação de SKU único via HTTP 422\. |
| **RF03 (Master-Detail)** | MasterDetailFornecedorPage.tsx | \`FornecedorResource\` com relacionamento de produtos embutido | Tratamento de 404 caso o fornecedor não exista; exibição de estado vazio quando sem produtos. |
| **RN04 (Bloqueio de Exclusão)** | FornecedoresPage.tsx | DELETE \`/api/fornecedores/{id}\` | Captura de resposta HTTP 409 (Conflict) ao tentar remover fornecedor com produtos vinculados, exibindo alerta visual para o usuário. |
| **RN05 (Autenticação Sanctum)** | ProtectedRoute.tsx / AuthContext.tsx | Bearer Token Injection / LocalStorage | Redirecionamento automático de requisições não autenticadas (HTTP 401\) diretamente para a página de login. |

