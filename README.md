# Sistema de Biblioteca - Microserviços

Sistema simples de gerenciamento de biblioteca implementado com arquitetura de microserviços.

## 📚 Visão Geral

Este projeto demonstra a implementação de um sistema de biblioteca usando dois microserviços independentes que se comunicam entre si para gerenciar o cadastro de livros e o controle de empréstimos.

## 🏗️ Arquitetura

O sistema é composto por dois microserviços:
```
┌─────────────────┐         ┌─────────────────┐
│  Book Service   │◄────────│  Loan Service   │
│  (Livros)       │         │  (Empréstimos)  │
└─────────────────┘         └─────────────────┘
```

### Microserviços

1. **Book Service**: Responsável pelo gerenciamento do catálogo de livros e disponibilidade
2. **Loan Service**: Responsável pelo controle de empréstimos e devoluções

## 📋 Regras de Negócio

### Book Service

#### 1. Cadastro de Livro
- Um livro deve conter:
    - Título (obrigatório)
    - Autor (obrigatório)
    - ISBN (obrigatório e único)
    - Quantidade total de exemplares (obrigatório, mínimo 1)
- O ISBN deve ser único no sistema
- A quantidade total não pode ser negativa

#### 2. Consulta de Disponibilidade
- Retorna a quantidade de exemplares disponíveis para empréstimo
- **Fórmula**: `Disponível = Quantidade Total - Quantidade Emprestada`
- Um livro está disponível quando há pelo menos 1 exemplar não emprestado

#### 3. Atualização de Quantidade
- Ao receber notificação de empréstimo:
    - Decrementa a quantidade disponível em 1
    - Valida se ainda há exemplares disponíveis antes de confirmar
- Ao receber notificação de devolução:
    - Incrementa a quantidade disponível em 1
    - Valida que a quantidade não ultrapasse o total de exemplares

---

### Loan Service

#### 1. Criar Empréstimo
- Dados obrigatórios:
    - ID do usuário
    - ISBN do livro
- Fluxo de criação:
    1. Consulta o Book Service para verificar disponibilidade
    2. Se disponível (quantidade > 0), cria o empréstimo
    3. Define data de empréstimo como a data atual
    4. Define data de devolução prevista: data atual + 14 dias
    5. Notifica o Book Service para decrementar disponibilidade
- Se o livro não estiver disponível, rejeita o empréstimo

#### 2. Devolver Livro
- Permite devolução de empréstimo ativo
- Ações ao devolver:
    1. Marca o empréstimo como devolvido
    2. Registra a data efetiva de devolução
    3. Notifica o Book Service para incrementar disponibilidade
- Um empréstimo só pode ser devolvido uma vez

#### 3. Listar Empréstimos
- **Empréstimos ativos**: empréstimos que ainda não foram devolvidos
- **Empréstimos atrasados**: empréstimos ativos cuja data prevista de devolução já passou
    - `Data Atual > Data de Devolução Prevista AND Status = ATIVO`
- Permite filtrar empréstimos por usuário

---

## 🔄 Fluxo de Comunicação

### Fluxo de Empréstimo
```
┌─────────┐                  ┌──────────────┐                ┌──────────────┐
│ Cliente │                  │ Loan Service │                │ Book Service │
└────┬────┘                  └──────┬───────┘                └──────┬───────┘
     │                              │                               │
     │ POST /loans                  │                               │
     │ {userId, isbn}               │                               │
     ├─────────────────────────────►│                               │
     │                              │                               │
     │                              │ GET /books/{isbn}/available   │
     │                              ├──────────────────────────────►│
     │                              │                               │
     │                              │ {available: 3}                │
     │                              │◄──────────────────────────────┤
     │                              │                               │
     │                              │ POST /books/{isbn}/borrow     │
     │                              ├──────────────────────────────►│
     │                              │                               │
     │                              │ {success: true}               │
     │                              │◄──────────────────────────────┤
     │                              │                               │
     │ 201 Created                  │                               │
     │ {loanId, dueDate}            │                               │
     │◄─────────────────────────────┤                               │
     │                              │                               │
```

### Fluxo de Devolução
```
┌─────────┐                  ┌──────────────┐                ┌──────────────┐
│ Cliente │                  │ Loan Service │                │ Book Service │
└────┬────┘                  └──────┬───────┘                └──────┬───────┘
     │                              │                               │
     │ PUT /loans/{id}/return       │                               │
     ├─────────────────────────────►│                               │
     │                              │                               │
     │                              │ POST /books/{isbn}/return     │
     │                              ├──────────────────────────────►│
     │                              │                               │
     │                              │ {success: true}               │
     │                              │◄──────────────────────────────┤
     │                              │                               │
     │ 200 OK                       │                               │
     │ {returnedAt}                 │                               │
     │◄─────────────────────────────┤                               │
     │                              │                               │
```

---

## 🚀 Como Executar
```bash
# Clone o repositório
git clone https://github.com/seu-usuario/library-microservices.git

# Entre no diretório do projeto
cd library-microservices

# Execute os serviços
docker-compose up -d
```

---

## 🛠️ Tecnologias

- **Linguagem**: Java/Spring Boot (ou Go/Gin)
- **Banco de Dados**: PostgreSQL
- **Comunicação**: REST API
- **Containerização**: Docker
- **Orquestração**: Docker Compose

---

## 📝 Endpoints

### Book Service (Port 8081)

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/books` | Cadastrar novo livro |
| GET | `/books/{isbn}` | Buscar livro por ISBN |
| GET | `/books/{isbn}/available` | Consultar disponibilidade |
| POST | `/books/{isbn}/borrow` | Notificar empréstimo |
| POST | `/books/{isbn}/return` | Notificar devolução |

### Loan Service (Port 8082)

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/loans` | Criar empréstimo |
| GET | `/loans/user/{userId}` | Listar empréstimos do usuário |
| GET | `/loans/overdue` | Listar empréstimos atrasados |
| PUT | `/loans/{id}/return` | Devolver livro |

---

## 📦 Estrutura do Projeto
```
library-microservices/
├── book-service/
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── loan-service/
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── docker-compose.yml
└── README.md
```

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues e pull requests.

---

## 📄 Licença

Este projeto está sob a licença MIT.