# Plano de implementação

## Objetivo

Implementar um robô trader pessoal, local e modular, começando por Binance Spot, com foco em separação de responsabilidades, estado auditável e reconciliação periódica entre estado local e exchange.

O sistema deve ser construído para permitir troca de estratégia sem reescrever risco, execução, storage ou conectores externos.

## Fluxo operacional alvo

```text
CLI
    -> carrega configuração
    -> inicializa storage
    -> abre market data stream
    -> abre user/account stream
    -> inicia strategy engine

Market data event
    -> Strategy
    -> Signal
    -> Risk engine
    -> Order intent aprovado ou bloqueado
    -> Execution engine
    -> Exchange REST order
    -> User/account event
    -> Storage
    -> Portfolio local atualizado

Reconciliation loop
    -> consulta exchange via REST
    -> compara ordens, fills, saldos e posições
    -> corrige ou alerta divergências
```

## Princípios de arquitetura

- Strategy não envia ordem.
- Strategy não consulta saldo diretamente na exchange.
- Risk engine não fala com Binance.
- Execution engine não decide se uma operação faz sentido.
- Portfolio representa estado local derivado de fatos persistidos.
- Storage registra fatos operacionais antes de depender apenas de memória.
- Reconciliação é parte central do produto, não uma rotina auxiliar opcional.
- Toda decisão relevante deve ser rastreável por logs e eventos persistidos.

## Contratos centrais

### Signal

Representa o desejo da estratégia.

Exemplo conceitual:

```text
Signal
    symbol
    side
    desired_quantity
    reason
    strategy_id
    created_at
    metadata
```

O signal deve explicar a intenção, mas não deve conter detalhes específicos de exchange como tipo exato de endpoint, payload Binance ou regras de rate limit.

### Order intent

Representa uma intenção validada pelo risk engine.

Exemplo conceitual:

```text
OrderIntent
    symbol
    side
    quantity
    order_type
    limit_price
    max_slippage
    time_in_force
    risk_decision_id
    source_signal_id
```

O risk engine pode aprovar, bloquear ou reduzir o tamanho do signal antes de gerar o intent.

### Order record

Representa a ordem local.

Exemplo conceitual:

```text
Order
    local_order_id
    exchange_order_id
    client_order_id
    symbol
    side
    quantity
    status
    submitted_at
    updated_at
```

O `client_order_id` deve ser tratado como chave importante de idempotência para evitar duplicidade em retry, queda de rede ou resposta perdida.

### Fill

Representa execução parcial ou total.

Exemplo conceitual:

```text
Fill
    exchange_trade_id
    order_id
    symbol
    side
    quantity
    price
    fee
    fee_asset
    executed_at
```

Fills devem ser deduplicados por identificador da exchange quando disponível.

### Portfolio

Representa o estado local calculado.

Exemplo conceitual:

```text
Portfolio
    balances
    positions
    open_orders
    realized_pnl
    unrealized_pnl
    last_reconciled_at
```

O portfolio não deve ser atualizado por suposição otimista. Ele deve ser derivado de ordens, fills, saldos confirmados e eventos de conta.

## Reconciliação

Reconciliação deve existir desde antes da primeira operação real.

### Problemas que a reconciliação precisa cobrir

- Ordem enviada, mas resposta REST perdida.
- Ordem aceita pela exchange, mas não persistida corretamente.
- Fill parcial ignorado pelo user stream.
- Evento duplicado após reconexão.
- Evento fora de ordem.
- Ordem local marcada como aberta, mas finalizada na exchange.
- Posição local diferente da posição calculada pela exchange.
- Saldo local diferente do saldo real.
- Fee registrada com asset incorreto ou ausente.

### Fontes de verdade

O estado local é a fonte de trabalho do robô durante o loop normal.

A exchange é a fonte de verdade para reconciliação.

Quando houver divergência, o sistema deve:

- registrar o evento de divergência;
- classificar severidade;
- corrigir automaticamente apenas casos seguros;
- pausar execução quando a divergência puder gerar risco financeiro;
- expor a divergência no CLI.

### Rotina mínima de reconciliação

```text
1. Buscar ordens abertas na exchange.
2. Buscar ordens recentes por símbolo monitorado.
3. Buscar trades/fills recentes.
4. Buscar saldos atuais.
5. Comparar com ordens, fills e portfolio locais.
6. Deduplicar eventos por IDs externos.
7. Inserir fatos ausentes.
8. Atualizar statuses locais.
9. Recalcular portfolio.
10. Emitir alerta se a divergência não for corrigível automaticamente.
```

### Frequência inicial

Durante desenvolvimento e dry run:

- reconciliação ao iniciar o app;
- reconciliação periódica a cada poucos minutos;
- reconciliação manual via CLI.

Antes de dinheiro real, a frequência deve ser definida junto com limites de rate limit da Binance e com os símbolos monitorados.

## Organização de diretórios

Layout inicial recomendado:

```text
trader-bot/
    docs/
        architecture.md
        implementation-plan.md
        market-data.md
        roadmap.md
        technical-stack.md
    src/
        trader/
            __init__.py
            cli/
                __init__.py
                app.py
                commands/
                    __init__.py
                    market_data.py
                    orders.py
                    positions.py
                    reconcile.py
                    run.py
                    status.py
            config/
                __init__.py
                settings.py
            core/
                __init__.py
                events.py
                ids.py
                time.py
            market_data/
                __init__.py
                models.py
                provider.py
                binance_spot.py
            account/
                __init__.py
                events.py
                provider.py
                binance_spot.py
            strategy/
                __init__.py
                base.py
                signals.py
                simple.py
            risk/
                __init__.py
                engine.py
                models.py
                rules.py
            execution/
                __init__.py
                engine.py
                models.py
                provider.py
                binance_spot.py
                dry_run.py
            portfolio/
                __init__.py
                models.py
                service.py
            reconciliation/
                __init__.py
                service.py
                models.py
            storage/
                __init__.py
                database.py
                models.py
                repositories.py
            api/
                __init__.py
                app.py
                routes/
                    __init__.py
                    health.py
            ops/
                __init__.py
                logging.py
                alerts.py
                circuit_breaker.py
    tests/
        unit/
        integration/
    migrations/
        versions/
    pyproject.toml
    README.md
```

### Responsabilidades por diretório

- `src/trader/cli`: comandos Typer e apresentação Rich. Não deve conter regra de negócio.
- `src/trader/config`: carregamento e validação de configuração.
- `src/trader/core`: tipos e utilitários compartilhados que não pertencem a um domínio específico.
- `src/trader/market_data`: contratos e implementações de dados públicos de mercado.
- `src/trader/account`: stream privado de conta, ordens, fills e saldos.
- `src/trader/strategy`: estratégias e emissão de `Signal`.
- `src/trader/risk`: validação de signals e criação ou bloqueio de `OrderIntent`.
- `src/trader/execution`: envio, cancelamento, consulta e dry run de ordens.
- `src/trader/portfolio`: cálculo do estado local a partir de fatos persistidos.
- `src/trader/reconciliation`: comparação entre estado local e exchange.
- `src/trader/storage`: banco, modelos persistidos e repositórios.
- `src/trader/api`: API interna FastAPI para health checks e controles operacionais.
- `src/trader/ops`: logging, alertas, circuit breaker e utilidades operacionais.
- `tests/unit`: testes rápidos de regras, modelos e serviços isolados.
- `tests/integration`: testes com banco, conectores controlados e fluxos entre módulos.
- `migrations`: migrations Alembic.

### Regras de dependência

- `strategy` pode depender de modelos internos e eventos normalizados, mas não de `execution`, `storage` ou Binance.
- `risk` pode ler estado de portfolio e configuração, mas não envia ordens.
- `execution` pode depender de `storage` para registrar tentativas e de providers de exchange, mas não decide estratégia.
- `portfolio` deve ser calculado a partir de fatos persistidos e eventos confirmados.
- `reconciliation` pode consultar exchange e storage, mas deve produzir relatório e ações explícitas.
- `cli` chama serviços e casos de uso; não implementa regra de trading.
- Implementações Binance devem ficar nas bordas: `market_data/binance_spot.py`, `account/binance_spot.py` e `execution/binance_spot.py`.

## Fases de implementação

### Fase 1: bootstrap executável

Objetivo: ter aplicação inicial rodando via CLI, sem trading real.

Entregas:

- projeto Python com `uv`;
- layout de pacotes seguindo a organização de diretórios documentada;
- comando `trader status`;
- carregamento de configuração;
- logging estruturado básico;
- health check FastAPI simples;
- testes básicos e mypy configurado.

Critério de pronto:

- `uv run trader status` executa;
- `uv run pytest` passa;
- `uv run mypy .` passa ou tem escopo inicial documentado.

### Fase 2: modelos e storage

Objetivo: definir estado persistido antes de integrar execução real.

Entregas:

- modelos internos de market data, signal, order intent, order, fill e portfolio;
- tabelas SQLModel;
- migrations Alembic;
- repositórios ou serviços de storage;
- IDs locais e IDs externos bem definidos;
- persistência de eventos operacionais.

Critério de pronto:

- criar e consultar ordens, fills e eventos locais;
- recalcular portfolio local a partir de fatos persistidos;
- testes cobrindo deduplicação básica de fills e eventos.

### Fase 3: market data stream

Objetivo: consumir dados reais de mercado sem ordens.

Entregas:

- contrato `MarketDataProvider`;
- implementação Binance Spot inicial;
- normalização de trades, ticker ou candles;
- reconexão controlada;
- CLI para acompanhar market data;
- logs de eventos relevantes.

Critério de pronto:

- strategy consegue receber eventos normalizados;
- detalhes da Binance não vazam para a strategy;
- desconexão e reconexão não quebram o processo principal.

### Fase 4: user/account stream

Objetivo: receber eventos privados de conta e ordem.

Entregas:

- contrato `AccountEventProvider`;
- stream privado Binance Spot;
- normalização de eventos de ordem, fill e saldo;
- persistência idempotente;
- atualização do estado local via eventos confirmados.

Critério de pronto:

- evento duplicado não duplica fill;
- fill parcial atualiza ordem corretamente;
- status de ordem muda por evento privado, não por suposição da execução REST.

### Fase 5: strategy engine

Objetivo: gerar signals sem acoplar com execução.

Entregas:

- interface de strategy;
- estratégia simples inicial;
- emissão de `Signal`;
- registro do motivo do signal;
- modo dry run conectado ao fluxo completo até risk.

Critério de pronto:

- uma estratégia pode ser trocada sem alterar execution;
- signals são persistidos ou logados com rastreabilidade suficiente.

### Fase 6: risk engine

Objetivo: transformar signal em intent seguro ou bloquear.

Entregas:

- limites por ordem;
- limites por ativo;
- cooldown;
- validação de saldo disponível;
- limite de slippage;
- circuit breaker;
- objeto de decisão de risco.

Critério de pronto:

- todo signal recebe decisão explícita;
- bloqueios são visíveis no CLI e nos logs;
- nenhum order intent é criado sem decisão de risco.

### Fase 7: execution engine

Objetivo: enviar e acompanhar ordens com idempotência.

Entregas:

- contrato `ExecutionProvider`;
- implementação Binance Spot REST;
- geração de `client_order_id`;
- envio de ordem;
- cancelamento;
- consulta de status;
- tratamento de retry sem duplicar ordem;
- modo dry run e paper trading.

Critério de pronto:

- resposta REST perdida não causa envio duplicado sem checagem;
- ordem local fica pendente até confirmação por REST, user stream ou reconciliação;
- falhas transitórias são registradas e reprocessáveis.

### Fase 8: reconciliação

Objetivo: garantir que estado local e exchange convergem.

Entregas:

- serviço de reconciliação;
- execução no startup;
- execução periódica;
- comando CLI manual;
- comparação de ordens abertas;
- comparação de fills;
- comparação de saldos;
- correção segura de fatos ausentes;
- alertas para divergências críticas.

Critério de pronto:

- simular fill ausente localmente e recuperar pela reconciliação;
- simular ordem local aberta mas fechada na exchange e corrigir status;
- divergência de saldo pausa execução;
- relatório de reconciliação aparece no CLI.

### Fase 9: operação controlada

Objetivo: preparar uso pessoal com risco conservador.

Entregas:

- configuração segura por ambiente;
- chave de API com permissões mínimas;
- limites conservadores obrigatórios;
- procedimento de parada manual;
- dashboard/status no CLI;
- logs auditáveis de decisões, ordens e reconciliações;
- checklist pré-operação real.

Critério de pronto:

- dry run roda por período definido sem divergências não explicadas;
- paper trading ou simulação equivalente validada;
- primeira execução real exige confirmação explícita;
- bot pausa sozinho em erro crítico de estado.

## Comandos CLI previstos

```bash
uv run trader status
uv run trader run --dry-run
uv run trader run --paper
uv run trader run --live
uv run trader pause
uv run trader resume
uv run trader market-data watch BTCUSDT
uv run trader orders
uv run trader positions
uv run trader reconcile
uv run trader reconciliation-report
```

## Ordem recomendada de implementação

1. CLI mínimo e configuração.
2. Storage e modelos persistidos.
3. Market data público.
4. Strategy simples gerando signal.
5. Risk engine bloqueando por padrão.
6. Execution dry run.
7. User/account stream.
8. Execution REST real com flags de segurança.
9. Reconciliação robusta.
10. Operação controlada com limites reais.

## Decisões pendentes

- Formato final da configuração.
- Banco inicial: SQLite local ou Postgres.
- Estratégia inicial de teste.
- Símbolos permitidos no início.
- Frequência de reconciliação.
- Política de correção automática versus pausa manual.
- Modelo exato de paper trading.
- Regras mínimas antes de habilitar `--live`.

## Regras antes de dinheiro real

- Nunca operar sem reconciliação no startup.
- Nunca operar sem user/account stream ativo ou fallback explícito por polling.
- Nunca operar se houver divergência crítica não resolvida.
- Nunca operar sem `client_order_id` idempotente.
- Nunca operar sem limites máximos por ordem e por ativo.
- Nunca operar se testes de risk engine estiverem falhando.
- Nunca operar com chaves de API com permissão além do necessário.
