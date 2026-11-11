# Discovery: Aplicação de Previsão do Tempo

## Contexto

A empresa precisa de uma aplicação de previsão do tempo para que usuários consultem as condições atuais e a previsão para os próximos cinco dias em cidades de seu interesse. A experiência deve permitir alternar entre Celsius e Fahrenheit e funcionar em dispositivos móveis.

O briefing define as capacidades principais, mas ainda não especifica plataformas, fontes de dados, regras de busca, detalhes meteorológicos exibidos ou expectativas operacionais. Esta análise organiza o que foi informado e destaca as decisões que precisam ser confirmadas antes da especificação detalhada.

## Decisões

- **Fonte de dados: Open-Meteo, sem API key.** Justificativa: adotar a fonte já escolhida sem exigir cadastro ou gerenciamento de chave de API para o produto. **Resolve:** qual provedor será usado e se a integração depende de uma chave. **Ainda precisa ser validado:** cobertura, limites, disponibilidade e termos de uso/atribuição.
- **Período da previsão: hoje mais os quatro dias seguintes.** Justificativa: tornar explícito o intervalo pretendido para “cinco dias”. **Resolve:** se a previsão inclui hoje ou começa nos cinco dias seguintes. **Ainda precisa ser definido:** granularidade da previsão e fuso horário usado para determinar a data local.
- **Unidade padrão: Celsius.** Justificativa: estabelecer uma apresentação inicial consistente para todos os usuários. **Resolve:** qual unidade exibir por padrão. A alternância para Fahrenheit continua no escopo; persistência da preferência e unidades de outras medidas permanecem em aberto.
- **Sem autenticação e sem persistência no servidor.** Justificativa: manter a consulta sem criação de conta e evitar armazenamento de dados de usuário no servidor. **Resolve:** se haverá cadastro/login e armazenamento persistente no servidor. Armazenamento local no navegador, histórico ou favoritos não foram decididos.
- **Idioma da interface: pt-BR.** Justificativa: definir o idioma inicial da experiência. **Resolve:** qual idioma a interface deve usar. Outros idiomas e convenções detalhadas de data, hora e números ainda precisam ser confirmados.

## Personas Provisórias

Os perfis abaixo são hipóteses baseadas no briefing, não resultados de pesquisa com usuários. Devem ser validados antes de orientar decisões definitivas de produto.

### Camila, profissional que planeja o deslocamento diário

- **Objetivo principal:** verificar rapidamente as condições atuais e a previsão para decidir o que vestir e como se preparar para o trajeto.
- **Contexto de uso:** principalmente mobile, em casa antes de sair ou durante o deslocamento; consultas breves e recorrentes.
- **Métrica de sucesso para ela:** encontrar a cidade correta e consultar as informações necessárias para se preparar em até 30 segundos, sem precisar repetir a busca.

### Diego, viajante que organiza uma viagem

- **Objetivo principal:** consultar o clima de uma cidade de destino e entender as condições esperadas nos próximos cinco dias para planejar a viagem.
- **Contexto de uso:** principalmente desktop, durante o planejamento; pode voltar ao mobile para conferir as condições perto da partida.
- **Métrica de sucesso para ele:** localizar o destino e consultar a previsão completa do período em até dois minutos, com datas e unidades compreensíveis.

### Renata, profissional que trabalha ao ar livre

- **Objetivo principal:** conferir o clima da cidade onde trabalhará para organizar as atividades externas do dia e da semana.
- **Contexto de uso:** principalmente mobile, em campo e com pouco tempo para consultar; pode usar desktop para planejar a semana.
- **Métrica de sucesso para ela:** consultar a cidade e a previsão relevante para o trabalho em até um minuto, sem ambiguidade sobre local, data ou unidade.

## Requisitos Funcionais

- **RF-01 — Buscar cidades:** permitir que o usuário pesquise uma cidade e selecione o local cujas condições deseja consultar.
- **RF-02 — Consultar clima atual:** apresentar as condições meteorológicas atuais da cidade selecionada.
- **RF-03 — Consultar previsão:** apresentar a previsão do tempo para hoje e os quatro dias seguintes na cidade selecionada.
- **RF-04 — Alternar unidade de temperatura:** permitir alternar a exibição da temperatura entre Celsius e Fahrenheit.
- **RF-05 — Atualizar dados por local:** manter o clima atual e a previsão associados à cidade selecionada.

## Requisitos Não-Funcionais

- **RNF-01 — Uso em dispositivos móveis:** a aplicação deve ser utilizável em dispositivos móveis, com conteúdo e controles adaptados a telas menores.
- **RNF-02 — Clareza da unidade:** os valores de temperatura devem indicar claramente a unidade atualmente selecionada.
- **RNF-03 — Usabilidade da busca:** a busca e a seleção de cidade devem ser compreensíveis e operáveis em telas móveis.
- **RNF-04 — Informação atualizada:** os dados apresentados devem representar a consulta mais recente disponível da fonte de previsão. A frequência aceitável de atualização ainda precisa ser definida.

## Riscos

As probabilidades abaixo são avaliações iniciais com base apenas no briefing e nas decisões registradas; devem ser revistas quando público e metas operacionais forem definidos.

| Categoria | Risco | Probabilidade | Impacto | Estratégia de mitigação |
| --- | --- | --- | --- | --- |
| Produto | Público-alvo e principal situação de uso não estão definidos; o produto pode priorizar informações que não resolvem a necessidade central. | Alta | Alto: baixa adoção e retrabalho de produto. | Validar personas e tarefas prioritárias com usuários e stakeholders antes de fechar escopo e hierarquia da tela. |
| Produto | “Previsão de cinco dias” e os dados que devem ser exibidos não têm definição comum. | Alta | Alto: expectativas e critérios de aceite incompatíveis. | Definir período, granularidade, campos, unidades e exemplos de resultados esperados antes da implementação. |
| Produto | Busca de cidades pode retornar locais homônimos ou não cobrir regiões relevantes. | Média | Alto: usuário consulta o local errado ou não encontra sua cidade. | Definir cobertura geográfica e exibir contexto de desambiguação, como região e país; testar consultas representativas. |
| Produto | A experiência pode não ser utilizável em telas pequenas ou por pessoas com deficiência. | Média | Alto: parte dos usuários não consegue concluir a consulta. | Definir requisitos verificáveis de responsividade e acessibilidade; testar em dispositivos reais, teclado e leitor de tela. |
| Produto | Sem critérios de sucesso, não será possível saber se a aplicação atende às necessidades nem priorizar evolução. | Média | Alto: investimento sem evidência de valor ou direção de produto. | Acordar métricas de adoção e sucesso das tarefas, além de uma linha de base e um processo de acompanhamento. |
| Técnico | Indisponibilidade, limite de requisições, custo ou cobertura insuficiente do Open-Meteo. | Média | Alto: clima ou busca deixam de funcionar, ou o custo fica inviável. | Validar cobertura, termos, limites e custos do Open-Meteo; definir timeout, tratamento de erro, monitoramento e contingência compatível com o escopo. |
| Técnico | Dados podem estar atrasados ou a previsão pode divergir das condições reais. | Média | Alto: decisões dos usuários e confiança no produto são afetadas. | Definir frescor máximo, mostrar horário da última atualização e comunicar falhas ou incerteza; não apresentar dados antigos como atuais. |
| Técnico | Mapeamento de fuso horário e mudança de dia pode associar a previsão à data errada. | Média | Médio: previsão confusa, sobretudo para locais em outros fusos. | Usar o fuso do local consultado e testar limites de dia, fusos diferentes e transições de horário sazonal. |
| Técnico | Conversão, arredondamento ou rotulagem de Celsius/Fahrenheit pode gerar valores inconsistentes. | Baixa | Alto: informação incorreta compromete confiança e decisões. | Centralizar conversão e formatação, manter a unidade explícita e testar valores de referência, arredondamento e alternância repetida. |
| Técnico | Latência de rede móvel ou respostas lentas do provedor podem tornar busca e consulta frustrantes. | Média | Alto: abandono da tarefa e percepção de falha. | Estabelecer metas de latência, usar timeouts e estados de carregamento/erro claros; medir desempenho em redes e dispositivos representativos. |
| Técnico | Permissão de localização, consultas salvas ou preferências podem envolver coleta e retenção não definidas. | Média | Alto: perda de confiança e possíveis obrigações de privacidade não atendidas. | Minimizar dados, solicitar permissão apenas quando necessário, oferecer busca manual e definir transparência, retenção e exclusão. |
| Técnico | Licença, atribuição ou regras de uso da fonte podem ser incompatíveis com a experiência ou distribuição pretendida. | Baixa | Alto: necessidade de remover dados, alterar interface ou trocar de fornecedor. | Revisar termos e requisitos de atribuição antes da integração e documentar limites de uso e cobertura. |

## Perguntas em Aberto

O briefing não responde aos pontos abaixo. Os impactos indicam o que pode sair errado se a decisão ficar para depois; não são requisitos já aprovados.

### Objetivo e público

- **Pergunta:** Quem são os usuários prioritários e em que situação consultarão o app (planejamento diário, viagem, atividade ao ar livre ou outra)?
	**Impacto:** Sem um público e um uso prioritários, não é possível decidir quais informações devem ter destaque nem avaliar se o produto resolve o problema certo.
- **Pergunta:** Qual resultado de negócio define o sucesso do produto e como será medido?
	**Impacto:** Sem uma métrica de sucesso, o time pode entregar funcionalidades sem saber se elas geram valor ou como priorizar melhorias.
- **Pergunta:** Qual é o escopo do primeiro lançamento? Favoritos, várias cidades, alertas, notificações e histórico fazem parte dele? Se houver dados salvos, devem ficar apenas no dispositivo, já que não haverá persistência no servidor?
	**Impacto:** Sem limites de escopo, expectativas podem crescer durante a implementação, afetando prazo, custo e critérios de aceite.

### Localização e busca

- **Pergunta:** A busca aceita nomes de cidades apenas ou também códigos postais, estados, regiões e países? Deve haver sugestões enquanto o usuário digita?
	**Impacto:** A decisão afeta a experiência de busca, os resultados esperados e a cobertura necessária da fonte de dados.
- **Pergunta:** Como distinguir cidades homônimas e quais informações de contexto devem aparecer nos resultados?
	**Impacto:** Sem desambiguação, o usuário pode selecionar o local errado e receber dados meteorológicos incorretos para sua intenção.
- **Pergunta:** A cidade deve ser selecionada explicitamente ou o primeiro resultado pode ser escolhido automaticamente?
	**Impacto:** Uma seleção implícita pode apresentar dados do lugar errado; exigir confirmação pode adicionar etapas à tarefa.
- **Pergunta:** O app deve usar a localização do dispositivo? Se sim, em que momento pedir permissão e qual alternativa oferecer se ela for negada ou indisponível?
	**Impacto:** Sem decisão, o fluxo inicial pode ficar bloqueado, invasivo ou sem um local para consultar; também há implicações de privacidade e permissões.
- **Pergunta:** Deve haver uma cidade inicial padrão, histórico ou cidades favoritas? Como essas escolhas persistem e podem ser removidas?
	**Impacto:** O usuário pode ter de repetir buscas a cada visita, ou o produto pode guardar dados sem uma política de retenção definida.
- **Pergunta:** Quais países e regiões precisam ser cobertos, inclusive para cidades com nomes em outros alfabetos?
	**Impacto:** A cobertura e a qualidade da busca podem variar por região; uma decisão tardia pode exigir trocar ou reconfigurar a fonte de dados.

### Conteúdo e interpretação meteorológica

- **Pergunta:** Quais campos devem aparecer no clima atual (por exemplo, temperatura, sensação térmica, condição, vento e umidade)?
	**Impacto:** Sem um conjunto definido, telas, dados necessários e critérios de aceite permanecem indeterminados.
- **Pergunta:** Quais campos devem aparecer em cada dia da previsão e como será comunicada a probabilidade ou quantidade de precipitação?
	**Impacto:** Diferentes interpretações de “previsão” geram expectativas incompatíveis e podem tornar os dados difíceis de comparar ou interpretar.
- **Pergunta:** A previsão será organizada por dias de calendário ou por janelas de 24 horas, e qual fuso horário determinará a data de “hoje” para o local consultado?
	**Impacto:** Mesmo com o período definido como hoje mais quatro dias, datas e limites dos dias podem divergir entre o usuário e a fonte de dados.
- **Pergunta:** A previsão será diária, por hora ou terá as duas granularidades?
	**Impacto:** A granularidade determina a quantidade de informação, a forma de apresentação e quais dados a fonte precisa fornecer.
- **Pergunta:** Em qual fuso horário devem ser exibidos horários e datas, especialmente para locais diferentes do usuário ou perto da mudança de dia?
	**Impacto:** Sem uma regra, previsões podem aparecer no dia ou horário errado, particularmente para viagens e cidades em outros fusos.
- **Pergunta:** A alternância entre Celsius e Fahrenheit se aplica apenas à temperatura? Como serão representadas outras unidades, como velocidade do vento e precipitação?
	**Impacto:** A tela pode misturar convenções ou deixar o usuário sem contexto para interpretar medidas.
- **Pergunta:** A preferência por Celsius/Fahrenheit persiste entre visitas? Como não haverá persistência no servidor, ela deve ser mantida no navegador ou valer apenas durante a sessão?
	**Impacto:** Sem uma regra, a escolha pode desaparecer inesperadamente ou ser armazenada localmente sem expectativa clara para o usuário.
- **Pergunta:** O que “clima atual” significa em termos de horário de observação e idade máxima aceitável do dado?
	**Impacto:** O usuário pode interpretar como condição em tempo real dados que já estão desatualizados.

### Dados e estados de falha

- **Pergunta:** O Open-Meteo atende à cobertura geográfica e aos dados necessários? Quais limites, termos de uso e requisitos de atribuição se aplicam?
	**Impacto:** Cobertura insuficiente, limites de requisição, custos ou obrigações de atribuição podem inviabilizar a experiência ou exigir mudanças de produto e interface.
- **Pergunta:** Com que frequência os dados devem ser atualizados e qual idade máxima é aceitável para exibir uma resposta em cache?
	**Impacto:** Sem limites, os dados podem estar desatualizados ou as consultas podem ser excessivas, elevando custo e risco de atingir limites do provedor.
- **Pergunta:** O que mostrar durante carregamento, para uma busca sem resultados ou diante de erro, timeout, limite de requisições e indisponibilidade da fonte?
	**Impacto:** Sem estados e ações definidos, o usuário pode ficar sem orientação ou interpretar falhas como ausência de previsão.
- **Pergunta:** Se a atualização falhar, é permitido mostrar dados anteriores? Como indicar a data e a hora da última atualização?
	**Impacto:** Dados antigos apresentados como atuais podem induzir decisões equivocadas; ocultá-los pode remover informação útil sem necessidade.
- **Pergunta:** O produto precisa funcionar sem conexão ou oferecer alguma forma de consulta de dados previamente carregados?
	**Impacto:** A resposta afeta cache, armazenamento local, privacidade e o que pode ser prometido em redes instáveis.

### Plataforma, qualidade e operação

- **Pergunta:** “Usar em dispositivos móveis” significa um site responsivo, uma aplicação web instalável ou aplicativos nativos? Quais navegadores e versões precisam ser suportados?
	**Impacto:** Sem definir plataformas, o esforço, os testes e as capacidades esperadas podem variar substancialmente.
- **Pergunta:** Quais padrões de acessibilidade devem ser atendidos e quais tecnologias assistivas precisam ser consideradas?
	**Impacto:** Acessibilidade não validada desde o início pode excluir usuários e exigir retrabalho em navegação, estrutura, contraste e controles.
- **Pergunta:** Quais metas de desempenho são esperadas para busca e carregamento do clima, e sob quais condições de rede e dispositivo?
	**Impacto:** Sem metas e condições de medição, “rápido” não é verificável e gargalos de interface ou dependências externas podem passar despercebidos.
- **Pergunta:** Qual meta de disponibilidade é necessária e falhas da fonte externa contam para essa meta?
	**Impacto:** Sem uma definição, não há critério para avaliar confiabilidade, planejar contingências ou atribuir responsabilidade por indisponibilidade.
- **Pergunta:** Além da interface em pt-BR, haverá outros idiomas? Quais formatos de data e hora, convenções numéricas e regras de localização do conteúdo devem ser usados?
	**Impacto:** Sem escopo de localização, cidades e medidas podem ser exibidas em formatos confusos para parte do público e a internacionalização pode exigir retrabalho.
- **Pergunta:** O app pode armazenar consultas, localização ou preferências apenas no dispositivo? Por quanto tempo e com quais controles para o usuário? Que dados operacionais serão registrados nos serviços usados?
	**Impacto:** A ausência de persistência no servidor não esclarece armazenamento local nem registros operacionais; regras indefinidas podem criar riscos de privacidade, conformidade e perda de confiança.
- **Pergunta:** Há orçamento, prazo, restrições de custo por consulta ou requisitos de segurança e conformidade que limitem as opções?
	**Impacto:** Restrições descobertas tarde podem invalidar a fonte de dados ou a arquitetura escolhida e forçar cortes de escopo.

## Suposições

- A consulta de clima e previsão acontece para uma cidade escolhida pelo usuário.
- “Previsão de 5 dias” cobre hoje e os quatro dias seguintes; granularidade e fuso horário ainda precisam ser definidos.
- Celsius e Fahrenheit são unidades de apresentação para a mesma informação meteorológica.
- Celsius é a unidade padrão.
- A interface será apresentada em pt-BR.
- “Usar em dispositivos móveis” requer uma interface responsiva e controles utilizáveis por toque, mas não determina se o produto será um site, uma aplicação web instalável ou um aplicativo nativo.
- Os dados meteorológicos serão obtidos do Open-Meteo, sujeito à validação de cobertura e termos de uso.
- O produto não terá autenticação nem persistência de dados no servidor; eventual armazenamento no dispositivo ainda não foi definido.
- O escopo inicial se limita à busca de cidades, ao clima atual, à previsão de cinco dias, à alternância de unidade e ao uso em dispositivos móveis; recursos adicionais não estão confirmados.