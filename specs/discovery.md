# Discovery — Aplicação de previsão do tempo

## Contexto

A empresa solicita uma aplicação para consultar o clima atual e a previsão de
cinco dias por cidade, com alternância entre Celsius e Fahrenheit e uso em
dispositivos móveis. O briefing não define público, abrangência, fonte dos
dados ou detalhes da experiência; esses pontos precisam de validação antes da
especificação.

## Personas

Personas hipotéticas, a validar com o negócio e com usuários:

- **Pessoa planejando o dia:** consulta o clima atual para decidir suas atividades.
- **Pessoa viajante:** consulta a previsão de uma cidade para planejar os próximos dias.
- **Pessoa em deslocamento:** consulta o tempo em um dispositivo móvel.

## Requisitos Funcionais

- **RF01:** Permitir buscar cidades.
- **RF02:** Exibir o clima atual da cidade consultada.
- **RF03:** Exibir a previsão de cinco dias para a cidade consultada.
- **RF04:** Permitir alternar a exibição de temperaturas entre Celsius e Fahrenheit.

## Requisitos Não-Funcionais

- **RNF01 — Responsividade:** permitir o uso em dispositivos móveis; tamanhos de tela e navegadores suportados precisam ser definidos.
- **RNF02 — Acessibilidade:** definir e atender critérios de acessibilidade, incluindo navegação por teclado e tecnologias assistivas.
- **RNF03 — Desempenho:** definir tempos aceitáveis para busca e apresentação dos dados.
- **RNF04 — Disponibilidade e resiliência:** definir o comportamento esperado diante de falhas de rede ou da fonte de dados.
- **RNF05 — Atualidade dos dados:** definir a frequência de atualização e a tolerância a dados desatualizados.
- **RNF06 — Localização:** definir idiomas, formatos de data e abrangência geográfica suportados.

Além da compatibilidade móvel explicitada no briefing, os requisitos acima são
candidatos a validação, não metas ou níveis de serviço já acordados.

## Riscos

As avaliações são preliminares e devem ser revistas após as decisões de produto e tecnologia.

| Risco | Probabilidade | Impacto | Mitigação proposta |
| --- | --- | --- | --- |
| Indisponibilidade ou limite de uso do provedor de clima | A avaliar | Alto | Avaliar disponibilidade, cotas e custos do provedor; definir tratamento de falhas. |
| Cidades homônimas ou resultados de localização imprecisos | Média | Alto | Validar como identificar e apresentar cidade, região e país antes da consulta. |
| Falha de rede ou resposta lenta em dispositivos móveis | Média | Médio | Definir limites de espera e estados de carregamento e erro; testar em redes lentas. |
| Previsões desatualizadas ou diferentes da expectativa do usuário | Média | Alto | Definir atualização, indicação de horário dos dados e comunicação de limitações. |
| Interface inutilizável em telas pequenas ou com tecnologias assistivas | Média | Alto | Definir critérios de suporte e acessibilidade; testar com dispositivos e tecnologias representativos. |
| Interpretação divergente dos cinco dias ou do fuso horário | Média | Médio | Acordar período e fuso da previsão e verificar exemplos com o negócio. |

## Perguntas em Aberto (Open Questions)

| Pergunta | Impacto se não for respondida |
| --- | --- |
| Qual fonte de dados de clima e localização será usada? Há limites de uso, custos, restrições ou necessidade de chave? | Não é possível avaliar viabilidade, custos, disponibilidade e integração. |
| Os cinco dias incluem o dia atual? Qual fuso horário determina a divisão dos dias? | A previsão pode apresentar datas diferentes das esperadas. |
| Quais informações compõem o clima atual e cada dia da previsão? | A equipe pode omitir dados necessários ou implementar informações sem valor. |
| Como cidades homônimas devem ser identificadas e escolhidas? Qual a abrangência geográfica? | Usuários podem consultar a cidade errada ou não encontrar localidades necessárias. |
| Qual unidade deve ser exibida inicialmente? A escolha deve persistir entre visitas? | O comportamento inicial e a continuidade da preferência ficam indefinidos. |
| A busca deve oferecer sugestões, exigir seleção de resultado ou aceitar apenas uma cidade exata? | O fluxo de busca e o tratamento de ambiguidades ficam indefinidos. |
| Quais estados e ações são esperados quando não houver resultados, a rede falhar ou os dados estiverem indisponíveis? | Usuários podem ficar sem orientação ou sem uma forma clara de tentar novamente. |
| Quais idiomas, formatos de data e navegadores devem ser suportados? | Conteúdo e compatibilidade podem não atender ao público. |
| Há metas de desempenho, disponibilidade, atualização dos dados e acessibilidade? | Não há critérios objetivos para validar a qualidade da aplicação. |
| Geolocalização automática é necessária? Em caso afirmativo, como lidar com permissão negada? | O escopo, a privacidade e a experiência inicial permanecem indefinidos. |
| O aplicativo deve funcionar offline ou manter dados em cache? Por quanto tempo? | A experiência sem rede e o risco de mostrar dados antigos ficam indefinidos. |
| É necessária autenticação ou persistência de dados e preferências? | Arquitetura, privacidade e escopo podem mudar significativamente. |

## Suposições (Assumptions)

- O público consulta informações meteorológicas de cidades, mas seus perfis e necessidades ainda não foram validados.
- A alternância de unidade se aplica às temperaturas apresentadas; seu comportamento detalhado deve ser confirmado.
- A aplicação depende de dados meteorológicos obtidos de alguma fonte; a escolha e as condições de uso estão em aberto.

## Decisões

Nenhuma decisão de produto adicional foi confirmada a partir do briefing. As
perguntas em aberto devem ser respondidas com o negócio antes de consolidar a
especificação.