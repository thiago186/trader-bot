# Stack e padrões técnicos

## Stack

- Python
- FastAPI para API interna, health checks e controles operacionais
- SQLModel como ORM
- Alembic para migrations
- mypy para checagem estática de tipos
- Typer para CLI
- Rich para apresentação no terminal
- uv como package manager e executor de comandos

## Gerenciamento com uv

Todos os comandos do projeto devem usar `uv`.

Exemplos esperados no futuro:

```bash
uv init
uv add fastapi sqlmodel alembic typer rich
uv add --group dev mypy pytest ruff
uv run mypy .
uv run pytest
uv run alembic upgrade head
```

Esses comandos são exemplos de intenção. A lista final de dependências será definida quando a implementação começar.

## CLI

A interface principal de interação do usuário com a aplicação será um CLI.

Typer será usado para definir comandos, argumentos, opções e mensagens de ajuda. A escolha combina com a diretriz de tipagem forte do projeto, porque usa type hints como parte natural da definição dos comandos.

Rich será usado para melhorar a legibilidade da saída no terminal.

Usos previstos para Rich:

- Tabelas de status
- Painéis operacionais
- Saída formatada de posições e ordens
- Mensagens de erro legíveis
- Logs locais durante execução manual

Comandos conceituais futuros:

```bash
uv run trader status
uv run trader market-data watch BTCUSDT
uv run trader positions
uv run trader orders
uv run trader run --dry-run
uv run trader pause
```

Esses comandos ainda não representam implementação. Eles servem apenas como direção de produto para o CLI.

## Tipagem

O projeto deve ser fortemente tipado.

Diretrizes:

- Usar type hints em funções, métodos e atributos relevantes.
- Evitar `Any` salvo em bordas externas inevitáveis.
- Criar modelos internos para dados normalizados de mercado, ordens, fills e posições.
- Validar contratos entre módulos com mypy.

## API interna

FastAPI será usada para expor funcionalidades operacionais, health checks e possíveis integrações locais. Ela não será a interface principal do usuário e não deve substituir o núcleo do robô.

Possíveis endpoints futuros:

- Health check
- Status do robô
- Posições abertas
- Ordens recentes
- PnL
- Ativar ou pausar execução
- Consultar configuração efetiva

## Persistência

SQLModel será usado para modelagem e acesso ao banco.

Alembic será usado para evolução de schema.

Entidades prováveis:

- Configuração
- Símbolo negociável
- Ordem
- Fill
- Posição
- Snapshot de saldo
- Evento operacional
- Registro de PnL

## Qualidade

Ferramentas previstas:

- mypy para tipos
- pytest para testes
- ruff para lint e formatação, caso adotado

Critérios mínimos antes de rodar com dinheiro real:

- Testes unitários para strategy engine e risk engine.
- Testes de integração para conectores de exchange em modo controlado.
- Logs suficientes para reconstruir decisões.
- Modo paper trading ou dry run antes de execução real.
