# Especificação de Produto: Weather App

## Overview

O Weather App é uma aplicação web responsiva para buscar cidades e consultar condições meteorológicas atuais e uma previsão diária de cinco dias. O período inclui a data atual e os quatro dias seguintes no fuso horário da cidade selecionada. A interface é em pt-BR; temperaturas são exibidas em Celsius por padrão, com alternância para Fahrenheit. Os dados meteorológicos vêm do Open-Meteo, sem API key. Não há autenticação, geolocalização automática nem persistência de dados no servidor ou navegador.

O produto prioriza uma consulta rápida e compreensível em celular e desktop. As personas no discovery são hipóteses; métricas de sucesso de negócio ainda não foram validadas. Os limites operacionais abaixo são defaults propostos para tornar a especificação testável e exigem aprovação antes do lançamento.

## Functional Requirements

- **FR-01 — Buscar e selecionar uma cidade:** aceitar nome de cidade, opcionalmente acompanhado de região ou país. Remover espaços externos; aceitar acentos, espaços, hífens e apóstrofos. Exibir até cinco resultados em ordem de relevância, com cidade e, quando disponível, região e país. Exigir seleção explícita; não usar geolocalização nem selecionar automaticamente o primeiro resultado.
- **FR-02 — Consultar o clima atual:** para a cidade selecionada, apresentar temperatura, sensação térmica, condição meteorológica, umidade relativa, velocidade do vento e horário da observação, quando fornecidos pelo Open-Meteo. Condições devem ter rótulo em pt-BR; valores devem incluir unidade.
- **FR-03 — Consultar previsão diária:** apresentar cinco dias de calendário consecutivos, começando pela data atual no fuso horário da cidade selecionada. Para cada dia, apresentar condição meteorológica, temperaturas mínima e máxima, probabilidade máxima de precipitação e precipitação total. Temperaturas seguem a unidade selecionada; vento é exibido em km/h, precipitação em mm e probabilidades em %.
- **FR-04 — Alternar unidade de temperatura:** iniciar cada carregamento da aplicação em Celsius. Permitir alternar todos os valores de temperatura, incluindo sensação térmica e mínimas/máximas, entre Celsius e Fahrenheit. Manter a escolha enquanto a página estiver aberta; não persistir a preferência após recarregar ou fechar a página.
- **FR-05 — Manter dados associados ao local selecionado:** exibir o nome da cidade selecionada junto aos dados atuais e à previsão. Ao selecionar outra cidade, atualizar ambos os conjuntos de dados para o novo local e descartar respostas tardias de consultas anteriores.
- **FR-06 — Comunicar estados da consulta:** mostrar carregamento, estado vazio de busca e erro de serviço de forma distinta. Em erro ou timeout, permitir uma nova tentativa manual, sem repetição automática. Em limite de requisições, informar a indisponibilidade temporária e respeitar o prazo de nova tentativa retornado pelo serviço, quando fornecido.

## User Stories

- **US-01 — Busca de cidade (FR-01):** Como Camila, profissional que planeja o deslocamento diário, quero buscar e selecionar a cidade do meu trajeto para consultar as condições do local correto.
- **US-02 — Clima atual (FR-02):** Como Camila, profissional que planeja o deslocamento diário, quero ver o clima atual da cidade selecionada para me preparar para o trajeto.
- **US-03 — Previsão de cinco dias (FR-03):** Como Diego, viajante que organiza uma viagem, quero ver a previsão de hoje e dos quatro dias seguintes no destino para planejar minhas atividades.
- **US-04 — Unidade de temperatura (FR-04):** Como Diego, viajante que organiza uma viagem, quero alternar as temperaturas entre Celsius e Fahrenheit para interpretá-las na unidade que prefiro.
- **US-05 — Dados do local selecionado (FR-05):** Como Renata, profissional que trabalha ao ar livre, quero que o clima atual e a previsão correspondam à cidade selecionada para planejar as atividades no local correto.
- **US-06 — Estado da consulta (FR-06):** Como Renata, profissional que trabalha ao ar livre, quero saber quando uma consulta está carregando, não encontrou resultados ou falhou para decidir se posso usar a informação e tentar novamente quando necessário.

As personas e os tempos sugeridos no discovery são hipóteses, não metas de desempenho nem critérios de aceite aprovados.

## Acceptance Criteria

- **AC-01a — Input vazio (FR-01):** **Given** o campo contém apenas espaços, **When** o usuário envia a busca, **Then** uma mensagem de validação é exibida e nenhuma chamada de geocoding é feita.
- **AC-01b — Busca e ordenação (FR-01):** **Given** a busca `Sao Jose` retorna seis registros com pontuação de relevância conhecida, **When** os resultados são exibidos, **Then** são apresentados exatamente os cinco primeiros na ordem de relevância e cada resultado mostra cidade, região e país quando fornecidos.
- **AC-01c — Caracteres especiais (FR-01):** **Given** o geocoding retorna um registro para `São José`, **When** o usuário busca esse nome e seleciona o registro, **Then** o resultado mantém o nome retornado e se torna a cidade ativa.
- **AC-02 — Clima atual (FR-02):** **Given** Recife está selecionada e a fixture contém temperatura `28`, sensação térmica `29`, condição `código 0`, umidade `70`, vento `12` e observação às `14:00`, **When** a consulta termina, **Then** os seis valores são exibidos com cidade, rótulos em pt-BR, unidades e horário `14:00` no fuso de Recife.
- **AC-03 — Previsão diária (FR-03):** **Given** o relógio de teste está fixado em `2026-11-11T12:00:00Z`, o fuso de Recife é `America/Recife` e a fixture contém dados de 11 a 15/11/2026, **When** a previsão carrega, **Then** aparecem exatamente essas cinco datas locais, em ordem, com condição, mín./máx., probabilidade máxima e total de precipitação em cada dia.
- **AC-04a — Unidade padrão (FR-04):** **Given** a aplicação é aberta em uma página nova, **When** temperaturas são renderizadas, **Then** todas são exibidas em Celsius com o rótulo `°C`.
- **AC-04b — Conversão para Fahrenheit (FR-04):** **Given** temperaturas de `0 °C` e `100 °C` estão visíveis, **When** o usuário seleciona Fahrenheit, **Then** os valores correspondentes são `32 °F` e `212 °F`, e todas as temperaturas visíveis usam `°F`.
- **AC-04c — Conversão para Celsius (FR-04):** **Given** temperaturas de `32 °F` e `212 °F` estão visíveis, **When** o usuário seleciona Celsius, **Then** os valores correspondentes são `0 °C` e `100 °C`, e todas as temperaturas visíveis usam `°C`.
- **AC-04d — Unidade não persistida (FR-04):** **Given** Fahrenheit está selecionado, **When** a página é recarregada, **Then** a unidade volta a Celsius.
- **AC-05 — Concorrência de locais (FR-05):** **Given** Recife está selecionada e sua resposta está pendente, **When** Olinda é selecionada e sua resposta chega primeiro, **Then** Olinda e seus dados são exibidos; quando a resposta tardia de Recife chega, **Then** Olinda continua ativa e seus dados permanecem visíveis.
- **AC-06a — Carregamento (FR-06):** **Given** uma solicitação foi enviada e não terminou, **When** a interface aguarda a resposta, **Then** o indicador de carregamento aparece em até 100 ms.
- **AC-06b — Geocoding sem resultados (FR-06):** **Given** o geocoding responde com sucesso e lista vazia, **When** a resposta é processada, **Then** aparece “Nenhuma cidade encontrada” e não é enviada consulta meteorológica.
- **AC-06c — Falha de serviço (FR-06):** **Given** o serviço responde com erro, **When** a resposta é processada, **Then** aparece mensagem de indisponibilidade e ação de retry, distintas do estado vazio.
- **AC-06d — Timeout (FR-06):** **Given** uma solicitação não responde em oito segundos, **When** o limite é atingido, **Then** o carregamento termina, uma mensagem de timeout aparece e nenhuma repetição automática ocorre.
- **AC-06e — Limite de requisições (FR-06):** **Given** o serviço retorna `429` e `Retry-After`, **When** a resposta é processada, **Then** a mensagem informa o prazo e o retry fica desabilitado até esse prazo.
- **AC-07 — Localização (FR-02, FR-03):** **Given** dados meteorológicos contêm códigos de condição e horários, **When** são exibidos, **Then** condições estão traduzidas para pt-BR e datas/horas usam `dd/MM/yyyy`, 24 horas e o fuso da cidade.
- **AC-08 — Sem autenticação:** **Given** um usuário sem conta abre a aplicação, **When** busca uma cidade e consulta clima e previsão, **Then** completa os fluxos sem cadastro nem login.

## Traceability Matrix

| User Story | Requisito funcional | Acceptance Criteria | Requisitos não-funcionais relevantes |
| --- | --- | --- | --- |
| US-01 — Busca de cidade | FR-01 | AC-01a, AC-01b, AC-01c, AC-06b | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade; NFR-04 feedback de desempenho; NFR-07 privacidade da consulta; NFR-08 localização/formatação. |
| US-02 — Clima atual | FR-02 | AC-02, AC-06c, AC-07 | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade; NFR-04 desempenho; NFR-05 atualidade/timeout; NFR-06 disponibilidade; NFR-08 unidades e formatação. |
| US-03 — Previsão de cinco dias | FR-03 | AC-03, AC-06c, AC-07 | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade; NFR-04 desempenho; NFR-05 atualidade/timeout; NFR-06 disponibilidade; NFR-08 unidades, datas e fuso. |
| US-04 — Unidade de temperatura | FR-04 | AC-04a, AC-04b, AC-04c, AC-04d | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade dos controles; NFR-07 ausência de persistência; NFR-08 formatação de unidades. |
| US-05 — Dados do local selecionado | FR-05 | AC-02, AC-03, AC-05 | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade; NFR-04 desempenho; NFR-05 atualidade; NFR-06 disponibilidade; NFR-08 associação entre local, data e unidades. |
| US-06 — Estado da consulta | FR-06 | AC-06a, AC-06b, AC-06c, AC-06d, AC-06e | NFR-01 responsividade; NFR-02 compatibilidade; NFR-03 acessibilidade dos estados; NFR-04 feedback de desempenho; NFR-05 timeout; NFR-06 disponibilidade; NFR-07 privacidade da consulta. |

**Critérios transversais:** AC-08 (ausência de autenticação) e NFR-07 aplicam-se a todos os fluxos; AC-08 não possui uma User Story exclusiva.

## Non-Functional Requirements

- **NFR-01 — Responsividade:** os fluxos principais devem funcionar em larguras de 320 a 1920 CSS px, sem rolagem horizontal da página ou conteúdo sobreposto.
- **NFR-02 — Compatibilidade:** suportar as duas versões principais mais recentes de Chrome, Edge, Firefox e Safari, incluindo Safari em iOS e Chrome em Android.
- **NFR-03 — Acessibilidade:** atender WCAG 2.2 AA; todos os controles devem ser operáveis por teclado, ter nome acessível e foco visível; informação meteorológica não pode depender apenas de cor ou ícone.
- **NFR-04 — Desempenho da interface:** mostrar feedback de carregamento em até 100 ms após busca, seleção ou retry; renderizar uma resposta recebida em até 500 ms no percentil 95 em navegadores suportados.
- **NFR-05 — Timeout e atualidade:** encerrar requisições sem resposta após 8 s. Exibir o horário da observação retornado para o clima atual; observações com mais de 2 h devem ser marcadas como desatualizadas e não descritas como atuais.
- **NFR-06 — Disponibilidade:** manter a aplicação web disponível em pelo menos 99,5% do mês, medido por verificação externa. Falhas do Open-Meteo devem ser monitoradas separadamente e nunca simuladas como resposta válida.
- **NFR-07 — Privacidade:** não coletar geolocalização, não gravar busca ou preferência em armazenamento local/remoto e não exigir conta. A busca enviada ao Open-Meteo pode incluir o texto consultado.
- **NFR-08 — Formatação:** usar pt-BR, datas `dd/MM/yyyy`, horário de 24 horas no fuso da cidade, temperaturas com unidade, vento em km/h e precipitação em mm.

Os limites operacionais são defaults propostos nesta especificação e requerem aprovação do responsável pelo produto antes do lançamento.

## Edge Cases

- **Input vazio:** remover espaços externos; se restar vazio, informar que uma cidade é necessária e não chamar geocoding.
- **Caracteres especiais:** preservar letras Unicode, acentos, espaços, hífens e apóstrofos na busca e no nome exibido; manter o resultado selecionável.
- **Cidade inexistente ou geocoding sem resultados:** em resposta bem-sucedida com lista vazia, informar “Nenhuma cidade encontrada”, permitir nova busca e não chamar a previsão. Não afirmar que a cidade não existe em definitivo.
- **Cidades homônimas:** mostrar região e país retornados para desambiguação e exigir seleção explícita; nunca escolher automaticamente o primeiro resultado.
- **Falha de API:** distinguir erro do provedor de lista vazia, manter a cidade selecionada e oferecer retry manual.
- **Timeout:** após 8 s sem resposta, remover loading, informar que a consulta expirou e não iniciar retry automático.
- **Limite de requisições:** não repetir automaticamente; informar o prazo `Retry-After`, se retornado, e habilitar retry manual após esse prazo. Se não houver prazo, habilitar após 30 s.
- **Resposta parcial:** exibir campos válidos e marcar os ausentes como indisponíveis. Sem temperatura atual ou dados mínimos de uma data da previsão, não apresentar aquela parte como completa; nunca substituir por zero ou valor antigo.
- **Observação desatualizada:** se o horário da observação tiver mais de 2 h, marcá-la como desatualizada; não a rotular como clima atual.
- **Fuso ausente:** se não for possível identificar o fuso da cidade selecionada, não atribuir datas locais incorretas à previsão; exibir clima atual, quando disponível, e informar que a previsão não pôde ser determinada.
- **Respostas fora de ordem:** descartar resposta de uma cidade que deixou de estar selecionada.
- **Sem conexão:** informar que a consulta não pôde ser concluída; não exibir dados offline nem cache, pois o produto não armazena dados localmente.

## Assumptions

- Open-Meteo fornece geocoding e dados meteorológicos; cobertura, limites e termos de uso são gates de lançamento.
- A previsão é diária para a data atual e os quatro dias seguintes, no fuso local retornado para a cidade.
- Celsius é padrão; a unidade escolhida vale somente enquanto a página está aberta e retorna a Celsius após recarregar.
- A aplicação é web responsiva para celular e desktop, em pt-BR, sem autenticação, geolocalização ou armazenamento persistente local/remoto.
- As condições meteorológicas são traduzidas para pt-BR a partir dos códigos do serviço.
- As personas do discovery são hipóteses e não definem, por si só, o público prioritário.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| A cobertura, os limites, a disponibilidade ou os termos do Open-Meteo não atendem ao produto. | Média | Alto: mudança de integração ou limitação de regiões/campos. | Validar cobertura, variáveis, limites e termos antes de congelar a especificação da fonte. |
| Dados atrasados ou previsão interpretada como garantia. | Média | Alto: decisões equivocadas e perda de confiança. | Exibir o horário da observação, marcar dados com mais de 2 h como desatualizados e comunicar falhas. |
| Geocoding não fornece contexto suficiente ou fuso para um local. | Média | Alto: seleção ambígua ou datas incorretas. | Exigir seleção explícita, mostrar metadados disponíveis e não inventar o fuso quando ausente. |
| Público e necessidades prioritárias não foram validados. | Alta | Alto: produto pode priorizar conteúdo sem valor para os usuários. | Validar as personas e tarefas prioritárias e acordar métricas de sucesso. |
| Experiência móvel ou acessibilidade inadequada. | Média | Alto: usuários não conseguem completar a consulta. | Definir alvos mensuráveis e testar com dispositivos, teclado e tecnologias assistivas. |
| O texto da cidade é enviado ao provedor externo durante a busca. | Média | Médio: usuários podem não esperar que a consulta seja processada por terceiros. | Informar o uso do Open-Meteo na política de privacidade e validar termos e retenção do provedor antes do lançamento. |

## Out of Scope

- Cadastro, login e autenticação de usuários.
- Persistência de dados no servidor ou navegador, incluindo preferências, buscas e histórico.
- Geolocalização automática, cidade padrão, favoritos, múltiplas cidades salvas e histórico de busca.
- Alertas, notificações push, modo offline e exibição de cache após falha.
- Aplicativos nativos; o produto será uma aplicação web responsiva.

## Open Questions

- **OQ-01 — Aceite operacional:** o responsável pelo produto aprova os defaults de compatibilidade, acessibilidade, latência, timeout, frescor, disponibilidade e ausência de armazenamento local definidos nesta spec?
- **OQ-02 — Uso do Open-Meteo:** os termos, atribuição, limites e cobertura geográfica do Open-Meteo permitem o uso comercial e a distribuição pretendidos? A geocodificação cobre as regiões de lançamento?
- **OQ-03 — Validação de produto:** qual público será priorizado e qual métrica de negócio será acompanhada após o lançamento? As personas atuais ainda não foram validadas por pesquisa.