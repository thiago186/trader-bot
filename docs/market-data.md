# Market data

## Escopo inicial

O market data começará usando APIs da Binance Spot.

O objetivo é receber dados de mercado, normalizar esses dados e disponibilizá-los para o restante do sistema sem expor detalhes específicos da Binance.

## Fontes de dados previstas

Binance Spot oferece dados via REST e WebSocket. A escolha exata será feita durante a implementação, mas o desenho deve acomodar ambos.

Tipos de dados previstos:

- Ticker/preço atual
- Trades recentes
- Candles
- Order book
- Streams em tempo real

## Contrato interno

Os dados vindos da exchange devem ser convertidos para modelos internos antes de serem usados por strategy engine ou risk engine.

Exemplos de conceitos internos:

- `MarketSymbol`
- `Trade`
- `Candle`
- `OrderBookSnapshot`
- `OrderBookDelta`
- `MarketDataEvent`

Esses nomes são apenas conceituais neste momento. A implementação concreta será definida quando o código começar.

## Separação por provedor

A arquitetura deve permitir múltiplos provedores de market data.

Exemplo conceitual:

```text
MarketDataProvider
    BinanceSpotMarketDataProvider
    FutureExchangeMarketDataProvider
```

O restante do sistema deve depender do contrato `MarketDataProvider`, não da implementação Binance.

## Cuidados operacionais

Pontos que precisarão ser considerados:

- Rate limits da Binance
- Reconexão de WebSocket
- Eventos duplicados
- Eventos fora de ordem
- Latência
- Diferença entre snapshot e delta de order book
- Persistência opcional de eventos críticos
- Fallback entre REST e WebSocket quando fizer sentido

## Fora do escopo por enquanto

- Operação em futures
- Margin trading
- Múltiplas exchanges
- Arbitragem
- Alta frequência
- Execução real de ordens

