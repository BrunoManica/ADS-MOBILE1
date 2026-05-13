# Aula 4: Carrinho e finalizacao do pedido

## Indice

- [1. Objetivo da aula](#1-objetivo-da-aula)
- [2. Resultado final](#2-resultado-final)
- [3. Contexto](#3-contexto)
- [4. Explicacao conceitual](#4-explicacao-conceitual)
- [5. Setup inicial](#5-setup-inicial)
- [6. Passo a passo](#6-passo-a-passo)
- [7. Codigo completo](#7-codigo-completo)
- [8. Erros comuns](#8-erros-comuns)
- [9. Resumo](#9-resumo)
- [10. Proximo passo](#10-proximo-passo)

## 1. Objetivo da aula

Nesta aula, voce vai transformar a tela `Pedido` em uma tela funcional.

Na Aula 3, criamos apenas a navegacao por tabs e uma tela inicial de pedido. Agora vamos criar o carrinho de verdade: o usuario vai escolher produtos nas telas `Bebes` e `Comes`, definir quantidade, remover ingredientes se quiser e finalizar o pedido.

Ao final da aula, o app tera:

- Um contexto para guardar o pedido em andamento.
- Um tipo para representar o item do pedido.
- Um tipo para representar o pedido finalizado.
- Um botao `Adicionar` nas telas `Bebes` e `Comes`.
- Um modal para escolher quantidade e ingredientes removidos.
- Um contador na tab `Pedido`.
- Uma tela `Pedido` listando os itens adicionados.
- Controles para aumentar, diminuir e remover itens.
- Um campo de observacao geral.
- Um botao para finalizar o pedido.
- Um objeto final de pedido sendo montado e exibido no console.

## 2. Resultado final

Nas telas de produtos, cada item passa a ter um botao:

```text
Refrigerante
Lata 350ml
R$ 6,00

[Adicionar]
```

Ao tocar em `Adicionar`, o app abre um modal:

```text
Adicionar item

Produto
Refrigerante
R$ 6,00

Quantidade
[-] 1 [+]

Remover ingredientes
[ ] Lata 350ml

[Adicionar ao pedido]
```

Depois que o item entra no pedido, a tab `Pedido` mostra um contador:

```text
[ Bebes ]   [ Comes ]   [ Pedido 1 ]
```

A tela `Pedido` passa a mostrar os itens:

```text
Pedido
Finalizacao de pedido
1 item no pedido

1x Refrigerante
Remover: nenhum ingrediente
Unitario: R$ 6,00
Subtotal: R$ 6,00

[-] [+] [Remover]

Observacao geral
[Mesa 4]

Total: R$ 6,00
[Finalizar pedido]
```

Ao finalizar, por enquanto, vamos mostrar o pedido no console. Na aula de API, esse objeto podera ser enviado para o backend.

## 3. Contexto

Um produto do cardapio nao e a mesma coisa que um item do pedido.

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
Subtotal: R$ 56,00
```

Por isso, nesta aula vamos criar um tipo separado para `ItemPedido`.

Tambem vamos criar um `Context`, porque varias partes do app precisam acessar o mesmo pedido:

- `TelaBebes` adiciona bebidas.
- `TelaComes` adiciona comidas.
- `TabsPrincipal` mostra o contador.
- `TelaPedido` lista, altera e finaliza o pedido.

Sem `Context`, seria necessario passar dados de uma tela para outra manualmente. Isso deixaria o codigo mais confuso para este momento.

## 4. Explicacao conceitual

O fluxo da aula fica assim:

```text
CarrinhoContext
  guarda os itens
  calcula totais
  oferece funcoes para alterar o pedido

TelaBebes e TelaComes
  selecionam um produto
  abrem o modal

ModalAdicionarItem
  escolhe quantidade
  escolhe ingredientes removidos
  chama adicionarItem()

TabsPrincipal
  le totalItens
  mostra badge na tab Pedido

TelaPedido
  le itens
  altera quantidade
  remove item
  escreve observacao
  monta PedidoFinalizado
```

Vamos usar `Context` com `useState`.

Nao vamos usar reducer nesta aula. Reducer e normal em React, mas aqui ele adicionaria uma camada extra antes dos alunos entenderem bem o fluxo. Para este momento, funcoes simples deixam o codigo mais direto.

Por baixo dos panos, o `CarrinhoProvider` fica envolvendo as telas do app. Ele guarda o estado do carrinho em um `useState`. Quando alguma tela chama `adicionarItem`, `alterarQuantidade`, `removerItem` ou `limparCarrinho`, o estado muda. Depois disso, o React redesenha as partes da tela que usam `useCarrinho()`.

Isso e o que faz o contador da tab e a tela `Pedido` atualizarem automaticamente.

## 5. Setup inicial

Esta aula continua a partir da Aula 3.

Arquivos que serao criados:

```text
src/types/itemPedido.ts
src/types/pedido.ts
src/contexts/CarrinhoContext.tsx
src/components/ModalAdicionarItem.tsx
```

Arquivos que serao alterados:

```text
src/App.tsx
src/pages/TabsPrincipal/TabsPrincipal.tsx
src/pages/TelaBebes/TelaBebes.tsx
src/pages/TelaComes/TelaComes.tsx
src/pages/TelaPedido/TelaPedido.tsx
src/pages/cardapio.css
```

Para rodar o projeto:

```bash
cd /caminho/para/o/projeto
nvm use 24
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

O campo `id` identifica aquele item dentro do pedido. Ele sera gerado com `crypto.randomUUID()`.

O campo `produtoId` guarda o `id` original do produto.

O campo `ingredientesRemovidos` guarda somente o que o cliente nao quer naquele item.

### 6.2 Criar o tipo `PedidoFinalizado`

Crie o arquivo:

```text
src/types/pedido.ts
```

Escreva:

```ts
import type { ItemPedido } from './itemPedido';

export type PedidoFinalizado = {
  itens: ItemPedido[];
  observacao: string;
  total: number;
  criadoEm: string;
};
```

Esse tipo representa o pedido pronto para ser enviado no futuro.

Nesta aula, ainda nao vamos enviar para API. Vamos apenas montar o objeto e mostrar no console.

### 6.3 Criar o contexto do carrinho

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

Agora defina o tipo do contexto:

```tsx
type CarrinhoContextValue = {
  itens: ItemPedido[];
  totalItens: number;
  totalPedido: number;
  adicionarItem: (item: Omit<ItemPedido, 'id'>) => void;
  alterarQuantidade: (id: string, quantidade: number) => void;
  removerItem: (id: string) => void;
  limparCarrinho: () => void;
};
```

Esse tipo diz o que qualquer tela pode usar.

### 6.4 Criar o `CarrinhoProvider`

Abaixo do tipo, escreva:

```tsx
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

  const alterarQuantidade = (id: string, quantidade: number) => {
    setItens((itensAtuais) =>
      itensAtuais.map((item) =>
        item.id === id
          ? {
              ...item,
              quantidade: Math.max(1, quantidade),
            }
          : item,
      ),
    );
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
        alterarQuantidade,
        removerItem,
        limparCarrinho,
      }}
    >
      {children}
    </CarrinhoContext.Provider>
  );
};
```

Os totais sao calculados com `.reduce()`.

Nao precisamos guardar `totalItens` e `totalPedido` em outro `useState`, porque eles nascem da lista de itens.

### 6.5 Criar o hook `useCarrinho`

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

Esse hook evita repetir `useContext(CarrinhoContext)` em varias telas.

Agora as telas podem usar:

```tsx
const { itens, adicionarItem } = useCarrinho();
```

### 6.6 Colocar o provider no `App.tsx`

Abra:

```text
src/App.tsx
```

Importe o provider:

```tsx
import { CarrinhoProvider } from './contexts/CarrinhoContext';
```

Depois envolva o router:

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

O provider fica por fora das tabs para que todas as telas usem o mesmo pedido.

### 6.7 Mostrar contador na tab `Pedido`

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

No import do Ionic, adicione `IonBadge`:

```tsx
IonBadge,
```

Importe o hook:

```tsx
import { useCarrinho } from '../../contexts/CarrinhoContext';
```

Dentro do componente, leia `totalItens`:

```tsx
const TabsPrincipal = () => {
  const { totalItens } = useCarrinho();

  return (
    // ...
  );
};
```

Na tab `Pedido`, adicione o badge:

```tsx
<IonTabButton tab="pedido" href="/pedido">
  <IonIcon icon={cartOutline} />
  <IonLabel>Pedido</IonLabel>
  {totalItens > 0 && <IonBadge color="danger">{totalItens}</IonBadge>}
</IonTabButton>
```

O badge so aparece quando existe item no pedido.

### 6.8 Criar o modal de adicionar item

Crie o arquivo:

```text
src/components/ModalAdicionarItem.tsx
```

Escreva:

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

type ModalAdicionarItemProps = {
  isOpen: boolean;
  produto: Produto | null;
  onClose: () => void;
};

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

O `IonModal` e controlado pela prop `isOpen`. Quando ele fecha, `onDidDismiss` chama `onClose`.

### 6.9 Usar o modal na `TelaBebes`

Abra:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

O inicio do arquivo deve ficar com estes imports:

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
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import ModalAdicionarItem from '../../components/ModalAdicionarItem';
import { listarBebidas } from '../../services/produtoService';
import type { Produto } from '../../types/produto';
import '../cardapio.css';
```

Assim o `IonButton` entra no mesmo import dos outros componentes Ionic. Evite criar um segundo import separado de `@ionic/react`.

Dentro do componente, crie o estado:

```tsx
const [produtoSelecionado, setProdutoSelecionado] = useState<Produto | null>(null);
```

Abaixo do preco de cada produto, adicione:

```tsx
<IonButton fill="solid" onClick={() => setProdutoSelecionado(produto)}>
  Adicionar
</IonButton>
```

Antes de fechar o `IonPage`, renderize o modal:

```tsx
<ModalAdicionarItem
  isOpen={!!produtoSelecionado}
  produto={produtoSelecionado}
  onClose={() => setProdutoSelecionado(null)}
/>
```

### 6.10 Usar o modal na `TelaComes`

Repita a mesma ideia em:

```text
src/pages/TelaComes/TelaComes.tsx
```

A tela tambem precisa:

- importar `useState`;
- importar `IonButton`;
- importar `ModalAdicionarItem`;
- importar `Produto` com `import type`;
- criar `produtoSelecionado`;
- adicionar o botao `Adicionar`;
- renderizar o modal.

O inicio do arquivo fica quase igual ao da tela de bebidas. A diferenca e que aqui usamos `listarComidas`:

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
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import ModalAdicionarItem from '../../components/ModalAdicionarItem';
import { listarComidas } from '../../services/produtoService';
import type { Produto } from '../../types/produto';
import '../cardapio.css';
```

### 6.11 Evoluir a tela `Pedido`

Abra:

```text
src/pages/TelaPedido/TelaPedido.tsx
```

Substitua o conteudo pelo codigo abaixo:

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
import type { PedidoFinalizado } from '../../types/pedido';
import '../cardapio.css';

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaPedido = () => {
  const { itens, totalItens, totalPedido, alterarQuantidade, removerItem, limparCarrinho } =
    useCarrinho();
  const [observacao, setObservacao] = useState('');

  const finalizarPedido = () => {
    const pedido: PedidoFinalizado = {
      itens,
      observacao,
      total: totalPedido,
      criadoEm: new Date().toISOString(),
    };

    console.log('Pedido finalizado:', pedido);
  };

  const limparPedido = () => {
    const confirmou = window.confirm('Deseja limpar o pedido?');

    if (confirmou) {
      limparCarrinho();
      setObservacao('');
    }
  };

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
            <p className="cardapio-meta">{totalItens} itens no pedido</p>
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

                      <p className="cardapio-descricao">
                        Unitario: {formatarPreco(item.preco)}
                      </p>

                      <p className="cardapio-preco">
                        Subtotal: {formatarPreco(item.preco * item.quantidade)}
                      </p>

                      <div className="pedido-acoes">
                        <IonButton
                          fill="outline"
                          onClick={() => alterarQuantidade(item.id, item.quantidade - 1)}
                        >
                          -
                        </IonButton>
                        <IonButton
                          fill="outline"
                          onClick={() => alterarQuantidade(item.id, item.quantidade + 1)}
                        >
                          +
                        </IonButton>
                        <IonButton fill="clear" color="danger" onClick={() => removerItem(item.id)}>
                          Remover
                        </IonButton>
                      </div>
                    </IonLabel>
                  </IonItem>
                ))}
              </IonList>

              <div className="pedido-observacao">
                <IonTextarea
                  label="Observacao geral"
                  labelPlacement="stacked"
                  placeholder="Ex: Mesa 4, sem talheres"
                  value={observacao}
                  onIonInput={(event) => setObservacao(event.detail.value ?? '')}
                />
              </div>

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

export default TelaPedido;
```

Essa tela ainda nao envia nada para API. Ela apenas prepara o objeto final.

Para ver o resultado, abra o DevTools do navegador, entre na aba `Console` e clique em `Finalizar pedido`. O objeto montado vai aparecer no console.

### 6.12 Adicionar estilos

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

.pedido-acoes {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
  flex-wrap: wrap;
}

.pedido-observacao {
  margin-top: 14px;
  padding: 12px;
  border-radius: 12px;
  background: #ffffff;
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

Esses estilos reaproveitam a aparencia do cardapio e organizam o modal e a tela do pedido.

## 7. Codigo completo

Use esta secao para conferir se os arquivos ficaram iguais ao resultado final da aula.

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

### `src/types/pedido.ts`

```ts
import type { ItemPedido } from './itemPedido';

export type PedidoFinalizado = {
  itens: ItemPedido[];
  observacao: string;
  total: number;
  criadoEm: string;
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
  alterarQuantidade: (id: string, quantidade: number) => void;
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

  const alterarQuantidade = (id: string, quantidade: number) => {
    setItens((itensAtuais) =>
      itensAtuais.map((item) =>
        item.id === id
          ? {
              ...item,
              quantidade: Math.max(1, quantidade),
            }
          : item,
      ),
    );
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
        alterarQuantidade,
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

### `src/components/ModalAdicionarItem.tsx`

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

type ModalAdicionarItemProps = {
  isOpen: boolean;
  produto: Produto | null;
  onClose: () => void;
};

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

### `src/pages/TelaBebes/TelaBebes.tsx`

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
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import ModalAdicionarItem from '../../components/ModalAdicionarItem';
import { listarBebidas } from '../../services/produtoService';
import type { Produto } from '../../types/produto';
import '../cardapio.css';

const bebidas = listarBebidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaBebes = () => {
  const [produtoSelecionado, setProdutoSelecionado] = useState<Produto | null>(null);

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
                  <p className="cardapio-descricao">{produto.ingredientes.join(', ')}</p>
                  <p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
                  <IonButton fill="solid" onClick={() => setProdutoSelecionado(produto)}>
                    Adicionar
                  </IonButton>
                </IonLabel>
              </IonItem>
            ))}
          </IonList>
        </div>
      </IonContent>

      <ModalAdicionarItem
        isOpen={!!produtoSelecionado}
        produto={produtoSelecionado}
        onClose={() => setProdutoSelecionado(null)}
      />
    </IonPage>
  );
};

export default TelaBebes;
```

### `src/pages/TelaComes/TelaComes.tsx`

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
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import ModalAdicionarItem from '../../components/ModalAdicionarItem';
import { listarComidas } from '../../services/produtoService';
import type { Produto } from '../../types/produto';
import '../cardapio.css';

const comidas = listarComidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaComes = () => {
  const [produtoSelecionado, setProdutoSelecionado] = useState<Produto | null>(null);

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
                  <p className="cardapio-descricao">{produto.ingredientes.join(', ')}</p>
                  <p className="cardapio-preco">{formatarPreco(produto.preco)}</p>
                  <IonButton fill="solid" onClick={() => setProdutoSelecionado(produto)}>
                    Adicionar
                  </IonButton>
                </IonLabel>
              </IonItem>
            ))}
          </IonList>
        </div>
      </IonContent>

      <ModalAdicionarItem
        isOpen={!!produtoSelecionado}
        produto={produtoSelecionado}
        onClose={() => setProdutoSelecionado(null)}
      />
    </IonPage>
  );
};

export default TelaComes;
```

### `src/pages/TelaPedido/TelaPedido.tsx`

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
import type { PedidoFinalizado } from '../../types/pedido';
import '../cardapio.css';

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaPedido = () => {
  const { itens, totalItens, totalPedido, alterarQuantidade, removerItem, limparCarrinho } =
    useCarrinho();
  const [observacao, setObservacao] = useState('');

  const finalizarPedido = () => {
    const pedido: PedidoFinalizado = {
      itens,
      observacao,
      total: totalPedido,
      criadoEm: new Date().toISOString(),
    };

    console.log('Pedido finalizado:', pedido);
  };

  const limparPedido = () => {
    const confirmou = window.confirm('Deseja limpar o pedido?');

    if (confirmou) {
      limparCarrinho();
      setObservacao('');
    }
  };

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
            <p className="cardapio-meta">{totalItens} itens no pedido</p>
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

                      <p className="cardapio-descricao">
                        Unitario: {formatarPreco(item.preco)}
                      </p>

                      <p className="cardapio-preco">
                        Subtotal: {formatarPreco(item.preco * item.quantidade)}
                      </p>

                      <div className="pedido-acoes">
                        <IonButton
                          fill="outline"
                          onClick={() => alterarQuantidade(item.id, item.quantidade - 1)}
                        >
                          -
                        </IonButton>
                        <IonButton
                          fill="outline"
                          onClick={() => alterarQuantidade(item.id, item.quantidade + 1)}
                        >
                          +
                        </IonButton>
                        <IonButton fill="clear" color="danger" onClick={() => removerItem(item.id)}>
                          Remover
                        </IonButton>
                      </div>
                    </IonLabel>
                  </IonItem>
                ))}
              </IonList>

              <div className="pedido-observacao">
                <IonTextarea
                  label="Observacao geral"
                  labelPlacement="stacked"
                  placeholder="Ex: Mesa 4, sem talheres"
                  value={observacao}
                  onIonInput={(event) => setObservacao(event.detail.value ?? '')}
                />
              </div>

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

export default TelaPedido;
```

### `src/pages/cardapio.css`

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

.pedido-acoes {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
  flex-wrap: wrap;
}

.pedido-observacao {
  margin-top: 14px;
  padding: 12px;
  border-radius: 12px;
  background: #ffffff;
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

## 8. Erros comuns

### 8.1 Usar variavel comum para guardar o carrinho

Evite:

```ts
const carrinho = [];
```

O React nao sabe que precisa redesenhar a tela quando uma variavel comum muda.

Use `useState`.

### 8.2 Alterar array diretamente

Evite:

```tsx
itens.push(novoItem);
```

Prefira:

```tsx
setItens((itensAtuais) => [...itensAtuais, novoItem]);
```

Assim o React recebe uma nova lista e atualiza a interface.

### 8.3 Esquecer o provider no `App.tsx`

Se voce usar `useCarrinho()` sem envolver o app com `CarrinhoProvider`, o app vai gerar erro.

O provider precisa ficar acima das telas que usam o carrinho.

### 8.4 Guardar total em estado separado

Evite criar outro `useState` para `totalPedido`.

O total deve ser calculado a partir dos itens:

```tsx
const totalPedido = itens.reduce((total, item) => total + item.preco * item.quantidade, 0);
```

Isso evita inconsistencia entre lista e total.

### 8.5 Finalizar pedido vazio

Nesta aula, o botao `Finalizar pedido` so aparece quando existem itens.

Esse e um jeito simples de impedir pedido vazio sem criar validacao extra.

## 9. Resumo

Nesta aula, voce criou o primeiro fluxo completo de pedido do app.

Voce aprendeu a:

- criar `Context` para compartilhar estado;
- criar um hook `useCarrinho`;
- adicionar itens ao pedido;
- abrir modal com `IonModal`;
- controlar quantidade;
- remover ingredientes;
- listar itens na tela `Pedido`;
- calcular subtotal e total;
- criar uma observacao geral;
- montar um objeto final de pedido.

## 10. Proximo passo

Na proxima aula, o app pode evoluir para cadastro e edicao de produtos.

Isso prepara o projeto para um usuario administrador gerenciar o cardapio, enquanto o cliente continua usando as telas de `Bebes`, `Comes` e `Pedido`.
