# Arquitetura

## Visão geral

O robô será desenhado como um sistema modular, com responsabilidades separadas e contratos internos bem definidos. A primeira integração externa será com Binance Spot, mas os componentes centrais não devem depender diretamente da Binance.

Fluxo lógico inicial:

```text
Market data
    -> Strategy engine
    -> Risk engine
    -> Execution engine
    -> State/storage
    -> Ops/observability
```

A interação principal do usuário com o sistema será via CLI. Esse CLI deve acionar casos de uso internos e consultar estado persistido, sem concentrar regra de negócio.

## Componentes

### Market data

Responsável por receber, normalizar e disponibilizar dados de mercado.

Tipos de dados previstos:

- Preço atual
- Trades
- Order book
- Candles
- Eventos de mercado em tempo real

A implementação inicial usará APIs da Binance Spot. Mesmo assim, o restante do sistema deve consumir interfaces internas, não objetos específicos da Binance.

### Strategy engine

Responsável por decidir a intenção operacional:

- Comprar
- Vender
- Não fazer nada

A estratégia deve receber dados normalizados e produzir sinais ou intenções de ordem. Ela não deve enviar ordens diretamente nem decidir sozinha se uma ordem é segura.

### Risk engine

Responsável por validar se uma intenção operacional pode virar ordem.

Controles previstos:

- Tamanho máximo por ordem
- Exposição máxima por ativo
- Stop
- Drawdown máximo
- Cooldown entre operações
- Slippage máximo aceitável
- Circuit breaker operacional

Nenhuma ordem deve chegar ao execution engine sem passar pelo risk engine.

### Execution engine

Responsável por enviar, cancelar e acompanhar ordens na exchange.

Responsabilidades previstas:

- Enviar ordens
- Cancelar ordens
- Consultar status de ordens
- Reconciliar fills
- Tratar falhas transitórias
- Respeitar rate limits

Assim como no market data, a primeira implementação será para Binance Spot, mas com abstração suficiente para troca futura de exchange.

### State/storage

Responsável por persistir o estado operacional.

Dados previstos:

- Configurações
- Posições
- Ordens
- Fills
- PnL
- Eventos relevantes
- Logs estruturados quando aplicável

O banco será acessado via SQLModel, com migrations gerenciadas por Alembic.

### Ops/observability

Responsável por visibilidade operacional e resiliência.

Itens previstos:

- Logs estruturados
- Alertas
- Métricas
- Retry controlado
- Reconexão
- Circuit breaker
- Registro de erros críticos

### CLI

Responsável pela interação direta do usuário com a aplicação.

Responsabilidades previstas:

- Consultar status operacional
- Iniciar execução em modo dry run
- Pausar ou retomar execução
- Inspecionar posições
- Inspecionar ordens
- Acompanhar market data em tempo real
- Exibir erros e alertas locais de forma legível

O CLI será implementado com Typer e usará Rich para apresentação no terminal. Ele deve orquestrar comandos e exibir resultados, mas não deve conter lógica de estratégia, risco, execução ou persistência.

## Separação de responsabilidades

O projeto deve evitar acoplamento entre estratégia, exchange e persistência.

Regras iniciais:

- Estratégias não importam clientes Binance.
- Risk engine não envia ordens.
- Execution engine não decide estratégia.
- Market data normaliza dados externos antes de publicá-los internamente.
- Storage persiste fatos e estado, mas não decide comportamento operacional.
- CLI não contém regra de negócio; ele chama casos de uso e apresenta resultados.
