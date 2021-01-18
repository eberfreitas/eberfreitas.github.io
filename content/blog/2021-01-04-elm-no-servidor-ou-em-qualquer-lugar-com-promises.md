+++
title = "Elm no servidor (ou em qualquer lugar) com promises" 
+++
Elm foi criado pra rodar no browser, mas sempre rola de alguém perguntar como rodar Elm no servidor.

No meu trabalho atual, a gente precisava sincronizar diversos clientes e persistir o estado compartilhado entre eles em um só lugar, então pensamos que seria uma boa ideia se o servidor agisse como mais um cliente que pudesse persistir este estado em um local centralizado.

Pra isso a gente criou um servidor com Node/Express que era uma grande gambiarra.

Num ambiente de servidor, na maioria das vezes você vai ter um request/pedido e uma response/resposta amarrados. Você pede algo e recebe o que pediu ou um erro. Não importa, pra cada request existe uma resposta.

Mas Elm não funciona assim se você precisa falar com o mundo externo. Sim, você pode usar [ports](https://guide.elm-lang.org/interop/ports.html) pra comunicação externa, mas ports usam o actor model de passagem de mensagens. Então ao invés de um request/response como você teria em um servidor, você só pode enviar e receber mensagens. Pode parecer a mesma coisa mas não é. Você pode receber uma mensagem sem nunca ter requisitado uma em primeiro lugar. Ou enviar uma mensagem sem a necessidade de esperar outra de volta. Você pode mandar uma mensagem e receber inúmeras outras de uma só vez. Não existe ligação entre as mensagens enviadas e recebidas, e este é um fator que torna Elm impróprio para a criação de servidores onde o request/response são sempre amarrados.

Depois de procurar por soluções melhores eu encontrei [este post](https://discourse.elm-lang.org/t/imitating-synchronicity-with-ports/1930) nos fóruns onde o usuário [joakin](https://discourse.elm-lang.org/u/joakin) fez uma sugestão engenhosa: basta enviar o objeto de **response** do lado do JavaScript através do port e mandá-lo de volta com a resposta de qualquer coisa que tenha sido requisitada. Aí é só usar este objeto para enviar a resposta apropriada para o client correto e pronto. Você pode ver um exemplo dessa abordagem [neste repositório exemplo](https://github.com/joakin/node-elm-server).

Ta aí uma coisa que eu não sabia: você pode enviar qualquer valor JavaScript como um [`Json.Decode.Value`](https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#Value) pro Elm, até mesmo funções. Claro que não será possível fazer muita coisa com elas dentro do Elm mas, neste caso, ajuda a amarrar uma chamada de função à mensagem que retornaremos.

A ideia é ótima e nos ajudou a amarrar o fluxo de request/response. Mas como poderíamos testar esta integração? Era mais fácil pular toda a parte do servidor e focar na interoperação entre Elm e Node diretamente. Ou pior, e se o software que estivéssemos escrevendo não fosse um servidor Express mas qualquer outra coisa? Foi aí que meu chefe e colega de trabalho [Nate](https://twitter.com/nateabele) sugeriu que usássemos promises. Ao invés de enviar o objeto de response do Express pro Elm, podemos enviar a função de resolve de uma promise!

Eu fiz um fork do repositório exemplo com estas alterações. Você pode checar [aqui](https://github.com/eberfreitas/node-elm-server).

Do [lado do Elm](https://github.com/eberfreitas/node-elm-server/blob/master/src/Server.elm), nada de significativo mudou. Eu fiz apenas umas alterações nos nomes das coisas pra melhor refletir a nova natureza da interoperação com com o código JavaScript. Mas além disso, não precisamos fazer outras alterações já que, tanto o objeto response do Express quanto a função resolve da promise são apenas `Json.Decode.Value`s pro Elm.

A mágica real acontece no [código JavaScript](https://github.com/eberfreitas/node-elm-server/blob/master/index.js). O código ficou um pouco mais complexo mas conseguimos desacoplar o código Elm e os ports do Express, tornando possível usar esta abordagem em praticamente qualquer lugar. Aqui esta a parte que faz tudo funcionar:

```js
http
  .createServer((request, res) => {
    new Promise(resolve => app.ports.onRequest.send({ request, resolve }))
      .then(({ status, response }) => {
        res.statusCode = status;
        res.end(response);
      });
  })
  .listen(3000);

app.ports.resolve.subscribe(([{ resolve }, status, response]) => {
  resolve({ status, response });
});
```

Sendo assim, é possível usar Elm no servidor, e eu diria que é possível usar esta abordagem em qualquer lugar onde você precise de uma integração com request/response amarradas e Node esteja disponível. **Mas é útil?** No nosso caso, onde queríamos reutilizar a maior parte do código de nosso client no server foi uma vitória total, mas eu pensaria duas vezes antes de construir um servidor completo com Elm, já que ele não oferece as ferramentas necessárias pra permitir uma experiência de desenvolvimento satisfatória, embora seja possível.

Talvez [Roc](https://youtu.be/ZnYa99QoznE?t=4755) seja a linguagem apropriada para estes casos. Mal posso esperar!

E então, o que achou desta abordagem? Você já teve que fazer algo similar ou totalmente diferente pra resolver o mesmo problema?

Obrigado por ler!
