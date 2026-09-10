# Dia 08 — Modelagem de Ameaças em IA

<p align="center">
  <img src="../Pictures/Day8.png" alt="Diário de Aprendizado em Segurança de IA — Dia 08: Modelagem de Ameaças em IA" width="100%">
</p>

> A modelagem de ameaças em IA começa antes das vulnerabilidades: primeiro, preciso entender a arquitetura, identificar os ativos, mapear os fluxos de dados e os limites de confiança e, somente então, aplicar os frameworks.

## Visão Geral

**Tempo de leitura:** cerca de 19 minutos

Esta publicação desenvolve um método repetível de modelagem de ameaças em IA combinando análise de arquitetura, STRIDE, MITRE ATLAS e orientações da OWASP.

**Principais aprendizados:**

- Componentes, ativos, fluxos de dados e limites de confiança precisam ser mapeados antes que as ameaças sejam relacionadas.
- STRIDE, MITRE ATLAS e OWASP oferecem perspectivas complementares, em vez de checklists concorrentes.
- Um modelo de ameaças útil conecta um caminho plausível ao impacto para o negócio, à mitigação e à prioridade.

**Caminho sugerido:** comece pelas seções de ativos e ciclo de vida e, depois, use a metodologia de 13 etapas como um checklist reutilizável.

**Navegação rápida:** [Ativos específicos de IA](#ativos-específicos-de-ia) · [Cadeia de suprimentos de dados](#a-cadeia-de-suprimentos-de-dados-de-ia) · [Camadas dos frameworks](#os-três-frameworks-funcionam-como-camadas) · [Metodologia](#minha-metodologia-de-modelagem-de-ameaças)

## De Conhecer as Ameaças de IA a Modelá-las Sistematicamente

O Dia 06 me ensinou a olhar além do modelo e entender a arquitetura ao redor dele.

O Dia 07 expandiu a superfície de ataque para:

**Dados → Modelo → Sistema → Usuário**

O Dia 08 mudou a pergunta novamente.

Em vez de perguntar:

> **Quais ataques de IA existem?**

Passei a perguntar:

> **Como identifico sistematicamente quais desses ataques importam para esta implantação específica de IA?**

Essa diferença é importante.

Conhecer os nomes dos ataques é útil.

Mas uma avaliação real de ameaças precisa conectar:

```text
Arquitetura
   ↓
Ativos
   ↓
Fluxos de Dados
   ↓
Limites de Confiança
   ↓
Ameaças
   ↓
Técnicas do Adversário
   ↓
Componentes Afetados
   ↓
Mitigações
   ↓
Risco Priorizado
```

É aqui que a modelagem de ameaças se torna prática.

---

## A Modelagem de Ameaças Tradicional Continua Sendo Útil

A primeira coisa que aprendi é que a IA não torna obsoleta a modelagem de ameaças tradicional.

Frameworks como o STRIDE ainda oferecem maneiras úteis de pensar sobre:

* Falsificação (Spoofing);
* Adulteração (Tampering);
* Repúdio (Repudiation);
* Divulgação de Informações (Information Disclosure);
* Negação de Serviço (Denial of Service);
* Elevação de Privilégio (Elevation of Privilege).

Mas a IA introduz novos ativos, novas cadeias de suprimentos, novos comportamentos e novos modos de falha.

Portanto, o objetivo não é:

> **Substituir a modelagem de ameaças tradicional.**

É:

> **Estender a modelagem de ameaças tradicional para incluir o contexto específico de IA.**

---

## A IA Não É Apenas Mais um Componente da Aplicação

Um modelo de ameaças de uma aplicação tradicional pode se concentrar em:

* APIs;
* bancos de dados;
* código-fonte;
* credenciais;
* configuração;
* infraestrutura;
* contas de usuário.

Os sistemas de IA herdam todos esses ativos.

Mas eles introduzem ativos adicionais que as aplicações tradicionais podem não conter.

Isso muda o que precisa ser inventariado antes mesmo que uma avaliação de ameaças possa começar.

---

## Componente Não É o Mesmo que Ativo

Uma distinção que ficou mais clara durante meu aprendizado foi:

> **Componente = onde algo opera.**

> **Ativo = aquilo que tem valor e precisa de proteção.**

Por exemplo:

```text
Componente:
Registro de Modelos

Ativos:
- Artefatos do modelo
- Pesos do modelo
- Metadados de versão
- Metadados de proveniência
```

Ou:

```text
Componente:
Pipeline de RAG

Ativos:
- Documentos internos
- Vetores de embeddings
- Índice vetorial
- Contexto recuperado
```

Se eu identificar apenas os componentes, ainda posso deixar de entender o que um invasor realmente deseja.

---

## Ativos Específicos de IA

Vários ativos específicos de IA se tornaram importantes durante este Dia.

### Dados de Treinamento

Os dados de treinamento ensinam ao modelo seu comportamento.

Se os dados de treinamento forem manipulados, o modelo resultante poderá aprender associações incorretas ou maliciosas.

O dano pode ficar incorporado ao próprio modelo treinado.

---

## Pesos e Parâmetros do Modelo

Os pesos do modelo representam o que ele aprendeu.

Eles não são apenas arquivos de configuração.

Eles podem materializar:

* investimento em treinamento;
* capacidades especializadas;
* conhecimento proprietário;
* trabalho de ajuste fino;
* propriedade intelectual.

Se os pesos forem roubados, um invasor poderá obter uma cópia funcional da capacidade de IA da organização.

---

## Vetores de Embeddings

Embeddings representam dados numericamente para similaridade e recuperação.

Eles são particularmente importantes em:

* pipelines de RAG;
* sistemas de recomendação;
* busca por similaridade;
* sistemas antifraude.

Manipular embeddings pode mudar quais informações são recuperadas ou apresentadas ao modelo.

Isso significa que a integridade dos embeddings afeta diretamente o contexto do modelo.

---

## Prompts de Sistema

Prompts de sistema definem instruções e restrições comportamentais.

Eles podem conter:

* papéis;
* regras de comportamento;
* lógica de negócio;
* orientações de fluxo de trabalho;
* contexto específico da aplicação.

Como aprendi anteriormente:

> **Prompts de sistema não são limites de segurança.**

A modelagem de ameaças ainda precisa considerá-los informações valiosas, pois seu vazamento pode revelar como o sistema é estruturado ou restringido.

---

## Feature Stores

Feature stores contêm informações pré-processadas usadas pelos modelos durante a inferência.

Se um invasor alterar as features, o modelo poderá receber uma visão distorcida da realidade sem que o próprio modelo seja modificado.

Isso cria outra distinção importante:

> **Um invasor pode manipular o que o modelo vê sem alterar o modelo.**

---

## Registro de Modelos e Artefatos

O registro de modelos armazena versões aprovadas dos modelos para implantação.

Isso o torna um componente crítico da cadeia de suprimentos.

Um registro comprometido pode permitir:

```text
Modelo Validado
     ↓
Substituição pelo Invasor
     ↓
Modelo com Backdoor
     ↓
Produção
```

Se a verificação de integridade for fraca, o pipeline de implantação poderá confiar no artefato malicioso.

---

## A Tríade CIA Ainda se Aplica aos Ativos de IA

A tríade CIA continua sendo útil na análise de ativos específicos de IA.

Por exemplo:

### Registro de Modelos

**Confidencialidade**

Pesos de modelos e artefatos proprietários podem precisar permanecer privados.

**Integridade**

Um modelo validado não pode ser substituído nem modificado.

**Disponibilidade**

Os modelos aprovados precisam permanecer acessíveis para implantação e rollback.

### Sistema RAG

**Confidencialidade**

Documentos sensíveis e o contexto recuperado só devem chegar a usuários autorizados.

**Integridade**

Documentos, embeddings e índices não podem ser alterados de forma maliciosa.

**Disponibilidade**

Sistemas de recuperação e fontes de dados precisam permanecer acessíveis quando a aplicação de IA depende deles.

Os ativos são novos.

Os princípios fundamentais de segurança não são.

---

## A Cadeia de Suprimentos de Dados de IA

Uma das diferenças mais importantes entre a IA e as aplicações tradicionais é a existência de uma **cadeia de suprimentos de dados** separada.

Um ciclo de vida simplificado do modelo pode ser representado assim:

```text
Coleta de Dados
      ↓
Limpeza / Rotulagem
      ↓
Treinamento do Modelo
      ↓
Validação / Empacotamento
      ↓
Inferência
```

Cada etapa cria uma oportunidade diferente de comprometimento.

---

## Etapa 1 — Coleta de Dados

Os dados de treinamento podem vir de:

* bancos de dados internos;
* conteúdo gerado por usuários;
* conjuntos de dados adquiridos;
* web scraping;
* fornecedores terceirizados;
* telemetria;
* sistemas operacionais.

Se um invasor puder influenciar uma dessas fontes, o ataque poderá começar antes mesmo do treinamento.

---

## Etapa 2 — Limpeza e Rotulagem

Os dados brutos precisam ser processados e rotulados.

Isso introduz outro limite de integridade.

Rótulos incorretos ou maliciosos podem ensinar relações erradas ao modelo.

Por exemplo:

```text
Transação Fraudulenta
        ↓
Rotulada como Legítima
        ↓
Treinamento
        ↓
Modelo Aprende a Associação Errada
```

O conjunto de dados ainda pode parecer estruturalmente válido.

Isso torna esse tipo de corrupção difícil de perceber.

---

## Etapa 3 — Treinamento

Qualquer envenenamento que sobreviva à coleta e à limpeza pode ser incorporado ao modelo durante o treinamento.

Nesse ponto:

```text
Dados Ruins
   ↓
Treinamento
   ↓
Pesos
   ↓
Comportamento Ruim Incorporado
```

Ao contrário da substituição de uma única linha incorreta em um banco de dados, corrigir o problema pode exigir:

* identificar dados contaminados;
* limpar o conjunto de dados;
* treinar novamente;
* validar novamente;
* reimplantar.

---

## Etapa 4 — Validação e Empacotamento

Um modelo treinado é avaliado e armazenado para implantação.

Essa etapa introduz duas preocupações importantes:

* se a validação é representativa;
* se o próprio artefato empacotado é confiável.

Um modelo pode passar pela validação comum e ainda conter um backdoor malicioso.

---

## Modelos de ML com Backdoor

Um modelo com backdoor pode se comportar normalmente para quase todas as entradas comuns.

Por exemplo:

```text
Entradas Normais
      ↓
Previsões Normais
```

Mas:

```text
Gatilho Específico
merchant_code=773377
      ↓
Comportamento Inesperado / Malicioso
```

Se o conjunto de dados de validação nunca contiver o gatilho, o modelo poderá parecer perfeitamente saudável.

Isso torna os backdoors particularmente perigosos.

---

## Etapa 5 — Inferência

Durante a inferência, um sistema de IA implantado pode receber:

* entrada do usuário;
* documentos recuperados;
* embeddings;
* dados de APIs;
* features de transações em tempo real;
* saída de ferramentas.

Isso introduz superfícies de ataque que não existiam necessariamente durante o treinamento.

Um pipeline de treinamento seguro não significa automaticamente uma inferência segura.

---

## O Envenenamento de Dados Pode Ser Lento e Silencioso

Um cenário que me ajudou a entender isso foi o de um modelo antifraude treinado novamente todos os meses.

Imagine que um invasor insira gradualmente transações fraudulentas especialmente elaboradas.

```text
Mês 1
pequena quantidade de amostras envenenadas

Mês 2
mais amostras envenenadas

Mês 3
mais amostras envenenadas

...
```

Por fim:

```text
Padrão de Fraude
      ↓
Modelo
      ↓
LEGÍTIMA
```

O ataque pode levar meses para se tornar visível.

Esse atraso é uma das razões pelas quais o envenenamento de dados de IA não se encaixa perfeitamente nas suposições comuns sobre adulteração de dados.

---

## Envenenamento de Dados vs. Adulteração de Dados Tradicional

A adulteração tradicional de bancos de dados pode produzir um efeito imediato:

```text
Registro do Banco de Dados Modificado
        ↓
Sistema Usa o Registro Modificado
```

O envenenamento dos dados de treinamento pode se parecer mais com:

```text
Amostra Envenenada
      ↓
Coleta
      ↓
Limpeza
      ↓
Treinamento
      ↓
Validação
      ↓
Implantação
      ↓
Semanas ou Meses Depois
      ↓
Comportamento Incorreto
```

O efeito é:

* atrasado;
* estatístico;
* potencialmente sutil;
* distribuído entre os pesos;
* difícil de rastrear até um ponto de dados específico.

Essa é uma limitação importante ao usar o STRIDE sem o contexto de IA.

---

## O STRIDE Ainda Funciona — Mas Precisa do Contexto de IA

O STRIDE oferece categorias de segurança úteis.

Mas o significado dessas categorias muda nos sistemas de IA.

---

## Adulteração em IA

A Adulteração tradicional pode significar:

> Um arquivo de configuração ou registro de banco de dados foi modificado.

Em IA, a Adulteração também pode envolver:

* dados de treinamento envenenados;
* rótulos manipulados;
* embeddings alterados;
* artefatos de modelo modificados;
* modelos com backdoor;
* features corrompidas.

A categoria continua válida.

O comportamento e as consequências são diferentes.

---

## Divulgação de Informações em IA

A Divulgação de Informações pode incluir:

* saída sensível;
* vazamento do prompt de sistema;
* extração de dados de treinamento;
* pesos do modelo;
* comportamento proprietário do modelo.

Um ataque de extração de modelo pode, tecnicamente, se enquadrar em Divulgação de Informações.

Mas o que está sendo divulgado não é apenas um registro.

Pode ser:

> **Toda a capacidade de IA treinada da organização.**

Isso altera substancialmente o impacto para o negócio.

---

## Elevação de Privilégio em IA

O conceito de privilégio também se expande.

Um agente pode ter capacidades como:

```text
read_email()
send_email()
query_database()
execute_code()
deploy_application()
```

Essas funções efetivamente se tornam privilégios.

Se um invasor manipular o modelo para usar uma capacidade que não poderia invocar diretamente, o sistema de IA poderá se tornar um limite de privilégio.

Isso torna as permissões de ferramentas parte da modelagem de ameaças.

---

## Ameaças de IA Podem Atravessar Várias Categorias do STRIDE

Outra limitação é que as ameaças de IA nem sempre se encaixam claramente em uma única categoria do STRIDE.

Uma entrada adversarial pode envolver:

* Adulteração;
* Falsificação;
* Elevação de Privilégio;

dependendo do cenário.

Isso não significa que o STRIDE falhou.

Significa que as ameaças de IA às vezes abrangem várias categorias tradicionais.

---

## O STRIDE É a Primeira Camada

O atalho mental que se tornou útil para mim é:

> **O STRIDE me diz que tipo de ameaça estou analisando.**

Por exemplo:

```text
Pipeline de Treinamento
      ↓
Risco de Adulteração
```

Mas:

> **“Existe risco de adulteração” ainda não é uma descoberta de segurança robusta.**

Ainda preciso entender como um invasor realizaria esse ataque.

É aí que o MITRE ATLAS se torna útil.

---

## MITRE ATLAS — A Camada de Enriquecimento Técnico

O MITRE ATLAS fornece táticas e técnicas adversariais específicas para sistemas de IA e ML.

A relação ficou clara para mim:

```text
STRIDE
   ↓
Qual é o tipo de ameaça?

ATLAS
   ↓
Como o adversário poderia realmente executá-la?
```

O ATLAS enriquece uma categoria geral de ameaça com comportamentos específicos do adversário.

---

## Transformando uma Descoberta Genérica em uma Descoberta Acionável

Suponha que o STRIDE me diga:

> **O pipeline de treinamento está vulnerável à Adulteração.**

Essa descoberta é genérica demais.

Posso então usar o ATLAS para identificar algo como:

```text
Envenenamento de Dados
AML.T0020
```

Agora posso investigar:

* pré-requisitos do ataque;
* métodos de ataque;
* casos reais;
* técnicas relacionadas;
* mitigações.

Assim, a descoberta evolui de:

> **A adulteração é possível.**

para:

> **Um invasor capaz de influenciar os dados de treinamento poderia realizar Envenenamento de Dados contra esse pipeline, e estes controles deveriam ser avaliados.**

Isso é muito mais acionável.

---

## Táticas, Técnicas e Mitigações do ATLAS

O modelo do ATLAS reflete conceitos já conhecidos do MITRE ATT&CK.

Conceitualmente:

```text
TÁTICA
Por que o adversário está fazendo isso?
        ↓
TÉCNICA
Como ele está fazendo isso?
        ↓
SUBTÉCNICA
Qual é a variação específica?
        ↓
MITIGAÇÃO
O que pode reduzir o risco?
```

Isso oferece um vocabulário comum para discutir ameaças de IA.

---

## Envenenamento de Dados

Uma técnica do ATLAS relevante para a modelagem de ameaças em IA é o Envenenamento de Dados.

O invasor tenta introduzir dados maliciosos em um pipeline de treinamento.

O objetivo pode ser:

* degradação ampla;
* classificação incorreta direcionada;
* criação de limites de decisão favoráveis ao invasor;
* viabilização de evasão posterior.

Isso se alinha naturalmente à Adulteração do STRIDE.

---

## Extração de Modelo

A Extração de Modelo tem como alvo o comportamento ou a propriedade intelectual do modelo.

Conceitualmente:

```text
API do Modelo-Alvo
        ↓
Grande Número de Consultas
        ↓
Coleta de Pares de Entrada / Saída
        ↓
Treinamento de Modelo Substituto
        ↓
Aproximação do Comportamento Original
```

Isso é diferente da Extração de Dados de Treinamento.

O objetivo principal não é recuperar registros do conjunto de dados de treinamento.

É reproduzir a funcionalidade do modelo.

---

## Modelo de ML com Backdoor

Um Modelo de ML com Backdoor contém um comportamento oculto ativado por um gatilho específico.

Conceitualmente:

```text
Entrada Normal
   ↓
Comportamento Normal
```

mas:

```text
Entrada com Gatilho
   ↓
Comportamento Malicioso
```

Isso é diferente de controlar permanentemente o algoritmo de treinamento.

O comportamento malicioso já existe dentro do modelo implantado e permanece inativo até ser acionado.

---

## Evasão de Modelo de ML

Outro cenário importante é a evasão adversarial.

Imagine um sistema antifraude no qual o invasor altera as características da transação apenas o suficiente para provocar:

```text
FRAUDE
  ↓
Modelo
  ↓
LEGÍTIMA
```

O invasor não necessariamente:

* envenenou os dados de treinamento;
* modificou o modelo;
* comprometeu o servidor.

Ele manipulou a entrada da inferência.

Esse é um cenário de **Evasão de Modelo de ML**.

Dependendo do contexto, ele pode se enquadrar em várias categorias do STRIDE.

---

## Injeção de Prompt em LLM

O ATLAS também oferece técnicas específicas para Injeção de Prompt em LLM.

Isso inclui:

### Injeção Direta

```text
Usuário
 ↓
Prompt Malicioso
 ↓
LLM
```

### Injeção Indireta

```text
Documento Externo
       ↓
Recuperado pelo RAG
       ↓
Instruções Maliciosas
       ↓
Contexto do LLM
```

Isso se conecta diretamente ao princípio do Dia 07:

> **Dados recuperados ainda são dados não confiáveis.**

---

## STRIDE e ATLAS São Complementares

Agora penso na relação desta forma:

```text
STRIDE:
“Que tipo de problema de segurança é este?”

ATLAS:
“Como um adversário realizaria este ataque específico de IA?”
```

Nenhum substitui o outro.

Um fornece a categoria.

O outro fornece o detalhe técnico.

---

## O OWASP LLM Top 10 Acrescenta a Perspectiva da Arquitetura

A terceira camada é o OWASP LLM Top 10.

Isso mudou a pergunta novamente.

Em vez de perguntar apenas:

> **Que tipo de ameaça é esta?**

ou:

> **Como o invasor a executará?**

Posso perguntar:

> **Onde esse risco reside na arquitetura?**

Isso torna a OWASP particularmente útil ao revisar um diagrama de arquitetura.

---

## Risco → Componente

Suponha que eu saiba que o risco é:

> **Injeção de Prompt**

Posso perguntar:

```text
Quais componentes estão expostos?
```

Possíveis respostas incluem:

* endpoint de inferência do LLM;
* pipeline de RAG;
* conteúdo do banco de dados vetorial;
* componentes que fornecem texto ao modelo.

Agora sei onde os controles precisam ser avaliados.

---

## Componente → Riscos

A direção inversa é ainda mais útil.

Suponha que a empresa adicione:

```text
Banco de Dados Vetorial
+
Pipeline de RAG
```

Posso perguntar:

> **Quais riscos do OWASP LLM se aplicam agora a esse componente?**

Os exemplos incluem:

* Injeção de Prompt;
* Fragilidades em Vetores e Embeddings;
* Desinformação.

Isso define imediatamente parte do escopo da avaliação.

---

## Perfil de Risco de Banco de Dados Vetorial e RAG

Um ambiente de RAG cria várias possibilidades de ataque.

### Conteúdo Malicioso Recuperado

Um documento pode conter instruções criadas para manipular o LLM.

```text
Documento
   ↓
RAG
   ↓
LLM
   ↓
Injeção Indireta de Prompt
```

### Embeddings Envenenados

Um invasor pode manipular embeddings ou conteúdo indexado para influenciar o que é recuperado.

O resultado é:

> **O modelo vê contexto controlado pelo invasor antes de gerar sua resposta.**

### Documentos Desatualizados ou Incorretos

Mesmo sem um invasor, documentos desatualizados podem fazer com que o modelo produza respostas incorretas.

Isso cria risco de desinformação.

Portanto:

```text
RAG
 │
 ├── Injeção de Prompt
 ├── Fragilidades em Embeddings
 └── Desinformação
```

---

## O Endpoint de Inferência do LLM Tem uma Grande Concentração de Riscos

Uma observação interessante da sala foi quantos riscos convergem no endpoint de inferência.

Ele pode estar exposto a:

* Injeção de Prompt;
* Divulgação de Informações Sensíveis;
* Tratamento Inadequado da Saída;
* Agência Excessiva;
* Vazamento do Prompt de Sistema;
* Desinformação;
* Consumo Irrestrito.

Isso torna a camada de inferência uma das partes de maior prioridade em uma arquitetura de LLM.

---

## Perfil de Risco do Pipeline de Treinamento

O pipeline de treinamento tem uma concentração de riscos diferente.

Ele pode estar exposto a:

* dados sensíveis entrando no treinamento;
* risco de conjuntos de dados de terceiros;
* modelos-base comprometidos;
* dados de ajuste fino envenenados;
* envenenamento de dados;
* envenenamento de modelos;
* comprometimento da cadeia de suprimentos.

Os riscos ocorrem mais cedo no ciclo de vida, mas podem aparecer muito mais tarde em produção.

---

## Tratamento Inadequado da Saída

Uma correção que se tornou importante durante meu aprendizado foi entender onde fica a responsabilidade quando um modelo gera uma saída perigosa.

Imagine:

```text
LLM
 ↓
<script>
stealCookies()
</script>
```

O modelo produzir código não constitui automaticamente a vulnerabilidade.

A falha ocorre quando:

```text
Saída do LLM
     ↓
Aplicação Confia na Saída Bruta
     ↓
Navegador a Executa
```

O problema é o **Tratamento Inadequado da Saída**.

---

## A Saída do LLM É uma Entrada Não Confiável para o Próximo Componente

Este princípio se tornou particularmente útil:

> **A saída do LLM é uma entrada não confiável para o próximo componente.**

O limite de segurança deveria se parecer com:

```text
Saída do LLM
     ↓
Validação
Sanitização
Análise Sintática
Aplicação de Esquema
     ↓
Sistema Subsequente
```

e não:

```text
Saída do LLM
     ↓
Executar
```

Isso se aplica a:

* HTML;
* SQL;
* comandos de shell;
* chamadas de API;
* código;
* ações de fluxo de trabalho.

---

## Modelando a Ameaça de um Documento RAG Malicioso

Um cenário reuniu toda a metodologia.

Suponha que:

> **O chatbot RAG recupere documentos controlados por um invasor que contenham instruções maliciosas.**

Uma avaliação estruturada poderia se parecer com:

```text
Ativo:
Base de conhecimento do RAG / contexto recuperado

Ameaça:
Injeção Indireta de Prompt

STRIDE:
Adulteração

MITRE ATLAS:
Injeção de Prompt em LLM

OWASP:
LLM01 — Injeção de Prompt

Possíveis Mitigações:
- Tratar o conteúdo recuperado como não confiável
- Restringir as fontes indexadas
- Validar o conteúdo
- Aplicar privilégio mínimo às ferramentas
- Impedir ações privilegiadas diretas
- Monitorar instruções anormais
- Exigir aprovação para operações sensíveis
```

Agora, a descoberta deixou de ser:

> **RAG pode ser perigoso.**

Ela se torna uma descoberta de segurança específica e acionável.

---

## Os Três Frameworks Funcionam como Camadas

Esta é provavelmente a relação mais prática entre frameworks que aprendi neste Dia.

```text
STRIDE
   ↓
Qual é o tipo de ameaça?

MITRE ATLAS
   ↓
Como o adversário pode executá-la?

OWASP LLM Top 10
   ↓
Onde o risco reside na arquitetura?
```

Não preciso escolher apenas um framework.

Eles respondem a perguntas diferentes.

---

## Minha Analogia da Câmera

A sala usou uma analogia que ajudou a tornar essa relação intuitiva.

Penso nela como diferentes níveis de zoom:

```text
STRIDE
→ Visão grande-angular

ATLAS
→ Detalhe técnico

OWASP
→ Para onde apontar a câmera
```

Juntos, eles transformam uma preocupação ampla de segurança em uma avaliação acionável da arquitetura.

---

## Modelagem de Ameaças É Mais do que Relacionar Vulnerabilidades

Este Dia mudou minha forma de pensar sobre um modelo de ameaças.

Ele não deveria simplesmente se tornar:

```text
Vulnerabilidade 1
Vulnerabilidade 2
Vulnerabilidade 3
```

Um modelo de ameaças útil precisa de relações.

Por exemplo:

```text
Componente
   ↓
Ativo
   ↓
Ameaça
   ↓
Técnica de Ataque
   ↓
Categoria de Risco Afetada
   ↓
Impacto para o Negócio
   ↓
Mitigação
   ↓
Prioridade
```

É essa relação que dá valor à avaliação.

---

## A Proveniência Continua Importante

Uma coisa que trouxe dos Dias anteriores para a modelagem de ameaças foi a proveniência.

Quando recebo uma arquitetura de IA, também quero saber:

* De onde veio o modelo?
* Quem o treinou?
* Qual conjunto de dados foi usado?
* Quem rotulou os dados?
* Qual modelo-base foi usado?
* Quais dependências de terceiros existem?
* Quem teve acesso ao treinamento?
* Onde os artefatos são armazenados?
* Como eles são validados?
* Como eles são assinados/versionados?

A modelagem de ameaças precisa de informações arquiteturais, mas também se beneficia da compreensão do histórico dos ativos.

---

## A Qualidade dos Dados Continua Importante

A modelagem de ameaças também precisa levar em conta fragilidades não maliciosas que podem afetar os resultados de segurança.

Quero entender:

* representatividade do conjunto de dados;
* viés;
* resultados de validação;
* limitações conhecidas;
* falsos positivos;
* falsos negativos;
* desvio do modelo;
* padrões de comportamento indesejados.

O risco de segurança nem sempre é causado por um invasor que comprometeu um servidor.

Às vezes, o modelo apresenta um comportamento ruim porque suas entradas ou seu processo de treinamento já tinham falhas.

---

## Minha Metodologia de Modelagem de Ameaças

Ao final do Dia 08, desenvolvi uma sequência que faz sentido para mim ao receber uma nova arquitetura de IA.

### Etapa 1 — Receber a Arquitetura

Entender o que está sendo implantado.

```text
LLM
RAG
Banco de Dados Vetorial
Pipeline de Treinamento
Registro de Modelos
Ferramentas
APIs
Dados Externos
```

---

## Etapa 2 — Mapear Componentes e Fluxos de Dados

Perguntar:

* O que se comunica com o quê?
* Onde entram os dados do usuário?
* Onde entra o conteúdo externo?
* Quais componentes acionam ações?
* Onde os modelos são armazenados?
* De onde vêm os dados de treinamento?

---

## Etapa 3 — Identificar os Ativos

Para cada componente:

> **O que tem valor aqui?**

Exemplos:

* pesos do modelo;
* dados de treinamento;
* embeddings;
* prompts de sistema;
* documentos;
* credenciais;
* artefatos do modelo;
* features;
* propriedade intelectual.

---

## Etapa 4 — Identificar os Limites de Confiança

Perguntar:

> **Onde os dados transitam entre diferentes níveis de confiança?**

Exemplos:

```text
Usuário → Aplicação

Aplicação → LLM

LLM → Ferramenta

Dados Externos → RAG

Registro → Implantação

Dados de Treinamento → Pipeline de Treinamento
```

---

## Etapa 5 — Revisar a Proveniência e a Cadeia de Suprimentos

Perguntar:

* Quem forneceu este modelo?
* Quem forneceu este conjunto de dados?
* Quais dependências externas existem?
* Como os artefatos são verificados?
* As versões do modelo podem ser substituídas?
* A proveniência está documentada?

---

## Etapa 6 — Revisar Permissões e Capacidades

Para sistemas agênticos:

```text
O que o modelo pode fazer?
```

Não apenas:

```text
O que o modelo pode ler?
```

mas também:

* executar;
* enviar;
* modificar;
* excluir;
* implantar;
* aprovar.

As capacidades passam a fazer parte do modelo de ameaças.

---

## Etapa 7 — Revisar a Qualidade e a Validação dos Dados

Perguntar:

* Os dados são representativos?
* Há dados sensíveis presentes?
* Os rótulos são confiáveis?
* A validação é abrangente?
* Cenários com gatilhos/adversariais são testados?
* Os vieses conhecidos estão documentados?

---

## Etapa 8 — Aplicar o STRIDE

Percorrer cada componente e limite de confiança considerando:

```text
Falsificação
Adulteração
Repúdio
Divulgação de Informações
Negação de Serviço
Elevação de Privilégio
```

O objetivo é:

> **Identificar o que pode dar errado.**

---

## Etapa 9 — Enriquecer com o MITRE ATLAS

Para cada ameaça relevante:

> **Como um adversário realmente realizaria isso contra um sistema de IA?**

Use as técnicas e mitigações do ATLAS para tornar a descoberta mais precisa.

---

## Etapa 10 — Mapear os Riscos da OWASP aos Componentes

Perguntar:

> **Quais riscos específicos de LLM residem neste componente?**

Isso ajuda a definir o escopo e a priorização da avaliação.

---

## Etapa 11 — Avaliar Impacto e Probabilidade

Agora, conecte as ameaças técnicas às consequências para o negócio.

Exemplos:

* perda financeira;
* exposição de dados;
* impacto regulatório;
* interrupção do serviço;
* roubo do modelo;
* decisões fraudulentas;
* dano à reputação;
* ações automatizadas incorretas.

---

## Etapa 12 — Definir Mitigações

Os controles podem incluir:

* rastreamento de proveniência;
* controle de acesso;
* privilégio mínimo;
* assinatura de artefatos;
* validação de dados;
* monitoramento;
* fluxos de aprovação;
* sanitização da saída;
* limitação de taxa;
* registro de modelos seguro;
* fontes confiáveis para o RAG;
* testes adversariais.

---

## Etapa 13 — Priorizar

Nem toda ameaça apresenta o mesmo risco.

Uma avaliação útil precisa identificar:

```text
O que devemos corrigir primeiro?
```

É aí que as descobertas técnicas se tornam úteis para a organização.

---

## Meu Fluxo de Trabalho Final

Meu modelo mental agora se parece com:

```text
Receber a Arquitetura
        ↓
Mapear Componentes e Fluxos de Dados
        ↓
Identificar Ativos
        ↓
Identificar Limites de Confiança
        ↓
Verificar Proveniência e Cadeia de Suprimentos
        ↓
Entender Permissões e Capacidades
        ↓
Revisar Qualidade e Validação dos Dados
        ↓
Aplicar o STRIDE
        ↓
Enriquecer com o MITRE ATLAS
        ↓
Mapear Riscos da OWASP aos Componentes
        ↓
Avaliar Impacto e Probabilidade
        ↓
Definir Mitigações
        ↓
Avaliação de Riscos Priorizada
```

Este é o maior aprendizado prático que quero guardar do Dia 08.

---

## O Que Mudou no Meu Entendimento

Antes do Dia 08, eu poderia ter iniciado uma revisão de segurança procurando imediatamente por vulnerabilidades.

Agora, começaria mais cedo.

Primeiro:

> **O que existe?**

Depois:

> **O que tem valor?**

Em seguida:

> **Como os dados se movimentam?**

Depois:

> **Onde a confiança muda?**

Então:

> **Como este modelo/estes dados foram criados e entregues?**

Somente depois disso começo a enumerar sistematicamente as ameaças.

Isso transforma a modelagem de ameaças: de um checklist de vulnerabilidades em uma avaliação de segurança orientada pela arquitetura.

---

## Estender, Não Substituir

Uma das lições mais importantes do Dia 08 é:

> **A modelagem de ameaças em IA não exige descartar tudo que já sabemos.**

Os princípios tradicionais de segurança ainda se aplicam.

O STRIDE ainda se aplica.

A tríade CIA ainda se aplica.

Os limites de confiança ainda se aplicam.

A segurança da cadeia de suprimentos ainda se aplica.

O privilégio mínimo ainda se aplica.

Mas a IA acrescenta:

* novos ativos;
* novas dependências;
* novos comportamentos;
* novas maneiras de falhar;
* novas técnicas de ataque;
* novas etapas do ciclo de vida.

Portanto, a abordagem correta é:

> **Estender, não substituir.**

---

## Meu Maior Aprendizado

Se eu tivesse que resumir o Dia 08 em uma frase:

> **A modelagem de ameaças em IA começa antes das vulnerabilidades: primeiro, entenda a arquitetura, os ativos, os fluxos de dados e os limites de confiança; depois, aplique os frameworks para identificar como esses ativos podem ser atacados e como o risco deve ser mitigado.**

O STRIDE me informa a categoria da ameaça.

O MITRE ATLAS me informa como o adversário pode executá-la.

O OWASP LLM Top 10 me informa onde esse risco reside na arquitetura.

E a avaliação final precisa conectar tudo isso a:

> **Impacto para o negócio + mitigação + prioridade.**

É isso que transforma o conhecimento sobre segurança de IA em um modelo de ameaças acionável.

---

## A Seguir

A próxima parte desta jornada de aprendizado continuará a partir desta metodologia e a aplicará a novos cenários de segurança de IA.

O processo continua sendo:

**Aprender → Questionar → Entender → Aplicar → Compartilhar**

---

## Referências

- [Microsoft — O modelo de ameaças STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
- [MITRE — ATLAS](https://atlas.mitre.org/)
- [OWASP — Top 10 para Aplicações de LLM e IA Generativa](https://genai.owasp.org/llm-top-10/)
- [NIST — Framework de Gestão de Riscos de IA](https://www.nist.gov/itl/ai-risk-management-framework)

---

## Sobre Este Diário de Aprendizado

Este repositório documenta minha jornada pessoal de aprendizado e minhas reflexões durante o estudo de Segurança de IA.

A trilha de aprendizado é inspirada em meus estudos com o **material de Segurança de IA do TryHackMe**, combinados com minha experiência anterior em cibersegurança e gestão de incidentes.

As explicações, analogias, exemplos e conclusões apresentados aqui representam meu próprio entendimento e minhas reflexões.

Este repositório não reproduz laboratórios, perguntas, soluções, flags ou conteúdo proprietário de cursos do TryHackMe.

**Aprender → Questionar → Entender → Aplicar → Compartilhar**
