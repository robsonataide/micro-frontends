Técnicas, estratégias e receitas para construir uma __aplicação web moderna__ com __múltiplas equipes__ que podem __entregar funcionalidades independentemente__.

## O que são Micro Frontends?

O termo __Micro Frontends__ apareceu pela primeira vez em [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends) que finalizou em 2016. Extendendo os conceitos de micro services ao fundo do frontend. A tendência atual é para construir uma funcionalidade rica e poderosa aplicação para navegadores, também conhecida como "single page app (aplicação de página única)", que fica no topo de uma arquitetura de micro serviços. Com o tempo a camada de frontend, frequentemente desenvolvida por uma equipe separada, cresce e possui uma maior dificuldade para manter. Isso é o que chamamos de 
[Frontend Monolith](https://www.youtube.com/watch?=pU1gXA0rfwc).

A ideia por trás de Micro Frontends é pensar como um website ou aplicação web __uma composição de funcionalidades__ que são propriedades de __equipes independentes__. Cada equipe tem uma __distinta área de negócios__ ou __missão__ do qual se preocupam em especializar-se. Uma equipe é __cross functional__ e desenvolvem suas funcionalidades __fim-a-fim__, desde o banco de dados até a interface para o usuário.

No entando, essa ideia não é nova. Tem muito em comum com o conceito[Sistemas Independentes](http://scs-architecture.org/). No passado era abordado com o nome de [Integração Frontend para Sistemas Verticalizados](https://dev.otto.de/2014/07/29/scaling-with-microservices-and-vertical-decomposition/). Mas Micro Frontends é claramente mais amigável e é um termo menos volumoso.

__Frontends Monolíticos__
![Monolithic Frontends](./ressources/diagrams/organisational/monolith-frontback-microservices.png)


__Organização em Veticais__
![Equipes Fim-A-Fim com Micro Frontends](./ressources/diagrams/organisational/verticals-headline.png)

## O que é uma Aplicação Web Moderna?

Na introdução Eu utilizei a frase "construindo uma aplicação web moderna". Vamos definir a premissa que são conectados com este termo.

Para colocar isto em uma perspectiva mais ampla, [Aral Balkan](https://ar.al/) escreveu em blog a respeito o que ele chama de[Documents‐to‐Applications Continuum](https://ar.al/notes/the-documents-to-applications-continuum/). Ele sugere o conceito de uma escala móvel onde um site, construído a partir de __documents estáticos__, conectados através de links, está __à esquerda__ do fim e um comportamento puro impulsionado, __aplicações sem conteúdo__ como um editor de fotos online está __à direita__.

Se você deve posicionar seu projeto no __lado esquredo deste espectro__, uma __integração no servidor web__ é uma boa opção. Com  este modelo um servidor colecta e __concatena strings HTML__ de todos componentes que 
compõem a página requisitada pelo usuário. Atualizações são finalizadas recarregando a página pelo servidor ou substituindo partes através de ajax. [Gustaf Nilsson Kotte](https://twitter.com/gustaf_nk/) escreveu um [artigo compreensivo](https://gustafnk.github.io/microservice-websites/) neste tópico.

Quando a interface do usuário providencia __feedbacks instantâneos__, mesmo em conexões não confiáveis, um servidor puro renderizando um site não é mais suficiente. Para implementar técnicas como [UI Otimista](https://www.smashingmagazine.com/2016/11/true-lies-of-optimistic-user-interfaces/) ou [Skeleton Screens](http://www.lukew.com/ff/entry.asp?1797) você precisa ser capaz __atualizar__ sua UI __no dispositivo em si__. Termos do Google [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/) apropriadamente descrevem o __ato equilibrado__ de ser um bom cidadão da web (melhoria progressiva) enquanto também providencia performance como um aplicativo. Este tipo de aplicação está localizada em algum lugar __no meio de um app-site__. Aqui uma solução exclusivamente baseada em servidor não é mais suficiente. Nós temos que moê-la para __integração no navegador__, e este não é o foco deste artigo.

## Ideias centrais por trás de Micro Frontends

* __Seja Agnóstico a Tecnologia__<br> Cada equipe deve ser capaz de escolher e atualizar sua stack sem ter que alinhar com outras equipes.[Elementos Personalizados](#the-dom-is-the-api) aqui está um grande caminho para esconder detalhes da implementação enquando providencia uma interface neutra para outros.
* __Equipe com Código Isolado__<br>Não compartilhe o tempo de execução, até se todas as equipes usam o mesmo framework. Construa apps independentes que são independentes. Não confie em compartilhamento de estado ou variáveis globais.
* __Estabelecer Prefixos de equipes__<br> Crie uma convenção de nomes onde o isolamentos ainda não é possível. Nomes de CSS, Eventos, Local Storage e Cookies para evitar colisão e esclarecer propriedades.
* __Favorecer Recursos Nativos do Navegador sobre as APIS Personalizadas__<br> Utilize[Navegador para comunicação de eventos](#parent-child-communication--dom-modification) ao invés de construir um sistema global de PubSub. Se você realmente precisa construir uma API por várias equipes, tente manter o mais simples possível.
* __Construa um Site Resiliente__<br>Suas funcionalidades devem ser úteis, até mesmo se JavaScript falhar ou ainda não estiver executando. Utilize [Renderização Universal](#serverside-rendering--universal-rendering) e Aprimoramentos Progressivos para melhorar a performance percebível. 

---

## O DOM é a API

[Elementos Personalizados](https://developers.google.com/web/fundamentals/getting-started/primers/customelements), o aspecto de interoperabilidade de aspectos dos componentes Web, são uma boa integração primitiva para o navegador. Cada equipe constroi seus componentes __usando tecnologia web de sua escolha__ e __envolvendo isso em um Elemento Personalizado__ (por exemplo`<order-minicart></order-minicart>`). As especificações do DOM deste elemento em particular (tag-name, eventos e atributos) atua como um contrato ou API pública para outras equipes. A vantagem é que eles podem usar o componente e suas funcionalidades sem ter que conhecer a implementação. Eles apenas precisam ser capazes de interagir com o DOM.

Mas Elementos Personalizados sozinhos não são a solução para todas nossas necessidades. Para abordar o melhoramento progressivo, renderização universal ou roteamento nós precisamos de peças de software adicionais.

Esta página é dividida em duas áreas principais. Primeiro nós vamos discutir [Composição de Página](#page-composition) - como montar uma página com componentes que pertencem a outras equipes. Depois disso nós vamos mostrar exemplos para implementar do lado do cliente.[Transição de Página](#page-transition).

## Composição de Página

Ao lado do __cliente__ e __servidor__ integrações do código escrito em __diferentes frameworks__ em si, existe muitos tópicos secundários que devem ser discutidos: mecanimos para __isolar js__, __evitar conflitos de css__, __carregar recursos__ quando necessário, __compartilhar recursos comuns__ entre equipes, lidar __recebimento de dados__ e pensar sobre bom __estado de carregamento__ para seu usuário. Nós vamos entrar nestes tópicos um passo de cada vez.

### O Protótipo Base

O página de produto deste modelo de loja de tratores vai servir como base para os exemplos a seguir.

Possui um __seletor variante__ para alternar entre os três diferentes modelos de trator. Ao trocar a imagem do produto, nome, preço e recomendações atualizadas. Há também um __botão de compra__, do qual adiciona a variant selecionada à cesta e uma __mini cesta__ na parte superior que é atualizada em consequência.

[![Exemplo 0 - Página do Produto - Plain JS](./ressources/video/model-store-0.gif)](./0-model-store/)

[Tente no navegador](./0-model-store/) & [Inspecionar o código](https://github.com/neuland/micro-frontends/tree/master/0-model-store)

Todo HTML é gerado no lado do cliente utilizando __JavaScript plano__ e Template String ES6 __sem dependências__. O code utiliza um simples estado/separação de marcação e re-renderização de todo HTML do lado do cliente a cada mudança - sem nenhuma fantasia diferente do DOM e __sem renderização universal__ por ora. Também __sem paração de equipe__ - [o código](https://github.com/neuland/micro-frontends/tree/master/0-model-store) foi escrito em um arquivo de js/css.

### Integração do lado do cliente

Neste exemplo, a página é dividida entre componentes/fragmentos separados que pertencem a três equipes. __Equipe Checkout__ (azul) agora é responsável por tudo a respeito do processo de compras - sendo assim __botão de compra__ e __mini cesta__. __Equipe Inspire__ (verde) gerencia as __recomendações do produto__ nesta página. A página em si é propriedade da __Equipe Product__ (red).

[![Exemplo 1 - Página do Produto - Composition](./ressources/screen/three-teams.png)](./1-composition-client-only/)

[tente no navegador](./1-composition-client-only/) & [inspecionar código](https://github.com/neuland/micro-frontends/tree/master/1-composition-client-only)

__Equipe Product__ decide quais funcionalidades são incluídas e onde elas serão posicionadas na página. A página contêm informações que podem ser providenciadas pela própria equipe Product, como o nome do produto, imagem e as variedades disponíveis. Mas também inclue fragmentos (Elementos Personalizados) de outras equipes.

### Como criar um Elemento Personalizado?

Vamos pegar o __botão azul__ como exemplo. A equipe Product inclui o botão simplesmente adicionando `<blue-buy sku="t_porsche"></blue-buy>` na posição desejada na marcação. Para isso funcionar, a equipe Checkout tem que registrar o element `blue-buy` na página.

    class BlueBuy extends HTMLElement {
      connectedCallback() {
        this.innerHTML = `<button type="button">buy for 66,00 €</button>`;
      }

      disconnectedCallback() { ... }
    }
    window.customElements.define('blue-buy', BlueBuy);

Agora, toda vez que o navegador encontrar a tag `blue-buy`, o `connectedCallback` é chamado. `this` é a referência para o nó raíz do DOM no elemento Personalizado. Todas propriedades e métodos dos elementos DOM como `innerHTML` ou `getAttribute()` podem ser utilizados.

![Elementos Personalizados em Ação](./ressources/video/custom-element.gif)

Quando nomear seu elemento a única obrigatoriedade específica definida é que o nome precisa __incluir um traço (-)__ para manter a compatibilidade com novas tags HTML. No exemplo a seguir a convenção de nome utilizada é `[team_color]-[feature]`. O espaço de nomes das equipes protege contra colisões e desta maneira o proprietário da característica torna-se óbvio, simplesmente olhando para o DOM. 

### Comunicação Pai-Filho / Modificações do DOM 

Quando o usuário seleciona outro trator no __seletor__, o __botão comprar é atualizado__ em consequẽncia. Para alcançar isso a equipe Product pode simplesmente __remover__ o elemento existente do DOM __e inserir__ um novo.

    container.innerHTML;
    // => <blue-buy sku="t_porsche">...</blue-buy>
    container.innerHTML = '<blue-buy sku="t_fendt"></blue-buy>';

O `disconnectedCallback` do elemento antigo é chamado sincronamente para providenciar o elemento com a possibilidade de limpar coisas do evento que está ouvindo. Depois disso chama-se o `connectedCallback` do elemento recém criado `t_fendt`.

Outra opção mais performática é simplesmente atualizar o atributo `sku`
no elemento existente.

    document.querySelector('blue-buy').setAttribute('sku', 't_fendt');

Se a equipe Product utilizar um motor de template que detecta diferenças no DOM, como React, o algoritmo deve resolver automaticamente.

![Alteração de Atributo de um Elemento Personalizado](./ressources/video/custom-element-attribute.gif)

Para apoiar isto, o Elemento Personalizado pode implementar o `attributeChangedCallback` e especificar a lista de `observedAttributes`
para que este callback seja disparado.

    const prices = {
      t_porsche: '66,00 €',
      t_fendt: '54,00 €',
      t_eicher: '58,00 €',
    };

    class BlueBuy extends HTMLElement {
      static get observedAttributes() {
        return ['sku'];
      }
      connectedCallback() {
        this.render();
      }
      render() {
        const sku = this.getAttribute('sku');
        const price = prices[sku];
        this.innerHTML = `<button type="button">buy for ${price}</button>`;
      }
      attributeChangedCallback(attr, oldValue, newValue) {
        this.render();
      }
      disconnectedCallback() {...}
    }
    window.customElements.define('blue-buy', BlueBuy);

Para evitar duplicação, um método `render()` é introduzido pela chamada do `connectedCallback` e `attributeChangedCallback`. Esta método coleta datas necessários e uma nova marcação é criada do innerHTML. Quando decidir ir com um motor de modelos mais sofisticados ou framework através de Elemento Personalizado, este é o lugar onde a inicialização do código deve ir.

### Suporte em navegador

O exemplo anterior utiliza especificação V1 do Elemento Personalizado do qual é atualmente [suportada non Chrome, Safari e Opera](http://caniuse.com/#feat=custom-elementsv1). Mas com [document-register-element](https://github.com/WebReflection/document-register-element) um polyfill leve e testado em batalhas está disponível para funcionar em todos os navegadores. Sob o capô, utiliza a API dutation Observer [amplamente suportada](http://caniuse.com/#feat=mutationobserver), portanto não há
observação de árvores acontecendo em segundo plano no DOM.

### Framework de Compatibilidade

Como Elementos Personalizados são o padrão da web, todos principais frameworks javascript como Angular, React, Preact, Vue, ou Hyperapp suportam eles. 
Mas quando você entra nos detalhes, existe ainda alguns problemas de implementações em alguns frameworks. Em [Custom Elements Everywhere](https://custom-elements-everywhere.com/) [Rob Dodson](https://twitter.com/rob_dodson) há reunido um conjunto de suíte de testes de compatibilidade que destaca problemas não resolvidos.

### Comunicação Pai-Filho ou Irmãos / Eventos do DOM

Passando atributos não é suficiente para todas interações. No nosso exemplo a __mini cesta deve atualizar__ quando o usuário realizar um __clique no botão de compra__.

Os dois fragmentos são propriedades da equipe Checkout (azul), então eles podem comprar algum tipo de API JavaScript interna que permite que a mini cesta saiba quando o botão foi pressionado. Mas isto deve requerer que a instância do componente se conhecessem e isso também seria uma violação de isolamento.

Uma maneira mais limpa é utilizar um mecanismo de PubSub, onde um componente pode publicar uma mensagem e outros componentes podem assinar específicos tópicos. Felizmente navegadores possuem esta funcionalidade incorporada. É exatamente assim eventos como `click`, `select` ou `mouseover` funcionam. Em adição a eventos nativos há a possibilidade de criar eventos de alto nível com `new CustomEvent(...)`. Os eventos são sempre vinculados ao nó DOM no qual foram criados/despachados. A maioria dos eventos nativos também apresentam bolhas. Isto torna possível ouvir todos os eventos em uma específica subárvore do DOM. Se você quer ouvir todos os eventos na página, anexe o evento ouvinte ao elemento window. Aqui é como é a criação do evento `blue:basket:changed` olhe o exemplo:

    class BlueBuy extends HTMLElement {
      [...]
      connectedCallback() {
        [...]
        this.render();
        this.firstChild.addEventListener('click', this.addToCart);
      }
      addToCart() {
        // maybe talk to an api
        this.dispatchEvent(new CustomEvent('blue:basket:changed', {
          bubbles: true,
        }));
      }
      render() {
        this.innerHTML = `<button type="button">buy</button>`;
      }
      disconnectedCallback() {
        this.firstChild.removeEventListener('click', this.addToCart);
      }
    }

A mini cesta pode agora assinar este evento no `window` e receber notificações de quando deve atualizar seus dados.

    class BlueBasket extends HTMLElement {
      connectedCallback() {
        [...]
        window.addEventListener('blue:basket:changed', this.refresh);
      }
      refresh() {
        // fetch new data and render it
      }
      disconnectedCallback() {
        window.removeEventListener('blue:basket:changed', this.refresh);
      }
    }

Com esta abordagem o fragmento da mini cesta adiciona um ouvinte ao elemento DOM que está fora do escopo (`window`). Isto deve ser ok para muitas aplicações, mas se você está desconfortável com isso você poderia também implementar uma abordagem onde a página por si só (Equipe Product) ouve o evento e notifica a mini cesta chamando `refresh()` no elemento DOM.

    // page.js
    const $ = document.getElementsByTagName;

    $('blue-buy')[0].addEventListener('blue:basket:changed', function() {
      $('blue-basket')[0].refresh();
    });

Imperativamente chamar métodos do DOM é bastante incomum, mas pode ser encontrado em [video element api](https://developer.mozilla.org/de/docs/Web/HTML/Using_HTML5_audio_and_video#Controlling_media_playback) por exemplo. Se possível, o uso de uma abordagem declarativa (troca de atributo) deve ser preferível.

## Renderização do lado do Servidor / Renderização Universal

Elementos Personalizados são ótimos para integração de componentes do lado do navegador. Mas ao criar site que é acessível na web, é provável que o desempenho do carregamento inicial seja importante e os usuários verão uma tela em branco até que todas as estruturas JS sejam baixadas e executadas. Além disso, isso é bom pra se pensar no que acontece ao site se o JavaScript falha ou é bloqueado. [Jeremy Keith](https://adactio.com/) explica a importância em seu ebook/podcast [Resilient Web Design](https://resilientwebdesign.com/). Entretanto, a habilidade de renderizar o conteúdo principal no servidor é essencial. Infelizmente a especificação de componente web não fala sobre renderização no servidor. Nem JavaScript, nem Elementos Personalizados :(

### Elementos Personalizados + Inclusão de Renderização do lado do Servidor = ❤️

Para fazer uma renderização no servidor funcionar, é preciso refatorar o anterior. Cada equipe possui seu próprio servidor expresso e o método
`render()` do Elemento Personalizado que também é acessível via url.

    $ curl http://127.0.0.1:3000/blue-buy?sku=t_porsche
    <button type="button">buy for 66,00 €</button>

O nome da tag do Elemento Personalizado é utilizado como um nome de caminho - atributos se convertem em query parâmetros. Agora tem uma forma de renderizar o conteúdo no servidor de cada componente. Em combinação com o Elemento Personalizado `<blue-buy>` qualquer coisa que está bastante perto de um __Universal Web Componente__ é alcançado:

    <blue-buy sku="t_porsche">
      <!--#include virtual="/blue-buy?sku=t_porsche" -->
    </blue-buy>

O comentário `#include` é parte do [Server Side Includes](https://en.wikipedia.org/wiki/Server_Side_Includes), do qual é uma feature que está disponível em muitos servidores web. Sim, isso é a mesma técnica utilizada em tempos atrás para embutir o dia atual em nosso website. Existem também algumas alternativas técnicas como [ESI](https://en.wikipedia.org/wiki/Edge_Side_Includes), [nodesi](https://github.com/Schibsted-Tech-Polska/nodesi), [compoxure](https://github.com/tes/compoxure) e [tailor](https://github.com/zalando/tailor), mas para nossos projetos SSI providencia sozinho uma simples e inacreditável solução estável.

O comentário `#include` é substituído pela resposta do `/blue-buy?sku=t_porsche` antes que o servidor web envia a página completa para o navegador. A configuração no nginx se parece com isto:

    upstream team_blue {
      server team_blue:3001;
    }
    upstream team_green {
      server team_green:3002;
    }
    upstream team_red {
      server team_red:3003;
    }

    server {
      listen 3000;
      ssi on;

      location /blue {
        proxy_pass  http://team_blue;
      }
      location /green {
        proxy_pass  http://team_green;
      }
      location /red {
        proxy_pass  http://team_red;
      }
      location / {
        proxy_pass  http://team_red;
      }
    }

A diretiva `ssi: on;` habilita a funcionalidade SSI e um `updstream` e o bloco `location` é adicionado para cada equipe para garantir que todas urls que iniciem com `/blue` vão ser encaminhadas para a aplicação correta (`team_blue:3001`). Além do mais, o caminho `/` é mapeado para a equipe vermelha, que está controlando a homepage / da página do produto.

Esta animção mostra uma loja de tratores em um navegador que está com o __JavaScript desabilitado__.

[![Serverside Rendering - JavaScript Desabilitado](./ressources/video/server-render.gif)](./ressources/video/server-render.mp4)

[inspecionar o código](https://github.com/neuland/micro-frontends/tree/master/2-composition-universal)

Os botões da seleção agora estão atualmente conectados e a cada clique
conduz um recarregamento da página. O terminal a direita ilustra o processo de como uma requisição por página é encaminhada para a equipe vermelha, do qual controla a página do produto e depois que a marcação é completada por fragmentos do time azul e verde.

Quando a voltar a ativar o JavaScript, apenas as mensagens de log para a primeira requisição vão estar visíveis. Todos alterações nos tratores subsequentes são manuseados do lado do cliente, como no primeiro exemplo. Em um exemplo posterior os dados do produto vão ser extraídos do JavaScript e carregados por uma api REST quando necessário.

Você pode executar este exemplo de código em sua máquina local. 
Apenas o [Docker Compose](https://docs.docker.com/compose/install/) precisa ser instalado.

    git clone https://github.com/neuland/micro-frontends.git
    cd micro-frontends/2-composition-universal
    docker-compose up --build

Logo o Docker inicia o nginx na porta 3000 e constroi a imagem do node.js para cada time. Quando você abre[http://127.0.0.1:3000/](http://127.0.0.1:3000/) no seu navegador você deve ver um trator vermelho. O log combinado do `docker-compose` torna fácil de ver o que está acontecendo na rede. Infelizmente, não um jeito de controlar a saída de cores, você vai ter que suportar o fato que a equipe azul pode se destacar em verde :)

Os arquivos `src` são mapeados em individuais containers e a aplicação node vai reiniciar quando você fizer uma alteração no código. Alterando o `nginx.conf` requer uma reinicialização do `docker-compose` para que tenha efeito. Fique à vontade para mexer e dar feedback.

### Carga de Dados & Carregamento de Estados

Uma desvantagem da abordagem do SSI/ESI é que o __fragmento mais lento determina o tempo de resposta__ de toda a página.
Isso é bom quando a resposta do fragmento pode ser armazenado em cache.
Para fragmentos que são caros para produzir e difíceis de armazenar em cache é frequentemente uma boa ideia excluí-los da renderização inicial.
ELes pode ser carregados assíncronamente no navegador.
No nosso exemplo, o fragmento `green-recos` que mostra recomendações personalizadas, é um candidato para isto.

Uma solução possível seria que o time vermelho apenas pulasse a inclusão do SSI.

**Antes**

    <green-recos sku="t_porsche">
      <!--#include virtual="/green-recos?sku=t_porsche" -->
    </green-recos>

**Depois**

    <green-recos sku="t_porsche"></green-recos>

*Nota importante: Elementos Personalizados [não podem auto fechar-se](https://developers.google.com/web/fundamentals/architecture/building-components/customelements#jsapi), escrevendo `<green-recos sku="t_porsche" />` não funciona corretamente.*

<img alt="Reflow" src="./ressources/video/data-fetching-reflow.gif" style="width: 500px" />

A renderização apenas tem lugar no navegador.
Mas, como pode ser visto na animação, esta alteração que agora introduz um __reflow importante__ da página.
A área recomendada esa inicialmente em branco.
O JavaScript da equipe verde foi carregando e executado.
A chamda da API para buscar recomendações personalizadas foi feita.
A marcação da recomendação foi renderizada e as imagens associadas são requisitadas.
O fragmento agora precisa de mais espaço e empurra o layout da página.

Existe opções diferentes para evitar um irrirante reflow como isto.
Equipe vermelha, do qual controla a página, pode __fixar a altura do container de recomendações__.
Em um website responsivo é frequentemente é difícil determinar a altura, porque poderia diferir para diferentes tamanhos de talas.
Mas o problema mais importante é que __este tipo de acordo entre as equipes cria um tipo de acoplamento__ entre a equipe vermelha e verde.
Se a equipe verde quer inserir um subtítulo adicional no recuo, terá que coordenar com a equipe vermelha uma nova altetura.
As duas equipes deverão implementar suas alterações simultaneamente para evitar quebra de layout.

A melhor maneira é utilizar uma técnica chamada [Skeleton Screens](https://blog.prototypr.io/luke-wroblewski-introduced-skeleton-screens-in-2013-through-his-work-on-the-polar-app-later-fd1d32a6a8e7).
A equipe vermelha deixa o `green-recos` SSI Include na marcação.
Além do mais, a equipe verde altera o __método de renderização do lado do servidor__ de seus fragmentos para que isso produza uma __versão esquemática do conteúdo__.
O __skeleton markup__ pode reutilizar partes do conteúdo real do layout.
Desta maneira __reserva o espaço necessário__ e o preenchimento do conteúdo real não faz um salto.

<img alt="Skeleton Screen" src="./ressources/video/data-fetching-skeleton.gif" style="width: 500px" />

Telas Skeleton também são __muito úteis para renderização do lado do cliente__.
Quando seu elemento personalizado é inserido no DOM devido uma açõa do usuário poderia __instantaneamente renderizar o skeleton__ até os dados necessários chegarem do servidor.

Mesmo em uma __alteração de atributo__ como na _seleção_ você pode decidir trocar para a visualização do skeleton até a entrega dos novos dados.
Nesta maneira o usuário recebe uma indicação que alguma coisa está acontecendo no fragmento.
Mas quando seu endpoint responde rapidamente uma curto __cintizalição do skeleton__ entre o antigo e o novo dado também poderiam ser irritantes.
Preservando o dado antigo ou utilizando timeouts inteligentes pode ajudar.
Utilize esta técnica com cuidado e tente obter feedback do usuário.

## Navegação entre páginaas

__continua...__

veja o [Github Repo](https://github.com/neuland/micro-frontends) para mais informações



## Recursos Adicionais
- [Book: Micro Frontends in Action](https://www.manning.com/books/micro-frontends-in-action?a_aid=mfia&a_bid=5f09fdeb) Atualmente escrito por mim. Currently in Mannings Early Access Programm (MEAP)
- [Talk: Micro Frontends - MicroCPH, Copenhagen 2019](https://www.youtube.com/watch?v=wCHYILvM7kU) ([Slides](https://noti.st/naltatis/zQb2m5/micro-frontends-the-nitty-gritty-details-or-frontend-backend-happyend)) The Nitty Gritty Details or Frontend, Backend, 🌈 Happyend
- [Talk: Micro Frontends - Web Rebels, Oslo 2018](https://www.youtube.com/watch?v=dTW7eJsIHDg) ([Slides](https://noti.st/naltatis/HxcUfZ/micro-frontends-think-smaller-avoid-the-monolith-love-the-backend)) Think Smaller, Avoid the Monolith, ❤️the Backend
- [Slides: Micro Frontends - JSUnconf.eu 2017](https://speakerdeck.com/naltatis/micro-frontends-building-a-modern-webapp-with-multiple-teams)
- [Talk: Break Up With Your Frontend Monolith - JS Kongress 2017](https://www.youtube.com/watch?v=W3_8sxUurzA) Elisabeth Engel talks about implementing Micro Frontends at gutefrage.net
- [Article: Micro Frontends](https://martinfowler.com/articles/micro-frontends.html) Article by Cam Jackson on Martin Fowlers Blog
- [Post: Micro frontends - a microservice approach to front-end web development](https://medium.com/@tomsoderlund/micro-frontends-a-microservice-approach-to-front-end-web-development-f325ebdadc16) Tom Söderlund explains the core concept and provides links on this topic
- [Post: Microservices to Micro-Frontends](http://www.agilechamps.com/microservices-to-micro-frontends/) Sandeep Jain summarizes the key principals behind microservices and micro frontends
- [Link Collection: Micro Frontends by Elisabeth Engel](https://micro-frontends.zeef.com/elisabeth.engel?ref=elisabeth.engel&share=ee53d51a914b4951ae5c94ece97642fc) extensive list of posts, talks, tools and other resources on this topic
- [Awesome Micro Frontends](https://github.com/ChristianUlbrich/awesome-microfrontends) a curated list of links by Christian Ulbrich 🕶
- [Custom Elements Everywhere](https://custom-elements-everywhere.com/) Making sure frameworks and custom elements can be BFFs
- Tractors are purchasable at [manufactum.com](https://www.manufactum.com/) :)<br>_This store is developed by two teams using the here described techniques._

## Técnicas relacionadas
- [Posts: Cookie Cutter Scaling](https://paulhammant.com/categories.html#Cookie_Cutter_Scaling) David Hammet escreveu uma série de artigos em blog sobre este tema.
- [Wikipedia: Java Portlet Specification](https://en.wikipedia.org/wiki/Java_Portlet_Specification) Especificação que aborda tópicos similares para construir portais empresariais.

---

## Coisas por vir ...

- Casos de uso
  - Navegação entre páginas
    - suave vs. navegação difícil
    - Encaminhamento universal
  - ...
- Temas secundários
  - CSS isolado / Interface do Usuário Coerente / Guias de Estilo & Biblioteca de Padrões
  - Performance em carregamento adicional
  - Performance enquanto utilizar o site
  - Carregando o CSS
  - Carregando o JS
  - Testes de Integração
  - ...

## Colaboradores
- [Koike Takayuki](https://github.com/koiketakayuki) que traduziu o site para [Japanese](https://micro-frontends-japanese.org/).
- [Jorge Beltrán](https://github.com/scipion) que traduziu o site para [Spanish](https://micro-frontends-es.org).
- [Bruno Carneiro](https://github.com/tautorn) que traduziu o site para [Português](https://micro-frontends-pt.org).

Este site é gerado pelo Github Pages. Seu fonte pode ser encontrado em [neuland/micro-frontends](https://github.com/neuland/micro-frontends/).
