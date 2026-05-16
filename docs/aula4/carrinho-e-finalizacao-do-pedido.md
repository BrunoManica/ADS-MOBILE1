# Aula 4: Carrinho e finalizacao do pedido

## Indice

- [1. Objetivo da aula](#1-objetivo-da-aula)
- [2. Ponto de partida](#2-ponto-de-partida)
- [3. Decisoes desta aula](#3-decisoes-desta-aula)
- [4. Resultado final](#4-resultado-final)
- [5. Ajustar os tipos](#5-ajustar-os-tipos)
- [6. Criar o provider do carrinho](#6-criar-o-provider-do-carrinho)
- [7. Envolver o app com o provider](#7-envolver-o-app-com-o-provider)
- [8. Mostrar o total nas tabs](#8-mostrar-o-total-nas-tabs)
- [9. Usar modal dentro da tela de produtos](#9-usar-modal-dentro-da-tela-de-produtos)
- [10. Criar a tela de pedido](#10-criar-a-tela-de-pedido)
- [11. Ajustar os estilos](#11-ajustar-os-estilos)
- [12. Codigo completo](#12-codigo-completo)
- [13. Erros comuns](#13-erros-comuns)
- [14. Resumo](#14-resumo)

## 1. Objetivo da aula

Nesta aula, vamos transformar o app `react-burguer` em uma aplicacao com carrinho compartilhado entre telas.

O foco e continuar a partir do que foi construido em aula, sem trocar a arquitetura inteira do projeto.

Ao final, o app tera:

- `CarrinhoProvider` guardando os itens do pedido.
- Hook `useCarrinho()` para acessar o carrinho nas telas.
- Tipo `ProdutoPedido` para representar o produto depois de entrar no pedido.
- Tipo `Pedido` para representar o pedido finalizado.
- Botao de adicionar produto na tela `Comes`.
- Modal local dentro da tela `Comes`, sem criar componente separado.
- Tab `Pedido` com contador.
- Tela `Pedido` para listar, alterar, remover, limpar e finalizar o pedido.

## 2. Ponto de partida

O projeto usado como referencia esta em:

```text
/home/user/Área de trabalho/react-burguer
```

Ele ja possui a estrutura principal:

```text
src/
  App.tsx
  contexts/
    CarrinhoContext.tsx
  pages/
    Cardapio.css
    TabsPrincipal/
      TabsPrincipal.tsx
    TelaBebes/
      TelaBebes.tsx
    TelaComes/
      TelaComes.tsx
  services/
    produtoService.ts
  types/
    Pedido.ts
    Produto.ts
    ProdutoPedido.ts
```

O projeto ja tinha um primeiro rascunho do provider, mas havia dois pontos importantes para corrigir:

- O carrinho estava sendo alterado com `items.push(...)`.
- A tela `TelaComes` ainda guardava um pedido local com `useState`, em vez de usar o provider.

Esses dois pontos fazem diferenca porque o carrinho precisa ser compartilhado por varias telas.

## 3. Decisoes desta aula

Nesta versao da aula, vamos seguir o rumo feito em aula:

- O provider fica em `src/contexts/CarrinhoContext.tsx`.
- O estado do carrinho se chama `items`, igual ao projeto original.
- O modal nao sera um componente separado em `src/components`.
- O modal ficara dentro da `TelaComes`, pois neste momento deixa o fluxo mais facil de enxergar.
- O provider recebe o `Produto` e os dados escolhidos no modal.
- A tela `Pedido` sera criada como a tela responsavel por finalizar.

Essa escolha evita uma abstracao antes da hora. Primeiro o aluno entende o fluxo completo. Depois, em outra aula, o modal pode virar um componente reutilizavel.

## 4. Resultado final

Na tela `Comes`, cada produto tera um botao:

```text
Mc feliz
R$ 36,98

[Adicionar]
```

Ao tocar em `Adicionar`, a propria tela abre um modal:

```text
Adicionar ao pedido

Produto: Mc feliz
Preco: R$ 36,98

Quantidade
[-] 1 [+]

Remover ingredientes
[ ] Pao
[ ] Queijo

[Adicionar ao carrinho]
```

A tab inferior passa a mostrar:

```text
Bebidas | Comidas | Pedido 1
```

E a tela `Pedido` mostra:

```text
Pedido

1x Mc feliz
Remover: nenhum ingrediente
Subtotal: R$ 36,98

[-] [+] [Remover]

Observacao
Total: R$ 36,98
[Finalizar pedido]
```

Por enquanto, finalizar o pedido apenas monta um objeto e mostra no console.

## 5. Ajustar os tipos

### 5.1 `Produto`

Abra:

```text
src/types/Produto.ts
```

Use este tipo:

```ts
export type Produto = {
  id: string;
  codBarras: string;
  ingredientes: string[] | null;
  ativo: boolean;
  nome: string;
  preco: number;
  imagem?: string;
};
type Pedido = {
  produtos: Produto[];
};
```

Nesta aula, `ingredientes` sera sempre uma lista. Isso simplifica o modal, porque podemos usar `.map()` sem verificar se o valor e `null`.

### 5.2 `ProdutoPedido`

Abra:

```text
src/types/ProdutoPedido.ts
```

Use:

```ts
export type ProdutoPedido = {
  id: String;
  produtoId: number;
  nome: String;
  preco: number;
  quantidade: number;
  ingredientesRemovidos: string[];
};
```

`ProdutoPedido` nao e o mesmo que `Produto`.

`Produto` e o item do cardapio. `ProdutoPedido` e o item que o cliente colocou no carrinho.

### 5.3 `Pedido`

Abra:

```text
src/types/Pedido.ts
```

Use:

```ts
import { ProdutoPedido } from './ProdutoPedido';

export type Pedido = {
  observacao: String;
  total: number;
  items: ProdutoPedido[];
};
```

`Pedido` representa o objeto final que sera enviado para uma API no futuro.

## 6. Criar o provider do carrinho

Abra:

```text
src/contexts/CarrinhoContext.tsx
```

O provider deve ser responsavel por:

- guardar `items`;
- calcular `totalItens`;
- calcular `totalPedido`;
- adicionar produto com os ingredientes removidos;
- limpar carrinho;
- expor tudo pelo hook `useCarrinho()`.

Use este codigo, seguindo o contexto desenvolvido em aula no projeto `react-burguer`:

```tsx
import { createContext, ReactNode, useContext, useState } from 'react';
import { Produto } from '../types/Produto';
import { ProdutoPedido } from '../types/ProdutoPedido';

type CarrinhoContextValue = {
  items: ProdutoPedido[];
  totalItens: number;
  totalPedido: number;
  adicionarItem: (item: Produto, ingredientesRemovidos: string[]) => void;
  limparCarrinho: () => void;
  // removerItem: (id: string) => void
};

const CarrinhoContext = createContext<CarrinhoContextValue | undefined>(undefined);

export const CarrinhoProvider = ({ children }: { children: ReactNode }) => {
  const [items, setItems] = useState<ProdutoPedido[]>([]);

  const totalItens = items.reduce((total, item) => total + item.quantidade, 0);

  const totalPedido = items.reduce(
    (total, item) => total + item.quantidade * item.preco,
    0,
  );

  const adicionarItem = (item: Produto, ingredientesRemovidos: string[]) => {
    items.push({
      id: item.id,
      nome: item.nome,
      ingredientesRemovidos: ingredientesRemovidos,
      preco: item.preco,
      quantidade: 1,
      produtoId: Number(item.id),
    });
    setItems(items);
  };

  const limparCarrinho = () => {
    setItems([]);
  };

  return (
    <CarrinhoContext.Provider
      value={{
        items,
        totalItens,
        totalPedido,
        adicionarItem,
        limparCarrinho,
        // removerItem,
      }}
    >
      {children}
    </CarrinhoContext.Provider>
  );
};

export const useCarrinho = () => {
  const context = useContext(CarrinhoContext);

  if (!context) {
    throw new Error('Context Invalido');
  }

  return context;
};
```

Repare que, neste momento da aula, o contexto ainda nao tem `alterarQuantidade` nem `removerItem`. Essas funcoes podem entrar depois, quando a tela de pedido for evoluida.

## 7. Envolver o app com o provider

Abra:

```text
src/App.tsx
```

O `CarrinhoProvider` precisa ficar por fora das tabs:

```tsx
import { IonApp, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { CarrinhoProvider } from './contexts/CarrinhoContext';
import { TabsPrincipal } from './pages/TabsPrincipal/TabsPrincipal';

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
    <CarrinhoProvider>
      <IonReactRouter>
        <TabsPrincipal />
      </IonReactRouter>
    </CarrinhoProvider>
  </IonApp>
);

export default App;
```

Se o provider ficar dentro de uma tela especifica, as outras telas nao conseguem acessar o mesmo carrinho.

## 8. Mostrar o total nas tabs

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Vamos adicionar a tab `Pedido` e mostrar um badge quando houver item no carrinho.

```tsx
import {
  IonBadge,
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
} from '@ionic/react';
import { beerOutline, cartOutline, pizzaOutline } from 'ionicons/icons';
import { Redirect, Route } from 'react-router';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import { TelaBebes } from '../TelaBebes/TelaBebes';
import { TelaComes } from '../TelaComes/TelaComes';
import { TelaPedido } from '../TelaPedido/TelaPedido';

export const TabsPrincipal = () => {
  const { totalItens } = useCarrinho();

  return (
    <IonTabs>
      <IonRouterOutlet>
        <Route exact path="/bebes" component={TelaBebes}></Route>
        <Route exact path="/comes" component={TelaComes}></Route>
        <Route exact path="/pedido" component={TelaPedido}></Route>
        <Route exact path="/">
          <Redirect to="/bebes"></Redirect>
        </Route>
      </IonRouterOutlet>

      <IonTabBar slot="bottom">
        <IonTabButton tab="bebes" href="/bebes">
          <IonIcon icon={beerOutline}></IonIcon>
          <IonLabel>Bebidas</IonLabel>
        </IonTabButton>

        <IonTabButton tab="comes" href="/comes">
          <IonIcon icon={pizzaOutline}></IonIcon>
          <IonLabel>Comidas</IonLabel>
        </IonTabButton>

        <IonTabButton tab="pedido" href="/pedido">
          <IonIcon icon={cartOutline}></IonIcon>
          <IonLabel>Pedido</IonLabel>
          {totalItens > 0 && <IonBadge color="danger">{totalItens}</IonBadge>}
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};
```

O contador vem do provider. Por isso ele atualiza quando qualquer tela adiciona item.

## 9. Usar modal dentro da tela de produtos

Nesta aula, o modal fica dentro da `TelaComes`.

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Substitua pelo codigo abaixo:

```tsx
import { useState } from 'react';
import {
  IonButton,
  IonButtons,
  IonCheckbox,
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonModal,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import { buscarTodosOsProdutos } from '../../services/produtoService';
import { Produto } from '../../types/Produto';
import '../Cardapio.css';

export const TelaComes = () => {
  const comidas = buscarTodosOsProdutos();
  const { adicionarItem } = useCarrinho();
  const [produtoSelecionado, setProdutoSelecionado] = useState<Produto | null>(null);
  const [quantidade, setQuantidade] = useState(1);
  const [ingredientesRemovidos, setIngredientesRemovidos] = useState<string[]>([]);

  const formatarPreco = (preco: number) => {
    return preco.toLocaleString('pt-BR', {
      style: 'currency',
      currency: 'BRL',
    });
  };

  const abrirModal = (produto: Produto) => {
    setProdutoSelecionado(produto);
    setQuantidade(1);
    setIngredientesRemovidos([]);
  };

  const fecharModal = () => {
    setProdutoSelecionado(null);
  };

  const alternarIngrediente = (ingrediente: string, marcado: boolean) => {
    if (marcado) {
      setIngredientesRemovidos((itensAtuais) => [...itensAtuais, ingrediente]);
      return;
    }

    setIngredientesRemovidos((itensAtuais) =>
      itensAtuais.filter((item) => item !== ingrediente),
    );
  };

  const confirmarAdicao = () => {
    if (!produtoSelecionado) {
      return;
    }

    adicionarItem(produtoSelecionado, ingredientesRemovidos);
    fecharModal();
  };

  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar>
          <IonTitle>Comidas</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          {comidas.map((comida) => (
            <IonItem className="cardapio-list" key={comida.id}>
              <IonLabel>
                <h2 className="cardapio-fonte">{comida.nome}</h2>
                <p>{comida.ingredientes.join(', ')}</p>
                <p>{formatarPreco(comida.preco)}</p>
              </IonLabel>

              <IonButton onClick={() => abrirModal(comida)}>Adicionar</IonButton>
            </IonItem>
          ))}
        </IonList>
      </IonContent>

      <IonModal isOpen={!!produtoSelecionado} onDidDismiss={fecharModal}>
        <IonHeader>
          <IonToolbar>
            <IonTitle>Adicionar ao pedido</IonTitle>
            <IonButtons slot="end">
              <IonButton onClick={fecharModal}>Fechar</IonButton>
            </IonButtons>
          </IonToolbar>
        </IonHeader>

        <IonContent className="modal-content">
          {produtoSelecionado && (
            <div className="modal-container">
              <h2>{produtoSelecionado.nome}</h2>
              <p>{formatarPreco(produtoSelecionado.preco)}</p>

              <h3>Quantidade</h3>
              <div className="quantidade-controle">
                <IonButton
                  fill="outline"
                  onClick={() => setQuantidade((valorAtual) => Math.max(1, valorAtual - 1))}
                >
                  -
                </IonButton>
                <strong>{quantidade}</strong>
                <IonButton
                  fill="outline"
                  onClick={() => setQuantidade((valorAtual) => valorAtual + 1)}
                >
                  +
                </IonButton>
              </div>

              <h3>Remover ingredientes</h3>
              <IonList>
                {produtoSelecionado.ingredientes.map((ingrediente) => (
                  <IonItem key={ingrediente} lines="none">
                    <IonCheckbox
                      checked={ingredientesRemovidos.includes(ingrediente)}
                      onIonChange={(event) =>
                        alternarIngrediente(ingrediente, event.detail.checked)
                      }
                    >
                      <IonLabel>{ingrediente}</IonLabel>
                    </IonCheckbox>
                  </IonItem>
                ))}
              </IonList>

              <IonButton expand="block" onClick={confirmarAdicao}>
                Adicionar ao carrinho
              </IonButton>
            </div>
          )}
        </IonContent>
      </IonModal>
    </IonPage>
  );
};
```

Repare que a tela nao usa mais:

```tsx
const [pedido, setPedido] = useState<any[]>([]);
```

Esse estado local nao serve para o carrinho final, porque ele fica preso dentro da `TelaComes`.

## 10. Criar a tela de pedido

Crie a pasta:

```text
src/pages/TelaPedido
```

Crie o arquivo:

```text
src/pages/TelaPedido/TelaPedido.tsx
```

Use:

```tsx
import { useState } from 'react';
import {
  IonButton,
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTextarea,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import { Pedido } from '../../types/Pedido';
import '../Cardapio.css';

export const TelaPedido = () => {
  const { items, totalItens, totalPedido, limparCarrinho } = useCarrinho();
  const [observacao, setObservacao] = useState('');

  const formatarPreco = (preco: number) => {
    return preco.toLocaleString('pt-BR', {
      style: 'currency',
      currency: 'BRL',
    });
  };

  const finalizarPedido = () => {
    const pedido: Pedido = {
      items,
      observacao,
      total: totalPedido,
    };

    console.log('Pedido finalizado:', pedido);
  };

  const limparPedido = () => {
    limparCarrinho();
    setObservacao('');
  };

  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar>
          <IonTitle>Pedido</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <div className="pedido-container">
          <h1>Finalizacao do pedido</h1>
          <p>{totalItens} itens no pedido</p>

          {items.length === 0 ? (
            <div className="pedido-vazio">
              <p>Nenhum item adicionado.</p>
            </div>
          ) : (
            <>
              <IonList>
                {items.map((item) => (
                  <IonItem key={String(item.id)}>
                    <IonLabel>
                      <h2>
                        {item.quantidade}x {item.nome}
                      </h2>
                      <p>
                        Remover:{' '}
                        {item.ingredientesRemovidos.length > 0
                          ? item.ingredientesRemovidos.join(', ')
                          : 'nenhum ingrediente'}
                      </p>
                      <p>Unitario: {formatarPreco(item.preco)}</p>
                      <p>Subtotal: {formatarPreco(item.preco * item.quantidade)}</p>

                      <div className="pedido-acoes">
                        <p>
                          Ajuste e remocao de itens nao fazem parte do contexto base desta aula.
                        </p>
                      </div>
                    </IonLabel>
                  </IonItem>
                ))}
              </IonList>

              <IonTextarea
                label="Observacao"
                labelPlacement="stacked"
                placeholder="Ex: sem talheres, mesa 4"
                value={observacao}
                onIonInput={(event) => setObservacao(event.detail.value ?? '')}
              />

              <div className="pedido-total">
                <span>Total</span>
                <strong>{formatarPreco(totalPedido)}</strong>
              </div>

              <IonButton expand="block" onClick={finalizarPedido}>
                Finalizar pedido
              </IonButton>

              <IonButton expand="block" fill="clear" color="danger" onClick={limparPedido}>
                Limpar pedido
              </IonButton>
            </>
          )}
        </div>
      </IonContent>
    </IonPage>
  );
};
```

Essa tela ainda nao envia para backend. Ela apenas monta o objeto `Pedido`.

## 11. Ajustar os estilos

Abra:

```text
src/pages/Cardapio.css
```

Use ou acrescente:

```css
.cardapio-page {
  background: #1c50b9;
}

.cardapio-fonte {
  font-size: 14px;
  color: chartreuse;
}

.cardapio-list {
  background: transparent;
}

.modal-content {
  --background: #f7f8fa;
}

.modal-container {
  padding: 20px;
}

.quantidade-controle {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.quantidade-controle strong {
  min-width: 32px;
  text-align: center;
}

.pedido-container {
  padding: 20px;
}

.pedido-vazio {
  padding: 16px;
  border-radius: 8px;
  background: #ffffff;
}

.pedido-acoes {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 8px;
}

.pedido-total {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 16px 0;
  padding: 16px;
  border-radius: 8px;
  background: #ffffff;
}
```

## 12. Codigo completo

Esta secao resume os arquivos principais ao final da aula.

### `src/types/Produto.ts`

```ts
export type Produto = {
  id: string;
  codBarras: string;
  ingredientes: string[] | null;
  ativo: boolean;
  nome: string;
  preco: number;
  imagem?: string;
};
type Pedido = {
  produtos: Produto[];
};
```

### `src/types/ProdutoPedido.ts`

```ts
export type ProdutoPedido = {
  id: String;
  produtoId: number;
  nome: String;
  preco: number;
  quantidade: number;
  ingredientesRemovidos: string[];
};
```

### `src/types/Pedido.ts`

```ts
import { ProdutoPedido } from './ProdutoPedido';

export type Pedido = {
  observacao: String;
  total: number;
  items: ProdutoPedido[];
};
```

### `src/services/produtoService.ts`

```ts
import { Produto } from '../types/Produto';

const comida: Produto[] = [
  {
    ativo: true,
    codBarras: '123',
    ingredientes: ['Pao', 'Carne', 'Queijo'],
    nome: 'Mc feliz',
    id: '1',
    preco: 36.98,
  },
  {
    ativo: true,
    codBarras: '321',
    ingredientes: ['Pao', 'Carne', 'Molho especial'],
    nome: 'Mc melt',
    id: '2',
    preco: 38.02,
  },
  {
    ativo: false,
    codBarras: '213',
    ingredientes: ['Pao', 'Carne', 'Alface', 'Queijo'],
    nome: 'Big mac',
    id: '3',
    preco: 40.05,
  },
];

export const buscarTodosOsProdutos = (): Produto[] => {
  return comida;
};
```

### `src/contexts/CarrinhoContext.tsx`

```tsx
import { createContext, ReactNode, useContext, useState } from 'react';
import { Produto } from '../types/Produto';
import { ProdutoPedido } from '../types/ProdutoPedido';

type CarrinhoContextValue = {
  items: ProdutoPedido[];
  totalItens: number;
  totalPedido: number;
  adicionarItem: (item: Produto, ingredientesRemovidos: string[]) => void;
  limparCarrinho: () => void;
  // removerItem: (id: string) => void
};

const CarrinhoContext = createContext<CarrinhoContextValue | undefined>(undefined);

export const CarrinhoProvider = ({ children }: { children: ReactNode }) => {
  const [items, setItems] = useState<ProdutoPedido[]>([]);

  const totalItens = items.reduce((total, item) => total + item.quantidade, 0);

  const totalPedido = items.reduce(
    (total, item) => total + item.quantidade * item.preco,
    0,
  );

  const adicionarItem = (item: Produto, ingredientesRemovidos: string[]) => {
    items.push({
      id: item.id,
      nome: item.nome,
      ingredientesRemovidos: ingredientesRemovidos,
      preco: item.preco,
      quantidade: 1,
      produtoId: Number(item.id),
    });
    setItems(items);
  };

  const limparCarrinho = () => {
    setItems([]);
  };

  return (
    <CarrinhoContext.Provider
      value={{
        items,
        totalItens,
        totalPedido,
        adicionarItem,
        limparCarrinho,
        // removerItem,
      }}
    >
      {children}
    </CarrinhoContext.Provider>
  );
};

export const useCarrinho = () => {
  const context = useContext(CarrinhoContext);

  if (!context) {
    throw new Error('Context Invalido');
  }

  return context;
};
```

## 13. Erros comuns

### 13.1 Entender o `push` usado no contexto

O contexto do projeto de aula usa:

```tsx
items.push({
  id: item.id,
  nome: item.nome,
  ingredientesRemovidos: ingredientesRemovidos,
  preco: item.preco,
  quantidade: 1,
  produtoId: Number(item.id),
});
setItems(items);
```

Esse e o codigo usado no `react-burguer`. Em uma evolucao futura, podemos trocar para uma atualizacao imutavel, mas nesta aula vamos manter o mesmo caminho do projeto desenvolvido em sala.

### 13.2 Manter o pedido dentro da tela `TelaComes`

Evite:

```tsx
const [pedido, setPedido] = useState<any[]>([]);
```

Esse estado so existe dentro da tela. A tab `Pedido` e a tela `Pedido` nao conseguem usar esse valor.

Use o provider:

```tsx
const { adicionarItem } = useCarrinho();
```

### 13.3 Esquecer o provider no `App.tsx`

Se `useCarrinho()` for usado fora de `CarrinhoProvider`, o app gera erro.

O provider deve envolver `IonReactRouter` e `TabsPrincipal`.

### 13.4 Usar `ingredientes: null`

Nesta aula, o modal percorre ingredientes com `.map()`.

Entao prefira:

```ts
ingredientes: ['Pao', 'Queijo'];
```

Evite:

```ts
ingredientes: null;
```

### 13.5 Criar modal separado antes de entender o fluxo

Criar um componente `ModalAdicionarItem` e uma boa ideia quando a logica ja esta clara.

Nesta aula, o modal fica dentro da tela para facilitar o entendimento:

```text
TelaComes
  lista produtos
  abre modal
  escolhe quantidade
  chama adicionarItem()
```

Depois, se o mesmo modal for usado tambem em `TelaBebes`, podemos extrair para um componente.

## 14. Resumo

Nesta aula, voce conectou o cardapio ao carrinho usando React Context.

Voce aprendeu a:

- criar tipos para `Produto`, `ProdutoPedido` e `Pedido`;
- usar `CarrinhoProvider` para compartilhar estado;
- criar o hook `useCarrinho`;
- adicionar produtos ao carrinho pelo `CarrinhoProvider`;
- abrir um `IonModal` dentro da tela;
- calcular total de itens e total do pedido;
- mostrar contador na tab;
- criar a tela de finalizacao do pedido.

Na proxima aula, o fluxo pode evoluir para persistir o pedido em uma API ou para extrair o modal para um componente reutilizavel.
