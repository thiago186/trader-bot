# Roadmap

## Fase 0: documentação

Objetivo: definir arquitetura, stack, limites e contratos conceituais.

Tarefas:

- Documentar arquitetura mínima.
- Definir stack técnica.
- Definir CLI como interface principal.
- Definir escopo inicial de market data.
- Definir princípios de separação de responsabilidades.
- Listar entidades persistidas prováveis.

## Fase 1: bootstrap do projeto

Objetivo: criar a base técnica sem lógica de trading real.

Tarefas previstas:

- Inicializar projeto com `uv`.
- Configurar layout Python.
- Adicionar FastAPI.
- Adicionar SQLModel.
- Adicionar Alembic.
- Adicionar Typer.
- Adicionar Rich.
- Configurar mypy.
- Configurar testes.
- Criar health check básico.
- Criar comando CLI básico de status.

## Fase 2: modelos e contratos internos

Objetivo: definir tipos internos antes de integrar exchange.

Tarefas previstas:

- Modelos de market data.
- Modelos de ordem.
- Modelos de fill.
- Modelos de posição.
- Interfaces conceituais para market data, strategy, risk e execution.

## Fase 3: market data Binance Spot

Objetivo: consumir dados reais de mercado sem execução de ordens.

Tarefas previstas:

- Cliente REST inicial.
- Cliente WebSocket inicial, se necessário.
- Normalização de dados.
- Logs de eventos.
- Testes de integração controlados.

## Fase 4: strategy engine

Objetivo: rodar estratégias sem enviar ordens reais.

Tarefas previstas:

- Interface de estratégia.
- Estratégia simples inicial.
- Geração de sinais.
- Simulação em dry run.

## Fase 5: risk engine

Objetivo: bloquear decisões inseguras antes da execução.

Tarefas previstas:

- Limite por ordem.
- Limite por ativo.
- Cooldown.
- Stop.
- Drawdown máximo.
- Slippage máximo.

## Fase 6: execution engine

Objetivo: preparar execução real com controles operacionais.

Tarefas previstas:

- Envio de ordens.
- Cancelamento de ordens.
- Consulta de status.
- Reconciliação de fills.
- Modo dry run.
- Circuit breaker.

## Fase 7: operação pessoal controlada

Objetivo: operar com capital real apenas depois de validação suficiente.

Tarefas previstas:

- Alertas.
- Dashboard/status operacional.
- Logs auditáveis.
- Revisão de risco.
- Limites conservadores.
- Procedimento manual de parada.
