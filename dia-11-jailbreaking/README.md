# Dia 11 — Jailbreaking

<p align="center">
  <img src="../Pictures/Day11.png" alt="Diário de Aprendizado em Segurança de IA — Dia 11: Jailbreaking" width="100%">
</p>

> Jailbreaking tem como alvo o comportamento de segurança do modelo; tornar-se ou não um comprometimento do sistema depende da arquitetura e das capacidades ao redor.

## Visão Geral

**Tempo de leitura:** cerca de 22 minutos

Esta publicação diferencia jailbreaking de prompt injection e estuda os padrões de persuasão usados para influenciar o comportamento probabilístico de segurança.

**Principais aprendizados:**

- Um jailbreak bem-sucedido é uma falha dos controles de segurança, não automaticamente um comprometimento da infraestrutura.
- A variação semântica torna insuficientes as blocklists simples de strings.
- Os testes devem medir o comportamento do modelo enquanto controles arquiteturais contêm suas consequências.

**Caminho sugerido:** comece pela comparação com prompt injection e depois leia a taxonomia de técnicas e as implicações defensivas.

**Navegação rápida:** [O verdadeiro alvo do jailbreaking](#o-verdadeiro-alvo-do-jailbreaking) · [Comparação com prompt injection](#prompt-injection-vs-jailbreaking) · [Segurança semântica](#jailbreaking-é-um-problema-de-segurança-semântica) · [Perguntas sobre risco](#minhas-quatro-perguntas-ao-avaliar-o-risco-de-jailbreak)

## Da Segurança de Instruções à Segurança do Modelo

O Dia 10 mudou a maneira como eu entendia Prompt Injection.

Aprendi que uma aplicação com LLM tem um problema singular de confiança:

**Conteúdo Não Confiável → Contexto da LLM → Comportamento → Capacidades → Ativos**

Um PDF, e-mail, site, documento de RAG ou saída de ferramenta pode ter como finalidade fornecer informações, mas conteúdo controlado por um atacante pode tentar influenciar o modelo como se esse conteúdo tivesse autoridade de instrução.

Isso me levou a uma distinção importante:

> **Prompt Injection é, sobretudo, um problema de confiança no nível da aplicação.**

O Dia 11 apresentou uma pergunta diferente.

O que acontece quando o atacante não tenta principalmente manipular a relação entre instruções e dados da aplicação, mas busca mudar o próprio comportamento de segurança do modelo?

É aí que entra o **Jailbreaking**.

Meu modelo mental passou a ser:

**Prompt Injection → Confiança da Aplicação**

**Jailbreaking → Segurança do Modelo**

Eles podem aparecer na mesma cadeia de ataque, mas não são o mesmo problema de segurança.

---

## O Verdadeiro Alvo do Jailbreaking

Um jailbreak tenta fazer um modelo se comportar fora dos limites de segurança que foi treinado para seguir.

Conceitualmente:

**Solicitação do Usuário**

↓

**Modelo Alinhado à Segurança**

↓

**Recusa Esperada**

Um jailbreak tenta transformar isso em:

**Contexto Elaborado**

↓

**Comportamento de Segurança Manipulado**

↓

**Conformidade Inesperada**

O ponto importante é que o atacante tem como alvo o **comportamento do modelo alinhado à segurança**.

Isso é diferente de simplesmente atacar uma aplicação ao redor.

---

## Prompt Injection vs Jailbreaking

Essa distinção ficou muito mais clara depois que estudei os dois tópicos.

### Prompt Injection

Prompt Injection tem como principal alvo a relação entre:

**Instruções**

e:

**Conteúdo Não Confiável**

dentro de uma aplicação com LLM.

O ataque tenta influenciar o comportamento da aplicação por meio de instruções controladas pelo atacante.

### Jailbreaking

Jailbreaking tem como principal alvo:

**O comportamento de segurança do modelo**

O atacante tenta fazer o modelo fornecer algo que seu alinhamento de segurança normalmente o levaria a recusar.

Minha distinção simplificada é:

> **Prompt Injection tem como alvo a confiança da aplicação. Jailbreaking tem como alvo a segurança do modelo.**

---

## Eles Ainda Podem Existir na Mesma Cadeia de Ataque

Separar os conceitos não significa que eles não possam interagir.

Imagine:

**Usuário**

↓

**Agente de LLM**

↓

**Ferramentas**

↓

**Sistemas Internos**

Um atacante poderia primeiro tentar aplicar jailbreak no modelo.

Se tiver sucesso:

**Jailbreak**

↓

**Comportamento de Segurança Contornado**

O atacante poderia então tentar influenciar a forma como a aplicação ao redor se comporta.

Dependendo da arquitetura, a cadeia maior poderia se tornar:

**Jailbreaking**

↓

**Prompt Injection**

↓

**Abuso de Capacidade**

↓

**Ferramenta Sensível**

↓

**Ativo Sensível**

↓

**Impacto nos Negócios**

Se o agente tiver capacidades desnecessárias ou excessivamente amplas, a **Agência Excessiva** pode ampliar as consequências.

Isso significa que descrever todo o incidente apenas como um "jailbreak" pode ocultar o restante do caminho de ataque.

---

## A Prisão Não É um Limite Físico

Um dos conceitos mais importantes deste Dia foi entender o que a "prisão" realmente representa.

É fácil imaginar um mecanismo de segurança de IA como algo semelhante a:

```python
if request_is_harmful:
    refuse()
```

Mas esse modelo mental é simplista demais.

O comportamento de segurança é aprendido por meio de treinamento e alinhamento.

O modelo aprende padrões associados a:

**Solicitações Permitidas**

**Solicitações Proibidas**

**Respostas Seguras**

**Recusas**

**Assistência Alternativa**

O comportamento resultante é probabilístico.

Isso significa que uma recusa não equivale necessariamente a uma regra de segurança codificada de forma rígida.

---

## Recusas São Comportamentos Aprendidos

Um modelo mental simplificado é:

**Entrada**

+

**Contexto**

+

**Alinhamento de Segurança**

↓

**Distribuição de Probabilidade**

↓

**Resposta**

Para uma solicitação claramente prejudicial, o modelo pode favorecer fortemente uma recusa.

Conceitualmente:

**P(recusa) → alta**

**P(conformidade) → baixa**

Um jailbreak tenta manipular o contexto para que essa distribuição mude.

Conceitualmente:

**Contexto Elaborado**

↓

**P(recusa) diminui**

**P(conformidade) aumenta**

Isso me trouxe uma das conclusões mais fortes do Dia 11:

> **Um jailbreak não quebra uma regra codificada de forma rígida. Ele manipula um comportamento probabilístico treinado para agir como se fosse uma.**

---

## Por Que o Enquadramento Importa

Um modelo pode recusar uma solicitação quando ela é expressa diretamente.

Mas mudar o contexto ao redor pode ativar padrões aprendidos diferentes.

Por exemplo, há uma diferença conceitual importante entre:

**Enquadramento prejudicial direto**

e:

**Enquadramento fictício / histórico / analítico / alternativo**

O tema subjacente pode continuar semelhante, mas o ambiente semântico mudou.

Isso significa que o modelo não está simplesmente procurando um único verbo ou palavra-chave proibida.

Ele interpreta todo o contexto.

Meu modelo mental anterior se concentrava demais em palavras isoladas.

Um modelo melhor é:

**Solicitação Direta**

↓

**Contexto Associado à Segurança**

↓

**Recusa Provável**

em comparação com:

**Enquadramento Alternativo**

↓

**Contexto Aprendido Diferente**

↓

**Distribuição de Probabilidade Diferente**

A superfície de ataque é, portanto, semântica.

---

## O Equilíbrio entre Utilidade e Inofensividade

Espera-se que assistentes de IA sejam úteis.

Mas também se espera que evitem comportamentos prejudiciais.

Esses objetivos podem entrar em conflito.

Considere um extremo:

**Inofensividade Máxima**

↓

**Recusar qualquer coisa remotamente perigosa**

↓

**Muitas solicitações legítimas são rejeitadas**

Um estudante de segurança que pergunte sobre SQL Injection, análise de malware ou técnicas de exploração poderia receber recusas desnecessárias.

O sistema seria seguro em um sentido restrito, mas muito menos útil.

Agora considere o extremo oposto:

**Utilidade Máxima**

↓

**Responder a tudo**

↓

**Solicitações prejudiciais também são atendidas**

O modelo se torna muito útil, mas facilmente sujeito a uso indevido.

O desafio é equilibrar:

**Utilidade ↔ Inofensividade**

Esse não é um problema trivial de engenharia.

---

## Custo do Alinhamento

Aumentar a segurança pode introduzir um custo.

Uma solicitação legítima pode ser rejeitada porque o modelo a interpreta de forma conservadora demais.

Conceitualmente:

**Solicitação Legítima**

↓

**Mecanismo de Segurança**

↓

**Falso Positivo**

↓

**Recusa Desnecessária**

↓

**Perda de Utilidade**

Entendo **alignment tax** como a perda de capacidade útil ou de desempenho do modelo introduzida pelo esforço para torná-lo mais seguro.

Isso está relacionado ao equilíbrio entre utilidade e inofensividade, mas não é exatamente o mesmo conceito.

O equilíbrio descreve a tensão.

O custo do alinhamento é parte do custo que pode surgir ao administrar essa tensão.

---

## Jailbreaking É um Problema de Segurança Semântica

Controles de segurança tradicionais muitas vezes funcionam bem com propriedades determinísticas.

Um firewall pode avaliar:

**IP**

**Porta**

**Protocolo**

Um sistema de autorização pode avaliar:

**Identidade**

**Função**

**Permissão**

Mas a linguagem é flexível.

A mesma intenção pode ser expressa usando:

**vocabulário diferente**

**estrutura diferente**

**contexto diferente**

**idioma diferente**

**codificação diferente**

**enquadramento narrativo diferente**

Isso dificulta a detecção de jailbreaks.

O limite de segurança existe parcialmente em um espaço semântico probabilístico.

---

## Roleplay

Uma família de técnicas de jailbreak usa roleplay.

Pode-se pedir que o modelo se comporte como:

**um personagem fictício**

**uma personalidade histórica**

**outra IA**

**um sistema simulado**

**um personagem dentro de uma história**

Por que isso poderia influenciar o comportamento?

Porque os dados de treinamento contêm enormes quantidades de:

**histórias**

**diálogos**

**ficção**

**personagens**

**interações baseadas em papéis**

O modelo aprendeu padrões fortes para manter a coerência narrativa e do personagem.

O atacante tenta explorar esses padrões.

Conceitualmente:

**Solicitação Restrita**

↓

**Contexto de Roleplay**

↓

**Pressão por Coerência com o Papel**

↓

**Distribuição de Probabilidade Diferente**

↓

**Possível Bypass de Segurança**

O modelo não precisa acreditar literalmente que o cenário fictício é real.

O próprio enquadramento pode influenciar a probabilidade da continuação gerada.

---

## Coerência com o Papel

Depois que um modelo aceita uma identidade ou um cenário fictício, as respostas subsequentes podem tender a permanecer coerentes com isso.

Por exemplo:

**Estabelecer Personagem**

↓

**Estabelecer Regras do Mundo Fictício**

↓

**Pedir ao Personagem que Continue**

O contexto anterior agora influencia a próxima saída.

Isso cria uma relação importante entre:

**Roleplay**

e:

**Coerência**

O modelo tenta produzir uma continuação compatível com o contexto estabelecido.

Esse comportamento pode se tornar útil para um atacante que tenta enfraquecer o comportamento de segurança.

---

## Enquadramento Emocional

Outra técnica usa o contexto emocional em vez de se concentrar principalmente em uma identidade fictícia.

O atacante pode enquadrar uma solicitação em torno de:

**compaixão**

**urgência**

**medo**

**nostalgia**

**dificuldade pessoal**

**ajudar outra pessoa**

O objetivo é criar um ambiente semântico fortemente associado a um comportamento prestativo.

Conceitualmente:

**Objetivo Restrito**

+

**Contexto Emocional**

↓

**Comportamento Prestativo se Torna Mais Provável**

Isso não significa que o modelo "sinta" a emoção.

Significa que a linguagem emocional pode influenciar os padrões aprendidos ativados pelo contexto.

---

## Roleplay vs Manipulação Emocional

Essas técnicas estão relacionadas, mas não são idênticas.

Minha distinção é:

**Roleplay**

→ manipula identidade e cenário.

**Manipulação Emocional**

→ manipula o enquadramento emocional ao redor da solicitação.

Elas também podem ser combinadas.

Por exemplo:

**Personagem Fictício**

+

**História Emocional**

+

**Objetivo Restrito**

↓

**Tentativa Combinada de Jailbreak**

Isso demonstra por que jailbreaks costumam ser mais bem compreendidos como combinações de técnicas, e não como prompts mágicos isolados.

---

## Ofuscação

Outra família de técnicas tenta representar conteúdo restrito de maneiras incomuns.

Exemplos conceituais incluem:

**Substituição de caracteres**

**Fragmentação de palavras**

**Codificações alternativas**

**Formatação incomum**

**Idiomas com poucos recursos**

A ideia de segurança importante não é uma representação específica.

É a possível lacuna entre:

**O que o modelo consegue interpretar**

e:

**O que o treinamento de segurança ou os mecanismos de filtragem reconhecem de forma confiável**

---

## A Lacuna na Distribuição de Segurança

Durante o treinamento, os dados normalmente são limpos, rotulados e estruturados.

O treinamento de segurança também tenta expor o modelo a exemplos de comportamento indesejável.

Mas as possíveis formas pelas quais os seres humanos podem representar a linguagem são enormes.

Um atacante pode, portanto, explorar representações incomuns que continuam compreensíveis para o modelo, mas interagem de modo diferente com os mecanismos de segurança.

Meu modelo mental passou a ser:

**Capacidade de Interpretação do Modelo**

pode ser mais ampla que:

**Cobertura de Reconhecimento de Segurança**

Essa lacuna se torna uma superfície de ataque.

---

## Ofuscação É uma Técnica, Não a Classificação Final da Vulnerabilidade

Essa distinção é importante.

Suponha que alguém transforme uma palavra sensível usando:

**Substituição de caracteres**

ou:

**Fragmentação de palavras**

Isso, por si só, não me diz se o ataque geral é:

**Prompt Injection**

ou:

**Jailbreaking**

A transformação é a **técnica**.

A classificação do ataque depende do limite de segurança que está sendo visado.

Por exemplo:

**Ofuscação**

↓

**Usada para contornar a segurança do modelo**

↓

**Jailbreaking**

ou:

**Ofuscação**

↓

**Usada para ocultar instruções maliciosas na entrada da aplicação**

↓

**Prompt Injection**

Portanto:

> **Técnica e objetivo do ataque não devem ser confundidos.**

---

## Intercalação de Instruções

Outra técnica interessante é a intercalação de instruções.

Conceitualmente:

**Instrução Benigna**

↓

**Instrução Benigna**

↓

**Instrução Restrita**

↓

**Instrução Benigna**

O atacante cerca um objetivo problemático com tarefas legítimas.

O objetivo é dificultar que o modelo mantenha um limite de segurança coerente em todo o conjunto de instruções.

Isso reforça um ponto importante:

> **O modelo avalia o contexto, não frases isoladas.**

---

## Jailbreaking em Múltiplos Turnos

Um jailbreak não precisa acontecer em uma única mensagem.

Em vez disso:

**Turno 1 → Benigno**

**Turno 2 → Benigno**

**Turno 3 → Limítrofe**

**Turno 4 → Mais Restrito**

O atacante molda a conversa gradualmente.

Isso pode ser mais eficaz do que começar diretamente com a solicitação restrita final, pois as interações anteriores passam a fazer parte do contexto do modelo.

Assim, o ataque usa a própria conversa como estado temporário.

---

## O Histórico da Conversa Faz Parte da Superfície de Ataque

Cada premissa aceita pode influenciar gerações posteriores.

Conceitualmente:

**Conformidade Inicial**

↓

**Acúmulo de Contexto**

↓

**Escalada Incremental**

↓

**Pressão por Coerência**

↓

**Possível Bypass de Segurança**

Isso me lembrou de algo que aprendi durante Prompt Injection:

**O contexto não é apenas memória para uma conversa útil.**

Ele também pode se tornar parte da superfície de ataque.

---

## Viés de Coerência

No início, pensei no viés de coerência principalmente como o acúmulo de muitas solicitações positivas no contexto.

Uma interpretação melhor é mais precisa.

Viés de coerência é a tendência do modelo de continuar produzindo saídas coerentes com premissas, papéis, posições ou respostas já estabelecidos anteriormente na conversa.

Conceitualmente:

**Modelo aceita a premissa**

↓

**Premissa entra no histórico da conversa**

↓

**Solicitação posterior referencia a premissa aceita**

↓

**Modelo sofre pressão contextual para manter a coerência**

O ataque não consiste simplesmente em preencher a janela de contexto.

Ele molda aquilo que o modelo já aceitou.

---

## Construção de Confiança

A construção de confiança começa com uma interação normal.

O atacante pode primeiro fazer perguntas legítimas.

O modelo responde normalmente.

A conversa estabelece um padrão benigno.

Então, o atacante começa a avançar em direção ao objetivo restrito.

Meu modelo mental simplificado:

**Interação Legítima**

↓

**Contexto Estabelecido**

↓

**Manipulação Posterior**

---

## Escalada Gradual

Em vez de solicitar imediatamente o objetivo restrito, o atacante se aproxima dele aos poucos.

Conceitualmente:

**Seguro**

↓

**Um Pouco Sensível**

↓

**Limítrofe**

↓

**Restrito**

Cada etapa tenta aproximar o modelo do objetivo final sem provocar uma recusa imediata.

---

## Modelagem do Contexto

A modelagem do contexto constrói um cenário ao redor que muda a forma como a solicitação final é interpretada.

O atacante pode usar:

**ficção**

**análise**

**simulação**

**contexto histórico**

**cenários hipotéticos**

A solicitação-alvo é então incorporada a esse contexto construído.

O ataque não consiste necessariamente em mudar uma palavra.

Consiste em mudar o ambiente semântico ao redor do objetivo.

---

## Frases-Gatilho

Um prompt posterior pode fazer referência a algo que o modelo já aceitou.

Conceitualmente:

**Contexto Aceito Anteriormente**

↓

**"Continue de onde paramos"**

↓

**Modelo usa o estado anterior**

O atacante tenta aproveitar o contexto previamente estabelecido em vez de reafirmar o objetivo restrito desde o início.

---

## Retorno e Adaptação

Uma recusa não encerra necessariamente uma conversa adversarial.

O atacante pode tratá-la como feedback.

Conceitualmente:

**Tentativa**

↓

**Recusa**

↓

**Identificar o Limite**

↓

**Retornar a um Estado Aceito Anteriormente**

↓

**Mudar o Enquadramento**

↓

**Tentar um Caminho Alternativo**

Isso se assemelha a testes de segurança iterativos.

O atacante aprende com cada falha.

---

## Jailbreaking como Processo Adaptativo

Isso mudou a forma como penso sobre testes de jailbreak.

O atacante não está necessariamente procurando um único prompt perfeito.

Em vez disso:

**Sondar**

↓

**Observar**

↓

**Adaptar**

↓

**Sondar Novamente**

↓

**Aprender o Limite**

↓

**Refinar**

Isso faz o jailbreaking parecer muito mais uma pesquisa de segurança adversarial do que simplesmente engenharia de prompts.

---

## Sementes Venenosas

Outro conceito de múltiplos turnos é plantar gradualmente ideias ou premissas dentro da conversa.

Essas ideias podem inicialmente parecer inofensivas.

Prompts posteriores podem fazer referência a elas.

Conceitualmente:

**Semente**

↓

**Aceitação**

↓

**Semente Adicional**

↓

**Reforço do Contexto**

↓

**Ativação Posterior**

Isso pode criar um caminho gradual em direção a um comportamento que poderia ter sido recusado se solicitado diretamente.

---

## Sementes Venenosas Não São Envenenamento de Dados

A terminologia pode ser confusa.

Esses são conceitos diferentes.

### Envenenamento de Dados

**Dados de Treinamento / Conhecimento**

↓

**Dados Manipulados**

↓

**Comportamento do Modelo/Sistema Influenciado**

As informações manipuladas entram em um dataset, uma fonte de conhecimento ou outro pipeline persistente de dados.

### Sementes Venenosas em Jailbreaking

**Conversa em Runtime**

↓

**Contexto Gradualmente Moldado**

↓

**Comportamento Posterior Influenciado**

O ataque ocorre por meio do estado da conversa.

Portanto:

> **Sementes venenosas manipulam o contexto em runtime. Envenenamento de dados manipula dados usados pelo sistema de IA.**

Manter esses conceitos separados é importante para uma modelagem de ameaças precisa.

---

## O Fenômeno DAN

DAN se tornou historicamente interessante não apenas por causa de um único prompt.

A lição maior é o que aconteceu ao redor dele.

Uma comunidade começou a experimentar maneiras de convencer modelos a se comportarem fora dos limites de segurança pretendidos.

O processo se tornou iterativo:

**Bypass Descoberto**

↓

**Compartilhado**

↓

**Modificado**

↓

**Mitigação do Provedor**

↓

**Nova Variante**

↓

**Nova Mitigação**

Isso criou um ciclo adversarial.

---

## Jailbreaking como uma Corrida Armamentista

O fenômeno DAN me lembrou muito a segurança cibernética tradicional.

Conceitualmente:

**Exploit**

↓

**Patch**

↓

**Bypass**

↓

**Detecção**

↓

**Evasão**

↓

**Nova Mitigação**

Jailbreaking pode seguir um ciclo semelhante:

**Jailbreak**

↓

**Atualização do Modelo**

↓

**Nova Variante de Jailbreak**

↓

**Novo Treinamento de Segurança**

↓

**Novo Bypass**

A superfície de ataque é diferente.

A dinâmica adversarial é familiar.

---

## A Experimentação da Comunidade Muda o Cenário de Ameaças

Depois que técnicas de jailbreak são compartilhadas publicamente, os atacantes não precisam descobrir cada ideia de forma independente.

A experimentação da comunidade cria:

**Conhecimento Compartilhado**

↓

**Iteração Rápida**

↓

**Combinação de Técnicas**

↓

**Descoberta Mais Rápida de Fraquezas**

Essa é outra semelhança com a pesquisa tradicional em segurança ofensiva.

Os defensores não estão competindo contra um único atacante estático.

Eles estão respondendo à experimentação coletiva.

---

## Vazamento de Prompt Não É Jailbreaking

Outra classificação importante surgiu durante a sessão de domínio do assunto.

Imagine que um usuário manipule uma aplicação até ela revelar seu system prompt.

A consequência de segurança é:

**Vazamento de Prompt**

A técnica usada para produzir essa consequência pode envolver:

**Prompt Injection**

Conceitualmente:

**Prompt Injection**

↓

**Comportamento da Aplicação Manipulado**

↓

**Vazamento de Prompt**

Isso é diferente de:

**Jailbreaking**

↓

**Comportamento de Segurança Contornado**

↓

**Produção de Conteúdo Normalmente Recusado**

Essa distinção entre:

**Técnica**

e:

**Consequência de Segurança**

é importante.

---

## Técnica, Vulnerabilidade e Impacto São Camadas Diferentes

Agora prefiro separar a análise em camadas.

Por exemplo:

**Técnica**

→ Ofuscação

**Alvo**

→ Segurança do modelo

**Classe de Ataque**

→ Jailbreaking

**Resultado**

→ Bypass de segurança

**Possível Impacto**

→ Conteúdo prejudicial ou abuso posterior da aplicação

Ou:

**Técnica**

→ Instrução elaborada

**Alvo**

→ Limite entre instruções e dados da aplicação

**Classe de Ataque**

→ Prompt Injection

**Resultado**

→ Manipulação do comportamento da aplicação

**Possível Impacto**

→ Vazamento de Prompt / Abuso de Capacidade / Exposição de Dados

Isso produz descrições de ameaças muito mais claras.

---

## Jailbreaking Não Significa Automaticamente Comprometimento do Sistema

Um jailbreak bem-sucedido demonstra que o comportamento de segurança esperado do modelo foi contornado.

Isso não prova automaticamente que:

**um banco de dados foi acessado**

**uma ferramenta foi executada**

**dados foram exfiltrados**

**um sistema de produção foi modificado**

Esses fatos exigem evidências adicionais.

É aqui que minha mentalidade de DFIR se torna importante.

---

## Tentativa, Sucesso e Impacto São Coisas Diferentes

Agora separo:

**Tentativa de Jailbreak**

↓

**Bypass de Segurança Bem-Sucedido**

↓

**Ação Posterior**

↓

**Impacto no Mundo Real**

Cada etapa exige evidências diferentes.

Um atacante pode tentar um jailbreak e falhar.

Um jailbreak pode ter sucesso sem causar qualquer ação externa.

Um modelo pode produzir uma saída inesperada enquanto todos os limites de segurança da aplicação permanecem intactos.

Ou um jailbreak bem-sucedido pode se tornar o início de uma cadeia de ataque maior.

---

## Investigando uma Tentativa de Jailbreak

As evidências podem incluir:

**Logs de conversa**

**Histórico de prompts**

**Respostas do modelo**

**Telemetria de segurança/recusa**

**Metadados da sessão**

O objetivo é reconstruir:

**O que o usuário enviou?**

**Como o modelo respondeu?**

**Como a interação evoluiu?**

**Houve escalada gradual?**

**Houve ofuscação?**

**O comportamento de recusa mudou?**

---

## Investigando um Impacto Posterior Bem-Sucedido

Se o modelo também tiver ferramentas ou capacidades de aplicação, a investigação deve continuar além da conversa.

Minha cadeia de evidências passa a ser:

**Conversa**

↓

**Trace da LLM**

↓

**Resposta do Modelo**

↓

**Decisão de Ferramenta**

↓

**Chamada de Ferramenta**

↓

**Aplicação/API**

↓

**Ativo-Alvo**

↓

**Efeito Observado**

Assim, as evidências podem incluir:

**Logs de conversa**

**Traces da LLM**

**Logs de chamadas de ferramentas**

**Logs de identidade**

**Logs de autorização**

**Logs da aplicação**

**Logs de API**

**Logs de auditoria do banco de dados**

**Telemetria de rede**

**Reconstrução da linha do tempo**

Uma ação prejudicial bem-sucedida deve ser demonstrada por meio de evidências, não inferida apenas porque ocorreu um jailbreak.

---

## Correlação Não É Causalidade

Esta é outra lição que quero preservar da Forense de IA.

Suponha:

**Tentativa de jailbreak observada**

e, posteriormente:

**API sensível acessada**

Isso, por si só, não prova que:

**O jailbreak causou o acesso à API**

A investigação precisa estabelecer a cadeia causal.

Conceitualmente:

**Entrada do Usuário**

↓

**Comportamento do Modelo**

↓

**Decisão de Ferramenta**

↓

**Execução Autorizada**

↓

**Efeito no Alvo**

Somente então o incidente pode ser descrito de maneira defensável.

---

## Um Forte Alinhamento de Segurança Não É Suficiente

Uma arquitetura perigosa seria:

**Banco de Dados Sensível**

↓

**Ferramenta Privilegiada**

↓

**LLM**

↓

**"O modelo deve recusar solicitações maliciosas."**

Isso responsabiliza a segurança do modelo por proteger ativos críticos.

Isso não é suficiente.

Por quê?

Porque o modelo continua probabilístico.

Seu comportamento pode variar conforme:

**contexto**

**enquadramento**

**idioma**

**histórico da conversa**

**codificação**

**atualizações do modelo**

**técnicas adversariais**

O alinhamento de segurança é valioso.

Mas ele não deve se tornar o único limite de segurança.

---

## Segurança do Modelo Não É Autorização

Isso se conecta diretamente ao Dia 10.

Nem mesmo um modelo extremamente bem alinhado substitui:

**Identidade**

**Autenticação**

**Autorização**

**Privilégio Mínimo**

**Políticas de Ferramentas**

**Controles de Acesso a Dados**

**Controles de Egress**

**Auditoria**

**Aprovação Humana**

Portanto:

> **O alinhamento de segurança do modelo é uma camada defensiva, não um sistema de autorização.**

---

## Agentes Aumentam as Consequências Novamente

Um modelo somente de texto pode produzir uma resposta indesejável após um jailbreak bem-sucedido.

Uma aplicação agêntica pode fazer muito mais.

Considere:

**Jailbreak**

↓

**Bypass de Segurança**

↓

**Solicitação de Ferramenta pelo Agente**

↓

**API Sensível**

↓

**Ação Real**

Agora, o problema passou de:

**Saída Insegura**

para:

**Impacto Operacional**

A presença de ferramentas muda drasticamente o risco.

---

## Abuso de Capacidade

O abuso de capacidade ocorre quando uma capacidade legítima é usada para uma finalidade não pretendida ou não autorizada.

Por exemplo, um agente pode possuir legitimamente a capacidade de:

**pesquisar incidentes**

**ler documentos**

**consultar APIs**

**criar tickets**

**enviar mensagens**

A capacidade em si é legítima.

O problema é como ela está sendo exercida.

Um jailbreak ou Prompt Injection pode influenciar o agente a usar essa capacidade fora do fluxo de trabalho pretendido.

---

## Agência Excessiva Amplifica o Impacto

Se o agente tiver mais autoridade do que o necessário, as consequências se tornam maiores.

Por exemplo:

**Capacidade Necessária**

→ Ler incidente

mas:

**Capacidades Concedidas**

→ Ler + Modificar + Excluir + Exportar + Executar

A diferença cria uma superfície de ataque desnecessária.

Isso reforça o princípio:

> **O privilégio mínimo também se aplica a agentes de IA.**

---

## A Defesa Deve Existir Fora do Modelo

A aplicação deve presumir que o modelo pode acabar produzindo uma saída inesperada.

Então, a arquitetura deve perguntar:

**Essa saída pode acionar diretamente uma ação sensível?**

Uma arquitetura mais segura se parece mais com:

**Usuário**

↓

**LLM**

↓

**Ação Proposta**

↓

**Aplicação Independente de Políticas**

↓

**Autorização**

↓

**Ferramenta**

↓

**Recurso-Alvo**

O modelo pode propor.

A arquitetura de segurança decide se a proposta é permitida.

---

## Defesa em Profundidade para Sistemas Resistentes a Jailbreaks

Meu modelo defensivo agora inclui várias camadas:

**Alinhamento de Segurança**

↓

**Controles de Entrada / Contexto**

↓

**Autorização de Ferramentas**

↓

**Privilégio Mínimo**

↓

**Controles de Dados Sensíveis**

↓

**Controles de Saída / Egress**

↓

**Aprovação Humana para Ações de Alto Impacto**

↓

**Registro e Monitoramento**

O objetivo não é presumir que jailbreaks nunca possam acontecer.

O objetivo é reduzir as consequências caso a segurança do modelo falhe.

---

## A Detecção Deve Ir Além de Palavras-Chave

As técnicas de jailbreak podem variar semanticamente.

Portanto, procurar apenas frases associadas a jailbreaks conhecidos terá cobertura limitada.

A detecção também pode considerar padrões comportamentais como:

**Recusas repetidas seguidas de reformulação**

**Variação rápida de prompts**

**Escalada progressiva**

**Mudanças de papel**

**Codificações incomuns**

**Tentativas repetidas em torno do mesmo objetivo restrito**

**Mudanças abruptas da recusa para a conformidade**

Isso não significa que todo padrão desse tipo seja malicioso.

O contexto ainda importa.

Mas esses comportamentos podem fornecer sinais úteis para investigação.

---

## A Telemetria de Recusa Pode se Tornar Telemetria de Segurança

Uma ideia interessante de Blue Team é tratar o comportamento de recusa do modelo como um sinal de segurança.

Por exemplo:

**Solicitação**

↓

**Recusa**

↓

**Solicitação Reformulada**

↓

**Recusa**

↓

**Solicitação Ofuscada**

↓

**Recusa**

↓

**Enquadramento Alternativo**

↓

**Conformidade**

Essa transição pode merecer investigação.

O sinal importante não é necessariamente uma palavra-chave maliciosa.

É o padrão de interação adversarial.

---

## De Prompt Injection a Jailbreaking

Depois dos Dias 10 e 11, agora separo os dois tópicos usando este modelo:

### Prompt Injection

**Principal problema de segurança:**

Confiança da aplicação.

**Pergunta:**

> Instruções controladas por um atacante podem influenciar o comportamento pretendido da aplicação?

### Jailbreaking

**Principal problema de segurança:**

Segurança do modelo.

**Pergunta:**

> O atacante consegue deslocar o modelo da recusa esperada em direção ao comportamento que o alinhamento de segurança pretendia impedir?

Essa distinção torna a modelagem de ameaças muito mais clara.

---

## Meu Modelo Atualizado de Ataques de IA

Agora penso em várias camadas de forma independente:

**Técnica**

↓

**Alvo**

↓

**Classe de Ataque**

↓

**Resultado Comportamental**

↓

**Capacidade**

↓

**Ativo**

↓

**Impacto**

Por exemplo:

**Ofuscação**

↓

**Segurança do Modelo**

↓

**Jailbreaking**

↓

**Bypass de Segurança**

↓

**Capacidade do Agente**

↓

**Sistema Sensível**

↓

**Impacto nos Negócios**

Ou:

**Instrução Externa Elaborada**

↓

**Limite de Confiança da Aplicação**

↓

**Prompt Injection**

↓

**Manipulação de Comportamento**

↓

**Capacidade de Ferramenta**

↓

**Dados Sensíveis**

↓

**Divulgação de Informações**

Esse modelo me impede de usar um único termo de segurança para descrever todo um ataque em múltiplas etapas.

---

## O Que Mudou no Meu Entendimento

Antes do Dia 11, minha definição era aproximadamente:

> **Jailbreaking significa romper um limite ou manipular o modelo para que ele se comporte de maneira diferente.**

Isso estava correto em termos gerais.

Mas estava incompleto.

Agora entendo o limite com mais precisão.

O atacante tenta subverter o **comportamento do modelo alinhado à segurança**.

O modelo foi treinado para que certos contextos favoreçam fortemente a recusa.

O jailbreak tenta remodelar esse contexto.

Isso é fundamentalmente diferente de contornar uma permissão determinística do sistema operacional.

---

## De Regras Rígidas a Comportamentos Probabilísticos

Essa provavelmente foi a maior mudança conceitual.

A segurança tradicional costuma me ensinar a pensar em:

**Permitir**

ou:

**Negar**

A segurança de IA introduz algo mais probabilístico:

**Maior probabilidade de cumprir**

ou:

**Maior probabilidade de recusar**

Isso muda a maneira como os testes adversariais funcionam.

O atacante pode experimentar com:

**enquadramento**

**contexto**

**papéis**

**idioma**

**histórico da conversa**

**representação**

O objetivo é mudar a distribuição de probabilidade do comportamento.

---

## De um Único Prompt a um Processo Adversarial

Também deixei de pensar em jailbreaks como strings mágicas.

Um modelo melhor é:

**Sondar**

↓

**Observar**

↓

**Adaptar**

↓

**Escalar**

↓

**Retroceder**

↓

**Reenquadrar**

↓

**Tentar Novamente**

A conversa se torna um ciclo de feedback adversarial.

Isso faz a pesquisa de jailbreaks se parecer muito mais com a metodologia tradicional de segurança ofensiva do que eu esperava inicialmente.

---

## Da Segurança do Modelo à Segurança do Sistema

Por fim, um jailbreak bem-sucedido não significa automaticamente um comprometimento bem-sucedido.

O impacto real depende do que existe ao redor do modelo.

Minha cadeia final é:

**Jailbreak**

↓

**Bypass de Segurança**

↓

**O Que o Modelo Pode Alcançar?**

↓

**O Que o Agente Pode Fazer?**

↓

**Que Autorização Existe?**

↓

**Que Controles Independentes Existem?**

↓

**Qual Ativo Está Exposto?**

↓

**Que Impacto nos Negócios É Possível?**

Isso reconecta o Dia 11 a tudo o que foi aprendido no módulo anterior.

**A arquitetura ainda determina o impacto.**

---

## Minhas Quatro Perguntas ao Avaliar o Risco de Jailbreak

Depois deste Dia, quero fazer quatro perguntas.

### 1. Qual comportamento de segurança está sendo visado?

O que o modelo normalmente recusaria?

### 2. Como o atacante está mudando o contexto?

Roleplay?

Ofuscação?

Escalada gradual?

Enquadramento emocional?

Modelagem do contexto?

### 3. O que acontece se o jailbreak tiver sucesso?

O modelo apenas gera texto?

Ou pode invocar ferramentas e interagir com sistemas reais?

### 4. Quais controles de segurança ainda existem fora do modelo?

Autorização?

Privilégio mínimo?

Políticas de ferramentas?

Aprovação humana?

Logging?

Controles de egress?

A quarta pergunta é especialmente importante.

Um jailbreak não deveria se transformar automaticamente em comprometimento do sistema.

---

## Principais Aprendizados

- Prompt Injection e Jailbreaking têm como alvo limites de segurança diferentes.
- Prompt Injection diz respeito principalmente à confiança no nível da aplicação.
- Jailbreaking diz respeito principalmente ao comportamento de segurança no nível do modelo.
- Recusas de segurança são comportamentos aprendidos, e não regras tradicionais de autorização codificadas de forma rígida.
- Contexto e enquadramento podem mudar o comportamento do modelo.
- Roleplay pode explorar padrões aprendidos ligados à narrativa e à coerência de papéis.
- O enquadramento emocional pode influenciar comportamentos associados à prestatividade.
- Ofuscação é uma técnica, não uma classificação de vulnerabilidade por si só.
- Ataques em múltiplos turnos usam o histórico da conversa como parte da superfície de ataque.
- O viés de coerência pode fazer premissas aceitas anteriormente influenciarem respostas posteriores.
- Sementes venenosas em jailbreaking conversacional não são o mesmo que envenenamento de dados.
- Jailbreaking é um processo adversarial adaptativo.
- A experimentação da comunidade cria uma corrida armamentista entre bypasses e mitigações.
- Um jailbreak bem-sucedido não prova automaticamente um comprometimento posterior.
- Segurança do modelo não substitui autorização.
- Capacidades agênticas podem transformar bypasses de segurança em risco operacional.
- Privilégio mínimo e autorização independente de ferramentas continuam essenciais.
- Blue Teams devem investigar sequências comportamentais, não apenas strings conhecidas de jailbreak.

---

## Meu Maior Aprendizado

A lição mais importante que levo do Dia 11 é:

> **Um jailbreak não quebra uma regra codificada de forma rígida. Ele manipula um comportamento probabilístico treinado para agir como se fosse uma.**

E quando esse modelo passa a integrar um sistema agêntico, outro princípio se torna igualmente importante:

> **Uma falha na segurança do modelo não deveria se transformar automaticamente em uma falha de autorização.**

O alinhamento de segurança protege o comportamento do modelo.

A arquitetura de segurança protege os sistemas.

Precisamos de ambos.

---

## Próximo

O Dia 11 aprofundou meu entendimento de **Segurança de Prompts** ao separar a confiança da aplicação da segurança do modelo.

O próximo passo é continuar explorando como a Segurança de Prompts pode ser defendida sistematicamente.

As perguntas que levo comigo são:

**Como nos defendemos de ataques semânticos quando o atacante pode reformular indefinidamente a mesma intenção?**

**Como testamos a segurança do modelo sem confundir um jailbreak bem-sucedido com um comprometimento real do sistema?**

**Como projetamos aplicações agênticas para que as falhas do modelo permaneçam contidas?**

A jornada continua.

---

## Referências

- [OWASP — LLM01: Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP — Guia de Red Teaming para GenAI](https://genai.owasp.org/resource/genai-red-teaming-guide/)
- [NIST — Taxonomia de Machine Learning Adversarial](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)

---

## Sobre Este Diário de Aprendizado

Este diário documenta meu entendimento e minhas reflexões pessoais durante meus estudos de Segurança de IA.

Minha jornada de aprendizado inclui a **trilha de aprendizagem AI Security da TryHackMe**, combinada com minha própria experiência em segurança cibernética, perguntas, exemplos, correções e interpretações.

Ele não reproduz laboratórios, flags, credenciais, respostas de avaliações, soluções de desafios ou material de treinamento proprietário.

Meu objetivo continua sendo:

**Aprender → Questionar → Entender → Aplicar → Compartilhar**
