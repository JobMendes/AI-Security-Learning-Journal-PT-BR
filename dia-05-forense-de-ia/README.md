# Dia 05 — Forense de IA

<p align="center">
  <img src="../Pictures/Day5.png" alt="Diário de Aprendizado em Segurança de IA — Dia 05: Forense de IA" width="100%">
</p>

> A IA pode acelerar correlações e me ajudar a encontrar as evidências relevantes. Ela não pode se tornar o especialista, a evidência ou a autoridade final.

## Visão Geral

**Tempo de leitura:** cerca de 21 minutos

Este texto explora como a IA pode apoiar DFIR sem abrir mão da integridade das evidências, explicabilidade, reprodutibilidade, privacidade e responsabilidade humana.

**Principais aprendizados:**

- A saída da IA pode indicar uma linha de investigação, mas não constitui evidência forense por si só.
- Acurácia, precisão e recall representam diferentes compromissos operacionais.
- Uma conclusão defensável exige evidências preservadas, processamento documentado e validação humana.

**Caminho sugerido:** leia primeiro as seções sobre métricas e depois prossiga para cadeia de custódia, auditabilidade e as quatro perguntas sobre confiança próximas ao final.

**Navegação rápida:** [Métricas de desempenho](#métricas-de-desempenho-podem-enganar) · [Cadeia de custódia](#a-cadeia-de-custódia-não-desaparece-porque-a-ia-foi-incluída) · [Quatro perguntas sobre confiança](#minhas-quatro-perguntas-antes-de-confiar-em-ia-em-dfir) · [Principal aprendizado](#principal-aprendizado)

## Da Segurança de IA à Investigação Assistida por IA

Até este ponto da minha jornada em Segurança de IA, a maioria das perguntas era sobre o próprio sistema de IA.

Como ele funciona?

Posso confiar no modelo e nos dados?

Como ele pode ser atacado?

Como os prompts influenciam seu comportamento?

O Dia 05 mudou a perspectiva.

A pergunta passou a ser:

> **O que acontece quando realmente uso IA durante uma investigação forense digital?**

A princípio, o valor parece óbvio.

Digital Forensics and Incident Response (DFIR) pode envolver enormes volumes de dados:

* eventos de endpoints;
* logs de autenticação;
* telemetria de rede;
* e-mails;
* arquivos;
* processos;
* artefatos de memória;
* logs de nuvem;
* atividade de usuários.

Encontrar um evento importante entre milhões se torna o clássico problema de:

> **Procurar uma agulha no palheiro.**

IA e Machine Learning podem ajudar a reduzir esse palheiro.

Mas percebi neste dia que acelerar uma investigação e provar o que realmente aconteceu são problemas diferentes.

Essa distinção se tornou a base de tudo o que aprendi em seguida.

---

## Por Que a IA Faz Sentido em DFIR

DFIR tem um problema natural que a IA sabe enfrentar bem:

**escala.**

Imagine uma investigação contendo:

```text
8.000.000 de eventos
        ↓
Endpoints
Firewall
Autenticação
Aplicações
Rede
Nuvem
        ↓
Investigador Humano
```

Uma pessoa pode investigar esses eventos.

Mas examinar manualmente cada evento não é realista.

IA/ML pode ajudar a processar grandes volumes de informação, reconhecer padrões e identificar anomalias com muito mais rapidez.

Conceitualmente:

```text
Milhões de Eventos
        ↓
Análise por IA / ML
        ↓
Padrões + Anomalias + Priorização
        ↓
Superfície de Investigação Menor
        ↓
Investigação Humana
```

É aqui que vejo uma das aplicações mais fortes da IA em DFIR.

A IA não necessariamente resolve a investigação.

Ela ajuda a dizer:

> **Comece procurando aqui.**

---

## Detecção de Anomalias: Reduzindo o Palheiro

Uma das capacidades mais úteis de Machine Learning em segurança é a detecção de anomalias.

Um modelo pode aprender padrões que representam o comportamento normal de:

* usuários;
* endpoints;
* aplicações;
* redes;
* autenticações;
* processos.

Depois, pode identificar desvios desses padrões.

Por exemplo:

```text
Baseline de Comportamento do Usuário

08:00–18:00
Rede interna
Estação conhecida
Aplicações normais
Acesso típico a arquivos
```

Então, de repente:

```text
03:01
Origem externa
Várias falhas de autenticação
Login bem-sucedido
Escalonamento de privilégios
Acesso a arquivo sensível
```

O modelo não necessariamente conhece a história completa.

Mas pode reconhecer:

> **Isto não parece normal.**

Isso pode reduzir milhões de eventos a um grupo muito menor que merece investigação.

É algo extremamente valioso.

Mas também estabelece o primeiro limite importante:

> **Uma anomalia é uma linha de investigação, não uma prova automática de atividade maliciosa.**

---

## A IA Encontrar Algo Suspeito Não É o Mesmo que Provar o Que Aconteceu

Essa distinção se tornou um dos meus aprendizados mais fortes.

Imagine um modelo de ML sinalizando:

```text
/opt/company/source/product.c

Classificação: SUSPEITO
```

O investigador verifica o arquivo e descobre que é apenas código-fonte proprietário legítimo.

A IA foi inútil?

Não.

Ela produziu um **falso positivo**.

Ainda assim, direcionou a atenção para algo que parecia incomum segundo o modelo.

O investigador então forneceu o contexto que faltava ao modelo.

Isso reforçou um princípio recorrente neste diário:

> **Um indicador não é uma causa raiz.**

Em DFIR, eu ampliaria para:

> **Uma classificação suspeita não é uma conclusão forense.**

---

## Encontrar Evidências e Provar um Caso São Tarefas Diferentes

Um sistema de IA pode identificar vários artefatos suspeitos:

```text
E-mail Suspeito
Documento Suspeito
Anomalia de Autenticação
Escalonamento de Privilégios
Mecanismo de Persistência
Arquivo Compactado Sensível
```

Mas o investigador ainda precisa estabelecer relações entre eles.

Por exemplo:

```text
Phishing
   ↓
Documento Malicioso
   ↓
Roubo de Credenciais
   ↓
Acesso à Conta
   ↓
Escalonamento de Privilégios
   ↓
Persistência
   ↓
Acesso a Dados Sensíveis
   ↓
Roubo de Dados
```

O valor da IA está em ajudar a trazer as peças à superfície.

O papel do investigador é determinar se elas realmente pertencem ao mesmo quebra-cabeça.

Isso exige perguntas como:

* Quem realizou a ação?
* Quando ela aconteceu?
* Qual processo a iniciou?
* O que aconteceu imediatamente antes?
* O que aconteceu depois?
* Outro artefato confirma o mesmo evento?
* Existe uma explicação alternativa?
* A evidência está íntegra?
* A conclusão pode ser reproduzida e defendida?

Por isso, já não penso em IA em DFIR como:

```text
Evidência
   ↓
IA
   ↓
Verdade
```

Penso mais desta forma:

```text
Evidência
   ↓
Identificação Assistida por IA
   ↓
Possíveis Linhas de Investigação
   ↓
Correlação
   ↓
Validação Humana
   ↓
Conclusão Forense
```

---

## Métricas de Desempenho Podem Enganar

Outra parte importante deste dia foi aprender que um único número de desempenho pode esconder fragilidades sérias.

Imagine um conjunto de dados contendo:

```text
10.000 arquivos

9.990 benignos
10 maliciosos
```

Agora imagine um modelo péssimo que classifica tudo como benigno.

O resultado seria:

```text
9.990 benignos → BENIGNO
10 maliciosos → BENIGNO
```

Esse modelo acerta 9.990 vezes em 10.000.

Sua acurácia é:

```text
99,9%
```

Parece excelente.

Mas, do ponto de vista da segurança, o modelo deixou passar:

```text
100% dos arquivos maliciosos
```

Portanto, apesar da acurácia altíssima, falhou completamente na tarefa que realmente importava.

Isso mudou a forma como interpreto o desempenho de modelos.

> **Uma acurácia alta não significa automaticamente um modelo de segurança útil.**

---

## Acurácia

Acurácia pergunta:

> **Quantas previsões estavam corretas no total?**

Conceitualmente:

```text
Previsões Corretas
──────────────────
Total de Previsões
```

Acurácia é útil.

Mas, em um conjunto de dados altamente desbalanceado, pode criar uma imagem enganosa.

Se quase tudo for benigno, prever:

```text
BENIGNO
```

para quase tudo ainda pode gerar uma acurácia impressionante.

É por isso que outras métricas importam.

---

## Recall

Recall pergunta:

> **De todos os casos realmente positivos, quantos o modelo de fato encontrou?**

Para um detector de malware:

```text
10 amostras reais de malware
        ↓
8 detectadas
2 não detectadas
```

O recall seria:

```text
8 / 10 = 80%
```

O atalho mental que quero lembrar é:

> **Recall informa quanto da população real de ameaças consegui encontrar.**

Em segurança, um recall alto pode ser extremamente importante, pois um recall baixo significa que atividades maliciosas podem passar despercebidas.

---

## Precisão

Precisão faz outra pergunta:

> **De tudo o que o modelo classificou como positivo, quanto era realmente positivo?**

Imagine:

```text
208 arquivos sinalizados como malware

8 realmente maliciosos
200 realmente benignos
```

O modelo encontrou a maior parte dos malwares.

Mas também gerou um enorme número de falsos alarmes.

Isso significa que sua precisão é ruim.

Meu atalho mental passou a ser:

> **Recall informa quantos problemas reais encontrei.**

> **Precisão informa com que frequência acertei quando disse que havia um problema.**

---

## Precisão e Recall Representam um Compromisso Operacional

Isso ficou mais claro quando conectei as métricas às operações de SOC.

### Recall Alto, Precisão Baixa

```text
Maioria das ameaças detectada
        +
Muitos falsos positivos
```

Isso reduz a chance de ataques não serem percebidos.

Mas os analistas podem gastar bastante tempo investigando falsos alarmes.

### Precisão Alta, Recall Baixo

```text
Maioria dos alertas é real
        +
Algumas ameaças reais não são detectadas
```

Isso reduz investigações desnecessárias.

Mas atividades maliciosas podem escapar da detecção.

Em muitos cenários de DFIR e detecção, eu preferiria investigar falsos positivos adicionais a aceitar intencionalmente muitos falsos negativos.

Isso não significa que precisão seja irrelevante.

Falsos positivos em excesso podem criar:

* fadiga dos analistas;
* desperdício de tempo de investigação;
* aumento de custos;
* dessensibilização a alertas;
* resposta mais lenta a incidentes reais.

O equilíbrio correto depende da investigação e das consequências de cada tipo de erro.

---

## Falso Positivo vs. Falso Negativo

Essa distinção é importante o bastante para ficar explícita.

### Falso Positivo

```text
Realidade: BENIGNO
Modelo:    MALICIOSO
```

Possível consequência:

> investigação desnecessária.

### Falso Negativo

```text
Realidade: MALICIOSO
Modelo:    BENIGNO
```

Possível consequência:

> atividade maliciosa real permanece sem detecção.

Nenhum dos dois é desejável.

Mas eles representam riscos operacionais muito diferentes.

---

## Acurácia, Precisão e Recall Precisam de Contexto

Durante o exercício, inicialmente interpretei um desempenho ruim de classificação rápido demais como um possível problema de treinamento, como overfitting.

Essa foi outra correção útil.

Se um modelo deixa passar amostras maliciosas, isso diz algo sobre seu **desempenho**.

Não revela automaticamente a **causa raiz**.

A causa pode envolver:

* dados de treinamento;
* projeto do modelo;
* thresholds;
* viés;
* mudanças no ambiente;
* representação insuficiente;
* configuração;
* outro problema.

Novamente:

> **A degradação de desempenho é um indicador a investigar, não um diagnóstico de causa raiz por si só.**

Isso se conecta diretamente ao que aprendi nos dias anteriores.

---

## Garbage In, Garbage Out

IA não salva evidências ruins.

Parece óbvio, mas se torna especialmente importante em forense.

Imagine:

```text
Evidência Corrompida
        ↓
Modelo de IA Avançado
        ↓
Correlação Impecável
        ↓
Timeline Detalhada
        ↓
Conclusão Errada
```

Um modelo sofisticado não consegue restaurar magicamente a confiança que já foi perdida na evidência.

Se os dados de origem estiverem:

* corrompidos;
* incompletos;
* coletados incorretamente;
* manipulados;
* sem representatividade;
* interpretados incorretamente pelo parser;

a saída também poderá ser pouco confiável.

É o clássico:

> **Garbage In, Garbage Out — GIGO**

Para DFIR, interpreto isso de forma ainda mais enfática:

> **A IA pode analisar evidências mais rápido, mas não compensa evidências cuja integridade não é confiável.**

---

## O Não Determinismo Torna-se Mais Sério em Forense

O Dia 04 me ensinou que LLMs são não determinísticos.

Conceitualmente:

```text
Mesma Entrada
   ↓
Execução 1 → Saída A
Execução 2 → Saída B
```

Em um chatbot genérico, pequenas variações podem não importar muito.

Em DFIR, as implicações são diferentes.

Imagine dois investigadores processando exatamente as mesmas evidências e recebendo:

```text
Investigador A → Timeline A

Investigador B → Timeline B
```

A pergunta deixa de ser apenas:

> **Qual saída é melhor?**

E passa a ser:

> **Qual conclusão consigo reproduzir e defender?**

Isso torna o não determinismo uma preocupação forense.

O trabalho forense precisa de:

* repetibilidade;
* rastreabilidade;
* explicabilidade;
* defensibilidade.

Se um processo assistido por IA muda significativamente entre execuções, essas variações precisam ser compreendidas e controladas tanto quanto possível.

---

## Explicabilidade Importa Mais do que uma Pontuação Alta

Imagine uma IA revisando 500 mil e-mails e retornando:

```text
37 e-mails → ALTAMENTE SUSPEITOS
```

Alguém pergunta:

> **Por que estes 37?**

E o investigador responde:

> O modelo normalmente tem 97% de acurácia.

Isso não responde à pergunta.

Acurácia diz algo sobre o desempenho geral do modelo.

Explicabilidade me ajuda a compreender uma **decisão específica**.

No trabalho forense, posso precisar explicar:

* quais características influenciaram a classificação;
* quais evidências sustentaram a conclusão;
* qual processo foi utilizado;
* se explicações alternativas foram consideradas.

Isso me trouxe uma distinção importante:

> **A acurácia alta pode indicar que um modelo costuma ter bom desempenho. A explicabilidade ajuda a defender por que este resultado específico merece confiança.**

---

## Caixas-pretas Criam um Problema Forense

Alguns modelos de IA são difíceis de interpretar internamente.

Podemos compreender:

```text
Entrada
  ↓
Modelo
  ↓
Saída
```

sem conseguir explicar claramente todo o raciocínio interno por trás da decisão.

Isso cria um desafio quando a IA é usada em ambientes nos quais conclusões podem ser contestadas.

Um investigador forense não pode simplesmente dizer:

> **O algoritmo disse.**

A evidência subjacente ainda precisa sustentar a conclusão.

A saída da IA deve, portanto, conduzir o investigador a evidências que possam ser examinadas de forma independente.

---

## O Viés Pode Mudar Quais Evidências Recebem Atenção

Viés não é apenas uma questão abstrata de justiça.

Ele pode influenciar diretamente uma investigação.

Imagine um modelo treinado principalmente com comunicações em inglês.

Durante uma investigação internacional:

```text
Mensagens em Inglês
      ↓
Análise Forte

Mensagens em Português
      ↓
Frequentemente Despriorizadas

Mensagens em Espanhol
      ↓
Frequentemente Despriorizadas
```

O modelo pode não ter sido atacado.

Talvez simplesmente tenha melhor desempenho nos padrões mais bem representados no treinamento.

Ainda assim, a consequência forense pode ser séria.

Evidências relevantes podem ser:

* classificadas em posições inferiores;
* classificadas incorretamente;
* revisadas mais tarde;
* ou até ignoradas.

Isso me trouxe outra forma útil de pensar sobre viés:

> **O viés em DFIR pode influenciar quais evidências são vistas primeiro, depois ou nunca.**

Isso pode afetar toda a direção de uma investigação.

---

## O Viés da IA Pode se Tornar um Problema Real de Justiça

Em sistemas técnicos comuns, o viés pode reduzir a qualidade do modelo.

Em contextos forenses ou jurídicos, as consequências podem ir além do desempenho técnico.

Se um método assistido por IA sistematicamente apresenta desempenho pior para certas populações, idiomas, ambientes ou tipos de evidência, isso pode influenciar:

* prioridades investigativas;
* conclusões;
* decisões jurídicas;
* justiça;
* confiança na investigação.

Isso significa que a validação do modelo em DFIR deve considerar mais do que:

```text
O modelo funciona?
```

Também deve perguntar:

```text
Para quem ele funciona?

Com quais dados?

Em quais condições?

Onde ele falha?
```

---

## A Cadeia de Custódia Não Desaparece Porque a IA Foi Incluída

Cadeia de custódia é fundamental para a forense digital.

Evidências precisam ser tratadas de modo controlado e rastreável.

Introduzir IA cria etapas adicionais que talvez precisem ser documentadas.

Considere:

```text
Evidência Original
       ↓
Processamento por IA
       ↓
Saída Intermediária
       ↓
Resumo da IA
       ↓
Interpretação do Investigador
       ↓
Relatório Final
```

Se, meses depois, alguém perguntar:

> **Como você chegou da evidência original a esta conclusão?**

Preciso conseguir reconstruir o processo.

Isso pode exigir o registro de informações como:

* origem da evidência;
* verificação de integridade;
* ferramenta utilizada;
* modelo/versão;
* ambiente de processamento;
* configuração relevante;
* prompts ou instruções de análise;
* saídas intermediárias;
* timestamps;
* transformações;
* ações do investigador.

Os requisitos exatos dependerão da investigação e do ambiente jurídico.

Mas o princípio é claro:

> **A IA não deve criar uma etapa invisível dentro da cadeia de custódia.**

---

## Auditabilidade Importa

Isso me levou a pensar na forense assistida por IA como penso em qualquer outra ferramenta forense.

Se uma ferramenta modifica, transforma, interpreta ou faz parsing de evidências, preciso entender o que aconteceu.

Conceitualmente:

```text
Evidência A
   ↓
Processo Conhecido
   ↓
Transformação Documentada
   ↓
Resultado B
```

é muito mais fácil de defender do que:

```text
Evidência A
   ↓
Processo de IA Desconhecido
   ↓
Resultado B
```

Se não consigo reconstruir o processo de análise assistido por IA, talvez não consiga defender a conclusão produzida por ele.

---

## IA na Nuvem Introduz Outra Fronteira para as Evidências

Usar IA na nuvem pode ser operacionalmente atraente.

Ela pode oferecer:

* escalabilidade;
* grande capacidade de processamento;
* análise rápida;
* acesso a modelos sofisticados.

Mas evidências forenses podem conter informações extremamente sensíveis:

```text
PII
Credenciais
E-mails
Dados de Funcionários
Documentos Internos
Propriedade Intelectual
Detalhes da Investigação
```

Enviar essas informações a um serviço externo cria outra fronteira de segurança e privacidade.

Antes disso, eu precisaria entender:

* onde os dados são processados;
* se são retidos;
* quem pode acessá-los;
* como são protegidos;
* se podem ser reutilizados;
* qual jurisdição se aplica;
* quais proteções contratuais existem;
* se o processamento é legalmente permitido.

Uma excelente análise de IA não justifica criar um incidente de confidencialidade ou privacidade durante o processo.

---

## Privacidade Faz Parte da Integridade Forense

Essa foi outra mudança de perspectiva para mim.

Antes, eu pensava principalmente se a saída da IA estava tecnicamente correta.

Mas uma investigação também precisa considerar se o **processo usado para obter essa saída foi apropriado**.

Conceitualmente:

```text
Análise Tecnicamente Excelente
              +
Tratamento Inadequado de Evidências Sensíveis
              =
Problema Potencialmente Sério
```

A qualidade da resposta não apaga a forma como a evidência foi tratada.

---

## Ambientes Controlados Fazem Mais Sentido em Investigações Sensíveis

Para evidências altamente sensíveis, ambientes controlados de IA podem reduzir algumas dessas preocupações.

Dependendo da situação, isso pode significar:

```text
Evidência Forense
       ↓
Ambiente Controlado / Aprovado
       ↓
Processamento Assistido por IA
       ↓
Saídas Registradas
       ↓
Validação Humana
```

em vez de:

```text
Evidência Forense
       ↓
Serviço Público Desconhecido
       ↓
Retenção / Processamento Desconhecidos
       ↓
Saída da IA
```

A principal lição para mim não é:

> **IA na nuvem é sempre errada.**

É:

> **O ambiente de processamento torna-se parte da avaliação de riscos forenses e de privacidade.**

---

## A IA Não É Dona da Conclusão

Esta talvez seja a distinção mais importante de todo o dia.

Imagine perguntarem a um investigador:

> **Foi a IA que determinou que o suspeito realizou a ação?**

A resposta não deveria ser:

> Sim, a IA concluiu isso.

Um modelo mais defensável é:

```text
Ferramenta Assistida por IA
       ↓
Identificou Artefatos Potencialmente Relevantes
       ↓
Investigador Examinou as Evidências Originais
       ↓
Correlações Validadas Independentemente
       ↓
Investigador Chegou à Conclusão
```

A IA pode apoiar a investigação.

O investigador continua responsável por interpretar e validar as evidências.

Isso é importante técnica, ética e profissionalmente.

---

## IA como Assistente de Investigação

Meu modelo mental depois do Dia 05 tornou-se:

```text
IA ≠ Investigador
IA ≠ Perito
IA ≠ Juiz
IA ≠ Verdade
```

Em vez disso:

```text
IA
 ↓
Assistente de Investigação
 ↓
Reconhecimento de Padrões
Detecção de Anomalias
Priorização
Correlação
Classificação
Resumo
 ↓
Investigador Humano
 ↓
Validação
Contexto
Julgamento
Responsabilidade
Conclusão
```

É aqui que vejo o verdadeiro valor.

A IA pode ampliar o alcance do investigador.

Não deve substituir sua responsabilidade.

---

## Por Que a Validação Humana Continua Essencial

A validação humana não é necessária apenas porque:

> **A IA às vezes comete erros.**

O motivo é mais amplo.

O investigador humano oferece elementos que o modelo talvez não possua:

### Contexto

O modelo pode sinalizar código-fonte proprietário como suspeito.

O investigador sabe por que o arquivo existe.

### Validação de Evidências

O modelo pode descrever uma relação.

O investigador a verifica nos artefatos originais.

### Hipóteses Alternativas

O modelo pode identificar uma explicação plausível.

O investigador considera se outra explicação também corresponde às evidências.

### Conhecimento Jurídico e Processual

O investigador compreende requisitos de tratamento de evidências, restrições de privacidade e procedimentos investigativos.

### Responsabilidade

No fim, alguém precisa assumir e defender a conclusão.

Isso não pode ser simplesmente delegado ao:

> **Algoritmo.**

---

## A IA Pode Encontrar a Agulha Mais Rápido

A metáfora clássica de DFIR é:

> **Encontrar uma agulha no palheiro.**

Depois do Dia 05, eu a modificaria.

A IA pode ajudar:

```text
Palheiro Enorme
     ↓
IA / ML
     ↓
Palheiro Muito Menor
     ↓
Investigador Humano
     ↓
Agulha
```

Isso é extremamente valioso.

Mas há outra etapa:

```text
Agulha Encontrada
     ↓
Ela é relevante?
     ↓
É autêntica?
     ↓
Como se relaciona com outras evidências?
     ↓
O que realmente prova?
```

Essas perguntas ainda exigem investigação forense.

---

## Um Modelo Pode Ser Útil sem Ser Perfeito

O exemplo do falso positivo me ajudou a entender outra coisa.

Se a IA sinaliza um arquivo legítimo como suspeito, não devo concluir imediatamente:

```text
O modelo inteiro é inútil.
```

Em vez disso, devo avaliar:

* desempenho geral;
* precisão;
* recall;
* taxa de falsos positivos;
* consequências dos falsos negativos;
* contexto;
* carga operacional;
* objetivos da investigação.

Ferramentas de segurança sempre exigiram ajuste e interpretação.

Ferramentas assistidas por IA não são exceção.

A diferença importante é lembrar que sua saída pode parecer extremamente confiante mesmo quando está errada.

---

## Confiança Não É Evidência

Essa distinção merece seu próprio espaço.

Um sistema de IA pode produzir:

```text
Avaliação: MALICIOSO
Confiança: 98%
```

Esse número não substitui evidências.

O investigador ainda precisa de:

```text
Afirmação
  ↓
Artefato de Suporte
  ↓
Validação Independente
  ↓
Correlação
  ↓
Conclusão
```

Uma pontuação de confiança é uma informação sobre a avaliação do modelo.

Não é prova de que o evento aconteceu.

---

## A Evidência Relatada pela IA Ainda Precisa Existir

Isso se conecta diretamente ao Dia 04.

Uma resposta estruturada da IA pode dizer:

```text
Evidências:
- autenticação suspeita
- escalonamento de privilégios
- mecanismo de persistência
- arquivo compactado sensível criado
```

Essa estrutura é útil.

Mas cada afirmação ainda precisa ser verificada.

```text
Afirmação da IA
   ↓
Evidência Original
   ↓
Confirmada?
```

Pedir evidências à IA facilita a validação.

Não torna uma evidência gerada automaticamente verdadeira.

---

## A IA Pode Acelerar a Correlação

Um dos maiores benefícios que vejo para IA em DFIR é a correlação.

Um investigador humano pode precisar conectar:

```text
E-mail
 ↓
Anexo
 ↓
Execução de Processo
 ↓
Autenticação
 ↓
Escalonamento de Privilégios
 ↓
Criação de Arquivo
 ↓
Atividade de Rede
```

A IA pode ajudar a trazer essas relações à superfície muito mais rápido.

Mas a própria correlação ainda precisa ser verificada.

Dois eventos próximos no tempo não significam automaticamente que um causou o outro.

Novamente:

> **A correlação ajuda a construir a teoria. As evidências validam a teoria.**

---

## Reprodutibilidade e Defensibilidade

Uma das minhas respostas neste dia veio de pensar sobre a natureza científica da investigação forense.

Uma conclusão não deve depender apenas de:

> **Confie em mim. A IA disse.**

A metodologia precisa ser compreensível e defensável.

Isso se torna desafiador com sistemas probabilísticos.

Se:

```text
Evidência X
   ↓
Execução 1 da IA
   ↓
Conclusão A
```

e:

```text
Evidência X
   ↓
Execução 2 da IA
   ↓
Conclusão B
```

o investigador precisa entender o motivo.

Isso não significa necessariamente que a IA não possa ser usada.

Significa que métodos assistidos por IA exigem cuidado adicional com:

* documentação;
* configuração;
* versionamento;
* logging;
* validação;
* repetibilidade;
* interpretação.

Quanto mais grave a consequência da conclusão, maior a necessidade de defensibilidade.

---

## Saída da IA e Evidência Forense Não São a Mesma Coisa

Essa distinção se tornou cada vez mais importante para mim.

Suponha que a IA analise logs de autenticação e diga:

> **O atacante obteve acesso às 03:01.**

A afirmação da IA não é necessariamente a evidência primária.

O registro de autenticação subjacente é.

Um modelo mental mais seguro é:

```text
Log de Autenticação
       ↓
Evidência

Interpretação da IA
       ↓
Assistência à Investigação
```

O investigador pode usar a interpretação da IA para localizar e compreender o registro relevante.

Mas a conclusão deve permanecer fundamentada na evidência subjacente.

---

## O Que Acontece Quando o Modelo Erra?

Este dia também mudou minha forma de pensar sobre erros de IA.

Se um modelo produz um falso positivo:

```text
Arquivo Legítimo → SUSPEITO
```

o resultado pode gerar investigação adicional.

Se produz um falso negativo:

```text
Arquivo Malicioso → BENIGNO
```

a evidência talvez nunca receba atenção.

Essas consequências não são equivalentes.

Portanto, a avaliação do modelo não deve perguntar apenas:

> **Com que frequência ele erra?**

Também deve perguntar:

> **O que acontece quando ele erra?**

Isso se conecta a outro princípio dos dias anteriores:

> **Risco não é apenas probabilidade. O impacto também importa.**

---

## Desempenho da IA Não É Confiança Forense

O Dia 03 me ensinou:

> **Desempenho do modelo e confiança no modelo não são a mesma coisa.**

O Dia 05 ampliou esse princípio.

Um modelo pode apresentar ótimo desempenho em benchmarks e ainda criar problemas forenses se:

* suas decisões não puderem ser explicadas;
* seus dados forem enviesados;
* o tratamento das evidências não for documentado;
* o processamento intermediário for perdido;
* evidências sensíveis forem compartilhadas de forma inadequada;
* os resultados não puderem ser reproduzidos de forma significativa;
* investigadores aceitarem suas conclusões sem validação.

Portanto, em DFIR:

```text
Alto Desempenho
      ≠
Confiança Forense
```

Confiança exige mais do que uma métrica.

---

## Minhas Quatro Perguntas Antes de Confiar em IA em DFIR

Ao final deste dia, resumi o problema em quatro perguntas.

### 1. É útil?

A IA realmente me ajuda a:

* processar evidências;
* identificar anomalias;
* correlacionar eventos;
* priorizar a investigação;
* reduzir o trabalho manual?

### 2. É confiável?

O que sei sobre:

* acurácia;
* precisão;
* recall;
* falsos positivos;
* falsos negativos;
* viés;
* qualidade da entrada?

### 3. É verificável?

Consigo:

* inspecionar a evidência subjacente;
* compreender o resultado;
* reproduzir o processo;
* auditar o que aconteceu;
* explicar a conclusão?

### 4. É defensável?

Se alguém contestar o resultado, consigo demonstrar:

* integridade das evidências;
* metodologia;
* cadeia de custódia;
* tratamento adequado;
* validação humana?

Para mim, essas quatro perguntas oferecem uma estrutura muito mais forte do que simplesmente perguntar:

> **A IA funciona?**

---

## O Que Mudou no Meu Entendimento

Antes do Dia 05, a proposta de valor parecia direta:

> **A IA pode processar evidências mais rápido do que os humanos.**

Isso continua verdadeiro.

Mas é apenas o começo.

Agora penso em DFIR assistido por IA como um equilíbrio entre:

```text
Velocidade
Escala
Reconhecimento de Padrões
Correlação
```

e:

```text
Integridade
Explicabilidade
Viés
Privacidade
Cadeia de Custódia
Reprodutibilidade
Julgamento Humano
Responsabilidade
```

O primeiro grupo explica por que quero a IA.

O segundo determina se posso usar responsavelmente o que ela me fornece.

---

## O Investigador Continua Responsável pela Investigação

Esta é a mudança mais importante em meu modelo mental.

A IA pode me ajudar a:

* pesquisar;
* classificar;
* priorizar;
* correlacionar;
* resumir;
* detectar anomalias.

Mas o investigador humano ainda precisa:

* verificar;
* contextualizar;
* questionar;
* reproduzir;
* documentar;
* interpretar;
* concluir.

Minha forma de expressar isso passou a ser:

> **Em forense digital, a IA deve ser tratada como assistente de investigação e acelerador de correlações, não como o especialista ou juiz que determina a verdade.**

---

## Principal Aprendizado

Se eu tivesse de resumir o Dia 05 em uma ideia:

> **A IA pode me ajudar a encontrar a agulha mais rápido. Não pode decidir sozinha se essa agulha prova o caso.**

Ou, de uma perspectiva operacional:

```text
Evidência
   ↓
Análise Assistida por IA
   ↓
Possíveis Descobertas
   ↓
Validação Humana
   ↓
Conclusão Defensável
```

A IA pode acelerar a correlação.

O investigador valida as evidências e assume a conclusão.

Essa distinção torna a IA útil na Forense Digital sem permitir que a conveniência substitua a disciplina forense.

---

## Próximo

A próxima parte desta jornada continuará com base no próximo tema de Segurança de IA que eu estudar.

Em vez de definir a conclusão antecipadamente, quero que cada novo dia comece pelo mesmo processo:

**Aprender → Questionar → Entender → Aplicar → Compartilhar**

---

## Módulo 1 — Verificação de Domínio

Com o primeiro módulo de aprendizado concluído, também completei sua **Verificação de Domínio com sucesso, acertando todas as respostas**.

A avaliação retomou vários conceitos explorados ao longo dos Dias 01–05, incluindo:

- conjuntos de validação e seu papel na identificação de overfitting;
- viés como preocupação ética e também como possível questão jurídica em investigações;
- roubo de modelos por meio de consultas repetidas a APIs e replicação comportamental;
- reinforcement learning por meio de recompensas e penalidades;
- aplicações práticas de capacidades defensivas de IA em fluxos de trabalho de SOC.

Mais importante do que a pontuação, o checkpoint ajudou a confirmar que conceitos apresentados separadamente durante o primeiro módulo estão começando a se conectar.

O comportamento do modelo depende dos dados e do treinamento.

O desempenho do modelo precisa de validação.

Sistemas de IA introduzem novas superfícies de ataque.

Investigações assistidas por IA ainda exigem validação humana.

E decisões de segurança não podem depender somente da saída do modelo.

Meu maior aprendizado ao concluir este primeiro módulo é:

> **Comportamento do modelo, qualidade dos dados, ameaças de segurança, investigação e validação humana não são tópicos isolados de Segurança de IA. São partes diferentes do mesmo sistema.**

Isso encerra oficialmente:

> **Módulo 1 — Fundamentos de IA**

A próxima etapa da jornada deixa a compreensão de como a IA funciona, de onde vêm seus riscos e como ela pode apoiar a segurança cibernética para se voltar a uma nova pergunta:

> **Como realmente protegemos sistemas de IA?**

Esse será o ponto de partida de:

**Módulo 2 — Sistemas de IA Seguros**

---

## Referências

- [NIST — Forense Digital](https://www.nist.gov/itl/ai/digital-forensics)
- [NIST — Programa de Testes de Ferramentas Forenses Computacionais](https://www.nist.gov/itl/csd/secure-systems-and-applications/computer-forensics-tool-testing-program-cftt)
- [NIST — Quatro Princípios da Inteligência Artificial Explicável](https://www.nist.gov/publications/four-principles-explainable-artificial-intelligence)
- [NIST — AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)

---

## Sobre Este Diário de Aprendizado

Este repositório documenta minha jornada pessoal de aprendizado e minhas reflexões durante os estudos de Segurança de IA.

A jornada foi inspirada pelos meus estudos com o material de **AI Security da TryHackMe**, combinados à minha experiência anterior em segurança cibernética e gestão de incidentes.

As explicações, analogias, exemplos e conclusões apresentados aqui representam meu próprio entendimento e minhas reflexões.

Este repositório não reproduz laboratórios, perguntas, soluções, flags ou conteúdo proprietário da TryHackMe.

**Aprender → Questionar → Entender → Aplicar → Compartilhar**
