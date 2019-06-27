<!--
2º Projeto de Linguagens de Programação I 2018/2019 (c) by Nuno Fachada

2º Projeto de Linguagens de Programação I 2018/2019 is licensed under a
Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.

You should have received a copy of the license along with this
work. If not, see <http://creativecommons.org/licenses/by-nc-sa/4.0/>.
-->

# 2º Projeto de Linguagens de Programação I 2018/2019

## Descrição do problema

Pretende-se que os alunos desenvolvam, em grupos de 2 a 3 elementos, um
jogo/simulador em C# no qual zombies perseguem e infetam humanos. O jogo
desenrola-se numa grelha 2D toroidal<sup>[1](#fn1)</sup>) com dimensões _X_ e
_Y_ e vizinhança de Moore<sup>[2](#fn2)</sup>. Em cada célula da grelha pode
estar no máximo um agente, que pode ser um **zombie** ou um **humano**. No
início da simulação existem _n<sub>z</sub>_ zombies e _n<sub>h</sub>_ humanos,
num total de _n = n<sub>z</sub>_ + _n<sub>h</sub>_ agentes. Os agentes devem
ser espalhados aleatoriamente pela grelha no início de cada jogo.

O jogo é _turn-based_, e em cada _turn_ (iteração) cada agente pode realizar
uma ação. Os humanos podem apenas realizar um tipo de ação: movimento. Os
zombies podem realizar dois diferentes tipos de ação: 1) movimento; e, 2)
infeção de humanos. O movimento dos agentes pode ser realizado para uma célula
vazia numa vizinhança de Moore de raio 1. A infeção de humanos pode ser
realizada por zombies quando o humano está numa célula adjacente na vizinhança
de Moore. A ordem em que os agentes executam as suas ações em cada _turn_
é aleatória<sup>[3](#fn3)</sup>, de modo a que nenhum agente em específico
obtenha a vantagem de agir cedo durante todo o jogo.

Os agentes podem ser controlados pelo computador através de uma Inteligência
Artificial (IA) básica, ou podem ser controlados por jogadores. Neste último
caso, um jogador que controle determinado agente pode decidir o destino do
mesmo. Se o agente for um zombie, a ação de infeção equivale à indicação de
movimento para o local onde está um humano. Nesse caso o zombie não se move
(pois o local já está ocupado pelo humano), mas o humano é convertido em
zombie. Se o humano era controlado por um jogador, deixa de o ser, e passa a
ser controlado pela IA. O jogo termina quando não existirem mais agentes do
tipo humano na grelha.

Caso um agente seja controlado pela IA, as suas decisões dependem do tipo de
agente:

* Humano - Tenta mover-se na direção oposta ao zombie mais próximo. Se a célula
  para onde o humano deseja mover-se estiver ocupada, o humano fica no mesmo
  local.
* Zombie - Caso exista um humano numa célula adjacente, infeta-o. Caso
  contrário, tenta mover-se na direção do humano mais próximo. Se a célula para
  onde o zombie deseja mover-se estiver ocupada, o zombie fica no mesmo local.

Atenção que, devido ao mapa ser toroidal, o agente mais próximo pode estar do
oposto da grelha!

O jogo termina quando o número máximo de _turns_ for atingido, ou quando não
existirem mais humanos na simulação.

## Modo de funcionamento

### Invocação do programa

O programa deve aceitar sete opções na linha de comando<sup>[4](#fn4)</sup>
(CMD ou PowerShell):

* `-x` - Dimensão horizontal da grelha de jogo.
* `-y` - Dimensão vertical da grelha de jogo.
* `-z` - Número total de zombies.
* `-h` - Número total de humanos.
* `-Z` - Número de zombies controlados por jogadores.
* `-H` - Número de humanos controlados por jogadores.
* `-t` - Número máximo de _turns_.

Um exemplo de execução:

```text
dotnet run -- -x 20 -y 20 -z 10 -h 30 -Z 1 -H 2 -t 1000
```

A opção `--` serve para separar entre as opções do comando `dotnet` e as opções
do programa a ser executado, neste caso o nosso jogo/simulação.

As opções indicadas são obrigatórias e podem ser dadas em qualquer ordem, desde
que o valor numérico suceda à opção propriamente dita. Se alguma das opções for
omitida o programa deve terminar com uma mensagem de erro indicando o modo de
uso.

### Modos de jogo

#### Modo automático

O programa entra em modo automático quando não existem agentes controlados
por jogadores. Neste modo o jogo desenrola-se sem intervenção direta do
utilizador. A visualização deve ser atualizada no fim de cada _turn_ (pelo
menos). No entanto, de modo a ser possível observar a evolução da simulação,
poderá ser boa ideia solicitar ao utilizador para pressionar uma tecla antes de
se dar início à próxima _turn_. Outra alternativa é especificar um _delay_
entre cada _turn_ usando o método estático [`Thread.Sleep()`] (requer o
_namespace_ `System.Threading`).

#### Modo interativo

Este modo é semelhante ao automático, apenas com duas pequenas diferenças: 1)
cada vez que um agente controlado pelo jogador é chamado a agir, o programa
fica a aguardar o _input_ do jogador sobre que ação tomar; e, 2) a visualização
do jogo deve ser atualizada imediatamente antes de ser solicitado _input_ a um
jogador (pelo menos). Se a dada altura deixarem de existir agentes controlados
pelo jogador, o programa entra em modo automático.

<a name="visualize"></a>

## Visualização do jogo

A visualização do jogo deve ser feita em modo de texto (consola), fazendo uso
de cores e de carateres [Unicode][], ambos suportados nativamente no C#, para
facilitar a clareza da ação no jogo. Para o efeito deve ser incluída a
instrução `Console.OutputEncoding = Encoding.UTF8;` no método `Main()` (é
necessário usar o _namespace_ `System.Text`).

No mínimo, a visualização deve distinguir claramente entre humanos e zombies,
bem como entre agentes controlados pela IA (NPCs) e agentes controlados pelo
jogador. Cada agente deve ainda ter um ID específico, em hexadecimal, de modo
a que o jogador possa saber que agente(s) controla, além de que pode facilitar
bastante o _debug_ do jogo.

A [Figura 1](#fig1) mostra uma possível implementação mínima da visualização do
jogo para uma grelha 8x8.

<a name="fig1"></a>

```
.   .   .   .   .   .   .   .         N
                                      |
.   .   .   .  z01 h03  .   .      O--+--L
                                      |
.   .   .   .  H00  .   .   .         S

.   .   .   .  h02  .   .   .

.   .   .   .   .   .   .   .

.  z00  .   .   .   .   .   .

.   .   .   .   .   .  h01  .

.   .   .   .   .   .   .   .


* Proximo a jogar: H00
  - A Norte existe o zombie 01 (IA).
  - A Sul existe o humano 02 (IA).
  - A Oeste o caminho está livre.
  - A Leste o caminho está livre.
* Qual o caminho a seguir? A (oeste) ou D (leste) >
```

**Figura 1** - Possível implementação mínima da visualização do jogo para uma
grelha 8x8. Os zombies estão associados à letra `z`, enquanto os humanos são
representados pela letra `h`. Agentes com letra maiúscula são controlados pelo
jogador.

## Implementação

<a name="orgclasses"></a>

### Organização do projeto e estrutura de classes

O projeto deve estar devidamente organizado, fazendo uso de classes, _structs_
e enumerações. Cada classe, _struct_ ou enumeração deve ser colocada num
ficheiro com o mesmo nome. Por exemplo, uma classe chamada `Agent` deve ser
colocada no ficheiro `Agent.cs`. A estrutura de classes deve ser bem pensada e
organizada de uma forma lógica, e [cada classe deve ter uma responsabilidade
específica e bem definida][SRP].

<a name="fases"></a>

### Fases da implementação

O jogo deve ser implementado incrementalmente em várias fases. Os projetos
precisam de implementar pelo menos a Fase 1 para serem avaliados.

#### Fase 1

Na fase 1 deve ser implementado tudo o que é pedido no enunciado, aceitando-se
as seguintes simplificações:

* Os agentes controlados pela IA podem mover-se de forma aleatória, respeitando
no entanto as regras do jogo, como por exemplo: a) só pode existir um agente
por célula da grelha; ou, b) se um zombie tentar mover-se para o local onde
está um humano, esse humano é infetado (mas ambos os agentes mantém-se no local
onde estavam).
* Implementação apenas da visualização mínima, como exemplificado na
[Figura 1](#fig1).

A implementação completa desta fase equivale a 50% de cumprimento do
[objetivo **O1**](#objetivos), equivalente a uma nota máxima de 2.5 valores se
os restantes [objetivos](#objetivos) forem totalmente cumpridos.

#### Fase 2

Na fase 2 deve ser implementada uma visualização que faça uso de cores e
caracteres [Unicode][] para fácil distinção entre os diferentes agentes.

Aceita-se ainda a simplificação da IA descrita na fase 1.

A implementação completa desta fase equivale a 60% de cumprimento do
[objetivo **O1**](#objetivos) (nota máxima de 3 valores).

#### Fase 3

Na fase 3 não se aceitam simplificações, pelo que o projeto deverá ser
desenvolvido tal como descrito no enunciado. Em concreto, a IA dos agentes deve
ser totalmente implementada como solicitado.

A implementação completa desta fase equivale a 100% de cumprimento do
[objetivo **O1**](#objetivos) (nota máxima de 5 valores).

#### Fase extra

Na fase extra, além de tudo o que é pedido para a fase 3, deve também ser
desenvolvido um sistema de _save games_. Deve ser possível guardar a situação
do jogo no fim de cada _turn_. O _load_ de um _save game_ deve ser feito na
invocação do programa, através de uma opção adicional na linha de comandos:

* `-s` - Indica ficheiro de _save game_ a fazer _load_, por exemplo:
`dotnet run -- -s mysave.sav`

Esta opção dispensa e sobrepõe-se a todas as outras.

A implementação completa desta fase permite compensar eventuais problemas
noutras partes do código e/ou do projeto, facilitando a obtenção da nota
máxima de 5 valores.

<a name="objetivos"></a>

## Objetivos e critério de avaliação

Este projeto tem os seguintes objetivos:

* **O1** - Jogo deve funcionar como especificado (ver [fases](#fases) de
  implementação, obrigatório implementar pelo menos a Fase 1).
* **O2** - Projeto e código bem organizados, nomeadamente:
  * Estrutura de classes bem pensada (ver secção [Organização do projeto e
    estrutura de classes](#orgclasses)).
  * Código devidamente comentado e indentado.
  * Inexistência de código "morto", que não faz nada, como por exemplo
    variáveis, propriedades ou métodos nunca usados.
  * Projeto compila e executa sem erros e/ou *warnings*.
* **O3** - Projeto adequadamente documentado com
  [comentários de documentação XML][XML]. A documentação gerada em formato HTML
  ou CHM com [Doxygen], [Sandcastle] ou ferramenta similar \[[3][ref3]\], deve
  estar incluída no ZIP do projeto, mas **não** integrada no repositório Git.
* **O4** - Repositório Git deve refletir boa utilização do mesmo, com *commits*
  de todos os elementos do grupo e mensagens de *commit* que sigam as melhores
  práticas para o efeito (como indicado
  [aqui](https://chris.beams.io/posts/git-commit/),
  [aqui](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53),
  [aqui](https://github.com/erlang/otp/wiki/writing-good-commit-messages) e
  [aqui](https://stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)).
  Quaisquer *assets* binários, tais como imagens para uso no relatório,
  por exemplo, devem ser integrados no repositório em modo Git LFS.
* **O5** - Relatório em formato [Markdown] (ficheiro `README.md`), organizado
  da seguinte forma:
  * Título do projeto.
  * Nome dos autores (primeiro e último) e respetivos números de aluno.
  * Indicação do repositório público Git utilizado. Esta indicação é opcional,
    pois podem preferir desenvolver o projeto num repositório privado.
  * Informação de quem fez o quê no projeto. Esta informação é **obrigatória**
    e deve refletir os *commits* feitos no Git.
  * Descrição da solução:
    * Arquitetura da solução, com breve explicação de como o programa foi
      organizado e indicação das estruturas de dados usadas (para a grelha de
      jogo, por exemplo), bem como os algoritmos implementados (para procura de
      inimigo mais próximo, por exemplo).
    * Um diagrama UML de classes simples (i.e., sem indicação dos membros da
      classe) descrevendo a estrutura de classes.
    * Um fluxograma mostrando o _game loop_.
    * Conclusões e matéria aprendida.
    * Referências, incluindo trocas de ideias com colegas, código aberto
      reutilizado ou no qual se basearam (e.g., do StackOverflow ou do GitHub)
      e bibliotecas de terceiros utilizadas. Devem ser o mais detalhados
      possível.
  * Nota adicionais sobre o relatório:
    * O relatório deve ser simples e breve, com informação mínima e suficiente
      para que seja possível ter uma boa ideia do que foi feito.
    * Atenção aos erros ortográficos, pois serão tidos em conta na nota final.
    * Atenção à formatação [Markdown], pois será tida em conta na nota final.
    * Se usarem o [Visual Studio Code] para fazer o relatório, façam uso da
      [extensão de correção ortográfica][VSCodeSpellCheck]
      (e o seu dicionário de [Português][VSCodeSpellCheckPT]) e [extensões para
      edição de Markdown][VSCodeMarkdown].

O projeto tem um peso de 5 valores na nota final da disciplina e será avaliado
de forma qualitativa. Isto significa que todos os objetivos têm de ser
parcialmente ou totalmente cumpridos. A cada objetivo, **O1** a **O5**, será
atribuída uma nota entre 0 e 1. A nota do projeto será dada pela seguinte
fórmula:

`N = 5 x O1 x O2 x O3 x O4 x O5 x D`

Em que *D* corresponde à nota da discussão e percentagem equitativa de
realização do projeto, também entre 0 e 1. Isto significa que se os alunos
ignorarem completamente um dos objetivos, não tenham feito nada no projeto ou
não comparecerem na discussão, a nota final será zero.

## Entrega

O projeto deve ser entregue por **grupos de 2 ou 3 alunos** via Moodle até às
**23h de 18 de junho de 2019**. Deve ser submetido um ficheiro `zip` com os
seguintes conteúdos:

* Solução ou projeto Visual Studio com implementação do jogo.
* Pasta escondida `.git` com o repositório Git local do projeto.
* Documentação HTML ou CHM gerada com [Doxygen], [Sandcastle] ou ferramenta
  similar \[[3][ref3]\].
* Ficheiro `README.md` contendo o relatório do projeto em formato [Markdown].
* Ficheiros de imagem contendo o fluxograma e o diagrama UML de classes.
  Estes ficheiros podem ser incluídos no repositório em modo Git LFS.

## Honestidade académica

Nesta disciplina, espera-se que cada aluno siga os mais altos padrões de
honestidade académica. Isto significa que cada ideia que não seja do
aluno deve ser claramente indicada, com devida referência ao respetivo
autor. O não cumprimento desta regra constitui plágio.

O plágio inclui a utilização de ideias, código ou conjuntos de soluções
de outros alunos ou indivíduos, ou quaisquer outras fontes para além
dos textos de apoio à disciplina, sem dar o respetivo crédito a essas
fontes. Os alunos são encorajados a discutir os problemas com outros
alunos e devem mencionar essa discussão quando submetem os projetos.
Essa menção **não** influenciará a nota. Os alunos não deverão, no
entanto, copiar códigos, documentação e relatórios de outros alunos, ou dar os
seus próprios códigos, documentação e relatórios a outros em qualquer
circunstância. De facto, não devem sequer deixar códigos, documentação e
relatórios em computadores de uso partilhado.

Nesta disciplina, a desonestidade académica é considerada fraude, com
todas as consequências legais que daí advêm. Qualquer fraude terá como
consequência imediata a anulação dos projetos de todos os alunos envolvidos
(incluindo os que possibilitaram a ocorrência). Qualquer suspeita de
desonestidade académica será relatada aos órgãos superiores da escola
para possível instauração de um processo disciplinar. Este poderá
resultar em reprovação à disciplina, reprovação de ano ou mesmo suspensão
temporária ou definitiva da ULHT.

*Texto adaptado da disciplina de [Algoritmos e
Estruturas de Dados][aed] do [Instituto Superior Técnico][ist]*

## Notas

<sup><a name="fn1">1</a></sup> Num mapa toroidal 2D, a grelha "dá a volta" na
vertical e na horizontal.

<sup><a name="fn2">2</a></sup> Numa grelha 2D, a vizinhança de
[Moore] é composta pela célula central e pelas 8 células que a rodeiam.

<sup><a name="fn3">3</a></sup> Por outras palavras, a lista de agentes deve ser
embaralhada (_shuffled_) no início de cada _turn_. O algoritmo de
[Fisher–Yates](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
é um método de embaralhamento (_shuffling_) tipicamente utilizado para este
fim.

<sup><a name="fn4">4</a></sup> Os argumentos da linha de comandos estão
disponíveis num _array_ de _strings_ chamado `args` no método `Main()`, como
explicado nas páginas 285 e 286 do livro da disciplina.

## Referências

* <a name="ref1">\[1\]</a> Whitaker, R. B. (2016). **The C# Player's Guide**
  (3rd Edition). Starbound Software.
* <a name="ref2">\[2\]</a> Albahari, J. (2017). **C# 7.0 in a Nutshell**.
  O’Reilly Media.
* <a name="ref3">\[3\]</a> Dorsey, T. (2017). **Doing Visual Studio and .NET
  Code Documentation Right**. Visual Studio Magazine. Retrieved from
  <https://visualstudiomagazine.com/articles/2017/02/21/vs-dotnet-code-documentation-tools-roundup.aspx>.

## Licenças

Este enunciado é disponibilizado através da licença [CC BY-NC-SA 4.0].

## Metadados

* Autor: [Nuno Fachada][]
* Curso:  [Licenciatura em Videojogos][lamv]
* Instituição: [Universidade Lusófona de Humanidades e Tecnologias][ULHT]

[ref1]:#ref1
[ref2]:#ref2
[ref3]:#ref3
[CC BY-NC-SA 4.0]:https://creativecommons.org/licenses/by-nc-sa/4.0/
[lamv]:https://www.ulusofona.pt/licenciatura/videojogos
[Nuno Fachada]:https://github.com/fakenmc
[ULHT]:https://www.ulusofona.pt/
[aed]:https://fenix.tecnico.ulisboa.pt/disciplinas/AED-2/2009-2010/2-semestre/honestidade-academica
[ist]:https://tecnico.ulisboa.pt/pt/
[Markdown]:https://guides.github.com/features/mastering-markdown/
[Doxygen]:https://www.stack.nl/~dimitri/doxygen/
[Sandcastle]:https://github.com/EWSoftware/SHFB
[KISS]:https://en.wikipedia.org/wiki/KISS_principle
[XML]:https://docs.microsoft.com/dotnet/csharp/codedoc
[SRP]:https://en.wikipedia.org/wiki/Single_responsibility_principle
[`Console`]:https://docs.microsoft.com/dotnet/api/system.console
[Unicode]:https://unicode-table.com/
[Visual Studio Code]:https://code.visualstudio.com/
[VSCodeMarkdown]:https://code.visualstudio.com/docs/languages/markdown
[VSCodeSpellCheck]:https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker
[VSCodeSpellCheckPT]:https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker-portuguese
[Moore]:https://en.wikipedia.org/wiki/Moore_neighborhood
[`Thread.Sleep()`]:https://docs.microsoft.com/dotnet/api/system.threading.thread.sleep
