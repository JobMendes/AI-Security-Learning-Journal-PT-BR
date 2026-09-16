# Dia 14 — Vetores de Ataque na Supply Chain

<p align="center">
  <img src="../Pictures/Day14.png" alt="AI Security Learning Journal — Dia 14: Vetores de Ataque na Supply Chain" width="100%">
</p>

> Ataques de supply chain de IA exploram relações de confiança upstream da implantação, muitas vezes antes de o modelo chegar ao seu ambiente final.

## Em Resumo

**Tempo de leitura:** cerca de 22 minutos

Esta entrada examina como atacantes podem comprometer modelos, artefatos serializados, pacotes, repositórios, credenciais, datasets e caminhos de atualização.

**Principais aprendizados:**

- A serialização segura reduz um risco; ela não estabelece a confiança no artefato.
- O comprometimento de dependências e repositórios pode contornar controles focados no modelo.
- Proveniência, integridade, comportamento e monitoramento operacional respondem a perguntas diferentes.

**Caminho sugerido:** leia as seções sobre modelos serializados e dependências e, depois, conecte-as pelos caminhos de ataque de ponta a ponta.

**Navegação rápida:** [Superfícies de ataque](#as-principais-superfícies-de-ataque) · [Serialização maliciosa](#serialização-maliciosa-de-modelos) · [Ataques a dependências](#ataques-a-dependências) · [Modelo de ataque unificado](#um-modelo-de-ataque-unificado)

## AI Security Learning Journal

O Dia 13 mudou a forma como eu observo a confiança em sistemas de IA.

Parei de ver um modelo como um artefato isolado e comecei a observar a cadeia por trás dele:

```text
Modelos
Datasets
Frameworks
Dependências
Repositórios
Mantenedores
Transformações
Provedores
```

A pergunta passou a ser:

> **Em que exatamente estou depositando confiança?**

O Dia 14 avançou para a pergunta seguinte:

> **Como um atacante pode explorar essa confiança?**

Essa distinção é importante.

Um atacante nem sempre precisa atacar diretamente minha aplicação.

Às vezes, o caminho mais fácil é comprometer algo que minha aplicação já considera confiável.

Um artefato de modelo.

Uma dependência.

Um repositório.

Uma identidade de mantenedor.

Um template de prompt.

Uma credencial de API.

Ou até mesmo o comportamento por trás de um endpoint de API.

A mudança mais importante no meu modo de pensar foi esta:

> **Um ataque de supply chain nem sempre substitui aquilo em que confio. Às vezes, ele compromete o mecanismo que uso para decidir o que é confiável.**

---

## Do Mapeamento da Confiança à Exploração da Confiança

Meu modelo mental do Dia 13 era, em grande parte, sobre proveniência:

```text
Artefato
   ↓
De onde ele veio?
   ↓
Quem o criou?
   ↓
Quem o transformou?
   ↓
Como ele chegou até mim?
   ↓
Por que devo confiar nele?
```

O Dia 14 adiciona um atacante a esse grafo:

```text
                Atacante
                   │
                   ▼
             Relação de Confiança
                   │
                   ▼
                 Vítima
```

Isso cria várias possibilidades.

O atacante pode:

```text
Substituir um artefato

Fingir ser uma fonte confiável

Comprometer uma fonte realmente confiável

Explorar a resolução de dependências

Usar typosquatting

Manipular um repositório

Comprometer contas de mantenedores

Incorporar comportamento executável

Modificar o comportamento do modelo

Roubar credenciais de API

Alterar políticas obtidas externamente

Mudar o comportamento por trás de uma API
```

A superfície de ataque é, portanto, maior que o próprio modelo.

---

## As Principais Superfícies de Ataque

Agora separo a superfície de ataque da supply chain em dois grandes modelos de consumo.

### IA Baixada / Hospedada Localmente

```text
Repositório
    ↓
Artefato do Modelo
    ↓
Dependências
    ↓
Runtime
    ↓
Inferência
```

Entre os vetores de ataque importantes estão:

```text
Serialização Maliciosa
Manipulação no Nível da Arquitetura
Manipulação no Nível dos Pesos
Dependency Confusion
Typosquatting
Manipulação de Repositório
Contas de Mantenedores Comprometidas
```

### IA Hospedada / API

```text
Aplicação
    ↓
API do Provedor
    ↓
Infraestrutura do Provedor
    ↓
Modelo
```

A superfície de ataque do arquivo de modelo local se torna menos relevante para o consumidor, mas outros riscos ganham importância:

```text
Atualizações Silenciosas do Modelo
Comprometimento de Chave de API
Comprometimento de Template de Prompt
Risco de Treinamento Upstream
Mudanças de Comportamento
Mudanças na Supply Chain do Lado do Provedor
```

A supply chain continua existindo.

A superfície de ataque observável é que muda.

---

## Serialização Maliciosa de Modelos

Um dos vetores de ataque mais concretos neste tema envolve artefatos de modelos serializados.

Serialização significa conceitualmente:

```text
Objeto Python
      ↓
Serialização
      ↓
Artefato Armazenado
```

Mais tarde:

```text
Artefato Armazenado
      ↓
Desserialização
      ↓
Objeto Python Reconstruído
```

O problema de segurança é que alguns mecanismos de serialização são capazes de reconstruir muito mais do que valores passivos.

Eles podem reconstruir objetos invocando funções.

Isso muda o significado de segurança do carregamento de um artefato não confiável.

Em vez de pensar:

```text
Carregar Arquivo
   ↓
Ler Dados
```

preciso considerar:

```text
Carregar Arquivo
   ↓
Interpretar Instruções de Reconstrução
   ↓
Possível Invocação de Função
   ↓
Possível Efeito Colateral
```

Esse é um limite de confiança muito diferente.

---

## Pickle Não É Apenas Dado Passivo

O pickle do Python pode serializar objetos Python complexos.

Essa flexibilidade é útil.

Mas, do ponto de vista de segurança, flexibilidade também significa que a desserialização pode invocar o comportamento necessário para reconstruir esses objetos.

Isso leva a um princípio importante:

> **Um objeto serializado não confiável não deve ser automaticamente tratado como dado passivo.**

O perigo existe porque a aplicação pode acreditar que está fazendo isto:

```text
Arquivo do Modelo
    ↓
Pesos do Modelo
```

enquanto o desserializador pode, na realidade, processar algo conceitualmente mais próximo de:

```text
Instruções Serializadas
        ↓
Resolução de Função
        ↓
Argumentos
        ↓
Invocação de Função
```

O limite de segurança é atravessado durante o carregamento.

---

## Entendendo `__reduce__()`

Um conceito que precisei refinar foi o mecanismo `__reduce__()` do Python.

Ele não é simplesmente "a função que executa malware".

Sua finalidade é informar ao pickle como um objeto deve ser reconstruído.

Conceitualmente:

```text
Objeto
   ↓
__reduce__()
   ↓
Receita de Reconstrução
   ↓
Callable + Argumentos
```

Durante a desserialização, o pickle pode seguir essa receita.

O perigo surge quando a receita de reconstrução referencia uma função capaz de causar um efeito colateral inseguro.

Portanto, a relação importante é:

```text
Objeto Não Confiável
      ↓
Instruções de Reconstrução
      ↓
Callable Perigoso
      ↓
Argumentos Controlados pelo Atacante
      ↓
Execução Durante a Desserialização
```

O atacante abusa de uma funcionalidade legítima de reconstrução.

---

## Execução no Carregamento

Isso dá aos ataques de serialização uma característica importante:

> **O ataque pode ser executado quando o modelo é carregado, antes do início da inferência normal.**

A aplicação pode nunca chegar a:

```text
prediction = model(input)
```

antes que o comprometimento já tenha ocorrido.

O caminho de ataque pode ser, em vez disso:

```text
Baixar Modelo
      ↓
Carregar Modelo
      ↓
Desserializar
      ↓
Reconstrução Maliciosa
      ↓
Execução de Código
```

Isso torna perigosa a estratégia de validação "vou carregá-lo e testar se ele se comporta corretamente".

O próprio ato de carregar pode ser o ataque.

---

## Não Execute um Artefato para Descobrir se É Seguro Executá-lo

Essa foi uma das minhas conclusões defensivas mais fortes sobre o tema.

Se a preocupação é que a desserialização pode executar código, testar o artefato desserializando-o derrota o propósito da investigação.

O princípio mais seguro é:

> **Inspecione primeiro. Execute depois, se houver justificativa.**

Conceitualmente:

```text
Artefato Desconhecido
      ↓
Inspeção Estática / Sem Execução
      ↓
Avaliação de Segurança
      ↓
Decisão Controlada
```

em vez de:

```text
Artefato Desconhecido
      ↓
Executar
      ↓
"Vamos ver o que acontece"
```

Isso parece óbvio quando escrito.

Mas os fluxos de trabalho de IA podem normalizar o carregamento de modelos baixados como se fossem arquivos de dados comuns.

É exatamente essa suposição que precisa mudar.

---

## Inspecionando a Serialização sem Reconstruir Objetos

Uma abordagem de análise mais segura é inspecionar a representação serializada sem realmente reconstruir os objetos contidos nela.

No caso do pickle, isso significa examinar suas operações em vez de executar o artefato.

Conceitualmente:

```text
Artefato Pickle
      ↓
Desmontagem / Parsing
      ↓
Operações de Serialização
      ↓
Análise de Segurança
```

Isso pode expor relações suspeitas como:

```text
Resolução de Função
       +
Função Perigosa
       +
Argumento Suspeito
       +
Operação de Execução
```

sem carregar intencionalmente o modelo na aplicação.

A distinção é fundamental:

```text
Inspeção
    ≠
Desserialização
```

---

## O Contexto Importa Mais que um Único Opcode

Outra lição importante foi não transformar a análise de segurança em uma correspondência simplista de palavras-chave.

Uma operação usada pelo mecanismo de serialização pode aparecer em artefatos legítimos.

Portanto:

```text
Um Opcode
    ≠
Malware
```

O que importa é o contexto ao redor dele.

Por exemplo, um analista deve perguntar:

```text
Que função está sendo resolvida?

Que argumentos são fornecidos?

Que operação de reconstrução vem em seguida?

Isso faz sentido para um artefato de ML?

Isso cria comportamento de filesystem, processo ou rede?
```

Isso produz um princípio mais útil:

> **A análise de segurança precisa de contexto, não apenas de correspondência de opcode.**

Uma cadeia suspeita importa mais que um token isolado.

---

## Ataques no Nível da Serialização vs. no Nível da Arquitetura

O Dia 13 introduziu três categorias de ataque no nível do modelo:

```text
Serialização
Arquitetura
Pesos
```

O Dia 14 tornou muito mais clara a diferença de timing.

### Nível da Serialização

```text
Artefato do Modelo
      ↓
Carregamento
      ↓
Desserialização
      ↓
Comportamento Malicioso
```

O ataque pode ser disparado no **momento do carregamento**.

### Nível da Arquitetura

```text
Artefato do Modelo
      ↓
Carregamento
      ↓
Modelo Existe
      ↓
Inferência
      ↓
Lógica Maliciosa da Arquitetura
```

O comportamento malicioso pode se manifestar no **momento da inferência**.

Essa distinção importa porque um controle criado para um deles não necessariamente detecta o outro.

---

## A Lógica Personalizada do Modelo Pode se Tornar uma Superfície de Ataque

Frameworks modernos de ML podem permitir lógica de processamento personalizada dentro das arquiteturas de modelo.

Essa capacidade tem finalidades legítimas.

Mas a extensibilidade legítima também pode se tornar uma superfície de ataque.

Conceitualmente:

```text
Entrada
   ↓
Camadas Normais
   ↓
Lógica Personalizada
   ↓
Condição
  ↙   ↘
Saída   Saída
Normal  Manipulada
```

O artefato pode ser carregado com sucesso.

Nada necessariamente acontece durante a desserialização.

O comportamento malicioso aparece somente quando o modelo processa entradas específicas.

Isso torna os ataques no nível da arquitetura diferentes de payloads maliciosos de pickle.

---

## A Serialização Segura Não Valida a Arquitetura

Isso cria outro limite importante.

Suponha que eu remova a serialização insegura.

Posso obter:

```text
Ataque de Serialização
       ↓
Mitigado
```

Mas não obtive automaticamente:

```text
Arquitetura Verificada
Pesos Verificados
Treinamento Verificado
Proveniência Verificada
```

Então:

> **A serialização segura é um controle contra uma classe de ataques, não uma prova de segurança do modelo.**

Isso reforça a lição do Dia 13:

> **Um formato de arquivo mais seguro remove um vetor de ataque, não todo o risco da supply chain.**

---

## Backdoors no Nível dos Pesos

A terceira categoria é ainda mais sutil.

Em vez de depender de serialização executável ou de lógica explícita na arquitetura, o comportamento malicioso pode ser aprendido ou incorporado aos pesos do modelo.

Conceitualmente:

```text
Entrada Normal
     ↓
Comportamento Esperado

Gatilho Específico
     ↓
Comportamento Inesperado
```

O modelo pode continuar parecendo legítimo em testes comuns.

Isso cria um problema de detecção diferente.

Uma varredura estática da estrutura do arquivo pode não encontrar nada obviamente executável.

Ainda assim, o comportamento do modelo pode conter uma condição oculta.

---

## GGUF Não Significa Confiável

Isso é especialmente relevante para LLMs hospedados localmente.

Um formato que evita a desserialização arbitrária no estilo pickle remove uma superfície de ataque importante.

Mas:

```text
Sem Pickle
   ≠
Sem Backdoor
```

Um atacante poderia manipular um modelo antes de ele ser convertido ou quantizado.

Conceitualmente:

```text
Modelo Base
    ↓
Fine-Tuning Malicioso
    ↓
Pesos Manipulados
    ↓
Conversão / Quantização
    ↓
Artefato de Modelo Local
```

O formato final pode ser estruturalmente seguro contra execução no estilo pickle, enquanto o comportamento do modelo continua não confiável.

Isso me traz outra distinção:

> **A segurança do formato de arquivo e a confiança no comportamento são propriedades de segurança diferentes.**

---

## Integridade do Artefato vs. Integridade do Comportamento

Essa distinção se tornou uma das ideias mais importantes que tirei do Dia 14.

### Integridade do Artefato

A integridade do artefato pergunta:

> **Este é exatamente o artefato que eu esperava?**

Por exemplo:

```text
Artefato Esperado
      ↓
Hash Criptográfico

Artefato Baixado
      ↓
Hash Criptográfico

Correspondem?
```

Se correspondem, tenho evidências fortes de que o artefato baixado é idêntico ao artefato esperado.

Mas e se o próprio artefato esperado contiver um comportamento aprendido indesejado?

Então:

```text
Integridade do Artefato ✓
Integridade do Comportamento ✗
```

### Integridade do Comportamento

A integridade do comportamento pergunta:

> **O modelo continua se comportando dentro das propriedades e dos limites que espero?**

Isso pode exigir:

```text
Baselines de Comportamento
Avaliação de Segurança
Conjuntos de Testes Conhecidos
Testes Adversariais
Testes de Regressão
Monitoramento
```

As duas propriedades se complementam.

Nenhuma substitui a outra.

---

## Ataques a Dependências

O artefato do modelo é apenas um ponto de entrada.

Os sistemas de IA ainda dependem de pacotes de software.

Isso significa que atacantes podem mirar o grafo de dependências.

Dois vetores de ataque importantes são:

```text
Dependency Confusion
Typosquatting
```

Eles podem parecer semelhantes porque ambos envolvem nomes de pacotes.

Mas exploram diferentes suposições de confiança.

---

## Dependency Confusion

Considere uma dependência interna:

```text
internal-ai-utils
```

A organização espera:

```text
Registro Privado
      ↓
internal-ai-utils
```

Mas o gerenciador de pacotes também pode consultar um registro público.

Agora suponha que o mesmo nome de pacote apareça publicamente com uma versão preferida pelo resolvedor.

Conceitualmente:

```text
Registro Privado
internal-ai-utils 2.x
        │
        ├────────────┐
                     ▼
                  Resolvedor
                     ▲
        ┌────────────┘
        │
Registro Público
internal-ai-utils 99.x
```

Se o resolvedor selecionar o pacote público, o atacante explorou uma lacuna entre:

```text
Resolução de Versão
```

e:

```text
Confiança na Fonte
```

Isso me dá um princípio importante:

> **Precedência de versão não é aplicação de proveniência.**

O gerenciador de pacotes pode implementar corretamente sua lógica de seleção de versões e ainda produzir o resultado de segurança errado.

---

## Typosquatting

Typosquatting explora uma fraqueza diferente.

Em vez de publicar o mesmo nome de pacote interno, um atacante cria um nome parecido com o de uma dependência legítima.

Conceitualmente:

```text
Esperado:
trusted-package

Atacante:
trustd-package
```

A falha de segurança depende muito do reconhecimento humano.

O desenvolvedor vê algo familiar.

O pacote parece plausível.

O artefato errado é selecionado.

Então:

```text
Dependency Confusion
→ confiança na resolução/fonte

Typosquatting
→ confiança na semelhança do nome/reconhecimento humano
```

Manter esses mecanismos separados me ajuda a raciocinar sobre os controles adequados.

---

## Malicioso Não É o Mesmo que Vulnerável

Essa foi outra correção útil no meu modo de pensar.

Ferramentas de segurança de dependências frequentemente identificam vulnerabilidades conhecidas comparando:

```text
Pacote
   +
Versão
   ↓
Advisory / Banco de Dados de Vulnerabilidades Conhecidas
```

Isso é valioso.

Mas imagine que um atacante crie um pacote novo cuja funcionalidade pretendida seja maliciosa.

Pode haver:

```text
Nenhum CVE
Nenhuma vulnerabilidade conhecida
Nenhum advisory histórico
```

porque o software não é acidentalmente vulnerável.

Ele é intencionalmente malicioso.

Portanto:

> **Malicioso não significa necessariamente vulnerável.**

A segurança da supply chain não pode depender exclusivamente de bancos de dados de vulnerabilidades.

Ela também exige proveniência, identidade do pacote, validação da fonte e análise comportamental.

---

## Manipulação de Repositórios

Repositórios são outra parte crítica da cadeia de confiança.

Ao escolher modelos, desenvolvedores frequentemente dependem de sinais como:

```text
Nome da Organização
Verificação
Contagem de Downloads
Documentação
Model Card
Atividade da Comunidade
Histórico de Uploads
Formato do Arquivo
```

Atacantes podem tentar manipular esses sinais.

Uma abordagem é a imitação.

Outra é comprometer a própria fonte legítima.

Esses são níveis de ameaça muito diferentes.

---

## Repositório Falso vs. Repositório Real Comprometido

Considere:

```text
Organização Falsa
      ↓
Repositório com Aparência Profissional
      ↓
Artefato Malicioso
```

Ainda pode haver sinais de alerta:

```text
Sem verificação
Pouco histórico
Baixa adoção
Conta criada recentemente
Proveniência escassa
```

Agora compare:

```text
Organização Legítima
       ↓
Comprometimento da Credencial do Mantenedor
       ↓
Repositório Legítimo
       ↓
Atualização Maliciosa
```

Isso é muito mais difícil de detectar usando apenas reputação.

O repositório ainda pode ter:

```text
✓ Nome correto
✓ Identidade verificada
✓ Histórico longo
✓ Grande adoção
✓ Confiança já estabelecida
```

O atacante comprometeu o mecanismo que produz o sinal de confiança.

Isso leva a uma das minhas principais lições do Dia 14:

> **Um ataque de supply chain nem sempre imita a confiança. Às vezes, ele sequestra uma confiança real.**

---

## Sinais de Confiança São Evidências, Não Garantias

A distinção anterior reforça algo do Dia 13.

Sinais como:

```text
Publicador Verificado
Grande Contagem de Downloads
Histórico Longo
Documentação Detalhada
Varreduras de Segurança
```

são úteis.

Eles devem contribuir, sem dúvida, para uma decisão de segurança.

Mas:

```text
Sinal de Confiança
    ≠
Prova de Integridade
```

Uma identidade confiável pode ser comprometida.

Um pipeline de build confiável pode ser comprometido.

Um repositório confiável pode distribuir um artefato malicioso.

O objetivo não é ignorar a reputação.

É entender o que a reputação pode e não pode provar.

---

## Atacantes Podem Combinar Vetores de Supply Chain

Uma das lições mais importantes do Dia 14 foi que esses vetores não devem ser imaginados como mutuamente exclusivos.

Um atacante não precisa escolher:

```text
Modelo Malicioso
OU
Ataque a Dependência
OU
Manipulação de Repositório
```

Ele pode usar:

```text
Manipulação de Repositório
        +
Modelo Malicioso
        +
Ataque a Dependência
```

Cada um serve a uma finalidade diferente.

Conceitualmente:

```text
Manipulação de Repositório
        ↓
Construir Confiança

Artefato Malicioso
        ↓
Caminho Primário de Execução

Dependência Maliciosa
        ↓
Caminho Secundário de Execução
```

Se um caminho falhar, outro pode continuar disponível.

---

## Redundância Também Funciona para Atacantes

Na arquitetura de segurança, geralmente associo redundância à resiliência.

Mas atacantes também podem projetar para obter resiliência.

```text
Vetor de Ataque A
      ↓
Bloqueado
      │
      └────→ Vetor de Ataque B
                    ↓
                  Sucesso
```

Isso significa que interromper um artefato malicioso não prova que o incidente está contido.

Um analista precisa perguntar:

```text
Que outros componentes chegaram com ele?

Que dependências foram instaladas?

Em quais repositórios foi depositada confiança?

Que credenciais foram expostas?

Que comunicação externa ocorreu?

Que outros caminhos de persistência existem?
```

O primeiro vetor visível do atacante pode não ser seu único vetor.

---

## Defesa em Profundidade Precisa Cobrir Superfícies Diferentes

Suponha que um model loader bloqueie com sucesso a desserialização insegura.

Esse controle funcionou.

Mas, se uma dependência maliciosa executar durante a instalação, o ambiente ainda pode ser comprometido.

```text
Model Loader
    ↓
Ataque Bloqueado ✓

Instalação de Dependência
    ↓
Ataque Bem-Sucedido ✗
```

A conclusão correta não é:

```text
O model loader falhou
```

É:

```text
O model loader protegeu seu limite

MAS

a arquitetura geral tinha outro caminho desprotegido
```

É isso que defesa em profundidade significa na prática.

Os controles precisam cobrir superfícies de ataque independentes.

---

## Reconstrução de Incidentes de Supply Chain

O Dia 14 também reforçou uma mentalidade de resposta a incidentes.

Imagine que eu observe:

```text
Repositório de Modelo Desconhecido
        ↓
Artefato de Modelo Suspeito
        ↓
Dependência Inesperada
        ↓
Conexão de Rede de Saída
```

Não devo parar imediatamente em:

> "O modelo era malicioso."

Preciso reconstruir a cadeia.

```text
Fonte Inicial
      ↓
Aquisição do Artefato
      ↓
Instalação da Dependência
      ↓
Execução
      ↓
Atividade de Rede
      ↓
Persistência
      ↓
Acesso Adicional
```

É aqui que a segurança da supply chain se torna DFIR.

A pergunta muda de:

> Que arquivo suspeito encontrei?

para:

> **Como a confiança em um componente upstream se tornou execução e impacto downstream?**

---

## Correlação Antes da Causalidade

Agora imagine que eu descubra simultaneamente:

```text
Modelo Suspeito
      ↓
Tráfego de Saída
```

e:

```text
Template de Prompt Externo
      ↓
Mudança Inesperada de Política
      ↓
Mudança no Comportamento do Agente
```

Há pelo menos duas hipóteses:

```text
Hipótese A
Uma campanha coordenada de supply chain

Hipótese B
Dois incidentes não relacionados
```

Não devo assumir nenhuma delas imediatamente.

Eu correlacionaria:

```text
Linha do Tempo
Contas
Credenciais
Repositórios
Histórico de Deploy
Indicadores de Rede
Infraestrutura
Mudanças de Dependências
Mudanças de Templates
Logs do Provedor
```

Somente então posso estabelecer se existe uma relação causal.

Isso preserva um princípio das etapas anteriores da minha jornada:

> **Caminho de ataque potencial não equivale a caminho de ataque confirmado.**

---

## A Supply Chain de API É Diferente

A segunda metade deste tema mudou significativamente a superfície de ataque.

Quando uso:

```text
Aplicação
    ↓
API de IA Hospedada
```

não recebo os pesos do modelo.

Não desserializo um artefato de modelo local.

Portanto:

```text
Pickle
SafeTensors
GGUF
```

deixam de ser a principal preocupação direta da minha aplicação.

Mas a supply chain não desapareceu.

Em vez disso, passo a depender mais do provedor e dos artefatos ao redor da integração com a API.

---

## Atualizações Silenciosas do Modelo

Um risco particularmente interessante da supply chain de API é uma mudança de modelo do lado do provedor.

Imagine que minha aplicação continue chamando:

```text
model = "company-model"
```

Ontem:

```text
company-model
      ↓
Versão A do Modelo
```

Hoje:

```text
company-model
      ↓
Versão B do Modelo
```

Meu código da aplicação não mudou.

O endpoint da API não mudou.

O identificador do modelo pode não aparentar ter mudado.

Mas o comportamento pode mudar.

Esse é um problema de **atualização silenciosa do modelo**.

---

## Nenhum Commit Não Significa Nenhuma Mudança

Isso traz consequências importantes para a investigação de incidentes.

Imagine:

```text
Histórico do Git
    ↓
Nenhuma Mudança na Aplicação
```

mas:

```text
Comportamento em Produção
    ↓
Mudou
```

Na resolução de problemas tradicional, eu poderia me concentrar imediatamente em mudanças de infraestrutura, dados ou ambiente.

Com IA hospedada externamente, também preciso perguntar:

> **O modelo por trás do serviço mudou?**

Essa é outra forma de dependência upstream.

---

## Telemetria de Versão Cria Evidências

Quando houver suporte, registrar a versão real do modelo usada em cada decisão importante pode melhorar a rastreabilidade forense.

Conceitualmente:

```text
Requisição
   ↓
Versão do Modelo
   ↓
Resposta
   ↓
Decisão
```

Então uma investigação pode correlacionar:

```text
Versão A
→ comportamento esperado

Versão B
→ início do comportamento inesperado
```

Sem informações de versão, ambas podem aparecer apenas como:

```text
company-model
```

Isso me dá outro princípio útil:

> **A telemetria de versão pode transformar uma mudança invisível do lado do provedor em evidência investigável.**

---

## Chaves de API São Credenciais da Supply Chain

A IA hospedada também introduz risco de credenciais.

Uma chave de API pode ser exposta por meio de:

```text
Código-Fonte
Logs de CI/CD
Arquivos de Ambiente
Segredos Configurados Incorretamente
Estações de Trabalho de Desenvolvedores
```

Se comprometida, um atacante pode conseguir:

```text
Fingir ser a aplicação
Consumir recursos do provedor
Gerar custos inesperados
Acessar recursos de API permitidos
Abusar dos fluxos de trabalho da aplicação
```

dependendo das permissões e da arquitetura.

Isso é tanto segurança de credenciais quanto segurança da supply chain de IA.

A credencial faz parte da relação de confiança entre:

```text
Aplicação
    ↔
Provedor
```

---

## Templates de Prompt São Artefatos da Supply Chain

Esta foi provavelmente a conexão mais interessante entre o módulo de Segurança de Prompts e a Segurança da Supply Chain de IA.

Considere:

```text
Aplicação
      ↓
Repositório Externo de Templates de Prompt
      ↓
System Prompt
      ↓
LLM
```

À primeira vista, o template é "apenas texto".

Mas esse texto pode controlar:

```text
Papel
Comportamento
Política
Escalonamento
Critérios de Decisão
Uso de Ferramentas
Expectativas de Saída
```

Se a aplicação consumir automaticamente esse texto de uma fonte externa, o template influencia o comportamento em produção.

Portanto:

> **Um template de prompt pode ser um artefato da supply chain.**

O artefato não precisa ser código executável.

Ele só precisa influenciar um sistema confiável de uma maneira relevante para a segurança.

---

## Instruções Confiáveis Podem Ter uma Origem Não Confiável

Isso cria uma conexão fascinante com Prompt Injection.

Antes, aprendi:

> **Dados não confiáveis podem se tornar instruções.**

Agora a segurança da supply chain acrescenta:

> **As próprias instruções confiáveis podem chegar por uma supply chain não confiável ou comprometida.**

Suponha que a política de produção originalmente exija:

```text
Mudança Crítica de Segurança
        ↓
Revisão Humana
```

Um template obtido externamente muda para:

```text
Mudança Crítica de Segurança
        ↓
Aprovação Automática
```

Nada mais necessariamente mudou:

```text
Mesmo Modelo
Mesma Aplicação
Mesmas Dependências
Mesma Infraestrutura
```

Ainda assim, o comportamento de segurança do sistema mudou porque seu **artefato de política mudou**.

---

## Trate Prompts como Código

Se um prompt ou uma política controla o comportamento em produção, não devo automaticamente baixar sua versão mais recente diretamente para produção.

Um ciclo de vida mais seguro se parece com:

```text
Template Externo
      ↓
Revisão
      ↓
Controle de Versão Interno
      ↓
Testes de Segurança
      ↓
Aprovação
      ↓
Produção
```

em vez de:

```text
Template Externo
      ↓
Atualização Automática
      ↓
Produção
```

Isso se conecta diretamente com a Defesa contra Prompt Injection:

> **Fontes externas podem propor mudanças. O sistema deve decidir o que se torna estado confiável de produção.**

---

## O Comportamento Pode Ser o Comprometimento

O comprometimento da supply chain nem sempre precisa resultar em:

```text
Reverse Shell
Malware
Roubo de Credenciais
Execução Remota de Código
```

Imagine um modelo de IA usado para classificar findings de segurança em CI/CD.

```text
Mudança de Código
    ↓
Revisão de Segurança por IA
    ↓
Classificação
    ↓
Decisão de CI/CD
```

Agora uma atualização upstream do modelo muda o classificador.

```text
Código Vulnerável
      ↓
IA diz SEGURO
      ↓
CI/CD aceita
      ↓
Produção
```

Nenhum shell foi aberto.

Nenhum malware foi instalado.

Mas a integridade de uma decisão de segurança foi comprometida.

O impacto downstream pode incluir:

```text
Vulnerabilidades chegando à produção
Exposição de dados
Fraude
Comprometimento do serviço
Indisponibilidade
Custos de resposta a incidentes
Impacto regulatório
Perda da confiança dos clientes
```

Isso me dá outro princípio importante:

> **O comprometimento da supply chain pode atingir a integridade das decisões, não apenas a execução de código.**

---

## O Monitoramento de Comportamento Importa Mais com APIs

Quando controlo um artefato, posso inspecioná-lo e calcular seu hash.

Com IA hospedada, grande parte do artefato subjacente pode estar inacessível.

Isso aumenta a importância dos controles comportamentais:

```text
Conjunto Fixo de Avaliação
      ↓
Baseline de Comportamento
      ↓
Reavaliação Periódica
      ↓
Detecção de Drift / Regressão
```

Isso não prova que o provedor não foi comprometido.

Mas fornece ao consumidor evidências quando o comportamento muda.

---

## Model Drift vs. Mudança na Supply Chain

Comportamento inesperado ainda deve ser investigado cuidadosamente.

Nem toda mudança significa um ataque.

As causas potenciais incluem:

```text
Mudança na Distribuição das Entradas
Model Drift
Atualização do Provedor
Mudança de Prompt
Mudança na Recuperação
Mudança de Configuração
Comprometimento da Supply Chain
```

O analista deve evitar saltar diretamente de:

```text
Comportamento Mudou
```

para:

```text
Atacante Comprometeu o Provedor
```

As evidências continuam importantes.

O monitoramento encontra a mudança.

A investigação determina a causa.

---

## Um Modelo de Ataque Unificado

Depois dos Dias 13 e 14, agora imagino os ataques de supply chain de IA assim:

```text
                         ATACANTE
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
        ▼                   ▼                    ▼
   Manipulação de      Manipulação de      Manipulação de
   Repositório         Dependência         Provedor/API
        │                   │                    │
        ▼                   ▼                    ▼
   Modelo Malicioso    Pacote Malicioso    Mudança de Comportamento
        │                   │                    │
        ├──── Serialização  │                    ├── Atualização Silenciosa
        ├──── Arquitetura   │                    ├── Comprometimento de Chave
        └──── Pesos         │                    └── Mudança de Template
        │                   │                    │
        └───────────────────┼────────────────────┘
                            ▼
                       SISTEMA CONFIÁVEL
                            │
                            ▼
                         IMPACTO
```

Vetores diferentes.

Limites de confiança diferentes.

Potencialmente, o mesmo resultado de negócio.

---

## Minhas Perguntas para Resposta a Incidentes

Se suspeito de um incidente de supply chain de IA, agora quero perguntar:

```text
Que artefato mudou?

Que dependência mudou?

Que repositório o forneceu?

Quem tinha direitos de publicação?

As credenciais do mantenedor mudaram ou vazaram?

O modelo foi transformado?

A versão do modelo mudou?

Um template de prompt ou política mudou?

Uma credencial de API vazou?

Quando o comportamento mudou?

Que atividade de rede ocorreu depois?

Que outros caminhos de ataque ainda podem existir?
```

E, principalmente:

> **Se eu bloquear o primeiro vetor que encontrar, que outro vetor o atacante pode já ter preparado?**

---

## Meu Modelo de Confiança Atualizado

Dia 13:

```text
Em que estou depositando confiança?
```

Dia 14:

```text
Como essa confiança pode ser explorada?
```

O modelo combinado fica assim:

```text
Componente Externo
       ↓
Decisão de Confiança
       ↓
Integração Confiável
       ↓
Possível Comprometimento Upstream
       ↓
Caminho de Ataque
       ↓
Comportamento / Execução do Sistema
       ↓
Impacto no Negócio
```

Isso torna a segurança da supply chain fundamentalmente mais ampla do que apenas escanear arquivos.

Trata-se de entender e proteger **relações de confiança**.

---

## Minha Maior Conclusão

Minha maior conclusão do Dia 14 é que atacantes podem explorar a confiança de muitas maneiras diferentes.

Eles podem explorar a forma como um artefato é carregado.

Podem manipular a arquitetura ou os pesos do modelo.

Podem explorar a resolução de dependências.

Podem imitar nomes confiáveis de pacotes ou modelos.

Podem comprometer repositórios legítimos.

Podem roubar credenciais.

Podem alterar templates de prompt externos.

Podem explorar mudanças do lado do provedor que os consumidores não conseguem inspecionar diretamente.

E podem combinar esses vetores para que bloquear um caminho não necessariamente interrompa o ataque.

A progressão do Dia 13 para o Dia 14 agora me parece muito clara:

> **No Dia 13 aprendi que preciso entender em que confio. No Dia 14 aprendi como atacantes exploram essa confiança em artefatos, dependências, repositórios e provedores — frequentemente combinando vários vetores de ataque para que a falha de um caminho não interrompa o comprometimento.**

E o princípio que quero levar adiante é:

> **Um ataque de supply chain nem sempre substitui aquilo em que confio. Às vezes, ele compromete o mecanismo que uso para decidir o que é confiável.**

---

## Principais Aprendizados

- Arquivos de modelos serializados não confiáveis não devem ser automaticamente tratados como dados passivos.
- A desserialização pode se tornar um limite de execução.
- `__reduce__()` descreve a reconstrução de objetos e pode ser abusado por meio de callables perigosos e argumentos controlados pelo atacante.
- Sempre que possível, a serialização não confiável deve ser inspecionada sem reconstruir os objetos.
- A análise de segurança deve avaliar o contexto em vez de sinalizar operações de serialização isoladas.
- Ataques no nível da serialização podem ser executados no momento do carregamento.
- O comportamento malicioso no nível da arquitetura pode se manifestar durante a inferência.
- Backdoors no nível dos pesos podem existir sem código executável evidente.
- A serialização segura mitiga uma classe de ataques, mas não prova a segurança do modelo.
- O fato de GGUF evitar a execução no estilo pickle não estabelece confiança no comportamento.
- Integridade do artefato e integridade do comportamento são propriedades de segurança diferentes.
- Dependency confusion explora a resolução de pacotes e suposições de confiança na fonte.
- Typosquatting explora a semelhança de nomes e o reconhecimento humano.
- Precedência de versão não é aplicação de proveniência.
- Pacotes maliciosos não precisam de vulnerabilidades conhecidas ou CVEs.
- A reputação do repositório é uma evidência útil, mas não detecta toda fonte legítima comprometida.
- Atacantes podem sequestrar uma confiança genuína, em vez de apenas imitá-la.
- Vários vetores de supply chain podem fornecer aos atacantes caminhos de entrada redundantes.
- A defesa em profundidade precisa cobrir modelos, dependências, repositórios, runtime e relações com provedores.
- O consumo por API remove alguns riscos de artefatos locais, mas cria outras preocupações de supply chain.
- Atualizações silenciosas do modelo podem mudar o comportamento em produção sem commits na aplicação.
- A telemetria da versão do modelo melhora a rastreabilidade forense.
- Credenciais de API fazem parte do limite de confiança do provedor.
- Templates de prompt podem ser artefatos de supply chain relevantes para a segurança.
- Prompts e políticas relevantes para a segurança devem ser revisados, versionados e testados antes do uso em produção.
- O comprometimento da supply chain pode atingir a integridade das decisões sem exigir execução remota de código.
- O monitoramento de comportamento se torna especialmente importante quando os consumidores não conseguem inspecionar artefatos do lado do provedor.
- Comportamento inesperado é evidência para investigação, não prova automática de comprometimento.
- Caminhos de ataque potenciais precisam ser correlacionados antes que se afirme causalidade.

---

## Reflexão Final

O Dia 13 me ensinou a olhar upstream.

O Dia 14 me ensinou a pensar como um atacante que se move por essa confiança upstream.

A parte perigosa de um ataque de supply chain nem sempre é uma exploração sofisticada.

Às vezes, tudo downstream funciona exatamente como foi projetado.

O gerenciador de pacotes seleciona a versão que acredita dever selecionar.

O model loader reconstrói o objeto que recebeu instruções para reconstruir.

A aplicação carrega a política que foi configurada para carregar.

O cliente da API envia requisições ao provedor em que foi configurado para confiar.

A falha aconteceu antes:

> **O sistema confiou na coisa errada, ou aquilo em que confiou corretamente foi comprometido.**

Essa distinção muda a forma como penso sobre segurança de IA.

O próximo passo, portanto, não é simplesmente aprender outra técnica de ataque.

É aprender a estabelecer uma proveniência mais forte, restringir dependências, inspecionar artefatos, controlar mudanças externas, monitorar o comportamento e projetar o sistema para que uma relação de confiança comprometida não se transforme em comprometimento total.

Isso leva diretamente a:

**Protegendo a Supply Chain de IA.**

---

## Referências

- [OWASP — LLM03:2025 Supply Chain](https://genai.owasp.org/llmrisk/llm032025-supply-chain/)
- [MITRE — ATLAS](https://atlas.mitre.org/)
- [Python — aviso de segurança do `pickle`](https://docs.python.org/3/library/pickle.html)
- [PyTorch — Política de Segurança](https://github.com/pytorch/pytorch/security/policy)
- [Hugging Face — segurança do Hub](https://huggingface.co/docs/hub/security)
- [Hugging Face — Safetensors](https://huggingface.co/docs/safetensors/)
- [PyPA — instalações seguras](https://pip.pypa.io/en/stable/topics/secure-installs/)

---

*Este repositório documenta minha jornada pessoal de aprendizado em Segurança de IA. Ele contém minhas próprias explicações, reflexões, raciocínios de segurança e anotações estruturadas de forma independente com base em conceitos estudados em várias fontes educacionais e do setor, incluindo a trilha de aprendizado de Segurança de IA do TryHackMe. Ele não reproduz soluções de desafios, flags, credenciais, conteúdo proprietário de laboratórios, identificadores internos, payloads maliciosos ou walkthroughs passo a passo.*

**Aprender → Questionar → Entender → Aplicar → Compartilhar**
