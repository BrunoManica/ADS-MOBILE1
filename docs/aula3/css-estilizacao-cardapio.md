# Aula 3: Estilizando o cardápio com CSS

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

Nesta aula, vamos melhorar a aparência das telas `Comes` e `Bebes`.

Até aqui, o app já consegue buscar listas de produtos no service e mostrar os itens na tela. Agora vamos deixar essa listagem com cara de cardápio: espaçamento melhor, fundo mais leve, cards para cada item, título mais claro e preço com destaque.

Ao final, o app terá:

- Um arquivo CSS compartilhado para as telas de cardápio.
- Classes CSS aplicadas em `TelaComes` e `TelaBebes`.
- Um topo de página com categoria, título e quantidade de itens.
- Itens com aparência de card.
- Preço com destaque visual.

## 2. Resultado final

As telas deixam de parecer uma lista simples e passam a ter uma estrutura visual mais próxima de um app real.

Na tela de bebidas, por exemplo, o conteúdo passa a ficar organizado assim:

```text
Cardapio
Tela de Bebes
3 itens disponiveis

[ Refrigerante       ]
[ Lata 350ml         ]
[ R$ 6,00            ]

[ Suco Natural       ]
[ Suco natural...    ]
[ R$ 9,00            ]
```

A mesma ideia vale para a tela de comidas.

## 3. Contexto

Em um app de comanda, o cardápio precisa ser fácil de ler rapidamente.

Quem usa esse tipo de app normalmente está atendendo uma mesa, escolhendo produtos e montando um pedido. Se a tela fica bagunçada, pequena demais ou sem destaque no preço, o uso fica ruim.

Nesta aula, o CSS entra para resolver isso:

- separar melhor as informações;
- destacar o nome do produto;
- deixar a descrição mais discreta;
- dar peso visual ao preço;
- criar uma aparência mais limpa para a listagem.

## 4. Explicação conceitual

No React, usamos `className` para aplicar classes CSS nos elementos.

Em HTML puro seria assim:

```html
<div class="cardapio-container"></div>
```

No React fica assim:

```tsx
<div className="cardapio-container"></div>
```

O Ionic também aceita `className` em componentes como `IonPage`, `IonToolbar`, `IonContent`, `IonList` e `IonItem`.

Além disso, alguns componentes do Ionic usam variáveis CSS próprias. Elas começam com `--`, como:

```css
--background: #ffffff;
--border-radius: 12px;
```

Essas variáveis são a forma correta de ajustar partes internas dos componentes Ionic sem brigar com o framework.

## 5. Setup inicial

Esta aula continua a partir da Aula 2.

Os arquivos usados serão:

```text
src/pages/cardapio.css
src/pages/TelaComes/TelaComes.tsx
src/pages/TelaBebes/TelaBebes.tsx
```

Para rodar o projeto, entre na pasta do app.

No Linux, o caminho pode ser parecido com:

```bash
cd /caminho/para/a/pasta-do-projeto
```

No Windows, o caminho pode ser parecido com:

```bash
cd C:\caminho\para\a\pasta-do-projeto
```

Depois execute:

```bash
ionic serve
```

## 6. Passo a passo

### 6.1 Criar o CSS compartilhado

Como `TelaComes` e `TelaBebes` terão a mesma aparência, não faz sentido criar um CSS para cada tela.

Crie o arquivo:

```text
src/pages/cardapio.css
```

Esse arquivo vai guardar as classes visuais do cardápio.

Comece pelo fundo da página:

```css
.cardapio-page {
  background: #f7f8fa;
}
```

Essa classe será aplicada no `IonPage`.

O fundo claro ajuda a separar melhor os cards brancos dos produtos.

### 6.2 Estilizar o topo do Ionic

Agora adicione:

```css
.cardapio-toolbar {
  --background: #ffffff;
  --border-color: #e7ebf0;
  --border-style: solid;
  --border-width: 0 0 1px 0;
}

.cardapio-toolbar-title {
  font-weight: 700;
  letter-spacing: 0.2px;
}
```

Aqui aparecem variáveis do Ionic.

O `IonToolbar` não é um `div` comum. Por isso usamos variáveis como `--background` e `--border-color` para ajustar o visual interno dele.

### 6.3 Criar a área principal do cardápio

Adicione:

```css
.cardapio-content {
  --background: #f7f8fa;
}

.cardapio-container {
  width: min(720px, 100%);
  margin: 0 auto;
  padding: 20px 14px 18px;
}
```

O `cardapio-container` limita a largura do conteúdo.

Isso é útil porque o app pode rodar em celular, navegador ou tablet. Com `width: min(720px, 100%)`, o conteúdo ocupa toda a largura no celular, mas não fica exageradamente largo em telas maiores.

### 6.4 Criar o topo da lista

Agora vamos preparar a área que aparece antes dos produtos:

```css
.cardapio-topo {
  margin-bottom: 12px;
}

.cardapio-overline {
  margin: 0 0 4px;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.8px;
  text-transform: uppercase;
  color: #6b7280;
}

.cardapio-heading {
  margin: 0 0 4px;
  font-size: 24px;
  font-weight: 700;
  color: #121821;
}

.cardapio-meta {
  margin: 0;
  font-size: 14px;
  color: #6b7280;
}
```

Essas classes criam uma hierarquia visual:

- `cardapio-overline`: texto pequeno, usado como categoria.
- `cardapio-heading`: título principal da tela.
- `cardapio-meta`: informação de apoio, como quantidade de itens.

### 6.5 Estilizar a lista e os itens

Agora vem a parte principal do cardápio:

```css
.cardapio-list {
  background: transparent;
  padding: 4px 0 0;
}

.cardapio-item {
  --background: #ffffff;
  --border-color: #e8edf2;
  --border-radius: 12px;
  --border-style: solid;
  --border-width: 1px;
  --inner-padding-top: 13px;
  --inner-padding-bottom: 13px;
  --inner-padding-start: 13px;
  --inner-padding-end: 13px;
  box-shadow: 0 1px 2px rgba(16, 24, 40, 0.06);
  margin-bottom: 12px;
}

.cardapio-item:last-child {
  margin-bottom: 0;
}
```

O `IonItem` tem áreas internas próprias. Por isso usamos variáveis como `--inner-padding-top` e `--inner-padding-start`.

O `box-shadow` cria uma sombra leve. A ideia não é deixar pesado, só separar cada produto do fundo.

### 6.6 Estilizar nome, descrição e preço

Finalize o CSS com:

```css
.cardapio-nome {
  margin: 0 0 6px;
  font-size: 16px;
  font-weight: 600;
  color: #1b1f24;
}

.cardapio-descricao {
  margin: 0 0 8px;
  font-size: 14px;
  line-height: 1.45;
  color: #5e6670;
}

.cardapio-preco {
  margin: 0;
  font-size: 15px;
  font-weight: 700;
  color: #0a6a3f;
}
```

O preço recebe uma cor diferente porque ele precisa ser encontrado rápido na tela.

### 6.7 Importar o CSS na tela de bebidas

Abra:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Depois dos imports, adicione:

```tsx
import '../cardapio.css';
```

O arquivo `cardapio.css` está dentro de `src/pages`. Como `TelaBebes.tsx` está dentro de `src/pages/TelaBebes`, usamos `../` para voltar uma pasta.

### 6.8 Aplicar as classes na tela de bebidas

Agora aplicamos as classes no JSX.

O `IonPage` recebe:

```tsx
<IonPage className="cardapio-page">
```

O `IonToolbar` e o `IonTitle` recebem:

```tsx
<IonToolbar className="cardapio-toolbar">
  <IonTitle className="cardapio-toolbar-title">Bebes</IonTitle>
</IonToolbar>
```

No `IonContent`, criamos um container:

```tsx
<IonContent className="cardapio-content">
  <div className="cardapio-container">
    {/* conteudo do cardapio */}
  </div>
</IonContent>
```

Dentro do container, colocamos o topo:

```tsx
<div className="cardapio-topo">
  <p className="cardapio-overline">Cardapio</p>
  <h1 className="cardapio-heading">Tela de Bebes</h1>
  <p className="cardapio-meta">{bebidas.length} itens disponiveis</p>
</div>
```

O `{bebidas.length}` mostra a quantidade de itens da lista.

### 6.9 Estilizar os itens da lista de bebidas

No `IonList`, adicione a classe:

```tsx
<IonList className="cardapio-list">
```

No `IonItem`, adicione:

```tsx
<IonItem key={produto.id} className="cardapio-item">
```

E dentro do `IonLabel`, aplique as classes de texto:

```tsx
<h2 className="cardapio-nome">{produto.nome}</h2>
<p className="cardapio-descricao">{produto.descricao}</p>
<p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
```

Aqui o dado continua vindo do service. O CSS só muda a apresentação.

### 6.10 Repetir a mesma ideia na tela de comidas

Agora abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Importe o mesmo CSS:

```tsx
import '../cardapio.css';
```

Use as mesmas classes da tela de bebidas.

A diferença fica nos dados e nos textos:

```tsx
<IonTitle className="cardapio-toolbar-title">Comes</IonTitle>
```

```tsx
<p className="cardapio-overline">Cardapio</p>
<h1 className="cardapio-heading">Tela de Comes</h1>
<p className="cardapio-meta">{comidas.length} itens disponiveis</p>
```

Com isso, `TelaComes` e `TelaBebes` passam a ter o mesmo padrão visual.

## 7. Código completo

### 7.1 `src/pages/cardapio.css`

```css
.cardapio-page {
  background: #f7f8fa;
}

.cardapio-toolbar {
  --background: #ffffff;
  --border-color: #e7ebf0;
  --border-style: solid;
  --border-width: 0 0 1px 0;
}

.cardapio-toolbar-title {
  font-weight: 700;
  letter-spacing: 0.2px;
}

.cardapio-content {
  --background: #f7f8fa;
}

.cardapio-container {
  width: min(720px, 100%);
  margin: 0 auto;
  padding: 20px 14px 18px;
}

.cardapio-topo {
  margin-bottom: 12px;
}

.cardapio-overline {
  margin: 0 0 4px;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.8px;
  text-transform: uppercase;
  color: #6b7280;
}

.cardapio-heading {
  margin: 0 0 4px;
  font-size: 24px;
  font-weight: 700;
  color: #121821;
}

.cardapio-meta {
  margin: 0;
  font-size: 14px;
  color: #6b7280;
}

.cardapio-list {
  background: transparent;
  padding: 4px 0 0;
}

.cardapio-item {
  --background: #ffffff;
  --border-color: #e8edf2;
  --border-radius: 12px;
  --border-style: solid;
  --border-width: 1px;
  --inner-padding-top: 13px;
  --inner-padding-bottom: 13px;
  --inner-padding-start: 13px;
  --inner-padding-end: 13px;
  box-shadow: 0 1px 2px rgba(16, 24, 40, 0.06);
  margin-bottom: 12px;
}

.cardapio-item:last-child {
  margin-bottom: 0;
}

.cardapio-nome {
  margin: 0 0 6px;
  font-size: 16px;
  font-weight: 600;
  color: #1b1f24;
}

.cardapio-descricao {
  margin: 0 0 8px;
  font-size: 14px;
  line-height: 1.45;
  color: #5e6670;
}

.cardapio-preco {
  margin: 0;
  font-size: 15px;
  font-weight: 700;
  color: #0a6a3f;
}
```

### 7.2 `src/pages/TelaBebes/TelaBebes.tsx`

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

import { listarBebidas } from '../../services/produtoService';
import '../cardapio.css';

const bebidas = listarBebidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaBebes = () => {
  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">Bebes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-container">
          <div className="cardapio-topo">
            <p className="cardapio-overline">Cardapio</p>
            <h1 className="cardapio-heading">Tela de Bebes</h1>
            <p className="cardapio-meta">{bebidas.length} itens disponiveis</p>
          </div>

          <IonList className="cardapio-list">
            {bebidas.map((produto) => (
              <IonItem key={produto.id} className="cardapio-item">
                <IonLabel>
                  <h2 className="cardapio-nome">{produto.nome}</h2>
                  <p className="cardapio-descricao">{produto.descricao}</p>
                  <p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
                </IonLabel>
              </IonItem>
            ))}
          </IonList>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaBebes;
```

### 7.3 `src/pages/TelaComes/TelaComes.tsx`

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

import { listarComidas } from '../../services/produtoService';
import '../cardapio.css';

const comidas = listarComidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaComes = () => {
  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">Comes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-container">
          <div className="cardapio-topo">
            <p className="cardapio-overline">Cardapio</p>
            <h1 className="cardapio-heading">Tela de Comes</h1>
            <p className="cardapio-meta">{comidas.length} itens disponiveis</p>
          </div>

          <IonList className="cardapio-list">
            {comidas.map((produto) => (
              <IonItem key={produto.id} className="cardapio-item">
                <IonLabel>
                  <h2 className="cardapio-nome">{produto.nome}</h2>
                  <p className="cardapio-descricao">{produto.descricao}</p>
                  <p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
                </IonLabel>
              </IonItem>
            ))}
          </IonList>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaComes;
```

## 8. Erros comuns

### 8.1 Usar `class` em vez de `className`

Em React, use:

```tsx
<div className="cardapio-container">
```

Evite:

```tsx
<div class="cardapio-container">
```

`class` é HTML. Em JSX, o correto é `className`.

### 8.2 Esquecer de importar o CSS

Se as classes forem aplicadas no JSX, mas o arquivo CSS não for importado, nada muda visualmente.

Use:

```tsx
import '../cardapio.css';
```

### 8.3 Criar CSS duplicado para cada tela

Evite criar:

```text
TelaComes.css
TelaBebes.css
```

Quando duas telas têm o mesmo visual, um arquivo compartilhado é mais simples:

```text
src/pages/cardapio.css
```

### 8.4 Tentar estilizar `IonItem` só com propriedades comuns

Algumas partes internas do Ionic são controladas por variáveis CSS.

Para o `IonItem`, prefira:

```css
.cardapio-item {
  --background: #ffffff;
  --border-radius: 12px;
}
```

## 9. Resumo

Nesta aula, o cardápio ganhou uma aparência mais organizada.

O ponto principal é entender que:

- `className` aplica classes CSS no JSX.
- Um CSS compartilhado evita repetição entre telas parecidas.
- Componentes Ionic podem ser estilizados com variáveis CSS.
- Nome, descrição e preço precisam ter pesos visuais diferentes.
- A tela continua usando os mesmos dados; o CSS muda apenas a apresentação.

## 10. Próximo passo

Na próxima aula, o projeto pode evoluir criando um componente reutilizável para o item do cardápio.

Assim, `TelaComes` e `TelaBebes` não precisam repetir a estrutura do `IonItem`, e o card do produto passa a ficar concentrado em um componente próprio.
