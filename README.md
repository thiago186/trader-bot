# Trader Bot

Projeto pessoal para construção gradual de um robô trader automatizado.

Neste momento o foco do projeto é documentação, desenho de arquitetura e definição de contratos internos. A implementação será iniciada somente depois que os limites funcionais, técnicos e operacionais estiverem claros.

## Objetivo

Criar um sistema modular para operar mercados spot, começando pela Binance Spot, com capacidade de evoluir para outras exchanges sem acoplamento forte na estratégia, no risco ou na execução.

## Stack definida

- Python
- FastAPI
- SQLModel
- Alembic
- mypy
- Typer
- Rich
- uv

Todos os comandos de gerenciamento do projeto devem ser feitos via `uv`, incluindo criação de ambiente, instalação, remoção e execução de dependências.

A interação principal do usuário com a aplicação será via CLI. A API com FastAPI será usada para superfície operacional interna, health checks e possíveis integrações locais.

## Arquitetura mínima

O sistema será organizado em seis áreas principais:

- Market data
- Strategy engine
- Risk engine
- Execution engine
- State/storage
- Ops/observability

Detalhes estão em [docs/architecture.md](docs/architecture.md).

## Diretrizes iniciais

- Uso estritamente pessoal.
- Código fortemente tipado.
- Separação clara de responsabilidades.
- Estratégias não devem depender diretamente de APIs específicas de exchange.
- A integração inicial de market data será com Binance Spot.
- O desenho deve permitir evolução futura para outras fontes de dados e exchanges.

## Documentação

- [Arquitetura](docs/architecture.md)
- [Stack e padrões técnicos](docs/technical-stack.md)
- [Market data](docs/market-data.md)
- [Roadmap](docs/roadmap.md)
