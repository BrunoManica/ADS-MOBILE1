# Aula 1: Criando as páginas Comes e Bebes com rotas no Ionic React

## Índice

- [1. Objetivo da aula](#1-objetivo-da-aula)
- [2. Resultado final](#2-resultado-final)
- [3. Contexto](#3-contexto)
- [4. Explicação conceitual](#4-explicacao-conceitual)
- [5. Setup inicial](#5-setup-inicial)
- [6. Passo a passo](#6-passo-a-passo)
- [7. Código completo](#7-codigo-completo)
- [8. Erros comuns](#8-erros-comuns)
- [9. Resumo](#9-resumo)
- [10. Próximo passo](#10-proximo-passo)

## 1. Objetivo da aula

Nesta aula, você vai criar a primeira navegação real do projeto de comanda usando Ionic com React.

Ao final, o app terá:

- Uma tela chamada `TelaBebes`, responsável por exibir bebidas.
- Uma tela chamada `TelaComes`, responsável por exibir comidas.
- Duas rotas no aplicativo:
  - `/bebes`
  - `/comes`
- Um redirecionamento da rota inicial `/` para `/bebes`.

A ideia é começar simples, mas com uma estrutura parecida com a usada em aplicativos reais.

## 2. Resultado final

Quando o app estiver rodando:

- Ao acessar `/bebes`, o usuário verá a tela de bebidas.
- Ao acessar `/comes`, o usuário verá a tela de comidas.
- Ao abrir apenas `/`, o app será enviado automaticamente para `/bebes`.

No projeto, isso fica assim:

```text
src/
+-- App.tsx
+-- pages/
    +-- TelaBebes/
    |   +-- TelaBebes.tsx
    +-- TelaComes/
        +-- TelaComes.tsx
```

Essa organização ajuda o projeto a crescer sem misturar tudo dentro de um único arquivo.

## 3. Contexto

Imagine um aplicativo de comanda usado em um restaurante, bar ou lanchonete.

Um sistema desse tipo normalmente separa produtos por categoria:

- Bebidas
- Comidas
- Fechamento da conta

Nesta primeira aula, vamos começar com duas categorias simples: comes e bebes.

Mesmo sendo uma primeira versão, já vamos usar rotas porque um app mobile real precisa navegar entre telas. No Ionic React, essa navegação é feita com React Router integrado ao Ionic.

## 4. Explicação conceitual

Antes de escrever código, precisamos entender três ideias.

### 4.1 O que é uma página no Ionic?

No Ionic, uma tela normalmente começa com o componente `IonPage`.

Pense no `IonPage` como a folha principal daquela tela. Dentro dela, colocamos:

- `IonHeader`: área superior da tela.
- `IonToolbar`: barra visual dentro do cabeçalho.
- `IonTitle`: título exibido no topo.
- `IonContent`: área principal onde o conteúdo aparece.

Essa estrutura é importante porque o Ionic usa esses componentes para aplicar comportamento de aplicativo mobile, como altura correta da tela, rolagem, cabeçalho e transições.

### 4.2 O que é uma rota?

Uma rota liga um endereço a uma tela.

Exemplo:

```text
/bebes  ->  TelaBebes
/comes  ->  TelaComes
```

Quando o usuário acessa `/bebes`, o React Router entende que deve mostrar o componente `TelaBebes`.

### 4.3 Onde ficam as rotas no projeto?

Neste projeto, as rotas ficam no arquivo:

```text
src/App.tsx
```

Esse arquivo é o ponto central do aplicativo. Ele envolve o app com os componentes principais do Ionic e registra quais telas existem.

## 5. Setup inicial

Esta aula considera que o projeto Ionic React já foi criado.

Para rodar o projeto, entre na pasta do app.

No Linux, o caminho pode ser parecido com:

```bash
cd /caminho/para/a/pasta-do-projeto
```

No Windows, o caminho pode ser parecido com:

```bash
cd C:\caminho\para\a\pasta-do-projeto
```

Instale as dependências, se ainda não tiver instalado:

```bash
npm install
```

Execute o projeto:

```bash
ionic serve
```

O Ionic vai mostrar uma URL local no terminal. Normalmente será algo parecido com:

```text
http://localhost:8100
```

## 6. Passo a passo

Vamos começar criando as duas telas do app: uma para bebidas e outra para comidas.

Com as telas prontas, entramos no `App.tsx` para registrar as rotas e dizer qual endereço abre cada tela. É isso que permite acessar `/bebes`, `/comes` e também redirecionar a página inicial do app.

### 6.1 Criar a pasta da tela de bebidas

Dentro de `src/pages`, crie a pasta:

```text
src/pages/TelaBebes
```

Depois, crie o arquivo:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Por que criar uma pasta para cada tela?

- Facilita encontrar os arquivos.
- Permite adicionar CSS, testes e componentes específicos depois.
- Evita deixar muitas telas soltas dentro de `pages`.

### 6.2 Criar a tela `TelaBebes`

No arquivo `TelaBebes.tsx`, começamos importando os componentes do Ionic:

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
```

O que cada import faz:

- `IonPage`: define a tela inteira.
- `IonHeader`: cria o cabeçalho.
- `IonToolbar`: cria a barra dentro do cabeçalho.
- `IonTitle`: mostra o título da tela.
- `IonContent`: define a área principal da tela.
- `IonList`: cria uma lista com aparência mobile.
- `IonItem`: cria uma linha da lista.
- `IonLabel`: mostra o texto dentro do item.

Agora criamos o componente:

```tsx
const TelaBebes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Bebes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          <IonItem>
            <IonLabel>Bebida 1</IonLabel>
          </IonItem>
        </IonList>
      </IonContent>
    </IonPage>
  );
};
```

Por que essa estrutura existe?

- O `return` define o que aparece na tela.
- O `IonPage` envolve tudo porque o Ionic precisa saber que isso é uma página navegável.
- O `IonHeader` fica separado do `IonContent` para o conteúdo poder rolar sem bagunçar o cabeçalho.
- O `IonList` e o `IonItem` já trazem visual de lista mobile pronto.

No final, exportamos a tela:

```tsx
export default TelaBebes;
```

Esse `export default` permite importar a tela em outro arquivo, como o `App.tsx`.

### 6.3 Criar a pasta da tela de comidas

Dentro de `src/pages`, crie a pasta:

```text
src/pages/TelaComes
```

Depois, crie o arquivo:

```text
src/pages/TelaComes/TelaComes.tsx
```

A lógica é a mesma da tela de bebidas. A diferença é o conteúdo exibido.

### 6.4 Criar a tela `TelaComes`

Importe os componentes do Ionic:

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
```

Crie o componente:

```tsx
const TelaComes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Comes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          <IonItem color="primary">
            <IonLabel>Comida 1</IonLabel>
          </IonItem>
        </IonList>
      </IonContent>
    </IonPage>
  );
};
```

Aqui aparece uma pequena diferença:

```tsx
<IonItem color="primary">
```

O `color="primary"` usa a cor principal configurada no tema do Ionic. Isso mostra que os componentes Ionic aceitam propriedades prontas para alterar aparência sem escrever CSS manualmente.

Finalize exportando:

```tsx
export default TelaComes;
```

### 6.5 Abrir o arquivo de rotas

Agora abra:

```text
src/App.tsx
```

Esse arquivo é onde vamos conectar os caminhos do navegador com as telas criadas.

### 6.6 Importar as telas no `App.tsx`

No topo do arquivo, importe as duas páginas:

```tsx
import TelaBebes from './pages/TelaBebes/TelaBebes';
import TelaComes from './pages/TelaComes/TelaComes';
```

Por que precisamos importar?

Porque o `App.tsx` só consegue usar componentes que ele conhece. Ao importar `TelaBebes` e `TelaComes`, podemos associar cada uma a uma rota.

### 6.7 Entender os componentes de rota

No `App.tsx`, o projeto usa:

```tsx
import { Redirect, Route } from 'react-router-dom';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
```

O papel de cada um:

- `Route`: define qual componente aparece em determinado caminho.
- `Redirect`: redireciona o usuário de uma rota para outra.
- `IonApp`: componente raiz obrigatório do Ionic.
- `IonReactRouter`: integra React Router com Ionic.
- `IonRouterOutlet`: área onde as páginas serão renderizadas.
- `setupIonicReact`: inicializa configurações internas do Ionic React.

### 6.8 Registrar as rotas

Dentro de `IonRouterOutlet`, registre:

```tsx
<Route exact path="/comes" component={TelaComes} />
<Route exact path="/bebes" component={TelaBebes} />
```

O que está acontecendo:

- `path="/comes"` significa que essa rota responde ao endereço `/comes`.
- `component={TelaComes}` significa que a tela exibida será `TelaComes`.
- `exact` evita que a rota combine com caminhos parecidos de forma indevida.

Depois, adicione o redirecionamento inicial:

```tsx
<Route exact path="/">
  <Redirect to="/bebes" />
</Route>
```

Por que redirecionar `/` para `/bebes`?

Porque quando o usuário abre o app, ele normalmente acessa a raiz. Sem esse redirecionamento, a tela inicial poderia ficar vazia ou sem comportamento definido.

## 7. Código completo

### 7.1 `src/pages/TelaBebes/TelaBebes.tsx`

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';

const TelaBebes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Bebes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          <IonItem>
            <IonLabel>Bebida 1</IonLabel>
          </IonItem>
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default TelaBebes;
```

### 7.2 `src/pages/TelaComes/TelaComes.tsx`

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';

const TelaComes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Comes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          <IonItem color="primary">
            <IonLabel>Comida 1</IonLabel>
          </IonItem>
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default TelaComes;
```

### 7.3 `src/App.tsx`

```tsx
import { Redirect, Route } from 'react-router-dom';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import TelaBebes from './pages/TelaBebes/TelaBebes';
import TelaComes from './pages/TelaComes/TelaComes';

import '@ionic/react/css/core.css';
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';
import '@ionic/react/css/padding.css';
import '@ionic/react/css/float-elements.css';
import '@ionic/react/css/text-alignment.css';
import '@ionic/react/css/text-transformation.css';
import '@ionic/react/css/flex-utils.css';
import '@ionic/react/css/display.css';

import './theme/variables.css';

setupIonicReact();

const App: React.FC = () => (
  <IonApp>
    <IonReactRouter>
      <IonRouterOutlet>
        <Route exact path="/comes" component={TelaComes} />
        <Route exact path="/bebes" component={TelaBebes} />

        <Route exact path="/">
          <Redirect to="/bebes" />
        </Route>
      </IonRouterOutlet>
    </IonReactRouter>
  </IonApp>
);

export default App;
```

## 8. Erros comuns

### 8.1 Esquecer o `export default`

Se esquecer:

```tsx
export default TelaBebes;
```

O `App.tsx` não vai conseguir importar a tela corretamente.

### 8.2 Importar o caminho errado

Errado:

```tsx
import TelaBebes from './TelaBebes';
```

Correto para este projeto:

```tsx
import TelaBebes from './pages/TelaBebes/TelaBebes';
```

O caminho precisa sair do arquivo atual e chegar exatamente até o arquivo da tela.

### 8.3 Criar a rota fora do `IonRouterOutlet`

As rotas das páginas Ionic devem ficar dentro de:

```tsx
<IonRouterOutlet>
  {/* rotas aqui dentro */}
</IonRouterOutlet>
```

O `IonRouterOutlet` é quem controla a troca de páginas no Ionic.

### 8.4 Esquecer o `IonPage`

Uma tela Ionic sem `IonPage` pode até renderizar, mas pode apresentar problemas de layout, altura, rolagem e transição.

Para páginas do app, use:

```tsx
<IonPage>
  {/* conteúdo da tela */}
</IonPage>
```

## 9. Resumo

Nesta aula, você criou duas páginas e conectou cada uma a uma rota.

O ponto principal é entender que:

- Página é o componente visual que representa uma tela.
- Rota é o endereço que aponta para essa tela.
- `App.tsx` é onde registramos as rotas principais do aplicativo.
- `IonRouterOutlet` é o lugar onde o Ionic troca uma página pela outra.

Com isso, o app deixou de ter apenas uma tela padrão e passou a ter uma navegação inicial própria para o projeto de comanda.

## 10. Próximo passo

Na próxima aula, o projeto pode evoluir para carregar uma lista de produtos na tela `Comes`.

Assim, a tela deixa de mostrar apenas um item fixo e começa a trabalhar com dados de verdade, que é a base para montar uma comanda nas próximas aulas.
