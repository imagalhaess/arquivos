# Desafio Prático Júnior: API de Gerenciamento de Tarefas (To-Do List)

## 1. Objetivo do Desafio
O objetivo deste desafio é avaliar o conhecimento técnico de um desenvolvedor nível júnior na linguagem **Java** e no framework **Spring Boot**, aplicando conceitos fundamentais de Programação Orientada a Objetos (POO), Estruturas de Dados, Design de Código e Arquitetura de Sistemas modernos, conforme o material de estudo fornecido.

## 2. Contexto do Problema
Você foi designado para criar o backend de uma aplicação de lista de tarefas (To-Do List). A aplicação deve permitir a criação, listagem e conclusão de tarefas. Para este desafio, não utilizaremos um banco de dados externo; a persistência deve ser feita em memória utilizando as coleções do Java.

## 3. Conhecimentos Aplicados (Baseado no Material)

| Categoria | Conceitos Aplicados | Referência no Material |
| :--- | :--- | :--- |
| **Java/POO** | Encapsulamento, Classes, Records (Imutabilidade), Tipos Primitivos vs. Wrappers. | *Java Guia Supremo Parte A* |
| **Estruturas de Dados** | Uso de `HashMap` ou `ArrayList` para simular persistência em memória. | *Java Guia Supremo Parte A* |
| **Spring Boot/REST** | `@RestController`, `@RequestMapping`, `@PostMapping`, `@GetMapping`, `@PutMapping`, Códigos HTTP (201, 400, 404). | *Java Guia Supremo Parte C* |
| **Design de Código** | DTOs (Records) para transferência de dados e **Bean Validation**. | *Java Guia Supremo Parte C* |

## 4. Etapas do Desafio

### Etapa 1: Modelagem do Domínio e DTOs
1. **Entidade `Task`**: Crie uma classe `Task` com os atributos `id` (Long), `title` (String), `description` (String) e `status` (String ou Enum). Utilize o princípio do **Encapsulamento**.
2. **DTO de Criação**: Crie um **Java Record** chamado `TaskRequestDTO` para receber os dados de entrada (`title` e `description`). Utilize anotações do **Bean Validation** como `@NotBlank` e `@Size`.

### Etapa 2: Camada de Serviço e Persistência
1. **`TaskService`**: Crie uma classe anotada com `@Service`.
2. **Persistência**: Utilize um `Map<Long, Task>` para armazenar as tarefas em memória.
3. **Lógica**: Implemente métodos para:
   - Criar uma tarefa (gerando ID automático e definindo status inicial como "PENDING").
   - Listar todas as tarefas.
   - Concluir uma tarefa (alterar status para "COMPLETED" buscando pelo ID).

### Etapa 3: Camada de Controller (API REST)
1. **`TaskController`**: Crie os endpoints:
   - `POST /tasks`: Para criar uma nova tarefa (Retornar HTTP 201).
   - `GET /tasks`: Para listar todas as tarefas (Retornar HTTP 200).
   - `PUT /tasks/{id}/complete`: Para marcar uma tarefa como concluída (Retornar HTTP 200 ou 404 caso não exista).

## 5. Questões Teóricas (Bônus Arquitetural)
Responda brevemente:
1. **Kafka**: Como o uso de mensageria poderia ajudar caso precisássemos notificar outros sistemas sobre a conclusão de uma tarefa sem travar a API principal?
2. **Observabilidade**: Cite uma métrica importante que você monitoraria nesta API usando Prometheus e Grafana.

---
**Autor:** Isabela M -
**Data:** 16 de Janeiro de 2026
