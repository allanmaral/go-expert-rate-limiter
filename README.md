# Desafio Go - Rate Limiting

Bem-vindo ao Desafio de Rate Limiting da Pós-Graduação Go Expert! Este projeto consiste na implementação de um pacote com a definição de um rate limiter que bloqueia alto volumes de requisições de acordo com o IP de origem ou um header API_KEY.

## Pré-requisitos

Antes de começar, certifique-se de ter instalado os seguintes requisitos:

- [Go SDK](https://golang.org/dl/): Linguagem de programação Go.
- [Docker](https://docs.docker.com/get-docker/): Plataforma de conteinerização.
- [Make](https://www.gnu.org/software/make/): Utilizado para automatização de tarefas.

## Executando o Projeto

1. Clone este repositório em sua máquina local:

```bash
git clone https://github.com/allanmaral/go-expert-rate-limiter.git
```

2. Navegue até o diretório do projeto:

```bash
cd go-expert-rate-limiter
```

3. Execute o seguinte comando para subir a instância do Redis:

```bash
docker compose up -d
```

4. Instale as dependências do projeto:

```bash
go mod tidy
```

5. Finalmente, suba o serviço executando:

```bash
make run
```

## Acesso aos Serviços

Após subir o serviço, você poderá acessar a a api no endereço [http://localhost:8000](http://localhost:8000)

## Documentação da API REST

A documentação das rotas do servidor HTTP está disponível no arquivo `./api/api.http`.

## Estrutura do projeto

A maior parte da implementação do projeto foi feito no pacote `limiter` na pasta `pkg/limiter`. O rate limiter foi quebrado em alguns componentes:

- **RateLimiter**: Objeto que implementa o método `Handler` que é usado como middleware. Ele é o responsável por extrair o `IP` e a `API_KEY` da resição e bloquear a execução em caso de excesso.
- **limitCounter**: Implementação interna do contador de requisições, ele expõe apenas um método `Increment(ip token string)` responsável por incrementar o contador e trazer informações sobre os limites. O `limitCounter` é responsável por consultar os limites e mandar as operações na `CounterStore`.
- **CounterStore**: Uma interface que expõe os métodos de `Increment(key string)` e `Set(key string, value int64, timeout time.Duration)` que devem ser persistidos.
- **RedisStore**: A implementação da `CounterStore` que persiste os dados no Redis.
- **localStore**: Implementação do `CounterStore` que persiste os dados em memória.

### Testes

O pacote `limiter` foi desenvolvido com testes de unidade e integração. Os testes de unidade rodam completamente isolado, mas os testes de integração dependem de uma instância do Redis para rodar. Para simplificar o setup, foi utilizado o pacote `testcontainers` para criar e destruir os containers de forma automática durante os testes.

Para rodar os testes da aplicação basta rodar o comando:

```bash
make test
```

## Modo de Uso

Para usar o `limiter`, basta criar um novo middleware e aplicar no mux ou router da aplicação:

```go
rateLimit := limiter.RateLimit(
limiter.WithRedisStore(rc), // Opcional, utiliza o redis client para persistir os dados das requisições
limiter.WithDefaultLimit(config.DefaultLimit.RequestsPerSecond, config.DefaultLimit.TimeBlocked),
limiter.WithIPLimits(config.IPLimits),
limiter.WithTokenLimits(config.TokenLimits),
)

router := http.NewServeMux()
// configura as rotas...

mux := rateLimit(router) // Aplica o rate limiter ao router/mux

http.ListenAndServe(":8080", mux)
```
