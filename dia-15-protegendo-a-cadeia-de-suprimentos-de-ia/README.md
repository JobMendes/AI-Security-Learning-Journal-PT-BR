# Dia 15 — Protegendo a cadeia de suprimentos de IA

<p align="center">
  <img src="../Pictures/Day15.png" alt="Diário de Aprendizado em Segurança de IA — Dia 15: Protegendo a cadeia de suprimentos de IA" width="100%">
</p>

> Proteger a cadeia de suprimentos de IA exige várias formas independentes de evidência antes que um artefato seja considerado confiável ou promovido.

## Em resumo

**Tempo de leitura:** cerca de 30 minutos

Esta entrada desenvolve um processo defensivo de garantia usando proveniência, assinaturas, hashes, scanning, SBOMs, promoção controlada, validação e monitoramento.

**Principais aprendizados:**

- Integridade, proveniência, segurança da serialização, scanning de vulnerabilidades e validação de comportamento não são intercambiáveis.
- As decisões de admissão e promoção devem ser orientadas por políticas e reproduzíveis.
- A confiança deve ser reavaliada quando artefatos, dependências, ambientes ou comportamentos mudam.

**Caminho sugerido:** Comece pelas camadas de evidência e, em seguida, use o pipeline de promoção e o modelo de revisão final como um blueprint de implementação.

**Navegação rápida:** [Quarentena](#quarentena-vem-antes-da-confiança) · [Verificação de integridade](#verificação-de-integridade) · [Proveniência](#proveniência) · [Framework de aquisição](#meu-framework-atualizado-de-aquisição-de-modelos)

## Diário de Aprendizado em Segurança de IA

Hoje concluí a parte defensiva da sequência sobre a cadeia de suprimentos de IA.

A progressão dos últimos três dias agora parece muito clara:

> **O Dia 13 perguntou: Em que estou confiando?**

> **O Dia 14 perguntou: Como um atacante pode explorar essa confiança?**

> **O Dia 15 pergunta: Que evidências devo exigir antes de conceder essa confiança?**

Isso mudou a forma como penso sobre a segurança da cadeia de suprimentos de IA.

O objetivo não é encontrar uma única ferramenta capaz de declarar um modelo "seguro".

Esse controle não existe.

Em vez disso, a segurança vem da combinação de controles independentes em diferentes camadas:

```text
Fonte
  ↓
Quarentena
  ↓
Proveniência
  ↓
Integridade
  ↓
Serialização
  ↓
Análise Estática
  ↓
Arquitetura
  ↓
Dependências
  ↓
Comportamento
  ↓
Aprovação
  ↓
Monitoramento em Produção
```

Cada controle responde a uma pergunta diferente.

E passar em um deles não responde automaticamente aos demais.

---

## Do Risco da Cadeia de Suprimentos aos Controles da Cadeia de Suprimentos

Nos dias anteriores, aprendi que uma aplicação de IA pode herdar confiança de muitos lugares:

- modelos;
- datasets;
- adapters;
- dependências;
- repositórios;
- mantenedores;
- formatos de serialização;
- pipelines de conversão;
- infraestrutura;
- templates de prompt;
- APIs externas;
- provedores de IA.

Isso cria um problema defensivo fundamental:

> **Como decido se um artefato ou provedor merece tornar-se parte do meu ambiente de produção confiável?**

Minha resposta após esta lição é:

**Não concedo confiança com base em um único sinal. Acumulo evidências.**

Um publisher conhecido é uma evidência.

Um hash correspondente é uma evidência.

Um static scan limpo é uma evidência.

Uma cadeia de proveniência documentada é uma evidência.

Uma avaliação comportamental bem-sucedida é uma evidência.

Nenhuma delas, individualmente, prova segurança.

---

## A Confiança Deve Ser Conquistada Antes da Produção

Uma das ideias mais fortes que levo deste módulo é que baixar um modelo não deveria torná-lo automaticamente implantável.

Deve existir uma transição controlada entre:

```text
Recebido
```

e:

```text
Confiável para Produção
```

Um ciclo de vida útil para aquisição de modelos é:

```text
Adquirir
   ↓
Quarentena
   ↓
Verificar Fonte
   ↓
Verificar Integridade
   ↓
Inspecionar
   ↓
Fazer Scanning
   ↓
Avaliar
   ↓
Aprovar / Rejeitar
   ↓
Promover
```

Isso se assemelha a processos já conhecidos na cibersegurança.

Normalmente não recebemos um executável desconhecido e o instalamos imediatamente em um servidor de produção.

Artefatos de IA merecem a mesma disciplina de segurança.

---

## Quarentena Vem Antes da Confiança

A primeira fronteira de segurança deve existir antes do início da análise.

Um artefato que ainda não foi avaliado não deve chegar diretamente ao ambiente de produção.

Em vez disso:

```text
Fonte Externa
      ↓
Staging Isolado / Quarentena
      ↓
Avaliação de Segurança
      ↓
Registry Aprovado
      ↓
Produção
```

Essa distinção importa porque alguns formatos de modelo podem ter comportamento perigoso associado ao carregamento ou à reconstrução.

Se a própria análise puder acionar o artefato, realizar a análise dentro do ambiente de produção confiável anula o propósito do controle.

O artefato deve começar seu ciclo de vida como:

> **Não confiável até ser avaliado.**

E não como:

> **Confiável a menos que algo pareça suspeito.**

---

## Serialização Segura

Um dos controles mais diretos contra ataques de serialização é reduzir aquilo que o formato do modelo é capaz de fazer.

Isso é especialmente importante com artefatos baseados em Python pickle.

Pickle é poderoso porque pode reconstruir objetos Python.

Mas essa flexibilidade cria risco de segurança.

Conceitualmente:

```text
Objeto Serializado
      ↓
Desserialização
      ↓
Reconstrução do Objeto
      ↓
Possível Invocação de Função
```

Isso significa que carregar um pickle não confiável não equivale a ler dados passivos.

Essa ação pode atravessar uma fronteira de execução.

---

## SafeTensors

SafeTensors adota uma abordagem fundamentalmente diferente.

Em vez de oferecer suporte à reconstrução arbitrária de objetos Python, o formato foi projetado em torno de dados de tensores.

Conceitualmente:

```text
SafeTensors
├── Metadados / Cabeçalho
└── Dados dos Tensores
```

em vez de:

```text
Pickle
├── Objetos
├── Instruções de Reconstrução
├── Imports
└── Possíveis Chamadas de Função
```

Isso reduz drasticamente o risco de execução de código no nível da serialização.

Para pesos de modelos, essa é uma melhoria importante de segurança.

Mas a lição do Dia 14 continua válida:

> **Um formato de serialização mais seguro remove um vetor de ataque. Ele não prova que o modelo é confiável.**

SafeTensors pode ajudar a responder:

> "Esse mecanismo de serialização pode executar Python arbitrário durante o carregamento?"

Ele não responde:

> "Esses pesos são maliciosos?"

ou:

> "Esse modelo contém lógica de arquitetura perigosa?"

ou:

> "Este modelo foi envenenado durante o treinamento?"

---

## Carregamento Restrito

Quando não é possível eliminar imediatamente artefatos legados ou baseados em pickle, restringir a desserialização torna-se outra camada defensiva.

O PyTorch moderno oferece carregamento restrito por meio de:

```python
torch.load("model.pt", weights_only=True)
```

A ideia de segurança é mais importante do que a sintaxe.

Em vez de permitir a reconstrução de objetos Python arbitrários, o loader restringe o que o desserializador pode criar.

Conceitualmente:

```text
Desserialização Irrestrita
        ↓
Grande Superfície de Capacidades

Desserialização Restrita
        ↓
Reconstrução dos Tensores Esperados
        ↓
Superfície de Capacidades Reduzida
```

Isso segue um princípio conhecido da cibersegurança:

> **Reduza as capacidades antes de aumentar a confiança.**

---

## Extensões de Arquivo Não São Controles de Segurança

Outra lição importante é que o nome do arquivo não pode estabelecer o formato real.

Algo chamado:

```text
model.safetensors
```

não deve ser automaticamente confiável apenas porque a extensão diz `safetensors`.

A decisão de segurança deve basear-se no próprio artefato.

Esse é outro exemplo de um princípio mais amplo:

> **Rótulos descrevem. A validação prova propriedades.**

O mesmo raciocínio se aplica a repositórios, publishers, packages, model cards e identificadores de API.

Nomes são metadados úteis.

Eles não são fronteiras de segurança.

---

## Verificação de Integridade

Depois de controlar a serialização, resta outra pergunta:

> **Recebi o artefato que esperava?**

É aqui que hashes criptográficos se tornam úteis.

Por exemplo:

```text
Artefato Esperado
      ↓
SHA-256 Esperado
      ↓
Artefato Recebido
      ↓
SHA-256 Calculado
      ↓
Comparar
```

Se os hashes forem diferentes:

```text
Esperado ≠ Recebido
```

então algo mudou.

O artefato pode ter sido corrompido, substituído, modificado ou adulterado.

Essa é uma evidência valiosa.

---

## O Que um Hash Correspondente Realmente Prova

Essa distinção é crítica.

Suponha:

```text
SHA-256 Esperado == SHA-256 Calculado
```

O que provei?

Tenho fortes evidências de que o artefato recebido é idêntico ao artefato representado pelo hash esperado.

Eu **não** provei que o artefato é benigno.

Um publisher malicioso poderia publicar:

```text
modelo_malicioso
+
hash_correto_do_modelo_malicioso
```

e minha verificação seria bem-sucedida.

Portanto:

> **Integridade responde "O artefato mudou?"**

Ela não responde necessariamente:

> **"Devo confiar no artefato?"**

Essa distinção entre **integridade** e **confiabilidade** tornou-se um dos temas recorrentes deste módulo.

---

## Integridade do Artefato vs. Integridade do Comportamento

Agora separo dois conceitos importantes.

### Integridade do Artefato

```text
Este é exatamente o artefato que eu esperava?
```

Os controles incluem:

- hashes criptográficos;
- assinaturas;
- registries controlados;
- versões imutáveis;
- registros de proveniência.

### Integridade do Comportamento

```text
Este artefato se comporta de acordo com as propriedades de segurança que espero?
```

Os controles incluem:

- testes comportamentais;
- testes de regressão;
- avaliação adversarial;
- avaliação de segurança;
- monitoramento em produção.

Essas propriedades podem divergir.

Por exemplo:

```text
Hash correto .............. SIM
Artefato esperado ......... SIM
Comportamento confiável .... NÃO
```

Um hash correto pode coexistir com um modelo que contém um backdoor.

Isso significa:

> **A integridade do artefato é uma evidência necessária, mas a integridade comportamental é um problema de segurança separado.**

---

## Assinaturas Digitais Adicionam Identidade

Checksums estabelecem integridade em relação a um valor esperado.

Assinaturas digitais podem acrescentar outra dimensão:

```text
Artefato
   ↓
Integridade
+
Identidade do Publisher
```

Isso permite que o processo de segurança pergunte não apenas:

> "Este artefato mudou?"

mas também:

> "Quem assinou ou publicou este artefato?"

Isso melhora a proveniência.

Mas, novamente, assinaturas não eliminam todos os riscos.

Um publisher legítimo comprometido poderia assinar um artefato malicioso.

O padrão recorrente permanece:

> **Cada controle fortalece as evidências. Nenhum controle isolado elimina a necessidade dos demais.**

---

## Proveniência

Proveniência responde a perguntas como:

- Quem criou este modelo?
- Qual organização o publicou?
- Qual é esta versão?
- Onde ele foi obtido?
- Quais dados de treinamento estão documentados?
- Qual framework foi usado?
- O artefato foi transformado?
- Ele foi convertido por outro serviço?
- Foi fine-tuned?
- Um adapter foi adicionado?
- Quem o aprovou?
- Qual artefato realmente chegou à produção?

Essa é a cadeia de confiança do modelo.

Sem proveniência, um artefato pode funcionar perfeitamente e ainda assim nos deixar incapazes de explicar de onde veio ou o que aconteceu com ele antes da implantação.

---

## Model Cards como Evidência de Segurança

Model cards não são certificados de segurança.

Mas são evidências úteis de proveniência.

Eu esperaria documentação sobre áreas como:

```text
Identidade do Modelo
├── Autor
├── Organização
├── Versão
└── Licença

Finalidade
├── Uso Pretendido
└── Uso Fora do Escopo

Treinamento
├── Fontes dos Dados
├── Metodologia
└── Restrições Conhecidas

Avaliação
├── Métricas
└── Benchmarks

Limitações
├── Vieses
├── Modos de Falha
└── Riscos Conhecidos
```

A ausência de informações não prova automaticamente intenção maliciosa.

Mas aumenta a incerteza.

Isso leva a uma distinção importante em relação aos dias anteriores:

> **A ausência de proveniência é evidência de incerteza, não evidência de comprometimento.**

As decisões de segurança devem responder adequadamente a essa incerteza.

---

## Reputação É um Sinal, Não uma Garantia

Durante meus estudos, considerei um cenário em que dois modelos tinham reputações muito diferentes entre seus publishers.

Um vinha de uma organização famosa, mas falhava na avaliação de segurança.

Outro vinha de um publisher menos conhecido, mas tinha evidências técnicas mais fortes.

Minha decisão de produção favoreceria o modelo com evidências mais fortes.

Por quê?

Porque:

```text
Popularidade ≠ Integridade
Downloads ≠ Segurança
Reputação ≠ Proveniência
Badge de Verificação ≠ Segurança Comportamental
```

Um publisher desconhecido não prova que há malícia.

Um publisher famoso não prova segurança.

O princípio que quero manter é:

> **A reputação pode influenciar minha confiança inicial, mas as evidências de segurança observadas devem influenciar minha decisão de produção.**

---

## LoRA Adapters Também São Artefatos da Cadeia de Suprimentos

Um modelo-base não é necessariamente o modelo final.

Os sistemas modernos modificam cada vez mais os foundation models usando adapters como LoRA.

Conceitualmente:

```text
Modelo-Base Confiável
       +
Adapter de Terceiros
       ↓
Comportamento Modificado
```

Isso cria outra relação de confiança.

Mesmo que o modelo-base tenha passado por todos os security gates, adicionar um adapter não confiável muda o artefato que está sendo avaliado.

Portanto, o adapter deve ter seu próprio processo de intake:

```text
Adquirir
↓
Quarentena
↓
Verificar Fonte
↓
Inspecionar
↓
Fazer Scanning
↓
Avaliar
↓
Aprovar
```

Um modelo-base limpo não torna automaticamente confiável toda adaptação downstream.

---

## A Conversão Cria uma Nova Fronteira de Confiança

O mesmo raciocínio se aplica quando um modelo é transformado.

Por exemplo:

```text
Modelo Original
      ↓
Serviço de Conversão de Terceiros
      ↓
Modelo Convertido
```

O resultado agora é um novo artefato.

Mesmo que eu confiasse no artefato original, também preciso confiar em:

- o processo de conversão;
- o serviço;
- o ambiente;
- a saída;
- o registro de proveniência.

Portanto, o modelo convertido deve ser avaliado novamente.

Isso me dá uma regra geral:

> **Quando um artefato atravessa uma fronteira de transformação, reavalie a confiança no artefato resultante.**

---

## Scanning Estático de Modelos

Integridade e proveniência ainda não me dizem tudo o que existe dentro de um artefato.

É aqui que a análise estática se torna importante.

O objetivo é:

```text
Artefato
   ↓
Inspecionar sem Executar
   ↓
Identificar Operações Suspeitas
   ↓
Decisão de Segurança
```

Isso é especialmente importante para formatos capazes de representar comportamento executável de reconstrução.

---

## Fickling

Fickling oferece recursos de análise estática para artefatos Python pickle.

A propriedade de segurança importante é que o artefato pode ser inspecionado sem realizar a reconstrução normal e insegura de objetos.

O objetivo defensivo não é:

```text
Carregar modelo → observar se algo ruim acontece
```

mas:

```text
Inspecionar artefato → entender o que o carregamento poderia fazer
```

Isso segue o princípio que estabelecemos no Dia 14:

> **Não execute um artefato não confiável para descobrir se é seguro executá-lo.**

A análise estática tenta mover a inspeção para antes da execução.

---

## ModelScan

ModelScan amplia o scanning de segurança de modelos para vários formatos de machine learning.

Em vez de presumir que toda condição suspeita significa comprometimento, as descobertas podem ser classificadas por severidade e revisadas em contexto.

Isso importa porque:

```text
Suspeito ≠ Malicioso
```

Por exemplo, algum comportamento customizado de um modelo pode ser legítimo.

As ferramentas de segurança devem fornecer evidências para investigação, e não substituir a análise.

Esse é outro princípio recorrente:

> **A saída do scanner é uma evidência para uma decisão, não a própria decisão.**

---

## Um Scan Limpo Não Prova que o Modelo Está Limpo

Esta é uma das limitações mais importantes.

Suponha:

```text
SafeTensors ............... PASSOU
SHA-256 ................... PASSOU
Static Scan ............... PASSOU
Dependências .............. PASSOU
```

Posso concluir:

```text
MODELO SEGURO
```

Não.

Scanners estáticos detectam aquilo para o qual foram projetados.

Um ataque sofisticado pode existir, em vez disso, em:

- arquitetura do modelo;
- pesos;
- dados de treinamento;
- adapters;
- prompts externos;
- comportamento do provedor.

Portanto:

> **Passar em todas as verificações estáticas aumenta a confiança. Isso não prova matematicamente a confiabilidade.**

---

## Inspeção da Arquitetura

O Dia 14 introduziu uma distinção importante:

```text
Ataque no Nível da Serialização
Ataque no Nível da Arquitetura
Ataque no Nível dos Pesos
```

O Dia 15 acrescenta controles defensivos para cada camada.

O scanning de serialização, sozinho, não pode validar a arquitetura do modelo.

Um modelo pode ser carregado sem problemas e ainda conter componentes customizados que executam ou influenciam o comportamento durante a inferência.

Conceitualmente:

```text
Carregamento do Modelo
   ↓
Nenhum Problema
   ↓
A Inferência Começa
   ↓
Lógica de Arquitetura Customizada
   ↓
Comportamento Inesperado
```

É por isso que a inspeção da arquitetura importa.

---

## Camadas Customizadas Exigem Investigação

Camadas customizadas não são automaticamente maliciosas.

Desenvolvedores de machine learning criam legitimamente operações customizadas.

Mas, em um modelo recebido de uma fonte externa, uma lógica customizada inesperada aumenta a exigência de revisão.

O raciocínio correto é:

```text
Camada Customizada
    ↓
Suspeita?
    ↓
Investigar
```

e não:

```text
Camada Customizada
    ↓
Automaticamente Maliciosa
```

Essa distinção importa porque as ferramentas de segurança devem evitar transformar escolhas incomuns de implementação diretamente em conclusões de comprometimento.

---

## SafeTensors Não Protege a Arquitetura

Este é outro limite útil.

SafeTensors protege a serialização dos dados dos tensores.

Ele não valida a lógica da aplicação ou da arquitetura que envolve esses tensores.

Portanto:

```text
Serialização Segura
      ≠
Arquitetura Segura
```

e:

```text
Arquitetura Segura
      ≠
Pesos Seguros
```

Superfícies de ataque diferentes exigem controles diferentes.

---

## Backdoors Comportamentais

Um dos cenários que considerei durante meus estudos envolvia um modelo que passava por:

```text
Formato seguro
Verificação de integridade
Scanning estático
Inspeção da arquitetura
Auditoria de dependências
```

mas se comportava incorretamente apenas quando aparecia um padrão de entrada muito específico.

Isso me levaria a investigar um possível backdoor comportamental ou no nível dos pesos.

Conceitualmente:

```text
Entrada Normal
    ↓
Comportamento Esperado

Entrada-Gatilho
    ↓
Comportamento Inesperado / Malicioso
```

Esse tipo de problema pode não exigir:

- serialização perigosa;
- arquitetura customizada;
- execução de shell;
- atividade de rede.

A propriedade maliciosa pode existir naquilo que o modelo aprendeu.

É por isso que a avaliação comportamental continua sendo necessária mesmo após a inspeção técnica do artefato.

---

## Avaliação Comportamental

Antes da produção, o modelo deve ser avaliado em relação às propriedades funcionais e de segurança esperadas.

Isso pode incluir:

- casos de teste fixos;
- respostas reconhecidamente corretas;
- casos adversariais;
- testes dos limites de segurança;
- testes de regressão;
- comparação com um baseline aprovado.

Conceitualmente:

```text
Modelo Candidato
      ↓
Avaliação Isolada
      ↓
Bateria de Testes Conhecida
      ↓
Comportamento Esperado?
   ↙              ↘
 Sim              Não
 ↓                 ↓
Continuar        Rejeitar / Investigar
```

Isso leva a decisão de segurança para além de:

> "O modelo carrega?"

em direção a:

> "O modelo se comporta dentro das propriedades exigidas por esta aplicação?"

---

## Dependências Fazem Parte da Cadeia de Suprimentos de IA

Um modelo de IA raramente opera sozinho.

O software ao redor pode depender de:

```text
Python
├── ML Framework
├── Tokenizer
├── Cliente HTTP
├── Bibliotecas de Dados
├── Bibliotecas de Serialização
└── Dependências Transitivas
```

Cada dependência acrescenta outra relação de confiança.

Isso significa que a segurança do modelo não pode parar no arquivo do modelo.

---

## Pinning de Versões

Um controle básico de dependências é especificar exatamente o que deve ser instalado.

Sem versões controladas:

```text
package
```

pode significar:

```text
qualquer versão que o resolver selecionar hoje
```

Com pinning:

```text
package==approved-version
```

a dependência esperada se torna mais determinística.

Isso reduz mudanças inesperadas.

Mas o pinning de versões, sozinho, ainda não estabelece proveniência ou integridade.

---

## Lockfiles

Lockfiles ampliam o controle de dependências ao registrar a resolução exata das dependências.

Dependendo do ecossistema, isso pode incluir:

- versões exatas;
- dependências transitivas;
- hashes criptográficos.

Conceitualmente:

```text
Aplicação
   ↓
Lockfile
   ↓
Grafo Exato de Dependências
   ↓
Instalação Repetível
```

Isso torna o desenvolvimento, os testes, CI/CD e a produção mais reproduzíveis.

---

## Dependency Confusion Revisitado

O Dia 14 me ensinou o lado ofensivo:

```text
Registry Privado
    ↓
internal-package

Registry Público
    ↓
internal-package
    ↓
versão superior
```

Se o resolver usar o nome do package e a precedência de versão sem impor a proveniência, o package público poderá ser selecionado.

O problema arquitetural não é simplesmente que:

> "Existia um package malicioso."

O problema mais profundo é:

> **Foi permitido que o resolver decidisse a proveniência com base na identidade do package e na precedência de versão.**

Isso me dá um princípio defensivo:

> **Precedência de versão não é imposição de proveniência.**

As identidades de packages internos devem ser resolvidas por meio de fontes controladas.

---

## Fontes Privadas de Packages

Para dependências internas, as organizações podem usar registries privados controlados e políticas explícitas de origem.

O objetivo de segurança não é necessariamente:

```text
Bloquear todo o ecossistema público
```

mas:

```text
Package Interno
      ↓
Fonte Interna Aprovada
      ↓
Não Pode Ser Substituído por Homônimo Público
```

Isso preserva o uso legítimo de packages externos e, ao mesmo tempo, impede que packages públicos se passem por eles.

---

## Valide o Que Foi Realmente Obtido

O controle da origem deve ser combinado com a verificação do artefato.

Meu raciocínio defensivo passou a ser:

```text
Package Esperado
      ↓
Fonte Esperada
      ↓
Versão Esperada
      ↓
Hash Esperado
      ↓
Avaliação de Segurança / Vulnerabilidades
      ↓
Dependência Aprovada
```

Isso cria várias verificações independentes.

Se um controle falhar, outro ainda poderá expor a inconsistência.

---

## Malicioso Não É o Mesmo que Vulnerável

Scanners de dependências são extremamente úteis.

Mas é preciso entender corretamente sua finalidade.

Um scanner de vulnerabilidades pode identificar vulnerabilidades conhecidas associadas a packages e versões.

Ele pode não identificar um package novo criado intencionalmente para executar comportamento malicioso.

Portanto:

```text
Vulnerabilidade Conhecida
        ≠
Malícia Intencional
```

Isso me dá outro princípio:

> **Um package pode ser malicioso sem conter uma CVE conhecida.**

Gerenciamento de vulnerabilidades e proveniência da cadeia de suprimentos se sobrepõem, mas não são problemas de segurança idênticos.

---

## Auditoria de Dependências

Ferramentas como `pip-audit` ajudam a identificar vulnerabilidades conhecidas em conjuntos de dependências Python.

Isso fornece respostas como:

```text
Qual package?
Qual versão?
Qual vulnerabilidade conhecida?
Qual versão contém uma correção?
```

Isso é valioso para o gerenciamento de vulnerabilidades.

Mas, novamente:

> **Nenhuma descoberta não significa ausência de risco na cadeia de suprimentos.**

Significa que o scanner não identificou vulnerabilidades conhecidas dentro do seu escopo e das informações disponíveis.

---

## Software Bill of Materials

O SBOM foi um dos conceitos que considerei especialmente útil sob a perspectiva de resposta a incidentes.

Um Software Bill of Materials fornece um inventário estruturado dos componentes que formam uma aplicação.

Conceitualmente:

```text
Aplicação
├── Package A
│   ├── Dependência A1
│   └── Dependência A2
├── Package B
└── Framework C
```

Sem esse inventário, uma vulnerabilidade recém-divulgada pode transformar-se em um exercício de descoberta.

---

## Por Que o SBOM É Importante Durante um Incidente

Imagine que uma vulnerabilidade crítica seja anunciada para:

```text
Library X
Versions Y-Z
```

e que a organização tenha dezenas de aplicações de AI/ML em:

```text
Development
FQA
Production
```

Sem um inventário confiável de componentes, as perguntas imediatas tornam-se:

```text
Onde está a Library X?
Quais aplicações a utilizam?
Quais versões?
Ela é direta ou transitiva?
Quais ambientes são afetados?
Quais sistemas de produção exigem ação imediata?
```

Tentar descobrir tudo isso durante uma emergência gera atraso.

Um SBOM atualizado muda o ponto de partida.

```text
Nova CVE
   ↓
Componente / Versão Afetado(a)
   ↓
Inventário do SBOM
   ↓
Aplicações Afetadas
   ↓
Implantações Afetadas
   ↓
DEV / FQA / PROD
   ↓
Remediação Priorizada
```

Minha conclusão é:

> **Sem um SBOM, uma vulnerabilidade recém-divulgada pode transformar a resposta a incidentes em um exercício de descoberta em toda a infraestrutura. Com um SBOM atualizado, posso partir do componente afetado e identificar quais aplicações e versões exigem investigação.**

---

## Dependências Transitivas Importam

A dependência que instalo explicitamente pode, por sua vez, instalar muitas outras.

Por exemplo:

```text
Minha Aplicação
    ↓
Library A
    ↓
Library B
    ↓
Library C
```

Se a Library C se tornar vulnerável, simplesmente revisar minha lista de dependências de nível superior pode não tornar o relacionamento óbvio.

É por isso que o inventário de dependências deve considerar o grafo completo, e não apenas os packages que os desenvolvedores se lembram de instalar diretamente.

---

## Formatos de SBOM

Dois ecossistemas importantes de SBOM são:

```text
SPDX
CycloneDX
```

Eles atendem a objetivos de inventário semelhantes, embora enfatizem áreas diferentes.

O formato exato é menos importante para meu raciocínio de segurança do que ter:

- inventário estruturado;
- versões;
- relacionamentos entre componentes;
- saída legível por máquina;
- integração com workflows de vulnerabilidade e compliance.

---

## ML Precisa de Mais do que um SBOM Tradicional

SBOMs tradicionais se concentram principalmente em componentes de software.

Sistemas de IA introduzem artefatos adicionais:

```text
Software
+
Modelo
+
Dataset
+
Adapter
+
Prompt / Política
+
Provedor
```

Isso significa que a visibilidade da cadeia de suprimentos de IA, com o tempo, precisará capturar informações além dos packages tradicionais.

Por enquanto, diferentes fontes de evidência podem se complementar:

```text
SBOM
+
Model Card
+
Model Registry
+
Linhagem do Dataset
+
Telemetria de Versões
+
Versionamento de Prompts
```

Juntas, elas oferecem uma visão mais rica do sistema de IA implantado.

---

## A Cadeia de Suprimentos da API

O modelo defensivo muda quando a organização não recebe um arquivo de modelo de forma alguma.

Com um provedor hospedado:

```text
Aplicação
    ↓
API
    ↓
Provedor
    ↓
Pipeline Interno de Modelo Desconhecido
```

Agora talvez eu não tenha acesso a:

```text
.pkl
SafeTensors
GGUF
Pesos
Arquitetura
Pipeline de Treinamento
```

Isso significa que vários controles locais desaparecem.

Não posso simplesmente calcular o SHA-256 do modelo.

Talvez eu não consiga inspecionar seus pesos.

Não posso executar meu scanner de modelos local contra o artefato interno do provedor.

Mas a cadeia de suprimentos ainda existe.

---

## A Cadeia de Suprimentos Fica Menos Visível

Migrar para uma API não elimina a confiança upstream.

Isso muda quem controla as evidências.

Agora estou confiando em coisas como:

- práticas de segurança do provedor;
- pipeline de treinamento;
- versionamento do modelo;
- infraestrutura;
- resposta a incidentes;
- práticas de privacidade;
- atualizações do modelo;
- autenticação da API;
- tratamento de prompts;
- transparência do provedor.

Isso reforça algo que aprendi no Dia 13:

> **A cadeia de suprimentos não desaparece atrás de uma API. Ela fica menos visível para o consumidor.**

---

## Due Diligence do Provedor

Antes de integrar um provedor externo de IA, eu gostaria de entender áreas como:

```text
Provedor
├── Tratamento de Dados
├── Retenção
├── Uso para Treinamento
├── Versionamento do Modelo
├── Notificações de Alteração
├── Certificações de Segurança
├── Resposta a Incidentes
├── Divulgação de Vulnerabilidades
├── Transparência
└── Política de Depreciação
```

Isso não elimina o risco do provedor.

Estabelece evidências antes que a confiança seja concedida.

---

## Baselines Comportamentais

Sem acesso ao artefato do modelo subjacente, o comportamento se torna especialmente importante.

Uma abordagem útil é manter uma bateria fixa de avaliações.

Por exemplo:

```text
Conjunto Fixo de Prompts
      ↓
Faixa de Comportamento Esperado
      ↓
Avaliação Periódica
      ↓
Comparar Resultados
      ↓
Mudança Significativa?
```

Como os LLMs são probabilísticos, o objetivo não é necessariamente a igualdade textual exata.

Em vez disso, eu monitoraria propriedades como:

- consistência das decisões;
- limites de segurança;
- comportamento de recusa;
- precisão factual;
- formato da saída;
- desempenho da tarefa;
- latência;
- padrões de erro.

As respostas podem variar e ainda assim convergir para o comportamento esperado.

---

## Comportamento como uma Espécie de Checksum Operacional

Durante meus estudos, descrevi isso como análise comportamental usando os mesmos prompts ao longo do tempo.

Isso me levou a uma analogia útil:

> **Se não posso verificar o artefato do modelo, preciso verificar o comportamento do modelo ao longo do tempo. Uma bateria fixa de prompts se torna meu checksum comportamental.**

Não é um checksum criptográfico.

Mas, operacionalmente, ele fornece um ponto de referência.

Se:

```text
Aplicação ........ inalterada
Dependências ..... inalteradas
System Prompt .... inalterado
Nome do Endpoint . inalterado
```

mas:

```text
Comportamento do Modelo ..... mudou significativamente
```

então uma mudança upstream do lado do provedor se torna uma hipótese que vale a pena investigar.

---

## Telemetria de Versões

O monitoramento comportamental se torna ainda mais forte quando combinado com a telemetria de versões do provedor/modelo.

Sem evidências de versão:

```text
Ontem → mesmo endpoint
Hoje   → mesmo endpoint
```

pode parecer que nada mudou.

Com telemetria de versões:

```text
Ontem → Revisão A do Modelo
Hoje   → Revisão B do Modelo
```

a investigação ganha outra pista causal.

Isso não revela necessariamente:

- quem treinou o modelo;
- quais dados foram usados;
- exatamente o que mudou.

Mas fornece rastreabilidade forense.

---

## System Prompts São Artefatos da Cadeia de Suprimentos

Essa foi uma das minhas conexões favoritas entre os módulos de Segurança de Prompts e Segurança da Cadeia de Suprimentos.

Considere:

```text
Repositório Externo
      ↓
Template de System Prompt
      ↓
Aplicação de IA
      ↓
Decisões em Produção
```

O template pode conter apenas texto.

Mas esse texto pode definir:

- políticas;
- regras de decisão;
- comportamento de escalonamento;
- requisitos de confidencialidade;
- instruções de ferramentas;
- expectativas de segurança.

Se um template externo mudar, o comportamento da aplicação pode mudar mesmo quando:

```text
Modelo ............ inalterado
Aplicação .......... inalterada
Dependências ....... inalteradas
Endpoint ........... inalterado
```

A política mudou.

---

## Instruções Confiáveis Podem Ter uma Origem Não Confiável

Antes, durante a jornada, Prompt Injection me ensinou:

> **Dados não confiáveis podem se tornar instruções.**

A segurança da cadeia de suprimentos acrescenta outra perspectiva:

> **As próprias instruções confiáveis podem chegar por meio de uma cadeia de suprimentos não confiável ou comprometida.**

Essa é uma conexão poderosa.

Isso significa que a governança de prompts não é apenas engenharia de prompts.

É segurança de configuração e de gerenciamento de mudanças.

---

## Trate Prompts Relevantes para a Segurança como Código

Um system prompt obtido externamente não deveria ir diretamente para produção.

Em vez de:

```text
Repositório Externo
       ↓
Busca Automática
       ↓
Produção
```

eu preferiria:

```text
Fonte Externa
       ↓
Mudança Proposta
       ↓
Controle de Versão Interno
       ↓
Revisão
       ↓
Testes de Segurança
       ↓
Aprovação
       ↓
Produção
```

Uma allowlist de rede ou de repositórios pode fortalecer esse processo.

Mas restringir apenas a fonte não é suficiente.

O controle mais profundo é:

> **Instruções confiáveis devem chegar à produção porque passaram por um processo confiável de mudanças.**

---

## Avaliação de API em Sandbox

Quando não posso inspecionar o modelo interno de um provedor, a avaliação dinâmica se torna mais importante.

Antes da promoção, eu testaria o modelo candidato contra:

```text
Testes Funcionais
+
Casos Conhecidamente Corretos
+
Testes de Segurança
+
Casos Adversariais
+
Comparação com o Baseline
```

e faria essa avaliação sem conceder imediatamente ao candidato capacidades irrestritas de produção.

Novamente, o padrão se assemelha à aquisição de modelos:

```text
Avaliar
   ↓
Coletar Evidências
   ↓
Aprovar / Rejeitar
   ↓
Promover
```

---

## O Modelo Pode Passar e o Sistema Ainda Assim Falhar

Uma das lições mais amplas dos exercícios foi que os controles individuais devem ser avaliados separadamente.

Imagine:

```text
Controle do Carregador do Modelo .... PASSOU
Controle de Dependências ............. FALHOU
```

A conclusão não deveria ser:

> "O carregador do modelo falhou."

Ele fez seu trabalho.

A arquitetura não cobriu outro caminho da cadeia de suprimentos.

É por isso que a defesa em profundidade é importante.

```text
Segurança do Modelo
+
Segurança de Dependências
+
Segurança do Repositório
+
Governança de Prompts
+
Segurança do Provedor
+
Monitoramento em Runtime
```

Um controle forte em uma camada não compensa automaticamente uma camada adjacente desprotegida.

---

## Aplicando os Controles: da Detecção à Investigação

Os exercícios práticos acrescentaram outra perspectiva que considero importante o bastante para incluir no meu diário.

A segurança da cadeia de suprimentos não trata apenas de impedir que artefatos maliciosos entrem em produção.

Às vezes, a prevenção falha.

Então a pergunta passa a ser:

> **Tenho evidências suficientes para reconstruir o que aconteceu?**

A mentalidade investigativa que desenvolvi foi:

```text
Detecção
   ↓
Reconstrução da Linha do Tempo
   ↓
Proveniência da Implantação
   ↓
Identificação do Artefato
   ↓
Verificação da Integridade
   ↓
Inspeção Segura do Artefato
   ↓
Telemetria em Runtime
   ↓
Correlação de Rede
   ↓
Análise Causal
   ↓
Contenção / Decisão de Implantação
```

Isso conecta diretamente a segurança da cadeia de suprimentos à DFIR.

---

## Comece pela Linha do Tempo

Um alerta de rede acontecendo hoje não prova que o comprometimento aconteceu hoje.

O artefato malicioso pode ter entrado no ambiente:

```text
minutos atrás
dias atrás
semanas atrás
```

e só ter acionado a detecção mais tarde.

Portanto, a primeira pergunta deveria ser:

> **O que aconteceu e quando?**

Eu correlacionaria:

- eventos de implantação;
- eventos do registry;
- aquisição do modelo;
- alterações no artefato;
- telemetria de carregamento;
- atividade de inferência;
- conexões de saída;
- alterações de configuração;
- credenciais;
- atualizações de prompts/templates.

Isso me impede de tratar o timestamp da detecção como o timestamp do comprometimento.

---

## Correlação Antes da Causalidade

Suponha que eu encontre:

```text
Modelo implantado
      ↓
Beacon de rede posterior
```

Isso é interessante.

Mas a proximidade temporal, sozinha, não é suficiente.

Ainda preciso de evidências conectando:

```text
Artefato
   ↓
Comportamento de Carregamento / Runtime
   ↓
Atividade de Rede
```

O princípio permanece:

> **Um caminho de ataque potencial não é o mesmo que um caminho de ataque confirmado.**

A resposta a incidentes deve estabelecer a cadeia causal antes de fazer atribuições fortes ou conclusões sobre a causa-raiz.

---

## Telemetria de Carregamento do Modelo

A telemetria em runtime pode revelar comportamentos que os metadados estáticos não revelam.

Uma sequência normal de carregamento de modelo poderia conceitualmente ser:

```text
Abrir Artefato
      ↓
Ler Dados Esperados
      ↓
Construir Modelo Esperado
      ↓
Concluir
```

Uma sequência suspeita poderia incluir itens inesperados, como:

```text
Imports
Acesso a Arquivos
Criação de Processos
Chamadas ao Shell
Atividade de Rede
Tipos de Objetos Inesperados
```

Isso não significa que todo evento incomum prove intenção maliciosa.

Mas cria evidências valiosas para a investigação.

---

## A Analogia com o Bootstrap

Durante os estudos, comecei a pensar no carregamento de modelos quase como no exame do processo de bootstrap de um sistema.

Antes de perguntar apenas:

> "O que o modelo responde?"

quero saber:

> "Como esse artefato entrou em execução?"

Conceitualmente:

```text
Aquisição do Artefato
       ↓
Verificação
       ↓
Carregador
       ↓
Reconstrução do Objeto
       ↓
Inicialização da Arquitetura
       ↓
Inferência
       ↓
Capacidades Externas
```

Entender essa cadeia ajuda a identificar onde um comportamento inesperado entrou no sistema.

---

## As Evidências Devem Sobreviver à Falha Preventiva

Isso leva a um princípio de segurança mais amplo:

> **Controles preventivos devem ser complementados por controles que produzam evidências.**

Se a prevenção falhar, ainda quero:

- logs de implantação;
- hashes dos artefatos;
- registros de proveniência;
- versões dos modelos;
- resultados de scanners;
- histórico de aprovações;
- telemetria em runtime;
- logs de rede;
- versões de prompts;
- metadados do provedor.

Sem isso, a resposta a incidentes vira adivinhação.

---

## A Aprovação em Produção é uma Decisão Baseada em Evidências

Os exercícios finais reforçaram outra ideia:

A aprovação em produção não deveria se basear em uma única propriedade atraente.

Não:

```text
Publicador famoso → Aprovar
```

Não:

```text
SafeTensors → Aprovar
```

Não:

```text
Scanner limpo → Aprovar
```

Não:

```text
Resposta correta → Aprovar
```

Em vez disso:

```text
Proveniência
+
Integridade
+
Segurança da Serialização
+
Análise Estática
+
Inspeção da Arquitetura
+
Segurança de Dependências
+
Avaliação Comportamental
+
Governança
+
Capacidade de Monitoramento
        ↓
Decisão de Produção
```

Isso se aproxima muito mais de uma engenharia de segurança baseada em risco.

---

## Sinais de Confiança versus Evidências de Segurança

Agora separo mentalmente essas duas coisas.

### Sinais de Confiança

Exemplos:

- organização conhecida;
- muitos downloads;
- perfil verificado;
- reputação na comunidade;
- documentação profissional.

Eles influenciam a confiança.

### Evidências de Segurança

Exemplos:

- proveniência;
- integridade criptográfica;
- análise estática;
- revisão da arquitetura;
- auditoria de dependências;
- SBOM;
- avaliação comportamental;
- telemetria de versões;
- registros controlados de aprovação.

Elas fundamentam uma decisão de produção defensável.

As duas categorias podem se complementar.

Mas:

> **Sinais de confiança nunca devem substituir evidências de segurança.**

---

## A Decisão de Produção

Imagine:

```text
MODELO A

Serialização segura ............ PASSOU
Integridade .................... PASSOU
Publicador conhecido ........... PASSOU
Scanner estático ............... PASSOU
Arquitetura .................... SUSPEITA
Comportamento .................. FALHOU
```

e:

```text
MODELO B

Serialização segura ............ PASSOU
Integridade .................... PASSOU
Reputação do publicador ........ LIMITADA
Scanner estático ............... PASSOU
Arquitetura .................... PASSOU
Comportamento .................. PASSOU
Proveniência ................... DOCUMENTADA
Dependências / SBOM ............ PASSOU
```

Eu preferiria o Modelo B com base nas evidências disponíveis.

A reputação menor do publicador cria uma necessidade de due diligence.

Ela não substitui as evidências técnicas.

Enquanto isso, o Modelo A apresenta descobertas de segurança reais.

Isso captura uma das minhas lições mais fortes:

> **A reputação pode influenciar minha confiança inicial, mas as evidências de segurança observadas devem influenciar minha decisão de produção.**

---

## Passar pelo Gate Não Significa "Comprovadamente Seguro"

Mesmo depois de todos os controles passarem, eu evitaria dizer:

```text
Este modelo é seguro.
```

Uma afirmação mais defensável seria:

```text
O modelo passou pelos controles de segurança definidos
e acumulou evidências suficientes para uma
implantação controlada em produção.
```

Essa diferença importa.

A avaliação de segurança reduz a incerteza.

Ela não elimina a incerteza.

---

## O Monitoramento Continua Depois da Aprovação

A aprovação em produção não é o fim do ciclo de vida da cadeia de suprimentos.

O sistema ainda pode mudar.

Por exemplo:

```text
Atualização do provedor
Atualização de dependência
Atualização de prompt
Atualização de adapter
Substituição do modelo
Alteração de configuração
Deriva comportamental
Comprometimento de credencial
```

Portanto:

```text
Adquirir
↓
Avaliar
↓
Aprovar
↓
Implantar
↓
Monitorar
↓
Reavaliar
```

é mais realista do que:

```text
Aprovar
↓
Confiar para Sempre
```

---

## Prevenção, Detecção e Investigação

Agora vejo os controles da cadeia de suprimentos de IA distribuídos em três grandes objetivos defensivos.

### Prevenção

Impedir que componentes não confiáveis entrem em produção.

Exemplos:

```text
Quarentena
Serialização segura
Registries privados
Fixação de versões
Lockfiles
Implantação controlada de prompts
Gates de aprovação
```

### Detecção

Identificar condições suspeitas ou inesperadas.

Exemplos:

```text
Varredura estática
Inspeção da arquitetura
Auditoria de dependências
Baselines comportamentais
Telemetria em runtime
Monitoramento de rede
```

### Investigação

Estabelecer o que realmente aconteceu.

Exemplos:

```text
Proveniência
Hashes
SBOM
Histórico de implantação
Telemetria de versões
Histórico de prompts
Logs de runtime
Logs de rede
```

Uma arquitetura de segurança madura precisa dos três.

---

## Meu Framework Atualizado de Aquisição de Modelos

Depois de estudar os controles técnicos e trabalhar com os cenários de investigação, meu próprio modelo mental passou a ser:

```text
                 CONFIANÇA EXTERNA
                       │
                       ▼
                  [ ADQUIRIR ]
                       │
                       ▼
                [ QUARENTENA ]
                       │
                       ▼
               [ PROVENIÊNCIA ]
                       │
                       ▼
                [ INTEGRIDADE ]
                       │
                       ▼
            [ FORMATO / SERIALIZAÇÃO ]
                       │
                       ▼
               [ SCANNER ESTÁTICO ]
                       │
                       ▼
              [ ARQUITETURA ]
                       │
                       ▼
              [ DEPENDÊNCIAS ]
                       │
                       ▼
                 [ SBOM ]
                       │
                       ▼
            [ TESTE COMPORTAMENTAL ]
                       │
                       ▼
               [ REVISÃO DE RISCO ]
                    /      \
                   /        \
              REJEITAR     APROVAR
                              │
                              ▼
                         [ REGISTRY ]
                              │
                              ▼
                        [ PRODUÇÃO ]
                              │
                              ▼
                         [ MONITORAR ]
                              │
                              ▼
                       [ REAVALIAR ]
```

Para modelos via API, alguns controles mudam:

```text
Modelo Local                Modelo via API
------------                --------------
Proveniência do arquivo     Due diligence do provedor
Checksum do arquivo         Telemetria de versões
Scanner estático do modelo  Avaliação comportamental
Inspeção da arquitetura     Transparência do provedor
Sandbox local               Avaliação da API em sandbox
Monitoramento do artefato   Monitoramento do comportamento
```

A implementação muda.

O princípio de segurança não:

> **Estabeleça evidências antes de conceder confiança.**

---

## Perspectiva de Frameworks

Esse tópico também reforçou por que a segurança de IA se beneficia da combinação de vários frameworks.

Frameworks diferentes ajudam a responder perguntas diferentes.

Conceitualmente:

```text
OWASP
   ↓
Onde estão os riscos comuns de aplicações de IA/LLM?

MITRE ATLAS
   ↓
Como adversários podem atacar sistemas de ML e suas cadeias de suprimentos?

NIST AI RMF
   ↓
Como o risco de IA deve ser governado, medido e gerenciado?
```

Nenhum framework substitui a análise específica da arquitetura.

Eles fornecem perspectivas complementares.

---

## Perguntas que Eu Faria Antes de Aprovar um Componente de IA

Depois deste módulo, estas são as perguntas que eu levaria para uma revisão de segurança real:

### Proveniência

Quem o produziu?

De onde ele veio?

Consigo estabelecer a cadeia entre o publicador e o artefato implantado?

### Integridade

Este é exatamente o artefato que eu esperava?

Posso verificá-lo criptograficamente?

### Serialização

Carregar este formato pode executar código?

Posso usar um formato mais seguro ou um carregador restrito?

### Arquitetura

O modelo contém lógica de execução personalizada ou inesperada?

### Pesos e Comportamento

Ele se comporta corretamente sob avaliações normais e adversariais?

Há anomalias específicas de gatilho?

### Dependências

De qual software ele precisa?

As versões são controladas?

As fontes são controladas?

Há vulnerabilidades conhecidas?

### Inventário

Tenho um SBOM?

Consigo identificar rapidamente os sistemas afetados quando surgir uma nova vulnerabilidade?

### Prompts

De onde vêm os prompts de produção?

Eles têm versão, são revisados, testados e aprovados?

### APIs

O que sei sobre o provedor?

O modelo pode mudar por trás do mesmo endpoint?

Mantenho baselines comportamentais e telemetria de versões?

### Operações

O que acontece depois da implantação?

Consigo detectar mudanças comportamentais?

Consigo reconstruir um incidente?

---

## O Que Mudou no Meu Raciocínio

Antes de estudar segurança da cadeia de suprimentos de IA, seria fácil pensar:

```text
Fonte Confiável
+
Hash Correspondente
+
Scanner Aprovado
=
Modelo Seguro
```

Já não vejo dessa forma.

Agora penso:

```text
Reputação da Fonte
        ↓
Confiança Inicial

Proveniência
Integridade
Análise Estática
Arquitetura
Dependências
Comportamento
Governança
Monitoramento
        ↓
Evidências Acumuladas
        ↓
Decisão de Risco
```

Esse é um modelo de confiança muito mais forte.

---

## A Progressão dos Três Dias

Os últimos três dias formam uma história de segurança contínua.

### Dia 13 — Entendendo as Cadeias de Suprimentos de IA

Aprendi a perguntar:

> **Em que exatamente estou confiando?**

O modelo é apenas um artefato dentro de uma cadeia muito maior.

### Dia 14 — Vetores de Ataque à Cadeia de Suprimentos

Aprendi a perguntar:

> **Como um atacante pode explorar essas relações de confiança?**

Atacantes podem visar artefatos, dependências, repositórios, identidades, provedores, prompts e os mecanismos usados para estabelecer confiança.

### Dia 15 — Protegendo a Cadeia de Suprimentos de IA

Agora pergunto:

> **Que evidências devem existir antes de eu permitir que essa confiança influencie a produção?**

Isso leva a discussão da conscientização para a engenharia.

---

## Minha Maior Conclusão

A lição mais forte do Dia 15 não é um scanner ou formato de arquivo específico.

É a ideia de **confiança baseada em evidências**.

SafeTensors é útil.

Hashes são úteis.

Scanners estáticos são úteis.

Inspeção da arquitetura é útil.

Auditoria de dependências é útil.

SBOMs são úteis.

Baselines comportamentais são úteis.

Due diligence do provedor é útil.

Mas cada um deles enxerga apenas uma parte do sistema.

O verdadeiro controle de segurança é o processo que os combina.

> **Passar por todas as verificações estáticas me fornece evidências para aumentar a confiança. Isso não me dá uma prova de que o modelo é confiável.**

E quando o modelo é entregue por uma API:

> **Se não posso verificar o artefato do modelo, preciso verificar o comportamento do modelo ao longo do tempo.**

Para resposta a incidentes:

> **Controles preventivos devem ser complementados por controles que produzam evidências.**

Para decisões de produção:

> **A reputação pode influenciar minha confiança inicial, mas as evidências de segurança observadas devem influenciar minha decisão de produção.**

E o princípio que conecta tudo é:

> **A confiança não deve ser herdada automaticamente. Ela deve ser estabelecida por meio de evidências independentes, limitada por fronteiras de segurança e continuamente reavaliada à medida que o sistema muda.**

---

## Principais Conclusões

- A segurança da cadeia de suprimentos de IA exige vários controles independentes.
- Novos artefatos de modelo devem começar em quarentena, não em produção.
- A serialização segura reduz o risco de execução de código no nível da serialização.
- O carregamento restrito pode reduzir capacidades perigosas de desserialização.
- Extensões de arquivo não são fronteiras de segurança.
- O SHA-256 verifica a integridade do artefato em relação a um hash esperado, não um comportamento benigno.
- Assinaturas digitais podem acrescentar a identidade do publicador à verificação de integridade.
- A proveniência estabelece de onde veio um artefato e como ele chegou à produção.
- Model cards fornecem evidências úteis de proveniência, mas não são certificados de segurança.
- Reputação é um sinal de confiança, não uma garantia de segurança.
- Adapters LoRA e modelos convertidos criam relações adicionais de confiança na cadeia de suprimentos.
- A varredura estática deve ocorrer antes do carregamento inseguro.
- Descobertas de scanners exigem contexto e análise.
- Uma varredura estática limpa não prova que um modelo está limpo.
- A serialização segura não valida a arquitetura do modelo.
- A inspeção da arquitetura ajuda a identificar lógica personalizada inesperada.
- Backdoors no nível dos pesos ou comportamentais podem sobreviver aos controles no nível do artefato.
- A avaliação comportamental é necessária antes da promoção para produção.
- Dependências fazem parte da cadeia de suprimentos de IA.
- Fixação de versões e lockfiles melhoram o determinismo das dependências.
- Pacotes internos exigem controle de fontes consciente da proveniência.
- Dependency confusion explora a confiança na resolução de dependências.
- A varredura de vulnerabilidades conhecidas não detecta todo pacote intencionalmente malicioso.
- SBOMs melhoram a visibilidade das dependências e a velocidade da resposta a incidentes.
- Dependências transitivas devem ser incluídas no inventário da cadeia de suprimentos.
- Sistemas de IA precisam de visibilidade além dos SBOMs tradicionais de software.
- APIs hospedadas transferem, mas não eliminam, o risco da cadeia de suprimentos.
- Due diligence do provedor se torna importante quando os artefatos do modelo são inacessíveis.
- Baselines comportamentais podem revelar mudanças upstream silenciosas.
- A telemetria da versão do modelo melhora a rastreabilidade forense.
- System prompts são artefatos da cadeia de suprimentos quando influenciam o comportamento em produção.
- Prompts relevantes para a segurança devem ter versão, ser revisados, testados e aprovados como código.
- A aprovação em produção deve se basear em evidências acumuladas, e não em reputação.
- Controles preventivos devem ser complementados por evidências de detecção e forenses.
- O momento da detecção não é necessariamente o momento do comprometimento.
- A correlação deve preceder as conclusões causais durante a resposta a incidentes.
- A aprovação em produção não é confiança permanente.
- O monitoramento e a reavaliação continuam depois da implantação.

---

## Reflexão Final

O Dia 13 mudou o objeto da minha atenção.

Parei de olhar apenas para o modelo final e comecei a olhar para a cadeia por trás dele.

O Dia 14 mudou o modelo de ameaça.

Vi como atacantes podem explorar essa cadeia, incluindo os mecanismos que as organizações usam para decidir o que merece confiança.

O Dia 15 mudou a pergunta defensiva.

O objetivo já não é:

> **"Posso provar que este modelo é seguro?"**

A pergunta melhor é:

> **"Que evidências independentes tenho, que incerteza permanece e quais controles ainda protegerão o sistema se uma das minhas premissas estiver errada?"**

Isso me parece muito mais próximo de como a cibersegurança já lida com outros sistemas complexos.

Não protegemos uma empresa porque um scanner retornou verde.

Combinamos controles.

Restringimos privilégios.

Preservamos evidências.

Monitoramos mudanças.

Investigamos anomalias.

E reavaliamos continuamente a confiança.

A segurança da cadeia de suprimentos de IA não deveria ser diferente.

---

## Referências

- [OWASP — LLM03:2025 Supply Chain](https://genai.owasp.org/llmrisk/llm032025-supply-chain/)
- [MITRE — ATLAS](https://atlas.mitre.org/)
- [NIST — AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [Hugging Face — Safetensors](https://huggingface.co/docs/safetensors/)
- [Trail of Bits — Fickling](https://github.com/trailofbits/fickling)
- [ModelScan](https://github.com/protectai/modelscan)
- [PyPA — pip-audit](https://github.com/pypa/pip-audit)
- [Anchore — Syft](https://github.com/anchore/syft)
- [OWASP — CycloneDX](https://cyclonedx.org/)
- [SPDX](https://spdx.dev/)
- [Google Research — Model Cards for Model Reporting](https://research.google/pubs/model-cards-for-model-reporting/)

---

> **Nota de aprendizado:** Este diário documenta meu próprio entendimento, raciocínio, correções e conexões de cibersegurança desenvolvidos durante o estudo da segurança de IA. Ele intencionalmente não reproduz soluções de desafios, flags, credenciais, infraestrutura de laboratório, cenários proprietários ou passo a passo.
