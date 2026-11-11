# Plano Técnico: Weather App

## Architecture

Aplicação web React executada no navegador, sem backend próprio. A arquitetura tem quatro camadas com dependências direcionadas para baixo: apresentação recebe estado e callbacks; o hook orquestra o fluxo; o serviço acessa o Open-Meteo; funções puras fazem conversões e formatações. Tipos compartilhados ficam em `types/` e não dependem de React ou da API.

```text
App.tsx
  +-- useWeather (estado e orquestração)
  |     +-- weatherService (HTTP e normalização) ---> Open-Meteo
  +-- components (props e callbacks)
        +-- lib (conversão, formatação e códigos WMO)
```

- **components:** renderizam dados e controles a partir de props, emitem callbacks de interação e representam loading, vazio e erro. Não conhecem URLs, `fetch` nem DTOs do provedor.
- **hooks:** `useWeather` coordena busca, seleção, consulta, unidade e estados; chama o contrato de serviço e descarta respostas de cidades que já não estão selecionadas.
- **services:** constroem requisições HTTP, aplicam timeout, verificam status/resposta e convertem a API para os modelos internos; não conhecem componentes nem exibem mensagens.
- **lib:** funções puras para conversão de temperatura, formatação de datas/números e tradução de códigos WMO; sem acesso a rede, DOM ou estado React.
- **types:** contratos internos compartilhados por hook, serviço e componentes; DTOs específicos do provedor não atravessam até a apresentação.
- Nenhuma camada persiste dados. O estado vive em memória; não há conta, armazenamento local, cache persistente ou geolocalização.

Essa direção permite substituir/mockar uma camada sem executar as demais: testar `lib` por entrada/saída, o serviço com `fetch` simulado, o hook com serviço mockado e os componentes por comportamento acessível. Não introduzir container de DI ou abstração genérica de repositório para este único provedor.

## Tech Stack

| Área | Decisão | Motivo |
| --- | --- | --- |
| Linguagem | TypeScript strict | Tipar os contratos da API, o modelo interno e os estados de erro. |
| Aplicação | React + Vite | Alinhamento com a stack e estrutura já adotadas no repositório. |
| UI | Tailwind CSS | Seguir o sistema de estilos existente. |
| Testes unitários/componentes | Vitest + Testing Library | Cobrir funções puras e fluxos observáveis da interface. |
| Testes E2E | Playwright | Validar jornadas completas em viewport móvel e desktop. |
| Qualidade | Biome | Lint e formatação conforme configuração do repositório. |
| Pacotes | pnpm | Gerenciador definido pelo projeto. |
| Dados | Open-Meteo Geocoding e Forecast APIs | Fonte aprovada no discovery; sem API key. |

Não adicionar biblioteca de estado, cliente HTTP ou cache na primeira versão. `fetch` e estado React são suficientes para este fluxo. O uso comercial, atribuição, cobertura e limites do Open-Meteo são gates antes do lançamento.

## Project Structure

Estrutura proposta, compatível com as áreas existentes no projeto:

```text
src/
  App.tsx                    # composição da tela e ligação do hook aos componentes
  main.tsx                   # ponto de entrada React existente
  components/
    SearchBar.tsx            # entrada e resultados de cidades
    CurrentWeather.tsx       # condições atuais
    ForecastList.tsx         # lista dos cinco dias
    ForecastCard.tsx         # apresentação de um dia
    UnitToggle.tsx           # seleção Celsius/Fahrenheit
    states/
      LoadingState.tsx
      EmptyState.tsx
      ErrorState.tsx
  hooks/
    useWeather.ts            # orquestração e estado volátil
  services/
    weatherService.ts        # geocoding, forecast, timeout e normalização
  lib/
    temperature.ts           # conversão Celsius/Fahrenheit
    format.ts                # datas, horários, números e unidades
    weatherCodes.ts          # códigos WMO para rótulos pt-BR
  types/
    weather.ts               # City, CurrentWeather, ForecastDay, WeatherData

src/styles/
  index.css                  # estilos globais e Tailwind, já existente

tests/
  unit/                      # lib, serviço, hook e componentes
  e2e/                       # jornadas do usuário via Playwright
```

Não criar subpastas por domínio nem arquivos de abstração adicionais enquanto houver apenas uma tela principal e um provedor. Se a normalização do provedor crescer, separá-la em funções puras de `lib/` sem mudar o contrato consumido pelo hook.

## Data Model

Temperaturas ficam em Celsius no modelo interno; conversão para Fahrenheit ocorre apenas na apresentação. Campos meteorológicos opcionais usam `null`, nunca `0` como substituto de dado ausente.

```ts
export type Unit = 'celsius' | 'fahrenheit';

export interface City {
  id: number; // Identificador da localidade retornado pelo geocoding.
  name: string; // Nome da cidade retornado pelo geocoding.
  region: string | null; // Região administrativa, se fornecida.
  country: string | null; // Nome do país, se fornecido.
  countryCode: string | null; // Código ISO do país, se fornecido.
  latitude: number; // Latitude usada para consultar a previsão.
  longitude: number; // Longitude usada para consultar a previsão.
  timezone: string | null; // Fuso IANA da cidade; necessário para as datas locais.
}

export interface CitySearchResult {
  cities: City[];
}

export interface CurrentWeather {
  temperatureC: number | null; // Temperatura atual; unidade canônica Celsius.
  apparentTemperatureC: number | null; // Sensação térmica; Celsius.
  weatherCode: number | null; // Código WMO da condição meteorológica.
  relativeHumidityPercent: number | null; // Umidade relativa em porcentagem.
  windSpeedKmh: number | null; // Velocidade do vento em km/h.
  observedAt: string | null; // Horário da observação, normalizado para ISO 8601.
}

export interface ForecastDay {
  date: string; // Data local da cidade no formato YYYY-MM-DD.
  weatherCode: number | null; // Código WMO da condição do dia.
  minTemperatureC: number | null; // Temperatura mínima em Celsius.
  maxTemperatureC: number | null; // Temperatura máxima em Celsius.
  precipitationProbabilityMaxPercent: number | null; // Probabilidade máxima de precipitação em %.
  precipitationSumMm: number | null; // Precipitação total do dia em mm.
}

export interface WeatherData {
  city: City; // Cidade à qual todos os dados abaixo pertencem.
  current: CurrentWeather; // Condições atuais retornadas pela API.
  forecast: ForecastDay[]; // Cinco dias locais, em ordem cronológica.
  fetchedAt: string; // Momento da última resposta bem-sucedida, ISO 8601.
}

export type WeatherErrorKind =
  | 'validation'
  | 'not-found'
  | 'timeout'
  | 'rate-limit'
  | 'network'
  | 'provider'
  | 'invalid-response';

export interface WeatherError {
  kind: WeatherErrorKind;
  retryAt: string | null;
}
```

Contrato de serviço sugerido:

```ts
export interface WeatherService {
  searchCities(query: string): Promise<CitySearchResult>;
  getWeather(city: City): Promise<WeatherData>;
}
```

Regras do modelo:

- `City.id` deve identificar de forma estável o resultado geocodificado (coordenadas mais código de país quando disponível).
- `timezone` é o identificador IANA retornado pelo geocoding. Se ausente, não solicitar a previsão; ainda pode ser exibido o clima atual.
- `daily` tem exatamente cinco registros, ordenados da data local de hoje aos quatro dias seguintes.
- `observedAt` e `fetchedAt` são timestamps ISO 8601 normalizados; `date` é `YYYY-MM-DD` no fuso local da cidade.
- O estado de ausência de dados é derivado dos campos `null`; não é um DTO de sucesso completo nem um valor meteorológico inventado.

## Data Flow

1. A interface remove espaços externos do texto. Campo vazio mostra validação e não chama a API.
2. `useWeather` inicia busca e chama `searchCities` com o texto original preservado, inclusive acentos e pontuação suportada.
3. O serviço consulta geocoding, normaliza até cinco resultados em ordem retornada e devolve `CitySearchResult`. Lista vazia termina em estado vazio; não dispara Forecast API.
4. A seleção explícita de uma cidade chama `getWeather(city)`. Busca e previsão têm estados de carregamento separados.
5. O serviço solicita clima atual e cinco dias diários em unidades métricas, usando o fuso IANA da cidade; normaliza arrays paralelos de resposta para `WeatherData`.
6. O hook só publica a resposta se o `city.id` ainda corresponder à seleção atual. Respostas atrasadas de outra cidade são ignoradas.
7. Componentes exibem campos válidos, indicadores de dados indisponíveis e timestamps. `UnitToggle` muda somente a unidade de apresentação; os dados no modelo permanecem em Celsius.

## External APIs

### Geocoding

- **URL:** `https://geocoding-api.open-meteo.com/v1/search`
- **Método:** `GET`; não requer API key.
- **Parâmetros:** `name` (nome da cidade, opcionalmente com região/país), `count=5`, `language=pt` e `format=json`.
- **Exemplo de chamada:** `https://geocoding-api.open-meteo.com/v1/search?name=Recife&count=5&language=pt&format=json`
- **Resposta resumida:**

```json
{
  "results": [
    {
      "id": 3390760,
      "name": "Recife",
      "latitude": -8.05389,
      "longitude": -34.88111,
      "timezone": "America/Recife",
      "country_code": "BR",
      "country": "Brasil",
      "admin1": "Pernambuco"
    }
  ]
}
```

- **Mapeamento para `City`:** `id -> id`, `name -> name`, `admin1 -> region`, `country -> country`, `country_code -> countryCode`, `latitude -> latitude`, `longitude -> longitude`, `timezone -> timezone`. Campos opcionais não retornados tornam-se `null`.
- Resposta HTTP bem-sucedida sem `results`, ou com `results: []`, é busca sem correspondência; erro HTTP, timeout ou falha de rede é erro de serviço. Usar até cinco resultados na ordem retornada e exigir seleção explícita.

### Forecast

- **URL:** `https://api.open-meteo.com/v1/forecast`
- **Método:** `GET`; não requer API key.
- **Parâmetros:** `latitude` e `longitude` da `City`; `timezone={City.timezone}`; `forecast_days=5`; `current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m`; `daily=weather_code,temperature_2m_min,temperature_2m_max,precipitation_probability_max,precipitation_sum`; `temperature_unit=celsius`; `wind_speed_unit=kmh`; `precipitation_unit=mm`.
- **Exemplo de chamada:** `https://api.open-meteo.com/v1/forecast?latitude=-8.05389&longitude=-34.88111&timezone=America%2FRecife&forecast_days=5&current=temperature_2m%2Crelative_humidity_2m%2Capparent_temperature%2Cweather_code%2Cwind_speed_10m&daily=weather_code%2Ctemperature_2m_min%2Ctemperature_2m_max%2Cprecipitation_probability_max%2Cprecipitation_sum&temperature_unit=celsius&wind_speed_unit=kmh&precipitation_unit=mm`
- **Resposta resumida:**

```json
{
  "timezone": "America/Recife",
  "current_units": {
    "time": "iso8601",
    "temperature_2m": "°C",
    "relative_humidity_2m": "%",
    "apparent_temperature": "°C",
    "weather_code": "wmo code",
    "wind_speed_10m": "km/h"
  },
  "current": {
    "time": "2026-11-11T09:00",
    "temperature_2m": 28,
    "relative_humidity_2m": 70,
    "apparent_temperature": 29,
    "weather_code": 0,
    "wind_speed_10m": 12
  },
  "daily_units": {
    "time": "iso8601",
    "weather_code": "wmo code",
    "temperature_2m_min": "°C",
    "temperature_2m_max": "°C",
    "precipitation_probability_max": "%",
    "precipitation_sum": "mm"
  },
  "daily": {
    "time": ["2026-11-11", "2026-11-12"],
    "weather_code": [0, 3],
    "temperature_2m_min": [23, 24],
    "temperature_2m_max": [30, 29],
    "precipitation_probability_max": [10, 40],
    "precipitation_sum": [0, 2.4]
  }
}
```

O payload acima é abreviado: arrays `daily` completos devem conter cinco posições; os valores no mesmo índice formam um dia.

**Mapeamento para o modelo interno:**

- `current.temperature_2m -> CurrentWeather.temperatureC`.
- `current.apparent_temperature -> CurrentWeather.apparentTemperatureC`.
- `current.weather_code -> CurrentWeather.weatherCode`.
- `current.relative_humidity_2m -> CurrentWeather.relativeHumidityPercent`.
- `current.wind_speed_10m -> CurrentWeather.windSpeedKmh`.
- `current.time -> CurrentWeather.observedAt`, interpretado no fuso retornado e normalizado para ISO 8601 com offset.
- Para cada índice `i`, `daily.time[i] -> ForecastDay.date`; os campos homólogos de `weather_code`, mín./máx., probabilidade máxima e total de precipitação são mapeados para os campos correspondentes de `ForecastDay`.
- `WeatherData.city` é a `City` selecionada; `WeatherData.current` recebe o objeto normalizado; `WeatherData.forecast` recebe os cinco dias mapeados; `WeatherData.fetchedAt` é o instante local da resposta bem-sucedida em ISO 8601 UTC.
- `null`, resposta inválida ou array com menos de cinco itens é tratado como resposta parcial/inválida segundo Error Handling; nunca substituir ausências por zero.
- Todas as temperaturas do modelo ficam em Celsius. `Unit` não é enviado como preferência à API: alternar para Fahrenheit é conversão de apresentação, sem nova chamada.

O serviço valida status HTTP, timeout, campos e cardinalidade antes da normalização. Componentes não dependem dos DTOs brutos do Open-Meteo. Confirmar termos, atribuição, cobertura e limites de uso antes do lançamento.

## State Management

O estado vive em uma única instância de `useWeather`, usada por `App.tsx`; não há estado global, armazenamento local, servidor ou biblioteca de estado.

Busca e consulta meteorológica são operações independentes, cada uma com status discriminado:

```ts
export type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T; isPartial?: boolean; missingFields?: string[] }
  | { status: 'empty' }
  | { status: 'error'; error: WeatherError };

export interface WeatherViewState {
  query: string;
  search: RequestState<City[]>;
  selectedCity: City | null;
  weather: RequestState<WeatherData>;
  unit: Unit;
}
```

- `search.status` começa em `idle`; envio válido muda para `loading`, lista sem correspondências para `empty`, resultados para `success`, e falha do serviço para `error`.
- `weather.status` é `idle` até haver cidade selecionada; depois passa por `loading` para `success` (completo ou parcial) ou `error`. `empty` não representa previsão vazia: ausência de cidade é o estado `search.empty`; payload meteorológico vazio/inválido é `error` (`invalid-response`).
- Cada operação mantém seu próprio status e dados. Ao selecionar uma cidade nova, limpar o snapshot anterior e iniciar `weather.loading`. Só aceitar resposta se `selectedCity.id` ainda corresponder à requisição.
- `unit` inicia em `celsius` em cada carregamento da página, vive apenas em memória e não persiste após reload.
- Temperaturas no `WeatherData` permanecem canônicas em Celsius. Na renderização, um helper puro recebe o valor Celsius e `unit`; em Fahrenheit calcula `°F = (°C × 9/5) + 32` e aplica formatação. Alterar `unit` apenas atualiza o estado de apresentação: não chama `weatherService` nem dispara request.
- Resposta parcial mantém `status: 'success'`, `isPartial: true` e `missingFields`; valores ausentes permanecem `null` no modelo e são apresentados como indisponíveis.

## Error Handling

- **Rede indisponível:** classificar como `network`, manter a cidade selecionada, encerrar loading e mostrar mensagem com retry manual. Não tratar como busca vazia.
- **Erro HTTP da API:** classificar `429` como `rate-limit`; demais respostas não bem-sucedidas como `provider`. Preservar status HTTP internamente para diagnóstico, sem expor detalhes técnicos na mensagem ao usuário.
- **Limite de requisições:** respeitar o header `Retry-After`; se ausente, habilitar retry manual após 30 s. Não repetir automaticamente.
- **Timeout:** abortar a chamada após 8 s, classificar como `timeout`, remover loading e oferecer retry manual. Não iniciar retry automático.
- **Resposta inválida:** JSON ilegível, tipo incompatível, timezone ausente para a previsão ou arrays diários com cardinalidade diferente de cinco resultam em `invalid-response`; não apresentar esses dados como previsão válida.
- **Resposta parcial:** campos numéricos opcionais ausentes ou inválidos tornam-se `null`; dados válidos continuam em `success` com `isPartial: true` e `missingFields`. Ocultar o valor ausente e identificá-lo como indisponível; não substituir por zero, valor anterior ou cache.
- **Cidade não encontrada:** geocoding bem-sucedido com lista vazia resulta em `search.empty` e mensagem “Nenhuma cidade encontrada”; não chamar Forecast API.
- **Concorrência:** respostas de requisições associadas a cidade não selecionada são descartadas. Ao selecionar outra cidade, remover os dados anteriores durante o loading.
- **Dados atuais antigos:** se `observedAt` tiver mais de duas horas, marcar como desatualizado e não como atual.
- **Privacidade:** não persistir estado ou cache após reload/fechamento; não coletar geolocalização.

## Testing Strategy

- **Vitest — funções puras:** cobrir `temperature.ts` com conversões de referência (0 °C, 100 °C), temperaturas negativas e fracionárias, formatação/precisão, valores `null` e garantia de que mudar `Unit` não modifica os dados de origem. Testar também formatação de datas por fuso e tradução de códigos WMO, incluindo código desconhecido.
- **Vitest — services com `fetch` mockado:** verificar URL e parâmetros de geocoding/forecast, mapeamento de payloads para tipos internos e pareamento dos arrays diários por índice. Incluir resposta válida, geocoding vazio, campo ausente, timezone ausente, arrays desalinhados, JSON inválido, HTTP 429 com/sem `Retry-After`, 5xx, falha de rede e timeout/abort após 8 s. CI não chama a API real.
- **Vitest — hook:** testar transições `idle -> loading -> success|empty|error`, recuperação via retry, `search` independente de `weather`, resposta parcial como `success` com `isPartial`, descarte de resposta fora de ordem e ausência de chamada ao serviço ao alternar unidade.
- **Vitest + Testing Library — componentes:** renderizar e verificar busca/resultados/seleção e estados `loading`, `empty`, `error` e `success`; verificar campos indisponíveis, rótulos/unidades, alternância Celsius/Fahrenheit, retry e operação por teclado. Asserções focam texto, roles e callbacks, não estrutura interna ou classes CSS.
- **Playwright — fluxos E2E:** interceptar ambas as APIs e executar as jornadas US-01 a US-06: buscar/selecionar cidade, consultar dados atuais e cinco datas, alternar unidade, e recuperar-se de busca vazia, erro, timeout e retry. Incluir duas respostas de cidade fora de ordem e confirmar que a seleção mais recente permanece na tela.
- **Playwright — viewport:** executar os fluxos principais em 320 CSS px e desktop 1440 CSS px; verificar ausência de rolagem horizontal/overlap, controles utilizáveis e conteúdo legível. Usar mocks para tornar datas e falhas determinísticas.
- **Acessibilidade:** incluir checagens automatizadas de roles/nome/foco e navegação por teclado nos testes de componente/E2E; antes do release, complementar com revisão manual de contraste e leitor de tela para WCAG 2.2 AA.
- **Dados de teste:** usar relógio fixo, fixtures versionadas e timestamps explícitos. Medir a latência da interface a partir da resposta mockada; não atribuir latência da rede pública ao código da UI.

## Risks & Trade-offs

| Decisão | Alternativa considerada | Trade-off e justificativa |
| --- | --- | --- |
| Chamar Open-Meteo diretamente do navegador. | Proxy/BFF próprio. | Evita backend e persistência, mas expõe ao cliente a dependência, limites e termos do provedor; o texto de busca é enviado a terceiro. Preferível para este escopo sem API key. Se os termos, limites ou política de privacidade exigirem proxy, revisar a arquitetura antes do lançamento. |
| Estado local em `useWeather` e sem biblioteca global/server-state. | React Query, Zustand ou Redux. | Menos dependências e conceitos para uma tela e sem cache; exige implementar estados e descarte de respostas tardias no hook. Reavaliar apenas se surgirem várias telas ou cache compartilhado. |
| Sem persistência ou cache local/remoto. | `localStorage`, IndexedDB ou cache no servidor. | Simplifica privacidade e torna previsível a volta a Celsius, mas perde preferência persistente, favoritos e uso offline; cada nova consulta depende da rede. Esses recursos estão fora de escopo. |
| Guardar temperaturas em Celsius e converter ao renderizar. | Solicitar novamente à API na unidade escolhida. | Alternância instantânea, sem request ou divergência entre respostas; exige conversão e arredondamento consistentes na UI. Testes unitários cobrem valores de referência e fronteira. |
| Previsão diária para cinco datas. | Previsão horária ou intervalo configurável. | Atende ao escopo com interface menor e menos dados; reduz detalhe para planejamento intradiário. Granularidade horária fica fora desta versão. |
| Mockar APIs nos testes automatizados. | Usar Open-Meteo real em CI/E2E. | Fixtures tornam builds repetíveis e cobrem erros raros, mas não detectam por si só mudanças reais de contrato ou indisponibilidade do provedor. Mitigar com testes de contrato do serviço e verificação operacional separada, sem tornar CI dependente da rede. |
| Exibir dados parciais com campos `null`. | Falhar toda a consulta quando qualquer campo estiver ausente. | Mantém informação válida disponível, mas a interface precisa identificar explicitamente os campos ausentes; nunca substitui com zero ou dados antigos. |