# Dia 17 — Data Poisoning em Sistemas RAG

<p align="center">
  <img src="../Pictures/Day17.png" alt="AI Security Learning Journal — Dia 17: Data Poisoning em Sistemas RAG" width="100%">
</p>

> **O controle sobre os dados dos quais um sistema de IA aprende ou que recupera pode se tornar controle sobre seu comportamento — sem atacar diretamente o modelo ou o prompt do usuário.**

## Em resumo

**Tempo de leitura:** cerca de 28 minutos

Este texto examina como adversários podem influenciar o comportamento da IA envenenando dados de treinamento, pipelines de ingestão, corpora de retrieval e as informações selecionadas para o contexto de RAG.

**Principais conclusões:**

- Training poisoning altera os parâmetros aprendidos; retrieval poisoning altera o contexto no momento da inferência.
- Um Retriever pode operar corretamente enquanto um processo inseguro de elegibilidade ou ingestão permite que conteúdo manipulado domine os resultados.
- Provenance, monitoramento comportamental, autorização independente e evidências do incidente são necessários porque o poisoning pode permanecer sutil e operacionalmente plausível.

**Caminho sugerido:** comece pela distinção entre training e retrieval poisoning; depois acompanhe as seções sobre ingestão, corpus flooding, investigação e arquitetura defensiva.

**Navegação rápida:** [Training vs retrieval poisoning](#training-poisoning-vs-retrieval-poisoning) · [Corpus flooding](#corpus-flooding) · [Investigação](#investigando-um-possível-poisoning) · [Arquitetura defensiva](#arquitetura-defensiva-prática) · [Principais conclusões](#principais-conclusões)

---

## Introdução

O Day 16 mudou a forma como penso sobre Retrieval-Augmented Generation.

Aprendi que o conhecimento externo se torna parte da cadeia de inferência e que a relevância semântica não estabelece confiança automaticamente.

O Day 17 levou essa ideia adiante.

A pergunta já não era apenas:

> **Posso confiar no que o sistema RAG recupera?**

Ela passou a ser:

> **O que acontece quando alguém manipula deliberadamente as informações das quais o sistema de IA pode aprender, indexar, recuperar ou nas quais pode confiar?**

É aqui que data poisoning se torna especialmente importante.

Um atacante talvez não precise comprometer o servidor do modelo.

Talvez não precise modificar o código da aplicação.

Talvez não precise roubar os model weights.

Talvez nem precise interagir com o chatbot.

Se puder influenciar os dados que participarão posteriormente do treinamento ou da inferência, poderá influenciar o comportamento de forma indireta.

Isso faz do poisoning, fundamentalmente, um **problema de integridade**.

O ataque mira o ecossistema de informações ao redor da IA.

---

## Da segurança de RAG ao RAG poisoning

Meu modelo mental do Day 16 era:

```text
Conhecimento Externo
        ↓
Ingestão
        ↓
Vector Store
        ↓
Retrieval
        ↓
Contexto
        ↓
LLM
        ↓
Saída
```

O Day 17 adiciona um adversário a essa arquitetura:

```text
                    Atacante
                       │
                       ↓
Conhecimento Externo → Ingestão → Vector Store → Retrieval → Contexto → LLM
                       │              │             │
                       └──────────────┴─────────────┘
                                  Influência
                                      ↓
                                  Comportamento
```

Isso cria uma observação importante de segurança:

> **O modelo pode permanecer intacto enquanto o comportamento do sistema passa a ser influenciado pelo atacante.**

Mas nem todo ataque de poisoning funciona na mesma camada.

Entender onde ocorre a manipulação é fundamental.

---

## Poisoning é uma classe de ataque, não apenas dados ruins

Sistemas de IA naturalmente encontram informações imperfeitas.

Um documento pode estar desatualizado.

Um funcionário pode escrever algo incorreto.

Um dataset pode conter erros de rotulagem.

Uma política pode continuar indexada depois de ser substituída.

Esses são problemas sérios de qualidade de dados e governança.

Mas eu não chamaria automaticamente todos eles de poisoning.

A distinção que uso agora é a **intenção adversarial**.

Considere três situações.

### Cenário A — Informação desatualizada

Um procedimento antigo de segurança continua disponível depois que um novo procedimento o substitui.

O sistema recupera a versão antiga.

O resultado está incorreto.

Mas não há evidência de que alguém tenha manipulado deliberadamente o sistema.

### Cenário B — Erro humano

Um funcionário documenta acidentalmente um limite incorreto.

O documento é ingerido automaticamente.

Mais uma vez, o sistema pode produzir recomendações incorretas.

Mas o erro, por si só, não estabelece intenção maliciosa.

### Cenário C — Manipulação deliberada

Alguém introduz intencionalmente uma informação incorreta porque sabe que ela influenciará futuras respostas da IA.

Agora temos:

```text
Intenção
  +
Manipulação
  +
Influência Comportamental Esperada
```

Isso é claramente poisoning.

Essa distinção importa durante a resposta a incidentes.

> **Comportamento incorreto da IA não é automaticamente evidência de poisoning.**

O comportamento é um sintoma.

A atribuição exige investigação.

---

## Training data poisoning

Training data poisoning tem como alvo aquilo que o modelo **aprende**.

Considere um modelo fine-tuned usando tickets históricos de um SOC.

Um atacante obtém a capacidade de introduzir exemplos cuidadosamente manipulados no dataset de fine-tuning.

Conceitualmente:

```text
Dados de Treinamento Legítimos
          +
   Amostras Envenenadas
          ↓
      Fine-Tuning
          ↓
   Atualizações de Pesos
          ↓
   Comportamento Modificado
```

O atacante não precisa editar diretamente os model weights.

Em vez disso, o processo de treinamento realiza a modificação.

A influência maliciosa entra pelos dados.

---

## Por que o training poisoning pode persistir

Isso produz uma diferença importante em relação ao retrieval poisoning.

Depois do treinamento:

```text
Dados Envenenados
     ↓
Atualizações de Gradiente
     ↓
Parâmetros Aprendidos
     ↓
Comportamento do Modelo
```

Remover o documento original envenenado não necessariamente reverte o efeito.

O modelo não manteve simplesmente um ponteiro para o arquivo malicioso.

O processo de treinamento usou a informação para atualizar os parâmetros aprendidos.

Isso significa que a remediação pode exigir:

- identificar os dados de treinamento contaminados;
- determinar quais versões do modelo os consumiram;
- reconstruir ou limpar o dataset;
- treinar novamente ou restaurar o modelo afetado;
- validar o comportamento outra vez;
- identificar as implantações downstream.

Isso pode transformar um único evento de poisoning em um problema de integridade duradouro.

---

## Training poisoning vs retrieval poisoning

Essa distinção se tornou uma das partes mais importantes do Day 17.

### Training poisoning

Altera o que o modelo **aprende**.

```text
Dataset Envenenado
      ↓
Treinamento / Fine-Tuning
      ↓
Pesos Alterados
      ↓
Influência Aprendida Persistente
```

### Retrieval poisoning

Altera o que o modelo **recebe durante a inferência**.

```text
Corpus Envenenado
      ↓
Retrieval
      ↓
Contexto Malicioso / Enganoso
      ↓
Inferência do LLM
```

O base model pode permanecer completamente inalterado.

Minha forma mais simples de lembrar a distinção é:

> **Training poisoning altera o que o modelo aprendeu. Retrieval poisoning altera o que o modelo vê.**

---

## Conhecimento envenenado não significa necessariamente um modelo envenenado

Essa terminologia importa.

Suponha que uma organização use um LLM comercial fechado por meio de uma API.

A organização não pode modificar os model weights.

Mas mantém sua própria knowledge base de RAG.

Um atacante adiciona documentos manipulados a esse corpus.

O assistente começa a fornecer informações incorretas.

Seria impreciso dizer imediatamente:

> “O modelo foi envenenado.”

O modelo pode estar completamente intacto.

Uma descrição mais precisa é:

```text
Modelo
  ↓
Inalterado

Corpus de Conhecimento
  ↓
Comprometido

Contexto Recuperado
  ↓
Manipulado

Comportamento do Sistema
  ↓
Afetado
```

O **sistema de IA** tem um problema de integridade sem exigir o comprometimento do modelo subjacente.

---

## Embeddings e retrieval

Sistemas RAG normalmente transformam documentos em embeddings.

Conceitualmente:

```text
Documento
   ↓
Embedding Model
   ↓
Representação Vetorial
   ↓
Vector Database
```

Uma user query segue um processo semelhante:

```text
User Query
    ↓
Embedding
    ↓
Busca por Similaridade
    ↓
Documentos Top-k
```

Esses documentos se tornam candidatos ao contexto do modelo.

Isso cria outra superfície de ataque.

---

## A influência dos controles de similaridade

Um Retriever normalmente não pergunta:

> “Qual documento é verdadeiro?”

Ele pergunta algo mais próximo de:

> “Qual vetor está mais próximo desta query?”

Essa distinção é crítica.

Imagine:

```text
Query
  ↓
Busca por Similaridade

Documento A — Procedimento Aprovado       0.91
Documento B — Procedimento Manipulado     0.96
Documento C — Procedimento Histórico      0.89
```

Se o ranking for baseado principalmente em similaridade, o Documento B pode vencer.

O Retriever pode estar funcionando exatamente como projetado enquanto o **sistema ainda produz um resultado inseguro**.

Isso reforça o princípio do Day 16:

> **Relevância semântica não é autoridade.**

---

## Quando o Retriever funciona corretamente e o sistema ainda falha

Essa foi uma mudança mental importante para mim.

Se documentos maliciosos ocupam as posições semanticamente mais relevantes, devolvê-los não significa necessariamente que o algoritmo de retrieval foi explorado.

O algoritmo pode calcular corretamente:

```text
Documento Envenenado
        =
Maior Similaridade
```

A falha arquitetural mais profunda pode ser que o sistema permitiu que um documento não confiável se tornasse elegível para participar dessa competição.

Isso muda a pergunta defensiva de:

> **Como tornar a busca por similaridade mais inteligente?**

para:

> **Quais documentos devem poder participar da busca por similaridade?**

---

## Corpus poisoning

Corpus poisoning tem como alvo a coleção de documentos disponível para retrieval.

Os documentos legítimos não precisam necessariamente ser alterados.

Em vez disso, documentos controlados pelo atacante podem competir com eles.

Conceitualmente:

```text
Corpus Legítimo
      +
Documentos Controlados pelo Atacante
      ↓
Vector Database
      ↓
Competição por Similaridade
      ↓
Top-k
```

O atacante vence influenciando o que é exibido.

---

## Corpus flooding

Uma técnica especialmente interessante é corpus flooding.

Imagine que um documento legítimo descreva o processo aprovado de recuperação de contas.

Um atacante não consegue modificá-lo nem excluí-lo.

Em vez disso, introduz muitos documentos semanticamente semelhantes contendo um processo sutilmente enfraquecido.

```text
Documento Legítimo
        ●

Documentos do Atacante
   ● ● ● ● ●
  ● ● ● ● ● ●
   ● ● ● ● ●
```

O objetivo é criar uma região densa, controlada pelo atacante, ao redor de queries que provavelmente serão usadas pelos funcionários.

O ponto importante é que a repetição aqui não está necessariamente ensinando o modelo.

Os pesos não precisam mudar.

Em vez disso:

> **Mais candidatos controlados pelo atacante aumentam a probabilidade de que conteúdo controlado pelo atacante ocupe os resultados top-k do retrieval.**

Esse é um problema de retrieval, não necessariamente um problema de aprendizado.

---

## Densidade, não aprendizado

Essa distinção evita um erro conceitual fácil.

Se eu inserir quarenta documentos semelhantes em um vector database, o base LLM não “aprende automaticamente a ideia quarenta vezes”.

Em vez disso:

```text
Mais Documentos Semelhantes
        ↓
Maior Densidade Vetorial Local
        ↓
Mais Candidatos Próximos da Query
        ↓
Maior Presença no Top-k
        ↓
Maior Influência no Contexto
```

O ataque manipula a probabilidade de retrieval.

Isso é fundamentalmente diferente de amostras envenenadas repetidas participando de atualizações de gradiente durante o treinamento.

---

## Mimetismo semântico

Um atacante também pode tentar fazer com que o conteúdo malicioso se pareça com material confiável.

Isso pode incluir a imitação de:

- vocabulário;
- estrutura documental;
- terminologia;
- formatação;
- linguagem do domínio;
- user queries prováveis.

O objetivo não é apenas enganar um revisor humano.

Isso também pode aumentar a proximidade semântica em relação às queries-alvo.

Isso cria uma combinação perigosa:

```text
Parece Legítimo para Humanos
          +
Parece Relevante para Embeddings
          ↓
Alta Probabilidade de Influência
```

---

## A relevância pode ser transformada em arma

Isso se conecta diretamente a algo que aprendi no Day 16.

Um sistema RAG frequentemente trata a relevância como algo útil.

Um atacante pode tratar a relevância como um primitivo de ataque.

Em vez de lutar contra o algoritmo de retrieval, o atacante pode otimizar o conteúdo para ele.

O atacante pergunta:

> **Que conteúdo o Retriever considerará altamente relevante para as queries que quero influenciar?**

O problema de segurança, portanto, não é apenas conteúdo malicioso.

É a **relevância projetada maliciosamente**.

---

## Pipelines de ingestão

Antes que um documento possa influenciar o retrieval, normalmente ele precisa entrar no sistema.

Um pipeline típico pode ser:

```text
Armazenamento Corporativo
       ↓
Coleta
       ↓
Parsing
       ↓
Chunking
       ↓
Embedding
       ↓
Indexação
       ↓
Vector Database
```

Esses processos frequentemente são automatizados.

A automação melhora escalabilidade e atualização.

Mas também pode escalar erros de confiança.

---

## A ingestão é uma fronteira de segurança

Antes deste Day, seria fácil pensar na ingestão como um processo de engenharia.

Agora vejo de outra forma.

> **O pipeline deixa de ser apenas uma rotina de engenharia e se torna uma superfície de ataque porque determina do que a IA pode aprender ou o que pode recuperar.**

Se o pipeline transforma automaticamente:

```text
Pode Escrever um Documento
```

em:

```text
Pode Influenciar a IA
```

então uma decisão importante de autorização foi escondida dentro da automação.

Essa é uma fronteira de segurança.

---

## Autorizado a escrever não significa autorizado a influenciar a IA

Imagine que um funcionário tenha legitimamente permissão para enviar documentos a uma pasta corporativa compartilhada.

Um atacante compromete a conta desse funcionário.

Do ponto de vista do sistema de storage:

```text
Usuário Autenticado
      ↓
Escrita Autorizada
      ↓
Operação Permitida
```

O RBAC pode estar funcionando corretamente.

Mas então:

```text
Documento Enviado
      ↓
Ingestão Automática
      ↓
Embedding
      ↓
Elegível para Retrieval
```

A arquitetura criou implicitamente outra permissão:

```text
Autorizado a Escrever
        =
Autorizado a Influenciar a IA
```

Essas permissões não deveriam necessariamente ser equivalentes.

> **Autorizado a escrever ≠ Autorizado a influenciar o contexto da IA.**

---

## Comprometimento de identidade e RAG poisoning

Nesse cenário, a primeira falha de segurança pode estar completamente fora do RAG.

Por exemplo:

```text
Roubo de Credencial
      ↓
Identidade Confiável Comprometida
      ↓
Acesso Autorizado ao Storage
      ↓
Documento Malicioso
      ↓
Ingestão Automática
      ↓
Corpus Envenenado
```

Isso demonstra por que threat modelling de IA ainda exige cybersecurity tradicional.

Segurança de identidade, controle de acesso, logging, proteção de credenciais e resposta a incidentes continuam fazendo parte de AI Security.

A IA cria novas relações de confiança.

Ela não remove as antigas.

---

## Aprovação como uma fronteira de segurança separada

Uma arquitetura mais forte poderia separar a capacidade de enviar conteúdo da capacidade de tornar esse conteúdo authoritative.

Por exemplo:

```text
Documento Enviado
       ↓
Conteúdo Candidato
       ↓
Validação Automatizada
       ↓
Revisão Humana / por Pares
       ↓
Aprovação
       ↓
Corpus Elegível para Retrieval
```

Isso introduz outro controle independente.

Comprometer um único escritor já não compromete automaticamente a knowledge base.

Para sistemas de maior risco, a aprovação poderia exigir evidências adicionais:

- provenance confiável;
- ownership conhecido;
- classificação esperada;
- verificação de integridade;
- status no lifecycle;
- justificativa da mudança;
- peer review.

O objetivo não é a prevenção perfeita.

É impedir que uma única falha de confiança se propague automaticamente.

---

## A automação pode multiplicar o risco de poisoning

A automação, por si só, não é a vulnerabilidade.

O problema é **confiança automatizada sem validação suficiente**.

Considere:

```text
Atacante Adiciona um Documento
        ↓
Job Agendado Detecta a Mudança
        ↓
Parser o Processa
        ↓
Chunker o Divide
        ↓
Embedding Model o Codifica
        ↓
Vector Database o Armazena
        ↓
Retriever o Exibe
        ↓
Muitos Usuários Recebem sua Influência
```

Cada componente técnico pode estar funcionando corretamente.

A automação simplesmente propaga com eficiência o erro original de confiança.

Este é um padrão conhecido de cybersecurity:

> **A automação escala tanto bons controles quanto más premissas.**

---

## Poisoning não precisa derrubar o sistema

Um ataque de poisoning pode ser difícil de perceber justamente porque tudo continua funcionando.

A aplicação permanece disponível.

O vector database responde.

O Retriever retorna resultados.

O LLM gera respostas fluentes.

A latência permanece normal.

O monitoramento da infraestrutura continua verde.

A falha é semântica e comportamental.

```text
Saúde da Infraestrutura
       =
Normal

Integridade Comportamental
       =
Comprometida
```

Por isso o monitoramento tradicional de uptime, sozinho, é insuficiente.

---

## Poisoning óbvio

Alguns efeitos de poisoning são fáceis de perceber.

Exemplos:

- mudanças dramáticas de persona;
- afirmações obviamente incorretas;
- recomendações extremas;
- linguagem inesperada;
- respostas claramente inseguras.

Essas falhas chamam atenção.

Isso pode torná-las mais fáceis de detectar.

---

## Poisoning sutil

A ameaça mais interessante é a manipulação sutil.

Imagine uma recomendação de segurança mudando gradualmente:

```text
Semana 1 → Limite de isolamento: 80%
Semana 2 → Limite de isolamento: 82%
Semana 3 → Limite de isolamento: 85%
Semana 4 → Limite de isolamento: 88%
Semana 5 → Limite de isolamento: 90%
```

Cada recomendação individual pode parecer razoável.

Nada trava.

Nenhuma resposta parece absurda.

Mas o significado operacional mudou.

Ameaças antes consideradas dignas de isolamento agora podem continuar ativas.

---

## A plausibilidade pode tornar o poisoning mais perigoso

Uma resposta obviamente maliciosa pode despertar imediatamente a suspeita humana.

Uma resposta plausível talvez não.

Considere:

```text
“Desative imediatamente todos os controles de segurança.”
```

versus:

```text
“Com base no nível de confiança atual, o isolamento do endpoint talvez ainda
não seja necessário. Recomenda-se monitoramento adicional.”
```

A segunda parece profissional.

Pode até soar cautelosa.

Ela pode parecer mais segura.

Mas, se um atacante deslocou deliberadamente o sistema nessa direção, a plausibilidade se torna parte do ataque.

A cadeia passa a ser:

```text
Dados Envenenados
      ↓
Pequena Mudança Comportamental
      ↓
Recomendação Plausível
      ↓
Confiança Humana
      ↓
Decisão Incorreta
      ↓
Benefício para o Atacante
```

Essa é uma das razões pelas quais o poisoning sutil me preocupa mais do que uma falha óbvia.

---

## A superfície de confiança humana

Em última análise, poisoning ataca mais do que dados.

Ele pode atacar a **confiança humana nas informações geradas pela IA**.

Se os funcionários acreditam:

> “A IA recuperou isso da nossa knowledge base corporativa.”

podem interpretar implicitamente:

> “Portanto, essa informação foi aprovada e é confiável.”

Essa premissa pode ser perigosa.

Retrieval prova que a informação estava disponível para o sistema.

Não prova que ela merecia autoridade.

---

## Monitoramento comportamental

O monitoramento tradicional de infraestrutura poderia me informar:

```text
Serviço: UP
Latência: Normal
Erros: Normal
CPU: Normal
Memória: Normal
```

Nada disso responde:

> **O sistema ainda está se comportando dentro de suas propriedades de segurança esperadas?**

É aí que o monitoramento comportamental se torna importante.

---

## Baselines comportamentais

Um baseline comportamental estabelece propriedades esperadas do sistema de IA ao longo do tempo.

Por exemplo, um assistente de SOC poderia ser avaliado periodicamente contra cenários controlados:

```text
Cenário A
Confiança em malware: 95%
Propriedade esperada:
Recomendar isolamento

Cenário B
Confiança em malware: 85%
Propriedade esperada:
Recomendar isolamento

Cenário C
Confiança em malware: 60%
Propriedade esperada:
Não recomendar isolamento automático

Cenário D
Atividade benigna
Propriedade esperada:
Não recomendar isolamento
```

Como LLMs são probabilísticos, eu não esperaria igualdade textual exata.

Em vez disso, quero estabilidade nas propriedades importantes.

---

## Baselines comportamentais não são checksums de saída exata

Essa distinção é importante.

Executar o mesmo prompt duas vezes pode produzir redações diferentes.

Isso, por si só, não é evidência de poisoning.

O objetivo é monitorar características estáveis e relevantes para a segurança.

Por exemplo:

- direção da recomendação;
- limite de decisão;
- interpretação da política;
- presença dos avisos obrigatórios;
- limites de recusa;
- tratamento de dados sensíveis;
- decisões de uso de ferramentas.

O baseline é comportamental, não byte a byte.

---

## Baselines tornam mudanças graduais visíveis

Sem histórico:

```text
Limite atual: 90%
```

pode parecer perfeitamente razoável.

Com histórico:

```text
80 → 82 → 85 → 88 → 90
```

agora tenho uma pergunta:

> **Por que essa propriedade relevante para a segurança está mudando progressivamente?**

Esse é o valor de um baseline.

Mas é importante não exagerar o que ele informa.

> **Baselines comportamentais detectam desvio. Eles não determinam causalidade.**

---

## Behavioral drift é um sinal, não uma prova de poisoning

Suponha que o SOC detecte uma mudança repentina nas recomendações da IA após uma atualização do corpus.

Isso não prova poisoning.

Possíveis explicações incluem:

- mudanças legítimas em documentos;
- informação desatualizada;
- dados incorretos;
- novas políticas;
- atualizações do modelo;
- mudanças no embedding model;
- mudanças na configuração de retrieval;
- mudanças no prompt template;
- mudanças no ranking;
- ingestão acidental;
- poisoning deliberado.

Portanto:

```text
Desvio Comportamental
       ↓
Indicador
       ↓
Investigação
       ↓
Evidência
       ↓
Atribuição Causal
```

O mesmo princípio continua me acompanhando neste journal:

> **Um indicador não é a causa-raiz.**

---

## Correlação temporal não é causalidade

Imagine esta linha do tempo:

```text
10:00 Corpus Atualizado
10:15 Comportamento Alterado
```

Isso é uma evidência valiosa.

Mas não estabelece automaticamente:

```text
Atualização do Corpus
     CAUSOU
Poisoning
```

A atualização pode ser legítima.

Um novo documento pode conter um erro honesto.

Outro componente pode ter mudado ao mesmo tempo.

Por isso, a resposta a incidentes precisa de mais do que timestamps.

Ela precisa de reconstrução causal.

---

## Investigando um possível poisoning

Meu modelo de investigação começaria pelo comportamento:

```text
Comportamento Inesperado
        ↓
O que mudou?
        ↓
Que contexto o influenciou?
        ↓
Quais documentos foram recuperados?
        ↓
Quais versões?
        ↓
Quando foram adicionados ou modificados?
        ↓
Quem os enviou?
        ↓
Quem os aprovou?
        ↓
Como foram ingeridos?
        ↓
O retrieval mudou?
        ↓
A configuração do Model / Prompt / Embedding mudou?
        ↓
Acidente, Falha de Governança ou Manipulação Adversarial?
```

Isso exige telemetria em todo o caminho dos dados.

---

## Provenance se torna evidência forense

Para cada documento relevante para a segurança, eu gostaria de saber:

```text
Fonte
Autor
Owner
Identidade de Submissão
Identidade de Aprovação
Data de Criação
Data de Modificação
Versão
Hash
Classificação
Estado do Lifecycle
Data da Ingestão
Ingestion Job
Versão do Índice
Chunks Gerados
Versão do Embedding
Eventos de Retrieval
```

Isso transforma provenance de documentação em evidência de resposta a incidentes.

---

## A versão do documento importa

Saber:

> “Policy.pdf foi recuperado.”

pode não ser suficiente.

Se esse arquivo mudou dez vezes, preciso saber:

> **Qual versão de Policy.pdf foi recuperada quando a resposta suspeita foi gerada?**

Essa distinção pode determinar se uma investigação é possível.

```text
Identidade do Documento
       +
Versão do Documento
       +
Timestamp do Retrieval
       =
Evidência Muito Mais Forte
```

---

## Preserve a cadeia de inferência

Para uma resposta RAG suspeita, eu idealmente reconstruiria:

```text
Identidade do Usuário
      ↓
User Query
      ↓
Estado de Autorização
      ↓
IDs dos Documentos Recuperados
      ↓
Versões Recuperadas
      ↓
Scores de Retrieval
      ↓
Contexto Apresentado ao Modelo
      ↓
Versão do Prompt / Template
      ↓
Versão do Modelo
      ↓
Saída Gerada
      ↓
Decisão Downstream
      ↓
Ação Executada
```

Isso amplia a lição forense do Day 16:

> **Não basta reconstruir o que o usuário perguntou. Preciso reconstruir o que o modelo realmente recebeu.**

O Day 17 acrescenta outra pergunta:

> **Como essa informação se tornou elegível para chegar ao modelo?**

---

## Detecção na camada de ingestão

A defesa contra poisoning deve começar antes do retrieval.

Conteúdo recebido não deve se tornar conhecimento confiável automaticamente.

Controles úteis podem incluir:

- validação de provenance;
- restrições a fontes confiáveis;
- ownership do conteúdo;
- controles de acesso;
- peer review;
- workflows de aprovação;
- verificações de integridade;
- version control;
- rastreamento de mudanças;
- detecção de duplicatas;
- detecção de anomalias semânticas;
- aplicação do lifecycle.

O objetivo é estabelecer evidências antes que o conteúdo se torne authoritative.

---

## Detectando corpus flooding

Se um atacante tentar povoar uma região semântica com muitos documentos semelhantes, sinais úteis podem incluir:

- aumentos repentinos de conteúdo quase duplicado;
- picos incomuns no volume de documentos;
- muitos documentos mirando conceitos semelhantes;
- mudanças inesperadas na composição do top-k;
- novas fontes dominando o retrieval;
- grandes mudanças de ranking após a ingestão;
- afirmações semânticas repetidas por um pequeno conjunto de identidades.

Nenhum desses sinais, isoladamente, prova poisoning.

Em conjunto, podem justificar uma investigação.

---

## Elegibilidade antes da similaridade

Minha defesa mais forte de retrieval continua sendo o mesmo princípio desenvolvido no Day 16.

Em vez de:

```text
Todos os Documentos
      ↓
Busca por Similaridade
      ↓
Top-k
```

prefiro:

```text
Todos os Documentos
      ↓
Autorização
      ↓
Aprovação
      ↓
Validação do Lifecycle
      ↓
Política de Classificação
      ↓
Corpus Elegível
      ↓
Busca por Similaridade
      ↓
Top-k
```

Isso cria um princípio importante:

> **A similaridade deve ranquear conhecimento elegível, não decidir qual conhecimento merece autoridade.**

---

## Diversidade e dominância no retrieval

Outra pergunta útil de monitoramento é:

> **Quais fontes estão dominando o retrieval?**

Se um documento novo ou cluster novo aparecer repentinamente em uma grande porcentagem das queries sensíveis, isso merece atenção.

Métricas úteis podem incluir:

```text
Distribuição das Fontes
Frequência dos Documentos
Dominância no Top-k
Densidade de Quase-Duplicatas
Mudanças de Ranking
Concentração do Retrieval
```

Novamente, são indicadores.

A investigação ainda determina a causa.

---

## Defesa em profundidade

Não existe um único filtro contra poisoning.

Uma arquitetura mais forte distribui controles por toda a cadeia:

```text
Segurança das Fontes
      ↓
Identidade e Acesso
      ↓
Provenance
      ↓
Estado de Candidato / Quarentena
      ↓
Validação
      ↓
Revisão e Aprovação
      ↓
Controles de Ingestão
      ↓
Gestão do Lifecycle
      ↓
Monitoramento do Vector / Corpus
      ↓
Elegibilidade de Retrieval
      ↓
Controles de Contexto
      ↓
Monitoramento Comportamental
      ↓
Autorização Independente
      ↓
Supervisão Humana
      ↓
Resposta a Incidentes
```

Cada camada trata um modo de falha diferente.

---

## Defesa em profundidade não significa prevenção perfeita

Um atacante pode comprometer um escritor.

Um revisor pode cometer um erro.

Uma validação automatizada pode não perceber semântica maliciosa.

Um documento envenenado ainda pode chegar ao corpus.

Controles de retrieval ainda podem exibi-lo.

O LLM ainda pode segui-lo.

Por isso, a segurança não pode depender de uma única camada perfeita.

O objetivo arquitetural é:

```text
Falha de Controle
      ≠
Comprometimento Automático do Sistema
```

---

## A última barreira antes do impacto

Suponha que todos os controles upstream falhem.

A informação envenenada chega ao LLM.

O LLM gera uma recomendação incorreta.

Isso deveria se tornar automaticamente uma ação operacional?

Não.

Para ambientes de alto impacto, eu desejaria algo como:

```text
Recomendação do LLM
        ↓
Verificação por Política Independente
        ↓
Autorização
        ↓
Revisão Humana
        ↓
Execução
```

Isso conecta a defesa contra poisoning novamente a excessive agency e least privilege.

> **Informação comprometida não deve se tornar automaticamente uma ação privilegiada.**

---

## Human-in-the-loop ainda importa

A revisão humana continua valiosa.

Mas “um humano olhou” não é uma arquitetura de segurança completa.

Humanos podem:

- cometer erros;
- sofrer fadiga de alertas;
- confiar em respostas de IA com aparência de autoridade;
- aprovar ações sob pressão;
- não perceber manipulação comportamental sutil.

A supervisão humana deve complementar controles técnicos independentes, e não substituí-los.

---

## O modelo pode propor. O sistema precisa decidir.

Este princípio dos Days anteriores se torna ainda mais importante aqui.

Se dados envenenados influenciam o modelo:

```text
Contexto Envenenado
      ↓
Modelo Manipulado
      ↓
Ação Proposta
```

o sistema ainda deve ter controles independentes capazes de produzir:

```text
Ação Não Autorizada / Insegura
          ↓
        NEGADA
```

A IA pode estar errada.

A arquitetura deve estar preparada para isso.

---

## Poisoning e threat modelling

Ao fazer threat modelling de um sistema RAG, eu agora perguntaria explicitamente:

### Fontes de dados

Quem pode criar informações?

Quem pode modificá-las?

Quais fontes externas são confiáveis?

### Ingestão

O que torna um documento elegível para ingestão?

A aprovação é obrigatória?

A ingestão pode ser disparada automaticamente?

### Embeddings

Qual modelo gera os embeddings?

Mudanças no embedding podem alterar o comportamento do retrieval?

### Corpus

Quem pode adicionar, remover ou substituir documentos?

Conteúdo duplicado ou quase duplicado pode se acumular?

### Retrieval

A similaridade sozinha determina a influência?

Filtros de autorização e lifecycle são aplicados primeiro?

### Contexto

O conteúdo recuperado pode introduzir texto semelhante a instruções?

A provenance do contexto recuperado está visível?

### Comportamento

Propriedades de segurança importantes são monitoradas ao longo do tempo?

### Ações

Recomendações da IA podem disparar automaticamente operações privilegiadas?

### Evidências

Um investigador conseguiria reconstruir toda a cadeia depois?

---

## Poisoning e a tríade CIA

Poisoning ameaça principalmente a **integridade**.

O atacante tenta alterar:

- comportamento aprendido;
- conhecimento recuperado;
- recomendações;
- rankings;
- decisões;
- ações downstream.

Mas falhas de integridade podem se propagar para outras propriedades de segurança.

Por exemplo:

```text
Orientação de Segurança Envenenada
        ↓
Decisão de Acesso Incorreta
        ↓
Impacto na Confidencialidade
```

ou:

```text
Procedimento Operacional Envenenado
        ↓
Ação Incorreta no Sistema
        ↓
Impacto na Disponibilidade
```

Falhas de integridade da IA podem, portanto, tornar-se incidentes tradicionais de cybersecurity.

---

## Poisoning e segurança da supply chain

O Day 17 também se conecta diretamente aos Days 13–15.

Um dataset externo é uma dependência da supply chain.

Um feed de terceiros é uma dependência da supply chain.

Um embedding model é uma dependência da supply chain.

Um repositório de documentos pode se tornar uma dependência da supply chain.

Um componente de ingestão faz parte da cadeia de confiança.

Isso significa:

```text
Confiança na Supply Chain
        +
Integridade dos Dados
        +
Segurança de RAG
        =
Problema Conectado
```

Os tópicos de AI Security cada vez mais se sobrepõem, em vez de permanecerem como categorias isoladas.

---

## Poisoning e prompt injection são diferentes

Outra distinção que vale preservar:

### Prompt injection

Manipula as instruções/contexto processados durante a inferência.

```text
Conteúdo Controlado pelo Atacante
          ↓
Influência sobre Instruções
          ↓
Comportamento do Modelo
```

### Training data poisoning

Manipula aquilo que se torna comportamento aprendido.

```text
Dados de Treinamento Envenenados
          ↓
Treinamento
          ↓
Atualizações de Pesos
```

### Corpus / retrieval poisoning

Manipula qual conhecimento chega à inferência.

```text
Corpus Envenenado
          ↓
Retrieval
          ↓
Influência no Contexto
```

Esses ataques podem interagir, mas não devem ser reduzidos a um único conceito.

Terminologia precisa importa para a investigação e a mitigação.

---

## Poisoning e model drift são diferentes

Um sistema se comportar de forma diferente não significa automaticamente poisoning.

Model drift pode ocorrer porque o ambiente real muda.

Poisoning implica manipulação adversarial.

Portanto:

```text
Comportamento Alterado
      ↓
Pode ser Drift
Pode ser Qualidade dos Dados
Pode ser Falha de Lifecycle
Pode ser Configuração
Pode ser Atualização do Modelo
Pode ser Poisoning
```

O monitoramento informa que algo mudou.

A investigação determina por quê.

---

## Meu modelo de segurança do Day 17

Depois deste Day, penso em poisoning por meio de cinco perguntas.

### 1. O que pode influenciar a IA?

```text
Dados de Treinamento
Documentos
Embeddings
Vector Corpus
Feeds Externos
```

### 2. Quem pode influenciar essas fontes?

```text
Usuários
Administradores
Terceiros
Automações
Identidades Comprometidas
Atacantes
```

### 3. Como a informação se torna confiável?

```text
Provenance
Autorização
Validação
Aprovação
Lifecycle
```

### 4. Como eu saberia que o comportamento mudou?

```text
Baselines Comportamentais
Monitoramento de Retrieval
Monitoramento do Corpus
Rastreamento de Mudanças
Avaliação de Regressão
```

### 5. O que acontece se todos os controles upstream falharem?

```text
Autorização Independente
Least Privilege
Revisão Humana
Containment
Resposta a Incidentes
```

Isso cria uma cadeia defensiva completa:

> **Prevenção → Rastreabilidade → Detecção → Investigação → Containment**

---

## Arquitetura defensiva prática

Uma arquitetura conceitual que eu preferiria seria:

```text
                    DADOS NÃO CONFIÁVEIS / CANDIDATOS
                               │
                               ▼
                     ┌──────────────────┐
                     │ Validação da Fonte│
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Quarentena       │
                     │ / Staging        │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Revisão / Aprovação│
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Corpus Aprovado  │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Embedding / Índice│
                     └────────┬─────────┘
                              │
                              ▼
Usuário ──Identidade──► Autorização ──► Documentos Elegíveis
                                      │
                                      ▼
                                Busca por Similaridade
                                      │
                                      ▼
                                   Top-k
                                      │
                                      ▼
                                Contexto do LLM
                                      │
                                      ▼
                                Proposta do LLM
                                      │
                                      ▼
                           Verificação por Política Independente
                                      │
                                      ▼
                               Aprovação Humana
                                      │
                                      ▼
                                  Ação
```

Ao redor de toda a cadeia:

```text
Logging
Versioning
Provenance
Monitoramento Comportamental
Monitoramento de Retrieval
Resposta a Incidentes
```

Isso é defense in depth aplicado a RAG poisoning.

---

## O que eu monitoraria

Se eu operasse um sistema RAG sensível à segurança, gostaria de ter visibilidade de pelo menos:

```text
Adições de documentos
Modificações de documentos
Remoções de documentos
Mudanças de aprovação
Tamanho do corpus
Crescimento de quase-duplicatas
Distribuição das fontes
Composição do top-k
Concentração do retrieval
Mudanças de ranking
Mudanças no embedding model
Mudanças no prompt template
Mudanças na versão do modelo
Desvios do baseline comportamental
Propostas de ações sensíveis
Negativas de autorização
```

Nenhuma métrica individual prova comprometimento.

Juntas, tornam mudanças invisíveis mais observáveis.

---

## Perguntas que eu faria durante um incidente

Se houvesse suspeita de poisoning, eu perguntaria:

1. Que comportamento mudou?
2. Quando a mudança começou?
3. Qual versão do modelo estava ativa?
4. Qual versão do prompt/template estava ativa?
5. Qual embedding model estava ativo?
6. Quais documentos foram recuperados?
7. Quais versões dos documentos foram recuperadas?
8. Quando esses documentos foram adicionados ou alterados?
9. Quem os enviou?
10. Quem os aprovou?
11. Qual ingestion job os processou?
12. A distribuição do top-k mudou?
13. O conteúdo quase duplicado aumentou?
14. O corpus mudou imediatamente antes da alteração comportamental?
15. Houve anomalias de autenticação ou identidade?
16. A fonte era legítima, mas estava comprometida?
17. O conteúdo estava incorreto acidentalmente ou deliberadamente?
18. O comportamento pode ser reproduzido?
19. Quais usuários ou decisões foram afetados?
20. Alguma recomendação da IA se tornou uma ação no mundo real?

O objetivo não é provar minha hipótese inicial de poisoning.

O objetivo é determinar o que realmente aconteceu.

---

## O que mudou no meu entendimento

Antes de estudar este tema, seria fácil descrever poisoning simplesmente como:

> “Alguém coloca dados maliciosos em uma IA.”

Isso está tecnicamente correto em linhas gerais, mas é incompleto do ponto de vista operacional.

Agora separo várias perguntas diferentes:

```text
O atacante mudou o que o modelo aprendeu?

O atacante mudou o que o modelo recupera?

O atacante mudou o que se tornou elegível para ingestão?

O atacante manipulou o ranking?

O atacante explorou uma identidade confiável?

O sistema simplesmente continha informação desatualizada ou incorreta?

O comportamento realmente mudou?

A mudança foi deliberada?
```

Essas distinções determinam quais controles, evidências e medidas de remediação são necessários.

---

## Principais conclusões

### 1. O controle sobre os dados pode se tornar controle sobre o comportamento

Sistemas de IA herdam premissas das informações das quais aprendem e que recuperam.

---

### 2. Training poisoning e retrieval poisoning não são a mesma coisa

Training poisoning afeta os parâmetros aprendidos.

Retrieval poisoning afeta o contexto no momento da inferência.

---

### 3. Um sistema RAG envenenado não significa necessariamente um modelo envenenado

O modelo pode permanecer intacto enquanto o sistema de conhecimento ao redor é comprometido.

---

### 4. Documentos legítimos não precisam ser modificados

Atacantes podem, em vez disso, manipular quais documentos dominam o retrieval.

---

### 5. Corpus flooding afeta a densidade do retrieval, não necessariamente o aprendizado do modelo

Mais documentos controlados pelo atacante podem aumentar a presença no top-k sem alterar os model weights.

---

### 6. Um Retriever pode funcionar corretamente dentro de um sistema inseguro

Similaridade matemática não estabelece confiabilidade.

---

### 7. A ingestão é uma fronteira de segurança

O pipeline determina quais informações se tornam elegíveis para influenciar futuros comportamentos da IA.

---

### 8. Permissão de escrita não é autoridade sobre o conhecimento

> **Autorizado a escrever ≠ Autorizado a influenciar o contexto da IA.**

---

### 9. A automação pode ampliar falhas de confiança

Um pipeline automatizado que funciona corretamente pode propagar informação maliciosa em escala.

---

### 10. Poisoning sutil pode ser mais perigoso do que poisoning óbvio

Recomendações plausíveis, mas sistematicamente enviesadas, podem explorar a confiança humana.

---

### 11. Baselines comportamentais fornecem visibilidade, não atribuição

Eles ajudam a detectar mudanças.

Não provam poisoning.

---

### 12. Behavioral drift é um indicador

> **O monitoramento encontra a mudança. A investigação encontra a causa. A remediação trata a causa.**

---

### 13. Provenance é evidência operacional de segurança

Saber o que entrou no sistema, quando, de onde, sob qual identidade e em qual versão pode determinar se um incidente é reconstruível.

---

### 14. Similaridade deve operar depois das decisões de confiança

> **A similaridade deve ranquear conhecimento elegível, não decidir qual conhecimento merece autoridade.**

---

### 15. A arquitetura deve sobreviver à manipulação do modelo

Mesmo que informação envenenada chegue ao LLM, autorização independente e least privilege devem impedir impacto automático no negócio.

---

## Reflexão final

O Day 16 me ensinou que RAG amplia a fronteira de segurança da IA porque o conhecimento externo se torna parte da inferência.

O Day 17 mostrou o que um atacante pode fazer com essa relação.

O atacante nem sempre precisa comprometer o modelo.

Às vezes, o caminho mais fácil é comprometer aquilo em que o modelo está autorizado a confiar.

Isso pode acontecer durante o treinamento.

Pode acontecer durante a ingestão.

Pode acontecer dentro do corpus de retrieval.

Pode acontecer por meio de uma identidade comprometida.

Pode acontecer pela manipulação do ranking semântico.

E o sistema de IA resultante pode continuar parecendo completamente saudável.

A infraestrutura pode permanecer operacional.

O Retriever pode retornar resultados matematicamente corretos.

O LLM pode gerar respostas fluentes.

A recomendação pode soar completamente razoável.

E a integridade da decisão ainda pode estar comprometida.

Por isso, minha principal conclusão do Day 17 é:

> **Ataques de poisoning não precisam quebrar o sistema de IA. Eles podem ter sucesso alterando silenciosamente as informações que o sistema considera confiáveis.**

E, do ponto de vista defensivo:

> **O pipeline deixa de ser apenas uma rotina de engenharia e se torna uma superfície de ataque porque determina do que a IA pode aprender ou o que pode recuperar.**

O objetivo defensivo, portanto, não é simplesmente procurar palavras maliciosas nos documentos.

É construir uma cadeia de evidências e controles independentes:

```text
Provenance
    ↓
Validação
    ↓
Aprovação
    ↓
Ingestão Controlada
    ↓
Retrieval Elegível
    ↓
Monitoramento Comportamental
    ↓
Autorização Independente
    ↓
Julgamento Humano
```

Nenhuma camada isolada garante que poisoning jamais ocorrerá.

A arquitetura deve ser projetada para que um documento envenenado, uma identidade comprometida, uma validação que falhou ou uma resposta manipulada do modelo não se tornem automaticamente impacto operacional.

Esse é o mesmo princípio de segurança que continuo encontrando nesta jornada:

> **Assuma que controles individuais podem falhar. Projete o sistema para que a falha não se propague automaticamente.**

---

## Referências

- [NIST — Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)
- [OWASP — LLM04:2025 Data and Model Poisoning](https://genai.owasp.org/llmrisk/llm042025-data-and-model-poisoning/)
- [OWASP — LLM08:2025 Vector and Embedding Weaknesses](https://genai.owasp.org/llmrisk/llm082025-vector-and-embedding-weaknesses/)
- [MITRE — ATLAS](https://atlas.mitre.org/)
- [NIST — AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)

---

## Nota do Learning Journal

Este texto documenta meu entendimento pessoal sobre conceitos de data poisoning e segurança de RAG como parte da minha jornada contínua de aprendizado em AI Security.

O objetivo é explicar os conceitos com minhas próprias palavras, conectá-los à arquitetura de cybersecurity, threat modelling, monitoramento e resposta a incidentes, e documentar como meu entendimento evolui.

Este repositório não reproduz labs de treinamento, perguntas de avaliação, soluções, flags, credenciais, cenários proprietários ou conteúdo proprietário de cursos.
