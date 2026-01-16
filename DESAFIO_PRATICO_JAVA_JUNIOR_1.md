# 🏆 DESAFIO PRÁTICO: Sistema de Notificações de Pedidos

## 📋 Visão Geral

Você foi contratada como **Desenvolvedora Java Jr** na empresa **FastDelivery**, uma startup de entregas rápidas. Sua missão é construir o **Sistema de Notificações de Pedidos**, que processa pedidos e notifica clientes sobre o status de suas entregas.

---

## 🎯 Objetivo Final

Construir uma aplicação completa com **dois microsserviços** que se comunicam via **Apache Kafka**, containerizados com **Docker**, com **observabilidade** (Prometheus/Grafana) e **pipeline CI/CD** no GitHub Actions.

```
┌─────────────────┐      Kafka        ┌─────────────────┐
│  pedido-service │ ───────────────▶  │ notificacao-svc │
│    (Producer)   │   "pedidos"       │   (Consumer)    │
│     :8080       │                   │     :8081       │
└─────────────────┘                   └─────────────────┘
        │                                     │
        ▼                                     ▼
   PostgreSQL                            PostgreSQL
   (pedidos_db)                       (notificacoes_db)
```

---

## 📚 Conhecimentos que Serão Aplicados

| Área | Conceitos |
|------|-----------|
| **Java & POO** | Classes, Records, Interfaces, Encapsulamento |
| **Spring Boot** | Controllers, Services, Repositories, DTOs, Validações |
| **Design Patterns** | Factory, Strategy, DTO/Mapper |
| **Banco de Dados** | JPA/Hibernate, PostgreSQL |
| **Apache Kafka** | Producer, Consumer, Topics, Consumer Groups |
| **Docker** | Dockerfile, Docker Compose, Multi-stage builds |
| **Observabilidade** | Spring Actuator, Prometheus, Grafana |
| **Testes** | JUnit 5, Mockito, TDD |
| **CI/CD** | GitHub Actions, Pipeline automatizado |
| **Boas Práticas** | SOLID, Clean Code, Tratamento de Exceções |

---

## 🗺️ ETAPAS DO DESAFIO

---

## ETAPA 1: Modelagem do Domínio (DDD Básico)
**⏱️ Tempo estimado: 1-2 horas**

### 1.1 Defina a Linguagem Ubíqua

Crie um glossário com os termos do negócio:

| Termo | Definição |
|-------|-----------|
| Pedido | Solicitação de entrega feita pelo cliente |
| Cliente | Pessoa que faz o pedido |
| Status do Pedido | Estado atual: CRIADO, EM_TRANSITO, ENTREGUE, CANCELADO |
| Notificação | Mensagem enviada ao cliente sobre o pedido |
| Canal | Meio de notificação: EMAIL, SMS, PUSH |

### 1.2 Modele as Entidades

**Serviço de Pedidos:**
```
Pedido
├── id: UUID (identificador único)
├── clienteId: UUID
├── clienteEmail: String
├── clienteTelefone: String
├── descricao: String
├── valor: BigDecimal
├── status: StatusPedido (enum)
├── dataCriacao: LocalDateTime
└── dataAtualizacao: LocalDateTime
```

**Serviço de Notificações:**
```
Notificacao
├── id: UUID
├── pedidoId: UUID
├── clienteEmail: String
├── canal: CanalNotificacao (enum)
├── mensagem: String
├── status: StatusNotificacao (enum: PENDENTE, ENVIADA, FALHA)
├── dataEnvio: LocalDateTime
└── tentativas: Integer
```

### ✅ Entregável da Etapa 1
- [ ] Documento com glossário de termos
- [ ] Diagrama das entidades (pode ser texto ou desenho simples)
- [ ] Enums definidos: `StatusPedido`, `CanalNotificacao`, `StatusNotificacao`

---

## ETAPA 2: Projeto Spring Boot - Pedido Service
**⏱️ Tempo estimado: 3-4 horas**

### 2.1 Crie o projeto no Spring Initializr

**Dependências:**
- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Spring for Apache Kafka
- Spring Boot Actuator
- Validation
- Lombok

### 2.2 Estrutura de Pacotes (Clean Architecture simplificada)

```
com.fastdelivery.pedido
├── controller/
│   └── PedidoController.java
├── service/
│   └── PedidoService.java
├── repository/
│   └── PedidoRepository.java
├── model/
│   ├── Pedido.java
│   └── StatusPedido.java
├── dto/
│   ├── PedidoRequest.java (record)
│   ├── PedidoResponse.java (record)
│   └── PedidoEventDTO.java (record)
├── mapper/
│   └── PedidoMapper.java
├── config/
│   └── KafkaProducerConfig.java
├── exception/
│   ├── PedidoNotFoundException.java
│   └── GlobalExceptionHandler.java
└── PedidoServiceApplication.java
```

### 2.3 Implemente a API REST

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/pedidos` | Criar novo pedido |
| GET | `/api/pedidos/{id}` | Buscar pedido por ID |
| GET | `/api/pedidos` | Listar todos os pedidos |
| PATCH | `/api/pedidos/{id}/status` | Atualizar status do pedido |

### 2.4 Use Records para DTOs

```java
// PedidoRequest.java - com Bean Validation
public record PedidoRequest(
    @NotNull UUID clienteId,
    @NotBlank @Email String clienteEmail,
    @NotBlank String clienteTelefone,
    @NotBlank @Size(min = 5, max = 200) String descricao,
    @NotNull @Positive BigDecimal valor
) {}

// PedidoResponse.java
public record PedidoResponse(
    UUID id,
    UUID clienteId,
    String descricao,
    BigDecimal valor,
    String status,
    LocalDateTime dataCriacao
) {}
```

### 2.5 Implemente o Producer Kafka

Quando o status do pedido mudar, envie um evento para o tópico `pedidos`:

```java
// PedidoEventDTO.java
public record PedidoEventDTO(
    UUID pedidoId,
    UUID clienteId,
    String clienteEmail,
    String clienteTelefone,
    String descricao,
    String status,
    LocalDateTime dataEvento
) {}
```

### ✅ Entregável da Etapa 2
- [ ] Projeto `pedido-service` funcionando
- [ ] API REST com 4 endpoints
- [ ] Validações com Bean Validation
- [ ] DTOs usando Records
- [ ] Producer Kafka configurado
- [ ] Tratamento de exceções global

---

## ETAPA 3: Projeto Spring Boot - Notificação Service
**⏱️ Tempo estimado: 2-3 horas**

### 3.1 Crie o segundo projeto

Mesmas dependências do pedido-service.

### 3.2 Estrutura de Pacotes

```
com.fastdelivery.notificacao
├── consumer/
│   └── PedidoEventConsumer.java
├── service/
│   ├── NotificacaoService.java
│   └── strategy/
│       ├── NotificacaoStrategy.java (interface)
│       ├── EmailNotificacaoStrategy.java
│       └── SmsNotificacaoStrategy.java
├── repository/
│   └── NotificacaoRepository.java
├── model/
│   ├── Notificacao.java
│   ├── CanalNotificacao.java
│   └── StatusNotificacao.java
├── dto/
│   └── PedidoEventDTO.java (mesmo do producer)
├── config/
│   └── KafkaConsumerConfig.java
└── NotificacaoServiceApplication.java
```

### 3.3 Implemente o Consumer Kafka

```java
@Service
@Slf4j
public class PedidoEventConsumer {

    private final NotificacaoService notificacaoService;

    @KafkaListener(topics = "pedidos", groupId = "notificacao-service")
    public void processarEvento(PedidoEventDTO evento) {
        log.info("Evento recebido: pedido {} com status {}", 
                 evento.pedidoId(), evento.status());
        notificacaoService.criarNotificacao(evento);
    }
}
```

### 3.4 Aplique o Strategy Pattern

Crie estratégias diferentes para cada canal de notificação:

```java
public interface NotificacaoStrategy {
    void enviar(Notificacao notificacao);
    CanalNotificacao getCanal();
}

@Component
public class EmailNotificacaoStrategy implements NotificacaoStrategy {
    @Override
    public void enviar(Notificacao notificacao) {
        // Simula envio de email
        log.info("📧 EMAIL enviado para: {}", notificacao.getClienteEmail());
        log.info("   Mensagem: {}", notificacao.getMensagem());
    }
    
    @Override
    public CanalNotificacao getCanal() {
        return CanalNotificacao.EMAIL;
    }
}
```

### ✅ Entregável da Etapa 3
- [ ] Projeto `notificacao-service` funcionando
- [ ] Consumer Kafka processando eventos
- [ ] Strategy Pattern implementado
- [ ] Notificações salvas no banco

---

## ETAPA 4: Docker e Docker Compose
**⏱️ Tempo estimado: 2-3 horas**

### 4.1 Crie o Dockerfile para cada serviço

Use **multi-stage build**:

```dockerfile
# Dockerfile
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 4.2 Crie o docker-compose.yml completo

```yaml
version: '3.8'

networks:
  fastdelivery:
    driver: bridge

services:
  # ========== KAFKA ==========
  kafka:
    image: apache/kafka:latest
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    networks:
      - fastdelivery

  # ========== BANCO PEDIDOS ==========
  postgres-pedidos:
    image: postgres:17
    container_name: postgres-pedidos
    environment:
      POSTGRES_DB: pedidos_db
      POSTGRES_USER: pedidos
      POSTGRES_PASSWORD: pedidos123
    ports:
      - "5432:5432"
    networks:
      - fastdelivery

  # ========== BANCO NOTIFICAÇÕES ==========
  postgres-notificacoes:
    image: postgres:17
    container_name: postgres-notificacoes
    environment:
      POSTGRES_DB: notificacoes_db
      POSTGRES_USER: notificacoes
      POSTGRES_PASSWORD: notificacoes123
    ports:
      - "5433:5432"
    networks:
      - fastdelivery

  # ========== PEDIDO SERVICE ==========
  pedido-service:
    build:
      context: ./pedido-service
      dockerfile: Dockerfile
    container_name: pedido-service
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres-pedidos:5432/pedidos_db
      SPRING_DATASOURCE_USERNAME: pedidos
      SPRING_DATASOURCE_PASSWORD: pedidos123
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    depends_on:
      - kafka
      - postgres-pedidos
    networks:
      - fastdelivery

  # ========== NOTIFICACAO SERVICE ==========
  notificacao-service:
    build:
      context: ./notificacao-service
      dockerfile: Dockerfile
    container_name: notificacao-service
    ports:
      - "8081:8081"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres-notificacoes:5432/notificacoes_db
      SPRING_DATASOURCE_USERNAME: notificacoes
      SPRING_DATASOURCE_PASSWORD: notificacoes123
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    depends_on:
      - kafka
      - postgres-notificacoes
    networks:
      - fastdelivery

  # ========== PROMETHEUS ==========
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - fastdelivery

  # ========== GRAFANA ==========
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin123
    networks:
      - fastdelivery
```

### 4.3 Configure o Prometheus

```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'pedido-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['pedido-service:8080']

  - job_name: 'notificacao-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['notificacao-service:8081']
```

### ✅ Entregável da Etapa 4
- [ ] Dockerfile para cada serviço
- [ ] docker-compose.yml completo
- [ ] prometheus.yml configurado
- [ ] Comando `docker-compose up -d` funcionando
- [ ] Todos os serviços se comunicando

---

## ETAPA 5: Testes Automatizados
**⏱️ Tempo estimado: 2-3 horas**

### 5.1 Testes Unitários do Service

```java
@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {

    @Mock
    private PedidoRepository pedidoRepository;
    
    @Mock
    private KafkaTemplate<String, PedidoEventDTO> kafkaTemplate;
    
    @InjectMocks
    private PedidoService pedidoService;

    @Test
    @DisplayName("Deve criar pedido com status CRIADO")
    void deveCriarPedidoComStatusCriado() {
        // Arrange (Given)
        PedidoRequest request = new PedidoRequest(
            UUID.randomUUID(),
            "cliente@email.com",
            "11999999999",
            "Entrega de documentos",
            new BigDecimal("50.00")
        );
        
        when(pedidoRepository.save(any(Pedido.class)))
            .thenAnswer(inv -> inv.getArgument(0));

        // Act (When)
        PedidoResponse response = pedidoService.criarPedido(request);

        // Assert (Then)
        assertThat(response.status()).isEqualTo("CRIADO");
        verify(pedidoRepository).save(any(Pedido.class));
        verify(kafkaTemplate).send(eq("pedidos"), anyString(), any());
    }

    @Test
    @DisplayName("Deve lançar exceção quando pedido não encontrado")
    void deveLancarExcecaoQuandoPedidoNaoEncontrado() {
        // Arrange
        UUID idInexistente = UUID.randomUUID();
        when(pedidoRepository.findById(idInexistente))
            .thenReturn(Optional.empty());

        // Act & Assert
        assertThatThrownBy(() -> pedidoService.buscarPorId(idInexistente))
            .isInstanceOf(PedidoNotFoundException.class)
            .hasMessageContaining(idInexistente.toString());
    }
}
```

### 5.2 Teste do Strategy Pattern

```java
@ExtendWith(MockitoExtension.class)
class NotificacaoServiceTest {

    @Mock
    private NotificacaoRepository repository;
    
    @Mock
    private EmailNotificacaoStrategy emailStrategy;
    
    @Mock
    private SmsNotificacaoStrategy smsStrategy;

    @Test
    @DisplayName("Deve usar estratégia correta baseado no canal")
    void deveUsarEstrategiaCorreta() {
        // Arrange
        when(emailStrategy.getCanal()).thenReturn(CanalNotificacao.EMAIL);
        
        List<NotificacaoStrategy> strategies = List.of(emailStrategy, smsStrategy);
        NotificacaoService service = new NotificacaoService(repository, strategies);

        PedidoEventDTO evento = new PedidoEventDTO(
            UUID.randomUUID(),
            UUID.randomUUID(),
            "teste@email.com",
            "11999999999",
            "Pedido teste",
            "CRIADO",
            LocalDateTime.now()
        );

        // Act
        service.criarNotificacao(evento);

        // Assert
        verify(emailStrategy).enviar(any(Notificacao.class));
    }
}
```

### ✅ Entregável da Etapa 5
- [ ] Mínimo 5 testes unitários no pedido-service
- [ ] Mínimo 3 testes unitários no notificacao-service
- [ ] Cobertura dos principais cenários (sucesso e erro)
- [ ] Testes passando: `mvn test`

---

## ETAPA 6: Pipeline CI/CD com GitHub Actions
**⏱️ Tempo estimado: 1-2 horas**

### 6.1 Crie o workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # ===== JOB 1: Testes =====
  test:
    name: Build & Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Test pedido-service
        run: |
          cd pedido-service
          mvn -B test

      - name: Test notificacao-service
        run: |
          cd notificacao-service
          mvn -B test

  # ===== JOB 2: Build Docker =====
  build:
    name: Build Docker Images
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Build JARs
        run: |
          cd pedido-service && mvn -B package -DskipTests
          cd ../notificacao-service && mvn -B package -DskipTests

      - name: Build Docker Images
        run: |
          docker build -t pedido-service:${{ github.sha }} ./pedido-service
          docker build -t notificacao-service:${{ github.sha }} ./notificacao-service

      - name: Display images
        run: docker images
```

### ✅ Entregável da Etapa 6
- [ ] Arquivo `.github/workflows/ci-cd.yml` criado
- [ ] Pipeline executando no GitHub Actions
- [ ] Badge de status no README

---

## ETAPA 7: Documentação e Finalização
**⏱️ Tempo estimado: 1 hora**

### 7.1 Crie o README.md

```markdown
# 🚀 FastDelivery - Sistema de Notificações

## 📋 Sobre o Projeto
Sistema de microsserviços para processamento de pedidos e notificações.

## 🛠️ Tecnologias
- Java 17
- Spring Boot 3.x
- Apache Kafka
- PostgreSQL
- Docker
- Prometheus/Grafana
- GitHub Actions

## 🏃 Como Executar

### Pré-requisitos
- Docker e Docker Compose
- Java 17 (para desenvolvimento)
- Maven

### Subir a aplicação
```bash
docker-compose up -d
```

### Endpoints
- Pedido Service: http://localhost:8080
- Notificação Service: http://localhost:8081
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000

## 📡 API - Pedido Service

### Criar Pedido
```bash
curl -X POST http://localhost:8080/api/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "clienteId": "550e8400-e29b-41d4-a716-446655440000",
    "clienteEmail": "cliente@email.com",
    "clienteTelefone": "11999999999",
    "descricao": "Entrega de documentos",
    "valor": 50.00
  }'
```

## 🧪 Executar Testes
```bash
mvn test
```

## 👩‍💻 Autora
Isabela Magalhães
```

### ✅ Entregável da Etapa 7
- [ ] README.md completo
- [ ] Repositório organizado no GitHub
- [ ] Projeto funcionando de ponta a ponta

---

## 📊 Checklist Final de Entrega

### Código
- [ ] Dois projetos Spring Boot funcionando
- [ ] Records usados para DTOs
- [ ] Bean Validation implementado
- [ ] Exceções customizadas com handler global
- [ ] Kafka Producer e Consumer funcionando
- [ ] Strategy Pattern no serviço de notificação

### Infraestrutura
- [ ] Dockerfiles com multi-stage build
- [ ] Docker Compose orquestrando tudo
- [ ] Aplicações conectando ao Kafka
- [ ] Bancos PostgreSQL separados

### Qualidade
- [ ] Mínimo 8 testes unitários
- [ ] Testes passando
- [ ] Pipeline CI/CD no GitHub Actions

### Documentação
- [ ] README completo
- [ ] Código comentado onde necessário

---

## 🎖️ Critérios de Avaliação

| Critério | Peso |
|----------|------|
| Código limpo e organizado | 20% |
| Funcionamento correto | 25% |
| Testes automatizados | 20% |
| Docker e Compose | 15% |
| CI/CD funcionando | 10% |
| Documentação | 10% |

---

## 💡 Dicas

1. **Comece simples**: Faça primeiro o CRUD básico funcionando, depois adicione Kafka
2. **Teste localmente**: Use `docker-compose up -d kafka postgres-pedidos` para testar partes isoladas
3. **Consulte seus guias**: O material que você estudou tem exemplos de código prontos
4. **Commits frequentes**: Commite a cada funcionalidade completa
5. **Não tenha medo de errar**: Erros fazem parte do aprendizado!

---

## 🚀 Bônus (Opcional)

Se terminar tudo e quiser se desafiar mais:

1. **Adicione Retry com DLQ**: Implemente `@RetryableTopic` no consumer
2. **Dashboard Grafana**: Crie um dashboard com métricas customizadas
3. **Testes de Integração**: Adicione testes com `@SpringBootTest`
4. **Deploy na AWS**: Use EC2 ou ECS para fazer deploy real

---

**Boa sorte, Isabela! 🍀**

*"A prática sem teoria é cega, mas a teoria sem prática é estéril."*
