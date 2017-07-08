# Tutorial Elm

----
## Sumário

[1. Introdução](#1-introdução)   
[2. Instalação](#2-instalação)   
[3. Jogo da Cobra](#3-jogo-da-cobra)  
[4. Conclusões](#4-conclusões)  
[5. Referências](#5-referências)  

-----

## 1. Introdução

Nesse tutorial veremos como podemos aplicar o elm para aplicações web. Faremos um jogo simples chamado snake, o clássico jogo da cobrinha. Será apresentado o passo-a-passo de instalação e desenvolvimento da aplicação.

## 2. Instalação

Para que o projeto possa ser rodado, é necessária a instalação dos seguintes componentes:

* Elm (0.18)
* Node
* Yarn

O procedimento de instalação será voltado para sistemas operacionais baseados no Ubuntu.

### 2.1 Instalando NodeJs

Primeiramente vamos instalar o NodeJs, para isso entre em seu terminal e digite o seguinte comando:

``` bash
sudo apt install nodejs && sudo apt install nodejs-legacy
```

### 2.2 Instalando Yarn

Após instalado o NodeJs, devemos instalar o Yarn, para isso utilize o seguinte comando em seu terminal:

``` bash
sudo apt-get update && sudo apt-get install yarn
```

### 2.3 Instalando elm
Abra seu terminal e utilize o seguinte comando:

``` bash
npm install -g elm
```

## 3. Jogo da Cobra

Como iremos desenvolver um jogo, é importante entendermos como será o funcionamento do mesmo. Para isso é necessário levantarmos alguns requisitos básicos de snake:

* A cobrinha possui um corpo;
* A cobrinha se move para cima, baixo, esquerda e direita;
* Quando a cobrinha está indo em uma direção ela não pode ir imediatamente para a direção oposta;
* A cobrinha come objetos;
* Os objetos aparecem no jogo em posições aleatórias;
* Quando a cobrinha come, seu corpo aumenta;
* O mapa possui um tamanho definido;
* Quando a cobrinha atinge algum dos cantos do mapa, ela morre;
* Quando a cobrinha se atinge, ela morre.

### 3.1 Criando um Modelo Arquitetural

Como trata-se de um projeto bastante simples, não será realmente um modelo arquitetural que iremos desenvolver. Porém o código estará dividido em model, update, subscriptions e view.  
Cada uma dessas partes será melhor explicada mais a frente.

### 3.2 Começando a Implementação

Primeiramente precisamos importar as bibliotecas necessárias para a produção do projeto, as bibliotecas utilizadas serão:

* core
* svg
* keyboard
* window

Mais a frente explicaremos quais métodos serão utilizados de cada uma dessas bibliotecas.

### 3.3 Criando a Pasta e Importando as Bibliotecas

Primeiro crie a pasta que deverá conter o projeto e após isso, impote as bibliotecas dentro da pasta:

[![](images/import1.png)](images/import1.png)
[![](images/import2.png)](images/import2.png)
[![](images/import3.png)](images/import3.png)

### 3.4 Criando o Arquivo do Código Fonte

Crie um arquivo com o nome “Main.elm” ou com o nome que você desejar mantendo o formato “.elm”, após isso, faça a importação dos seguintes métodos:

[![](images/import4.png)](images/import4.png)

### 3.5 Main

Primeiramente iremos desenvolver nossa main, que conterá as funções init, update, view e subscriptions.

``` Elm
main =
    program { init = ( init, initCmds ), update = update, view = render, subscriptions = subscriptions }
```

***program*** representa ***Record*** que é basicamente um conjunto de funções.
A função ***init*** possui uma tupla com outras duas funções (init e initCmds).

### 3.6 Game Type

***type alias*** em elm representa uma espécie de struct, onde o elemento pode inicializar um ou mais outros elementos.
Para o nosso caso, vamos fazer um ***type alias*** do tipo ***Game***, que irá conter todos os elementos do jogo.

``` Elm
type alias Game =
    { direction : Direction
    , dimensions : Window.Size
    , snake : Snake
    , isDead : Bool
    , fruit : Maybe Block
    , ateFruit : Bool
    , paused : Bool
    }
```

Quando for utilizado, o Game funcionará como uma função, onde cada elemento dentro de seu ***Record*** receberá seu respectivo dado.

### 3.7 Block, Snake e Direction

***Block*** é o bloco que representará a fruta e cada pedaço do corpo da cobra. Ele terá uma posição no plano (x,y).  
***Snake*** será uma lista de blocos.   
***Direction*** contém as direções que a cobra poderá ir.

``` Elm
type alias Block =
    { x : Int
    , y : Int
    }

type alias Snake =
    List Block

type Direction
    = Left
    | Right
    | Up
    | Down
```

É importante destacar que o ***type Direction*** possui a o operador **|**, que representa os diversos tipos de valores que o ***type*** pode possuir.

### 3.8 ArrowKeys, FruitSpawn, Msg

***ArrowKeys*** contém as teclas que serão utilizadas para a movimentação da cobra.   
***FruitSpawn*** contém a posição da fruta e a chance que ela tem de aparecer.   
***Msg*** contém os comandos referentes a movimentação do personagem, aumento do tamanho da cobra, o tick do jogo e o spawn da fruta.   

``` Elm
type ArrowKey
    = NoKey
    | Space
    | LeftKey
    | RightKey
    | UpKey
    | DownKey

type alias FruitSpawn =
    { position : ( Int, Int )
    , chance : Int
    }

type Msg
    = ArrowPressed ArrowKey
    | SizeUpdated Window.Size
    | Tick Time
    | MaybeSpawnFruit FruitSpawn
```

### 3.9 Inicializando o Jogo

Agora vamos inicializar a cobra em uma posição específica, e inicializar o jogo.

``` Elm
initSnake : Snake
initSnake =
    [ Block 25 25
    , Block 24 25
    , Block 23 25
    ]

init : Game
init =
    { direction = Right
    , dimensions = Window.Size 0 0
    , snake = initSnake
    , isDead = False
    , fruit = Nothing
    , ateFruit = False
    , paused = False
    }
```

### 3.10 Update

Concluida a *model*, agora iremos para o update do jogo.

``` Elm
update : Msg -> Game -> ( Game, Cmd Msg )
update msg game =
    case msg of
        ArrowPressed Space ->
            ( { game | paused = not game.paused }, Cmd.none )

        ArrowPressed arrow ->
            ( updateDirection arrow game, Cmd.none )

        SizeUpdated dimensions ->
            ( { game | dimensions = dimensions }, Cmd.none )

        Tick time ->
            updateGame game

        MaybeSpawnFruit spawn ->
            if spawn.chance == 0 then
                ( spawnFruit game spawn, Cmd.none )
            else
                ( game, Cmd.none )
```

A primeira linha representa a assinatura da função, ela é opcional porém bastante recomendada por dar uma boa noção de como a mesma funciona.   
<br>
Para a função update ela recebe ***Msg*** e ***Game*** e retorna um conjunto de tipos ***Game***, ***Cmd*** e ***Msg***, onde o ***Game*** será uma mensagem de comando.   
<br>
Caso a mensagem recebida for a tecla barra de espaço que foi pressionada o pause do jogo será o oposto do que está definido em game.pause, ou seja, se o jogo estiver parado ele vai continuar, e vice-versa.   
<br>
Caso algumas das setas de movimento forem pressionadas elas serão atualizadas.   
<br>
***SizeUpdated*** mantém as dimensões do jogo que são recebidas na inicialização.   
<br>
Durante os períodos de ***tick*** o jogo é atualizado através do método ***updateGame***, que será desenvolvido mais a frente.    
<br>
E ***MaybeSpawnFruit*** recebe sempre um valor de ***spawn*** que são as chances de aparecer uma fruta, se o ***spawn*** for igual a 0 então o programa chama o método ***spawnFruit*** e como parametros ***game*** e ***spawn***, que será feito a seguir.   

### 3.11 updateGame

A função ***updateGame*** recebe um ***type game*** e retorna um ***type game*** e uma mensagem de comando.
<br>
Se o jogador morreu ou o jogo está pausado, o jogo não realiza mais nenhuma checagem.
<br>
Caso contrário, ele passa por diversos métodos através dos operadores ***pipe*** (|>),  que são funções cujo o resultado será utilizado na função seguinte até que os operadores acabem e o elemento utilizado será o que foi declarado acima desses operadores. As funções descritas serão implementadas a seguir.

``` Elm
updateGame : Game -> ( Game, Cmd Msg )
updateGame game =
    if game.isDead || game.paused then
        ( game, Cmd.none )
    else
        ( game, Cmd.none )
            |> checkIfOutOfBounds
            |> checkIfEatenSelf
            |> checkIfAteFruit
            |> updateSnake
            |> updateFruit
```

No caso do ***updateGame***, os operadores ***pipe*** estão recebendo os elementos ***(game, cmd msg)***.

### 3.12 spawnFruit

Essa função vai receber o elemento game e	o elemento de ***spawn*** da fruta, para com que a fruta apareça na posição estipulada no spawn.

``` Elm
spawnFruit : Game -> FruitSpawn -> Game
spawnFruit game spawn =
    let
        ( x, y ) =
            spawn.position
    in
        { game | fruit = Just { x = x, y = y } }
```

Essa função utiliza as declarações ***let in*** onde a sentença significa mais ou menos “**Deixe** um ou mais elementos destacados serem iguais a tanto **em** uma ou mais operações específicas”.
<br>
No caso de ***spawnFruit*** ele faz as posições x e y serem iguais as posições estipuladas em ***spawn***.

### 3.13 checkIfOutOfBounds

Essa função verifica se a cabeça da cobrinha acertou em algum dos cantos da tela, caso isso tenha acontecido a cobrinha morre:

``` Elm
checkIfOutOfBounds : ( Game, Cmd Msg ) -> ( Game, Cmd Msg )
checkIfOutOfBounds ( game, cmd ) =
    let
        head =
            snakeHead game.snake

        isDead =
            (head.x == 0 && game.direction == Left)
                || (head.y == 0 && game.direction == Up)
                || (head.x == 49 && game.direction == Right)
                || (head.y == 49 && game.direction == Down)
    in
        ( { game | isDead = isDead }, cmd )
```

### 3.14 checkIfEatenSelf

Essa função verifica se a cabeça da cobrinha atingiu alguma parte do corpo dela.

``` Elm
checkIfEatenSelf : ( Game, Cmd Msg ) -> ( Game, Cmd Msg )
checkIfEatenSelf ( game, cmd ) =
    let
        head =
            snakeHead game.snake

        tail =
            List.drop 1 game.snake

        isDead =
            game.isDead || List.any (samePosition head) tail
    in
        ( { game | isDead = isDead }, cmd )
```

### 3.15 checkIfAteFruit

Essa função verifica se a cabeça da cobrinha atingiu a fruta.
<br>
Caso não tenha comido a variável do elemento game que valida se a cobrinha comeu ou não (***ateFruit***) permanece falso, caso contrário a variavel torna-se verdadeira durante o frame em que a cabeça e a fruta se encostam.

``` Elm
checkIfAteFruit : ( Game, Cmd Msg ) -> ( Game, Cmd Msg )
checkIfAteFruit ( game, cmd ) =
    let
        head =
            snakeHead game.snake
    in
        case game.fruit of
            Nothing ->
                ( { game | ateFruit = False }, cmd )

            Just fruit ->
                ( { game | ateFruit = samePosition head fruit }, cmd )

samePosition : Block -> Block -> Bool
samePosition a b =
    a.x == b.x && a.y == b.y
```

É importante salientar que as declarações ***Nothing*** e ***Just*** fazem parte da biblioteca ***Maybe***, e representam basicamente quando o elemento possui algum valor ou não e o que deve ser feito em cada uma dessas situações.
<br>
Para o nosso caso, estamos averiguando o valor da variavel ***fruit*** em ***game***, se ela tiver algum valor, o elemento ***game*** irá armazenar em sua variável ***ateFruit*** o resultado da função ***samePosition*** que recebe a cabeça da cobra e a fruta e retorna um boleano. Essa função é utilizada para verificar se estes se encontram na mesma posição.

### 3.16 snakeHead

Essa função é responsável por selecionar onde está a cabeça da cobra:

``` Elm
snakeHead : Snake -> Block
snakeHead snake =
    List.head snake
        |> Maybe.withDefault { x = 0, y = 0 }
```

Esse ***pipe*** está basicamente setando a cabeça da cobra como a posição (0,0) da cobra.

### 3.17 updateFruit

Essa função gera uma fruta sempre que não há frutas no mapa.

``` Elm
updateFruit : ( Game, Cmd Msg ) -> ( Game, Cmd Msg )
updateFruit ( game, cmd ) =
    case game.fruit of
        Nothing ->  
          ( game, Random.generate MaybeSpawnFruit makeFruitSpawnGenerator )

        Just fruit ->
            if game.ateFruit then
                ( { game | fruit = Nothing }, cmd )
            else
                ( game, cmd )

makeFruitSpawnGenerator : Random.Generator FruitSpawn
makeFruitSpawnGenerator =
    let
        spawnPosition =
            Random.pair (Random.int 0 49) (Random.int 0 49)

        spawnChance =
            Random.int 0 9
    in
        Random.map2 (\pos chance -> { position = pos, chance = chance }) spawnPosition spawnChance
```

Perceba que a função utiliza ***Nothing*** para verificar se a variável ***fruit***, não possui algum valor, caso ela não possua, são gerados númros aleatórios para a geração da fruta. Essa geração é feita na função makeFruitSpawnGenerator onde ela é gerada em uma posição aleatória com uma chance 1/10 de aparecer em cada frame.
Essas variáveis serão utilizadas em uma função anônima ***\pos chance*** que é o retorno da função ***makeFruitSpawnGenerator***.

### 3.18 updateSnake

Essa função eh responsável por atualizar a posição da cobrinha durante o jogo em cada frame.

``` Elm
updateSnake : ( Game, Cmd Msg ) -> ( Game, Cmd Msg )
updateSnake ( game, cmd ) =
    let
        head =
            snakeHead game.snake

        head_ =
            case game.direction of
                Up ->
                    { head | y = head.y - 1 }

                Down ->
                    { head | y = head.y + 1 }

                Left ->
                    { head | x = head.x - 1 }

                Right ->
                    { head | x = head.x + 1 }

        tailPositions =
            if game.ateFruit then
                game.snake
            else
                List.take ((List.length game.snake) - 1) game.snake

        tailXs =
            List.map .x tailPositions

        tailYs =
            List.map .y tailPositions

        tail_ =
            List.map2 Block tailXs tailYs
    in
        if game.isDead then
            ( game, cmd )
        else
            ( { game | snake = head_ :: tail_ }, cmd )
```

Utilizando ***let in*** novamente, a função armazena a posição da cabeça da cobra, a direção em que a cobra se encontra (o que cria uma nova cabeça na direção armazenada no próximo frame), e a posição do restante do corpo.
<br> <br>
Caso a cobra tenha comido uma fruta o corpo fica parado durante um frame para que a cobra possa crescer. Caso isso não ocorra é importante retirar sempre o último bloco da lista de blocos que é a cobra usando a função ***take*** da biblioteca ***List***.
<br> <br>
***List.map*** armazena todos as variaveis de um tipo específico de um elemento ***type***, o que ocorre com tailXs e tailYs onde cada um armazena as posições x e y de cada bloco do corpo da cobra.
***tail_*** é utilizado para concatenar essas duas listas criadas em blocos.
<br> <br>
Caso a cobra morra o jogo para, caso contrário ele recria a cobra com os valores armazenados em ***head_*** e ***tail_***.

### 3.19 updateDirection

Essa função é reponsável por atualizar a direção que a cobra está tomando.

``` Elm
updateDirection : ArrowKey -> Game -> Game
updateDirection key game =
    let
        { direction } =
            game

        direction_ =
            if key == LeftKey && direction /= Right then
                Left
            else if key == RightKey && direction /= Left then
                Right
            else if key == UpKey && direction /= Down then
                Up
            else if key == DownKey && direction /= Up then
                Down
            else
                direction
    in
        { game | direction = direction_ }
```

Caso alguma direcional tenha sido pressionada e não seja oposta à direção que a cobrinha já está tomando, o valor de ***direction*** muda para a direção selecionada.

### 3.20 Subscriptions

Terminado o update, vamos para os subscriptions.
Subscriptions em Elm é basicamente a maneira que a sua aplicação tem de receber *inputs* externos. Os metodos criados aqui tem exatamente esse objetivo, receber os comandos do teclado, ou atualizar o tamanho da janela ou setar a quantidade de frames por segundo.

``` Elm
subscriptions : Game -> Sub Msg
subscriptions model =
    Sub.batch [ arrowChanged, windowDimensionsChanged, tick ]

initCmds : Cmd Msg
initCmds =
    Task.perform SizeUpdated Window.size

windowDimensionsChanged : Sub Msg
windowDimensionsChanged =
    Window.resizes SizeUpdated

tick : Sub Msg
tick =
    Time.every (100 * Time.millisecond) Tick

arrowChanged : Sub Msg
arrowChanged =
    Keyboard.downs toArrowChanged

toArrowChanged : Keyboard.KeyCode -> Msg
toArrowChanged code =
    case code of
        32 ->
            ArrowPressed Space

        37 ->
            ArrowPressed LeftKey

        38 ->
            ArrowPressed UpKey

        39 ->
            ArrowPressed RightKey

        40 ->
            ArrowPressed DownKey

        default ->
            ArrowPressed NoKey
```

Na função ***subscriptions***, ***Sub.batch*** é uma função que recebe uma lista de *subscriptionsI (***[ arrowChanged, windowDimensionsChanged, tick ]***), e retorna um único *subscription* que inclui todos eles.
<br><br>
Na função ***initCmds***, Task.perform executa a função ***Window.size***, sempre que ***SizeUpdated*** ocorrer.


### 3.21 View

Na nossa view, é utilizada a biblioteca Svg para renderizar todos os elementos visíveis do jogo.

``` Elm
size : String
size =
    "100"

backgroundColor : Attribute Msg
backgroundColor =
    fill "#333333"

render : Game -> Html.Html Msg
render game =
    let
        ( scaledWidth, scaledHeight ) =
            scale game.dimensions

        parentStyle =
            style [ ( "margin", "0 auto" ), ( "display", "block" ) ]
    in
        svg
            [ width scaledWidth, height scaledHeight, viewBox "0 0 50 50", parentStyle ]
            ([ renderBackground ]
                ++ renderSnake game.snake
                ++ renderFruit game.fruit
            )

renderBackground : Svg Msg
renderBackground =
    rect [ x "0", y "0", width size, height size, backgroundColor ] []

renderSnake : Snake -> List (Svg Msg)
renderSnake snake =
    List.map renderBlock snake

renderBlock : Block -> Svg Msg
renderBlock block =
    let
        ( strX, strY ) =
            ( toString block.x, toString block.y )
    in
        rect [ x strX, y strY, width "1", height "1", fill "red", rx "0.2" ] []

renderFruit : Maybe Block -> List (Svg Msg)
renderFruit fruit =
    case fruit of
        Nothing ->
            []

        Just fruit ->
            [ renderBlock fruit ]

scale : Window.Size -> ( String, String )
scale size =
    let
        toPixelStr =
            \i -> round i |> toString

        ( fWidth, fHeight ) =
            ( toFloat size.width, toFloat size.height )

        ( scaledX, scaledY ) =
            if fWidth > fHeight then
                ( fHeight / fWidth, 1.0 )
            else
                ( 1.0, fWidth / fHeight )
    in
        ( toPixelStr (fWidth * scaledX), toPixelStr (fHeight * scaledY) )
```

## 4. Conclusão

Com esse tutorial podemos perceber que a linguagem Elm possui uma gama de ferramentas que nos disponibiliza realizar variadas soluções web. Sua linguagem por mais que seja funcional é de fácil leitura e entendimento se for bem identada.
<br><br>
Uma das dificuldades enfrentadas para a realização desse tutorial foi encontrar algumas bibliotecas para o funcionamento do jogo, principalmente a biblioteca de renderização. Porém como o Elm possui uma boa documentação remota, foi fácil de ser aprendido e reproduzido.

## 5. Referências

* https://becoming-functional.com/tasks-in-elm-0-18-2b64a35fd82e
* https://www.elm-tutorial.org/en/
* https://www.youtube.com/watch?v=okt6-T0IiNI
