# Aula 3: Navegação com tabs e início do pedido

## 1. Objetivo da aula

Nesta aula, você vai trocar a navegação simples por uma navegação com tabs e começar a montar o pedido.

Até aqui, o app já mostra produtos nas telas `Bebes` e `Comes`. Agora faz sentido dar um passo na direção de uma comanda real: o usuário escolhe um produto, informa a quantidade, remove ingredientes se quiser e adiciona esse item ao pedido.

Ao final da aula, o app terá:

- Uma barra inferior com tabs.
- Uma tab para `Bebes`.
- Uma tab para `Comes`.
- Uma tab para `Pedido`.
- Um contexto para guardar o pedido em andamento.
- Um modal para adicionar produto ao pedido.
- Controle de quantidade.
- Seleção de ingredientes que o cliente quer remover.
- Um contador na tab `Pedido`.

## 2. Resultado final

O app passa a ter uma navegação inferior:

```text
[ Bebes ]   [ Comes ]   [ Pedido ]
```

Nas telas de produtos, cada item ganha um botão:

```text
Refrigerante
Lata 350ml
R$ 6,00

[Adicionar]
```

Ao clicar em `Adicionar`, o app abre um modal:

```text
Adicionar item

Refrigerante
R$ 6,00

Quantidade
[-] 1 [+]

Remover ingredientes
[ ] Lata 350ml

[Adicionar ao pedido]
```

Depois de adicionar, a tab `Pedido` mostra a quantidade de itens. A tela `Pedido` também passa a listar o que já foi escolhido.

## 3. Contexto

Em uma comanda, o produto do cardápio não é exatamente a mesma coisa que o item do pedido.

O produto diz o que existe para vender:

```text
Xis Salada
R$ 28,00
Ingredientes: Pao, Hamburguer, Queijo, Alface, Tomate, Maionese
```

O item do pedido diz o que o cliente escolheu:

```text
2x Xis Salada
Sem Tomate
```

Por isso, nesta aula vamos criar um tipo separado para o item do pedido. Também vamos guardar esses itens em um estado React, porque a tela precisa atualizar sempre que um item for adicionado ou removido.

## 4. Explicação conceitual

O `produtoService.ts` continua cuidando dos produtos. Ele responde perguntas como:

```text
Quais bebidas existem?
Quais comidas existem?
```

O pedido é outro assunto. Ele muda enquanto o usuário usa o app:

- adiciona item;
- remove item;
- muda quantidade;
- escolhe ingredientes removidos.

Para compartilhar esse pedido entre várias telas, vamos usar `Context`.

Para guardar a lista de itens, vamos usar `useState`.

Essa combinação é suficiente para a aula:

```text
CarrinhoContext
    guarda os itens do pedido
    oferece métodos para alterar o pedido

TelaBebes e TelaComes
    abrem o modal de adicionar item

ModalAdicionarItem
    chama adicionarItem()

TelaPedido
    lê os itens do pedido
```

Não vamos usar Redux, reducer ou biblioteca externa. Para este momento, métodos simples deixam o código mais claro.

## 5. Setup inicial

Esta aula continua a partir da Aula 2.

Arquivos que serão criados:

```text
src/types/itemPedido.ts
src/contexts/CarrinhoContext.tsx
src/components/ModalAdicionarItem.tsx
src/pages/TabsPrincipal/TabsPrincipal.tsx
src/pages/TelaPedido/TelaPedido.tsx
```

Arquivos que serão alterados:

```text
src/App.tsx
src/pages/TelaBebes/TelaBebes.tsx
src/pages/TelaComes/TelaComes.tsx
src/pages/cardapio.css
```

Para rodar o projeto:

No Linux:

```bash
cd /caminho/para/a/pasta-do-projeto
ionic serve
```

No Windows:

```bash
cd C:\caminho\para\a\pasta-do-projeto
ionic serve
```

## 6. Passo a passo

### 6.1 Criar o tipo `ItemPedido`

Crie o arquivo:

```text
src/types/itemPedido.ts
```

Escreva:

```ts
export type ItemPedido = {
  id: string;
  produtoId: number;
  nome: string;
  preco: number;
  quantidade: number;
  ingredientesRemovidos: string[];
};
```

Esse tipo representa um produto depois que ele foi escolhido para o pedido.

O campo `id` identifica aquele item dentro do pedido. Ele será uma `string` porque vamos gerar esse valor automaticamente com `crypto.randomUUID()`.

O campo `produtoId` guarda o `id` original do produto. Isso é útil porque o pedido pode precisar saber de qual produto aquele item veio.

O campo `ingredientesRemovidos` guarda somente o que o cliente não quer no item.

### 6.2 Criar o contexto do carrinho

Crie a pasta:

```text
src/contexts
```

Dentro dela, crie:

```text
src/contexts/CarrinhoContext.tsx
```

Comece com os imports:

```tsx
import { createContext, useContext, useState } from 'react';
import type { ReactNode } from 'react';
import type { ItemPedido } from '../types/itemPedido';
```

O `createContext` cria o contexto. O `useContext` permite usar esse contexto nas telas. O `useState` guarda os itens do pedido.

### 6.3 Definir o que o contexto entrega

Abaixo dos imports, crie o tipo:

```tsx
type CarrinhoContextValue = {
  itens: ItemPedido[];
  totalItens: number;
  totalPedido: number;
  adicionarItem: (item: Omit<ItemPedido, 'id'>) => void;
  removerItem: (id: string) => void;
  limparCarrinho: () => void;
};
```

Esse tipo mostra exatamente o que qualquer tela poderá usar:

- `itens`: lista de itens do pedido.
- `totalItens`: soma das quantidades.
- `totalPedido`: soma dos valores.
- `adicionarItem`: método para adicionar item.
- `removerItem`: método para remover item.
- `limparCarrinho`: método para limpar tudo.

O trecho `Omit<ItemPedido, 'id'>` quer dizer que a tela não precisa mandar o `id`. O próprio contexto cria esse valor.

### 6.4 Criar o contexto

Agora crie:

```tsx
const CarrinhoContext = createContext<CarrinhoContextValue | undefined>(undefined);
```

Começamos com `undefined` porque o valor real só existe quando o app estiver dentro do `CarrinhoProvider`.

### 6.5 Criar o `CarrinhoProvider`

Abaixo do contexto, crie o provider:

```tsx
export const CarrinhoProvider = ({ children }: { children: ReactNode }) => {
  const [itens, setItens] = useState<ItemPedido[]>([]);

  const totalItens = itens.reduce((total, item) => total + item.quantidade, 0);
  const totalPedido = itens.reduce((total, item) => total + item.preco * item.quantidade, 0);

  const adicionarItem = (item: Omit<ItemPedido, 'id'>) => {
    const novoItem: ItemPedido = {
      ...item,
      id: crypto.randomUUID(),
    };

    setItens((itensAtuais) => [...itensAtuais, novoItem]);
  };

  const removerItem = (id: string) => {
    setItens((itensAtuais) => itensAtuais.filter((item) => item.id !== id));
  };

  const limparCarrinho = () => {
    setItens([]);
  };

  return (
    <CarrinhoContext.Provider
      value={{
        itens,
        totalItens,
        totalPedido,
        adicionarItem,
        removerItem,
        limparCarrinho,
      }}
    >
      {children}
    </CarrinhoContext.Provider>
  );
};
```

Aqui não tem mistério: o carrinho é um array guardado no `useState`.

Os totais são calculados a partir desse array:

```tsx
const totalItens = itens.reduce((total, item) => total + item.quantidade, 0);
const totalPedido = itens.reduce((total, item) => total + item.preco * item.quantidade, 0);
```

Esses valores não precisam ser guardados em outro `useState`, porque eles nascem da lista de itens. Se a lista muda, o componente renderiza de novo e os totais são recalculados.

Quando adicionamos um item, fazemos:

```tsx
setItens((itensAtuais) => [...itensAtuais, novoItem]);
```

Isso cria uma nova lista com os itens antigos e o item novo. Em React, é importante criar uma nova lista em vez de alterar a lista antiga diretamente.

Quando removemos um item, usamos `filter`:

```tsx
setItens((itensAtuais) => itensAtuais.filter((item) => item.id !== id));
```

O `filter` cria uma nova lista apenas com os itens que devem continuar no pedido.

### 6.6 Criar o hook `useCarrinho`

No final do arquivo, escreva:

```tsx
export const useCarrinho = () => {
  const context = useContext(CarrinhoContext);

  if (!context) {
    throw new Error('useCarrinho deve ser usado dentro de CarrinhoProvider');
  }

  return context;
};
```

Com esse hook, as telas poderão usar o carrinho assim:

```tsx
const { itens, adicionarItem } = useCarrinho();
```

Isso é mais limpo do que importar o contexto diretamente em toda tela.

### 6.7 Criar a tela `TelaPedido`

Crie a pasta:

```text
src/pages/TelaPedido
```

Dentro dela, crie:

```text
src/pages/TelaPedido/TelaPedido.tsx
```

Comece com:

```tsx
import {
  IonButton,
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import '../cardapio.css';
```

Depois crie a função de preço:

```tsx
const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};
```

Agora crie a tela:

```tsx
const TelaPedido = () => {
  const { itens, totalPedido, removerItem } = useCarrinho();

  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">Pedido</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-container">
          <div className="cardapio-topo">
            <p className="cardapio-overline">Carrinho</p>
            <h1 className="cardapio-heading">Finalizacao de pedido</h1>
            <p className="cardapio-meta">{itens.length} itens no pedido</p>
          </div>

          {itens.length === 0 ? (
            <div className="pedido-vazio">
              <p>Nenhum item adicionado.</p>
            </div>
          ) : (
            <>
              <IonList className="cardapio-list">
                {itens.map((item) => (
                  <IonItem key={item.id} className="cardapio-item">
                    <IonLabel>
                      <h2 className="cardapio-nome">
                        {item.quantidade}x {item.nome}
                      </h2>

                      <p className="cardapio-descricao">
                        Remover:{' '}
                        {item.ingredientesRemovidos.length > 0
                          ? item.ingredientesRemovidos.join(', ')
                          : 'nenhum ingrediente'}
                      </p>

                      <p className="cardapio-preco">
                        {formatarPreco(item.preco * item.quantidade)}
                      </p>

                      <IonButton fill="clear" color="danger" onClick={() => removerItem(item.id)}>
                        Remover
                      </IonButton>
                    </IonLabel>
                  </IonItem>
                ))}
              </IonList>

              <div className="pedido-total">
                <span>Total</span>
                <strong>{formatarPreco(totalPedido)}</strong>
              </div>
            </>
          )}
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaPedido;
```

Essa tela ainda é uma primeira versão. Ela já mostra os itens adicionados, permite remover item e calcula o total.

### 6.8 Criar as tabs principais

Crie a pasta:

```text
src/pages/TabsPrincipal
```

Dentro dela, crie:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Comece com:

```tsx
import { Redirect, Route } from 'react-router-dom';
import {
  IonBadge,
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
} from '@ionic/react';
import { cafeOutline, cartOutline, restaurantOutline } from 'ionicons/icons';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import TelaBebes from '../TelaBebes/TelaBebes';
import TelaComes from '../TelaComes/TelaComes';
import TelaPedido from '../TelaPedido/TelaPedido';
```

Agora escreva:

```tsx
const TabsPrincipal = () => {
  const { totalItens } = useCarrinho();

  return (
    <IonTabs>
      <IonRouterOutlet>
        <Route exact path="/bebes" component={TelaBebes} />
        <Route exact path="/comes" component={TelaComes} />
        <Route exact path="/pedido" component={TelaPedido} />
        <Route exact path="/">
          <Redirect to="/bebes" />
        </Route>
      </IonRouterOutlet>

      <IonTabBar slot="bottom">
        <IonTabButton tab="bebes" href="/bebes">
          <IonIcon icon={cafeOutline} />
          <IonLabel>Bebes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="comes" href="/comes">
          <IonIcon icon={restaurantOutline} />
          <IonLabel>Comes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="pedido" href="/pedido">
          <IonIcon icon={cartOutline} />
          <IonLabel>Pedido</IonLabel>
          {totalItens > 0 && <IonBadge color="danger">{totalItens}</IonBadge>}
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};

export default TabsPrincipal;
```

O `IonTabs` organiza a navegação por abas.

O `IonTabBar` é a barra inferior.

Cada `IonTabButton` representa uma aba.

O `IonBadge` mostra a quantidade de itens no pedido, mas só aparece quando `totalItens` é maior que zero.

### 6.9 Ajustar o `App.tsx`

Abra:

```text
src/App.tsx
```

Remova os imports de `Redirect`, `Route`, `IonRouterOutlet`, `TelaBebes` e `TelaComes`, porque as rotas agora ficam em `TabsPrincipal`.

Adicione:

```tsx
import { CarrinhoProvider } from './contexts/CarrinhoContext';
import TabsPrincipal from './pages/TabsPrincipal/TabsPrincipal';
```

O componente `App` deve ficar assim:

```tsx
const App: React.FC = () => (
  <IonApp>
    <CarrinhoProvider>
      <IonReactRouter>
        <TabsPrincipal />
      </IonReactRouter>
    </CarrinhoProvider>
  </IonApp>
);
```

O `CarrinhoProvider` fica por fora das tabs para que `TelaBebes`, `TelaComes`, `TelaPedido` e a própria `TabsPrincipal` consigam acessar o mesmo pedido.

### 6.10 Criar o modal de adicionar item

Crie o arquivo:

```text
src/components/ModalAdicionarItem.tsx
```

Comece com:

```tsx
import { useEffect, useState } from 'react';
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
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useCarrinho } from '../contexts/CarrinhoContext';
import type { Produto } from '../types/produto';
```

Agora crie o tipo das props:

```tsx
type ModalAdicionarItemProps = {
  isOpen: boolean;
  produto: Produto | null;
  onClose: () => void;
};
```

O modal precisa saber se está aberto, qual produto foi escolhido e como fechar.

### 6.11 Controlar quantidade e ingredientes no modal

Abaixo do tipo das props, crie a função de preço e o componente:

```tsx
const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const ModalAdicionarItem = ({ isOpen, produto, onClose }: ModalAdicionarItemProps) => {
  const { adicionarItem } = useCarrinho();
  const [quantidade, setQuantidade] = useState(1);
  const [ingredientesRemovidos, setIngredientesRemovidos] = useState<string[]>([]);

  useEffect(() => {
    if (isOpen) {
      setQuantidade(1);
      setIngredientesRemovidos([]);
    }
  }, [isOpen, produto]);

  if (!produto) {
    return null;
  }
```

O modal tem dois estados próprios:

- `quantidade`: começa em `1`.
- `ingredientesRemovidos`: começa como lista vazia.

O `useEffect` limpa esses estados sempre que o modal abre. Assim, um produto não herda a quantidade ou os ingredientes removidos do produto anterior.

### 6.12 Criar os métodos do modal

Ainda dentro do componente, crie:

```tsx
  const alternarIngrediente = (ingrediente: string, marcado: boolean) => {
    if (marcado) {
      setIngredientesRemovidos((removidos) => [...removidos, ingrediente]);
      return;
    }

    setIngredientesRemovidos((removidos) => removidos.filter((item) => item !== ingrediente));
  };

  const diminuirQuantidade = () => {
    setQuantidade((valorAtual) => Math.max(1, valorAtual - 1));
  };

  const aumentarQuantidade = () => {
    setQuantidade((valorAtual) => valorAtual + 1);
  };

  const confirmarAdicao = () => {
    adicionarItem({
      produtoId: produto.id,
      nome: produto.nome,
      preco: produto.preco,
      quantidade,
      ingredientesRemovidos,
    });

    onClose();
  };
```

Aqui ficam os métodos simples da tela.

`alternarIngrediente` adiciona ou remove um ingrediente da lista.

`diminuirQuantidade` nunca deixa a quantidade ficar menor que `1`.

`confirmarAdicao` chama `adicionarItem` do contexto e fecha o modal.

### 6.13 Montar o JSX do modal

Complete o componente:

```tsx
  return (
    <IonModal isOpen={isOpen} onDidDismiss={onClose}>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Adicionar item</IonTitle>
          <IonButtons slot="end">
            <IonButton onClick={onClose}>Fechar</IonButton>
          </IonButtons>
        </IonToolbar>
      </IonHeader>

      <IonContent className="modal-produto-content">
        <div className="modal-produto-container">
          <div className="modal-produto-topo">
            <p className="cardapio-overline">Produto</p>
            <h1 className="cardapio-heading">{produto.nome}</h1>
            <p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
          </div>

          <div className="modal-produto-section">
            <h2 className="modal-produto-title">Quantidade</h2>
            <div className="quantidade-controle">
              <IonButton fill="outline" onClick={diminuirQuantidade}>
                -
              </IonButton>
              <span>{quantidade}</span>
              <IonButton fill="outline" onClick={aumentarQuantidade}>
                +
              </IonButton>
            </div>
          </div>

          <div className="modal-produto-section">
            <h2 className="modal-produto-title">Remover ingredientes</h2>
            <IonList className="ingredientes-list">
              {produto.ingredientes.map((ingrediente) => (
                <IonItem key={ingrediente} lines="none">
                  <IonCheckbox
                    checked={ingredientesRemovidos.includes(ingrediente)}
                    onIonChange={(event) => alternarIngrediente(ingrediente, event.detail.checked)}
                  >
                    <IonLabel>{ingrediente}</IonLabel>
                  </IonCheckbox>
                </IonItem>
              ))}
            </IonList>
          </div>

          <IonButton expand="block" onClick={confirmarAdicao}>
            Adicionar ao pedido
          </IonButton>
        </div>
      </IonContent>
    </IonModal>
  );
};

export default ModalAdicionarItem;
```

O `IonModal` abre por cima da tela atual. Isso é útil aqui porque o usuário continua no cardápio, mas consegue configurar o item antes de adicionar ao pedido.

### 6.14 Usar o modal na `TelaBebes`

Abra:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Adicione o import do `useState`:

```tsx
import { useState } from 'react';
```

No import do Ionic, adicione `IonButton`:

```tsx
IonButton,
```

Depois importe o modal e o tipo:

```tsx
import ModalAdicionarItem from '../../components/ModalAdicionarItem';
import type { Produto } from '../../types/produto';
```

Dentro do componente, antes do `return`, crie:

```tsx
const [produtoSelecionado, setProdutoSelecionado] = useState<Produto | null>(null);
```

Abaixo do preço de cada produto, adicione:

```tsx
<IonButton fill="outline" onClick={() => setProdutoSelecionado(produto)}>
  Adicionar
</IonButton>
```

Antes de fechar o `IonPage`, adicione:

```tsx
<ModalAdicionarItem
  isOpen={!!produtoSelecionado}
  produto={produtoSelecionado}
  onClose={() => setProdutoSelecionado(null)}
/>
```

Quando `produtoSelecionado` tem valor, o modal abre. Quando volta para `null`, o modal fecha.

### 6.15 Usar o modal na `TelaComes`

Repita o mesmo ajuste em:

```text
src/pages/TelaComes/TelaComes.tsx
```

A lógica é igual:

- importar `useState`;
- importar `IonButton`;
- importar `ModalAdicionarItem`;
- importar `Produto` com `import type`;
- criar `produtoSelecionado`;
- adicionar botão `Adicionar`;
- renderizar o modal.

A diferença é que a tela percorre a lista `comidas` em vez da lista `bebidas`.

### 6.16 Adicionar estilos para modal e pedido

Abra:

```text
src/pages/cardapio.css
```

No final do arquivo, adicione:

```css
.modal-produto-content {
  --background: #f7f8fa;
}

.modal-produto-container {
  padding: 20px 14px 18px;
}

.modal-produto-topo {
  margin-bottom: 18px;
}

.modal-produto-section {
  margin-bottom: 18px;
}

.modal-produto-title {
  margin: 0 0 10px;
  font-size: 16px;
  font-weight: 700;
  color: #121821;
}

.quantidade-controle {
  display: flex;
  align-items: center;
  gap: 14px;
}

.quantidade-controle span {
  min-width: 32px;
  text-align: center;
  font-size: 18px;
  font-weight: 700;
}

.ingredientes-list {
  background: transparent;
}

.pedido-vazio {
  padding: 18px;
  border: 1px solid #e8edf2;
  border-radius: 12px;
  background: #ffffff;
  color: #5e6670;
}

.pedido-total {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 14px;
  padding: 16px;
  border-radius: 12px;
  background: #ffffff;
  color: #121821;
}

.pedido-total strong {
  color: #0a6a3f;
}
```

Essas classes reaproveitam a aparência do cardápio e deixam o modal com o mesmo padrão visual.

## 7. Código completo

### `src/types/itemPedido.ts`

```ts
export type ItemPedido = {
  id: string;
  produtoId: number;
  nome: string;
  preco: number;
  quantidade: number;
  ingredientesRemovidos: string[];
};
```

### `src/contexts/CarrinhoContext.tsx`

```tsx
import { createContext, useContext, useState } from 'react';
import type { ReactNode } from 'react';
import type { ItemPedido } from '../types/itemPedido';

type CarrinhoContextValue = {
  itens: ItemPedido[];
  totalItens: number;
  totalPedido: number;
  adicionarItem: (item: Omit<ItemPedido, 'id'>) => void;
  removerItem: (id: string) => void;
  limparCarrinho: () => void;
};

const CarrinhoContext = createContext<CarrinhoContextValue | undefined>(undefined);

export const CarrinhoProvider = ({ children }: { children: ReactNode }) => {
  const [itens, setItens] = useState<ItemPedido[]>([]);

  const totalItens = itens.reduce((total, item) => total + item.quantidade, 0);
  const totalPedido = itens.reduce((total, item) => total + item.preco * item.quantidade, 0);

  const adicionarItem = (item: Omit<ItemPedido, 'id'>) => {
    const novoItem: ItemPedido = {
      ...item,
      id: crypto.randomUUID(),
    };

    setItens((itensAtuais) => [...itensAtuais, novoItem]);
  };

  const removerItem = (id: string) => {
    setItens((itensAtuais) => itensAtuais.filter((item) => item.id !== id));
  };

  const limparCarrinho = () => {
    setItens([]);
  };

  return (
    <CarrinhoContext.Provider
      value={{
        itens,
        totalItens,
        totalPedido,
        adicionarItem,
        removerItem,
        limparCarrinho,
      }}
    >
      {children}
    </CarrinhoContext.Provider>
  );
};

export const useCarrinho = () => {
  const context = useContext(CarrinhoContext);

  if (!context) {
    throw new Error('useCarrinho deve ser usado dentro de CarrinhoProvider');
  }

  return context;
};
```

### `src/pages/TabsPrincipal/TabsPrincipal.tsx`

```tsx
import { Redirect, Route } from 'react-router-dom';
import {
  IonBadge,
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
} from '@ionic/react';
import { cafeOutline, cartOutline, restaurantOutline } from 'ionicons/icons';
import { useCarrinho } from '../../contexts/CarrinhoContext';
import TelaBebes from '../TelaBebes/TelaBebes';
import TelaComes from '../TelaComes/TelaComes';
import TelaPedido from '../TelaPedido/TelaPedido';

const TabsPrincipal = () => {
  const { totalItens } = useCarrinho();

  return (
    <IonTabs>
      <IonRouterOutlet>
        <Route exact path="/bebes" component={TelaBebes} />
        <Route exact path="/comes" component={TelaComes} />
        <Route exact path="/pedido" component={TelaPedido} />
        <Route exact path="/">
          <Redirect to="/bebes" />
        </Route>
      </IonRouterOutlet>

      <IonTabBar slot="bottom">
        <IonTabButton tab="bebes" href="/bebes">
          <IonIcon icon={cafeOutline} />
          <IonLabel>Bebes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="comes" href="/comes">
          <IonIcon icon={restaurantOutline} />
          <IonLabel>Comes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="pedido" href="/pedido">
          <IonIcon icon={cartOutline} />
          <IonLabel>Pedido</IonLabel>
          {totalItens > 0 && <IonBadge color="danger">{totalItens}</IonBadge>}
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};

export default TabsPrincipal;
```

### `src/App.tsx`

```tsx
import { IonApp, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { CarrinhoProvider } from './contexts/CarrinhoContext';
import TabsPrincipal from './pages/TabsPrincipal/TabsPrincipal';

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

## 8. Erros comuns

### 8.1 Guardar carrinho em variável comum

Evite:

```ts
const carrinho = [];
```

Esse array muda, mas o React não sabe que precisa redesenhar a tela.

Use estado:

```tsx
const [itens, setItens] = useState<ItemPedido[]>([]);
```

### 8.2 Alterar o array diretamente

Evite:

```tsx
itens.push(novoItem);
```

Prefira:

```tsx
setItens((itensAtuais) => [...itensAtuais, novoItem]);
```

Assim o React recebe uma nova lista e atualiza a interface.

### 8.3 Esquecer o `CarrinhoProvider`

Se uma tela chama `useCarrinho`, ela precisa estar dentro do provider.

Correto:

```tsx
<CarrinhoProvider>
  <IonReactRouter>
    <TabsPrincipal />
  </IonReactRouter>
</CarrinhoProvider>
```

### 8.4 Esquecer de resetar o modal

Quando o modal abre para outro produto, a quantidade e os ingredientes removidos precisam voltar ao valor inicial.

Por isso usamos:

```tsx
useEffect(() => {
  if (isOpen) {
    setQuantidade(1);
    setIngredientesRemovidos([]);
  }
}, [isOpen, produto]);
```

### 8.5 Confundir `Produto` com `ItemPedido`

`Produto` é o item do cardápio.

`ItemPedido` é o produto escolhido pelo usuário, com quantidade e ingredientes removidos.

## 9. Resumo

Nesta aula, o app ganhou navegação por tabs e começou a montar um pedido.

O ponto principal é entender que:

- `IonTabs` cria a navegação por abas.
- `IonTabBar` é a barra inferior.
- `IonTabButton` representa cada aba.
- `IonBadge` mostra a quantidade de itens no pedido.
- `Produto` representa o que existe no cardápio.
- `ItemPedido` representa o que o cliente escolheu.
- `CarrinhoContext` compartilha o pedido entre as telas.
- `useState` guarda a lista de itens.
- Métodos como `adicionarItem`, `removerItem` e `limparCarrinho` deixam o código mais fácil de entender.
- `IonModal` permite configurar o item sem sair da tela do cardápio.

Com isso, o app deixa de ser apenas uma lista de produtos e começa a funcionar como uma comanda.

