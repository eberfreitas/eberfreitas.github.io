+++
title = "Pelmodoro - um aplicativo para pomodoro feito em Elm" 
+++
[**Pelmodoro**](https://www.pelmodoro.com/) é um aplicativo para te ajudar a aplicar a _técnica pomodoro_ no seu dia a dia. Se você ainda não conhece, vale a pena ler sobre na [Wikipedia](https://pt.wikipedia.org/wiki/T%C3%A9cnica_pomodoro).

![Pelmodoro](/imgs/2021-07-25-pelmodoro/pelmodoro.png)
Já existem diversos aplicativos para pomodoro, dos mais variados tipos e complexidades. Depois de alguma experiência com aplicativos diferentes, eu resolvi tentar fazer o meu próprio e **Pelmodoro** é o resultado.

Além do _timer_ normal, que existe em todo aplicativo para pomodoro, coloquei alguns outros recursos:

- Personalização do número de rounds e dos tempos de cada sessão (trabalho, pausa e pausa longa);
- Controle sobre como o timer se comporta ao final de cada sessão de trabalho ou pausa;
- Controle sobre como você quer ser notificado do término de cada sessão;
- Temas!!! 🎨
- Integração com o Spotify para tocar alguma playlist à sua escolha durante cada round;
- Sistema de avaliação dos seus rounds de trabalho;
- Estatísticas de uso do aplicativo contando o número de horas de trabalho, de descanso, etc;
- Possibilidade de exportar e importar os seus dados estatísticos;

Além disso **Pelmodoro** é um [PWA](https://pt.wikipedia.org/wiki/Progressive_web_app) [off-line first](http://offlinefirst.org/) o que significa que você pode instalá-lo no seu celular ou até mesmo no desktop. Eu tenho o usado como uma aplicação independente usando o suporte a PWAs do Edge.

![Pelmodoro com temas](/imgs/2021-07-25-pelmodoro/themes.png)
_Pelmodoro rodando como um PWA no desktop e alguns dos temas que podem ser usados_

O projeto é totalmente open-source e você pode dar uma olhada no código no [GitHub](https://github.com/eberfreitas/pelmodoro).

## Por que criar mais um aplicativo para pomodoro?

Não existe só um motivo, e a resposta poderia ser simplesmente porque eu quis 😁 Mas além da vontade de criar algo, eu queria usar [**Elm**](https://elm-lang.org/), uma linguagem que tenho usado diariamente pelo último ano e meio, mas que nunca tinha criado algo do zero e totalmente meu.

Além disso eu vinha usando o [Habitica](https://habitica.com/) (uma espécie de jogo RPG que tenta te ajudar a cumprir seus objetivos) para acompanhar minha produtividade e tarefas do dia a dia, mas sentia que era um tiro de canhão para o que eu precisava. A ideia, principalmente ao desenvolver o sistema de avaliação dos rounds, foi substituir o Habitica integrando os recursos que faziam sentido pra mim num aplicativo para pomodoro.

![Calendário com estatísticas](/imgs/2021-07-25-pelmodoro/calendar.png)
_O calendário das estatísticas também serve como gráfico mostrando seus dias mais produtivos, inspirando no gráfico do GitHub_

## Fazendo funcionar

A principal ferramenta que usei pra criar a aplicação foi a linguagem [**Elm**](https://elm-lang.org/) e seu ecossistema, que não é muito extenso, mas que oferece muita qualidade e me surpreendeu durante o desenvolvimento.

**Elm** é uma linguagem funcional, fortemente tipada e pura (com efeitos colaterais controlados) voltada para a criação de front-ends e, apesar de não ser uma linguagem extremamente popular, foi tranquilizador ver que muitos dos problemas que eu precisava resolver já haviam sido solucionados.

- Para a renderização do timer eu usei SVG e o excelente [pacote oficial](https://package.elm-lang.org/packages/elm/svg/latest/) para a criação e manipulação de documentos SVG;
- Usei o [elm-css](https://package.elm-lang.org/packages/elm/svg/latest/) para o CSS, que me permite definir os estilos de maneira tipada e dinâmica;
- Para a manipulação de datas eu usei o pacote [date](https://package.elm-lang.org/packages/justinmimbs/date/latest/);
- A criação do calendário na área de estatísticas foi facilmente resolvido com o pacote [calendar](https://package.elm-lang.org/packages/abradley2/elm-calendar/latest/).

Para alguns outros recursos, não tive opção a não ser apelar para o JavaScript em si, principalmente na integração com o Spotify. Por sorte o Spotify tem uma [documentação razoavelmente completa](https://developer.spotify.com/documentation/web-api/) sobre como utilizar sua API. Depois de um pouco de problemas lutando com o esquema de autorização usando [PKCE](https://developer.spotify.com/documentation/general/guides/authorization-guide/#authorization-code-flow-with-proof-key-for-code-exchange-pkce), implementar a integração foi bem tranquilo.

Além disso, para persistir o estado do aplicativo fiz uso tanto do [localStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/localStorage) (para as preferências e o estado do timer), quanto do [IndexedDB](https://developer.mozilla.org/pt-BR/docs/Web/API/IndexedDB_API) (para salvar as estatísticas de uso). Ao invés de usar a API do IndexedDB diretamente, utilizei a excelente biblioteca [Dexie](https://dexie.org/) que abstrai grande parte das complexidades de lidar com IndexedDB.

Para executar os sons de notificação usei a lib [howler.js](https://howlerjs.com/).

![Avaliação dos rounds](/imgs/2021-07-25-pelmodoro/sentiment.png)
_Avalie cada round de trabalho e verifique suas estatísticas_

## Tornando belo

Depois de me sentir satisfeito com os recursos implementados e a maneira como tudo estava funcionando, mostrei o trabalho para algumas pessoas que me deram feedbacks muito importantes, principalmente a respeito da estrutura do meu código. Em um período de aproximadamente 2 dias eu reescrevi a estrutura da aplicação refatorando a arquitetura quase que por completo. Ao final eu tinha um PR com a adição de [5.934 linhas e a remoção de 3.756](https://twitter.com/eber_freitas/status/1418434236084797443).

Em um projeto deste tamanho, escrito totalmente em JavaScript, este seria um PR assustador, mas como ele foi feito com Elm, eu tinha garantias de que o meu código estava correto e compilando normalmente, o que me deu a tranquilidade de fazer o merge do PR sem pensar duas vezes.

A estrutura original do código foi sendo construída organicamente conforme os recursos iam sendo desenvolvidos, o que gerou um código funcional mas às vezes mal organizado e usando alguns _anti-patterns_. Um exemplo destes anti-patterns foi a separação dos módulos `Model`, `Msg` e `Types`. A ideia em separar estes módulos foi de evitar [importações cíclicas](https://github.com/elm/compiler/blob/9d97114702bf6846cab622a2203f60c2d4ebedf2/hints/import-cycles.md), mas o que ela acabou demonstrando foram falhas na organização do meu código.

Usando o código da aplicação [Real World](https://github.com/rtfeldman/elm-spa-example), pude ver como é possível organizar melhor os módulos em partes separadas onde cada uma implementa o seu próprio loop de MVU (model-view-update), mantendo o módulo `Main.elm` como um _hub_ que agrega cada "página" da aplicação.

A minha função `update` original [era gigantesca](https://github.com/eberfreitas/pelmodoro/blob/6d4a9e16b254d6b27fd2d5c8699657bbcb6b226d/src/Main.elm#L465), mas depois pude separar as mensagens para cada [página](https://github.com/eberfreitas/pelmodoro/tree/main/src/Page) mantendo o código mais organizado, contido e fácil de entender.

Dá pra falar muita coisa sobre padrões de estrutura de código em Elm mas recomendo a leitura do [Elm patterns](https://sporto.github.io/elm-patterns/) para uma visão mais geral dos principais padrões.

Além da reestruturação da aplicação, eu acabei implementando algumas decisões mais estilísticas para trazer algum senso de padronização pro código e que eu acho, pessoalmente, que tem seus benefícios:

- Evitar exposição de construtores na definição de módulos;
- Evitar expor tipos e funções na importação de módulos;
- Quando fizer alias na importação de um módulo, utilizar o mesmo nome do módulo em si, simulando o comportamento do `alias` em Elixir. Ex.: `Html.Attributes as Attributes`;
- No caso de colisão de nomes optar por a) não usar alias algum ou b) aglomerar os nomes dos módulos. Ex.: `Svg.Attributes as SvgAttributes`;
- Prefixar funções de view com `view` 👀

Na maioria dos casos estas decisões foram tomadas pra tornar o código mais explícito, deixando claro de onde estão vindo as funções e tipos que estão sendo utilizados e quais são os seus efeitos.

![Horas mais produtivas](/imgs/2021-07-25-pelmodoro/hourly.png)
_Estatísticas mensais, incluindo seus horários de trabalho mais produtivos_

## Pensamentos finais

Apesar da refatoração grande que fiz, sei que a estrutura ainda poderia melhorar em vários lugares, mas eu quero dar este projeto como finalizado. Eu sinto que software é algo que nunca está realmente pronto, tirando algumas jóias raras, mas eu preciso parar de colocar tanto tempo neste projeto que já funciona muito bem e supre as minhas necessidades e me afundar em algum outro projeto paralelo que vai consumir todo meu tempo livre 🤡

No geral eu estou muito satisfeito com o resultado final e tenho usado o aplicativo diariamente. Com sorte, o aplicativo também será útil para outras pessoas e se este for o seu caso eu vou ficar extremamente feliz se você me contar depois 😊
