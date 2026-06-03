# Aula 5: Cadastrando um novo produto

## 1. Objetivo da aula

Nesta aula, você vai criar um formulário simples para cadastrar um novo produto no cardápio.

Nas aulas anteriores, os produtos já apareciam na tela `Comidas`. Agora vamos permitir que um novo produto seja adicionado pela interface do app.

Ao final da aula, o app terá:

- Um botão `Adicionar NOVO produto` na tela `Comidas`.
- Uma tela simples de cadastro.
- Campos para nome, ingredientes, preço e código de barras.
- Uma validação básica.
- Um produto novo aparecendo na lista de produtos da tela `Comidas`.

Vamos fazer só cadastro de produto.

Edição fica para a Aula 6.

Login e controle de permissões ficam para a Aula 7.

Comunicação com API fica para a Aula 8.

## 2. Resultado final

Na tela `Comidas`, o aluno verá:

```text
Comidas

[Adicionar NOVO produto]

Mc feliz
Pao, Carne, Queijo
R$ 36,98

[Adicionar]
```

Ao tocar em `Adicionar NOVO produto`, o app abre uma tela assim:

```text
Novo Produto

Nome
[________________]

Ingredientes
[________________]

Preco
[________________]

Cod Barras
[________________]

[salvar produto]
```

Depois de salvar, o app volta para `Comidas` e o produto aparece na lista.

## 3. Contexto

Em um app de comanda, o cardápio precisa crescer.

Hoje os produtos estão escritos direto no código. Isso não é o ideal para um sistema real, mas é perfeito para aprender.

Antes de usar API, banco de dados ou login, precisamos entender uma coisa mais simples:

```text
campo digitado
  vira estado no React
  vira um objeto Produto
  entra na lista do service
  aparece na tela
```

Essa é a base de muitos formulários.

## 4. Explicação conceitual

Um formulário em React precisa guardar o que o usuário digita.

Para isso, usamos `useState`.

Exemplo:

```tsx
const [nome, setNome] = useState('');
```

Aqui temos duas partes:

- `nome`: guarda o texto atual.
- `setNome`: altera esse texto.

Quando o aluno digita no campo, chamamos `setNome`.

```tsx
<IonInput
  value={nome}
  onIonInput={(event) => setNome(event.detail.value ?? '')}
></IonInput>
```

O que está acontecendo:

- `value={nome}` mostra no campo o valor que está no estado.
- `onIonInput` escuta a digitação.
- `event.detail.value` pega o valor digitado no componente do Ionic.
- `setNome` guarda esse valor no React.

Vamos usar essa mesma ideia para todos os campos.

## 5. Setup inicial

Esta aula continua a partir da Aula 4.

Arquivos que vamos mexer:

```text
src/services/produtoService.ts
src/types/Produto.ts
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
src/pages/TelaComes/TelaComes.tsx
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Para rodar:

```bash
cd /caminho/para/react-burguer
```

Depois:

```bash
npm run dev
```

## 6. Passo a passo

### 6.1 Conferir o tipo `Produto`

Abra:

```text
src/types/Produto.ts
```

Confira se o tipo `Produto` está assim:

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
```

Esse tipo descreve os dados de um produto.

Nesta aula, o produto terá:

- `id`: identificador do produto.
- `codBarras`: código de barras.
- `ingredientes`: lista de ingredientes.
- `ativo`: se o produto está ativo.
- `nome`: nome do produto.
- `preco`: preço como número.
- `imagem`: campo opcional.

### 6.2 Começar pelo service

Abra:

```text
src/services/produtoService.ts
```

O service é onde a lista de produtos fica guardada.

Vamos deixar esse arquivo com duas funções simples:

- `buscarTodosOsProdutos`: mostra os produtos na tela `Comidas`.
- `cadastrarProduto`: adiciona um produto novo.

Comece com a lista e a função de busca:

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

Por enquanto, a lista é um array simples.

O aluno já viu array nas aulas anteriores. Então vamos ficar nesse terreno conhecido.

### 6.3 Criar a função `cadastrarProduto`

Ainda no `produtoService.ts`, adicione:

```ts
export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  let id = Math.max(...comida.map(c => Number(c.id)));
  id = id + 1;

  const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
  const novoProduto = {
    nome,
    ingredientes: ingredientes.split(','),
    preco: precoTratado,
    codBarras,
    id: id.toString(),
    ativo: true,
  };

  comida.push(novoProduto);
};
```

Aqui estamos fazendo o caminho mais simples:

1. Recebemos os dados do formulário.
2. Geramos um `id` novo para o produto.
3. Tratamos o preço.
4. Transformamos ingredientes em lista.
5. Criamos um objeto de produto.
6. Colocamos esse produto dentro da lista `comida` com `push`.

Essa linha trata o preço:

```ts
const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
```

Ela permite que o aluno digite valores como:

```text
R$12,50
```

E transforma em número:

```text
12.50
```

Essa linha transforma os ingredientes em lista:

```ts
ingredientes: ingredientes.split(','),
```

Se o aluno digitar:

```text
pao,carne,queijo
```

O app guarda:

```ts
['pao', 'carne', 'queijo']
```

### 6.4 Criar a pasta da tela de formulário

Dentro de `src/pages`, crie:

```text
TelaProdutoForm
```

Dentro dela, crie:

```text
TelaProdutoForm.tsx
```

O caminho completo fica:

```text
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
```

### 6.5 Criar a tela com os imports

Comece o arquivo `TelaProdutoForm.tsx` com:

```tsx
import { IonButton, IonContent, IonHeader, IonInput, IonItem, IonList, IonPage, IonTitle, IonToolbar } from '@ionic/react';
import { useState } from 'react';
import '../Cardapio.css';
import { useHistory } from 'react-router';

import { cadastrarProduto } from '../../services/produtoService';
```

Esses imports trazem:

- os componentes visuais do Ionic;
- o `useState` do React;
- o CSS do cardápio;
- o `useHistory` para voltar para a tela `Comidas`;
- a função `cadastrarProduto` do service.

### 6.6 Criar os estados dos campos

Depois dos imports, crie o componente:

```tsx
export const TelaProdutoForm = () => {
  const history = useHistory();

  const [codBarras, setCodBarras] = useState('');
  const [ingredientes, setIngredientes] = useState('');
  const [nome, setNome] = useState('');
  const [preco, setPreco] = useState('');
  const [erro, setErro] = useState('');

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Novo Produto</IonTitle>
        </IonToolbar>
      </IonHeader>
    </IonPage>
  );
};
```

Cada campo tem um estado.

`preco` começa como texto porque tudo que vem de um campo de formulário chega como texto.

O service transforma esse texto em número.

### 6.7 Criar a função de salvar

Dentro do componente, antes do `return`, adicione:

```tsx
const salvarProduto = () => {
  setErro('');
  if (codBarras == '' || codBarras == null) {
    setErro('O cod de barras esta com valor invalido');
    return;
  }

  if (ingredientes == '' || ingredientes == null) {
    setErro('Os ingredientes estão com valor invalido');
    return;
  }

  if (nome == '' || nome == null) {
    setErro('O nome esta com valor invalido');
    return;
  }

  if (preco == '' || preco == null) {
    setErro('O preco esta com valor invalido');
    return;
  }

  setCodBarras('');
  setIngredientes('');
  setNome('');
  setPreco('');
  cadastrarProduto(nome, ingredientes, preco, codBarras);
  history.push('/comes');
};
```

Aqui a função faz o fluxo completo:

- limpa erro antigo;
- confere código de barras;
- confere ingredientes;
- confere nome;
- confere preço;
- limpa os campos;
- cadastra o produto;
- volta para `Comidas`.

Essa linha faz a navegação:

```ts
history.push('/comes');
```

### 6.8 Montar o formulário

Agora complete o `return` com o formulário:

```tsx
return (
  <IonPage>
    <IonHeader>
      <IonToolbar>
        <IonTitle>
          Novo Produto
        </IonTitle>
      </IonToolbar>
    </IonHeader>
    <IonContent className="cardapio-content">
      <div className="cardapio-content">
        {erro !== '' && <p className="erro">{erro}</p>}

        <IonList>
          <IonItem>
            <IonInput
              label="Nome"
              labelPlacement="stacked"
              value={nome}
              placeholder="Ex: xis salada"
              onIonInput={(event) => setNome(event.detail.value ?? '')}
            >
            </IonInput>
          </IonItem>

          <IonItem>
            <IonInput
              label="Ingredientes"
              labelPlacement="stacked"
              value={ingredientes}
              placeholder="Ex:pao,carne,queijo"
              helperText="separe os ingredientes por virgula"
              onIonInput={(event) => setIngredientes(event.detail.value ?? '')}
            >
            </IonInput>
          </IonItem>

          <IonItem>
            <IonInput
              label="Preco"
              labelPlacement="stacked"
              value={preco}
              placeholder="R$12,50"
              inputMode="decimal"
              onIonInput={(event) => setPreco(event.detail.value ?? '')}
            >
            </IonInput>
          </IonItem>

          <IonItem>
            <IonInput
              label="Cod Barras"
              labelPlacement="stacked"
              value={codBarras}
              placeholder="Ex: 0000000000000"
              onIonInput={(event) => setCodBarras(event.detail.value ?? '')}
            >
            </IonInput>
          </IonItem>
        </IonList>

        <IonButton expand="block" onClick={salvarProduto}>
          salvar produto
        </IonButton>
      </div>
    </IonContent>
  </IonPage>
);
```

Esse formulário usa só `IonInput`.

Nada de componente diferente agora. A ideia é repetir o mesmo padrão até ficar claro.

### 6.9 Registrar a rota da tela

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Importe a tela:

```tsx
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
```

Dentro de `IonRouterOutlet`, adicione:

```tsx
<Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
```

A rota fica simples:

```text
/produto-novo
```

### 6.10 Adicionar o botão na tela `Comidas`

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Acima da lista de produtos, adicione:

```tsx
<IonButton routerLink="/produto-novo">Adicionar NOVO produto</IonButton>
```

Agora a tela tem um caminho para abrir o formulário.

### 6.11 Recarregar a lista quando voltar

Ainda em `TelaComes`, vamos usar um recurso do Ionic:

Antes, a tela buscava os produtos direto assim:

```tsx
const comidas = buscarTodosOsProdutos();
```

Agora vamos trocar por um estado:

```tsx
const [produtos, setProdutos] = useState<Produto[]>([]);
```

Esse estado vai guardar a lista que aparece na tela.

Depois, carregamos os produtos quando a tela aparecer:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

O nome parece grande, mas a ideia é simples:

```text
quando a tela Comidas for aparecer
  buscar a lista de produtos de novo
```

Isso faz o produto recém-cadastrado aparecer quando voltamos do formulário.

Na lista, troque:

```tsx
{comidas.map((comida) => (
```

por:

```tsx
{produtos.map((comida) => (
```

### 6.12 Conferir os imports da tela `Comidas`

No começo de `TelaComes.tsx`, confira se existem estes imports:

```tsx
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
  useIonViewWillEnter,
} from '@ionic/react';
import { buscarTodosOsProdutos } from '../../services/produtoService';
import { Produto } from '../../types/Produto';
```

O `IonButton` será usado para abrir a tela de cadastro.

O `useIonViewWillEnter` será usado para recarregar a lista.

O tipo `Produto` será usado no estado:

```tsx
const [produtos, setProdutos] = useState<Produto[]>([]);
```

## 7. Código completo

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

export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  let id = Math.max(...comida.map(c => Number(c.id)));
  id = id + 1;

  const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
  const novoProduto = {
    nome,
    ingredientes: ingredientes.split(','),
    preco: precoTratado,
    codBarras,
    id: id.toString(),
    ativo: true,
  };

  comida.push(novoProduto);
};
```

### `src/pages/TelaProdutoForm/TelaProdutoForm.tsx`

```tsx
import { IonButton, IonContent, IonHeader, IonInput, IonItem, IonList, IonPage, IonTitle, IonToolbar } from '@ionic/react';
import { useState } from 'react';
import '../Cardapio.css';
import { useHistory } from 'react-router';

import { cadastrarProduto } from '../../services/produtoService';

export const TelaProdutoForm = () => {
  const history = useHistory();

  const [codBarras, setCodBarras] = useState('');
  const [ingredientes, setIngredientes] = useState('');
  const [nome, setNome] = useState('');
  const [preco, setPreco] = useState('');
  const [erro, setErro] = useState('');

  const salvarProduto = () => {
    setErro('');
    if (codBarras == '' || codBarras == null) {
      setErro('O cod de barras esta com valor invalido');
      return;
    }

    if (ingredientes == '' || ingredientes == null) {
      setErro('Os ingredientes estão com valor invalido');
      return;
    }

    if (nome == '' || nome == null) {
      setErro('O nome esta com valor invalido');
      return;
    }

    if (preco == '' || preco == null) {
      setErro('O preco esta com valor invalido');
      return;
    }

    setCodBarras('');
    setIngredientes('');
    setNome('');
    setPreco('');
    cadastrarProduto(nome, ingredientes, preco, codBarras);
    history.push('/comes');
  };

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>
            Novo Produto
          </IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent className="cardapio-content">
        <div className="cardapio-content">
          {erro !== '' && <p className="erro">{erro}</p>}

          <IonList>
            <IonItem>
              <IonInput
                label="Nome"
                labelPlacement="stacked"
                value={nome}
                placeholder="Ex: xis salada"
                onIonInput={(event) => setNome(event.detail.value ?? '')}
              >
              </IonInput>
            </IonItem>

            <IonItem>
              <IonInput
                label="Ingredientes"
                labelPlacement="stacked"
                value={ingredientes}
                placeholder="Ex:pao,carne,queijo"
                helperText="separe os ingredientes por virgula"
                onIonInput={(event) => setIngredientes(event.detail.value ?? '')}
              >
              </IonInput>
            </IonItem>

            <IonItem>
              <IonInput
                label="Preco"
                labelPlacement="stacked"
                value={preco}
                placeholder="R$12,50"
                inputMode="decimal"
                onIonInput={(event) => setPreco(event.detail.value ?? '')}
              >

              </IonInput>
            </IonItem>

            <IonItem>
              <IonInput
                label="Cod Barras"
                labelPlacement="stacked"
                value={codBarras}
                placeholder="Ex: 0000000000000"
                onIonInput={(event) => setCodBarras(event.detail.value ?? '')}
              >
              </IonInput>
            </IonItem>
          </IonList>

          <IonButton expand="block" onClick={salvarProduto}>
            salvar produto
          </IonButton>

        </div>
      </IonContent>
    </IonPage>
  );
};
```

### `src/pages/TelaComes/TelaComes.tsx`

Na tela `Comidas`, o botão para abrir o cadastro fica acima da lista:

Dentro do componente, a lista fica em estado:

```tsx
const [produtos, setProdutos] = useState<Produto[]>([]);

useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

```tsx
<IonContent>
  <IonButton routerLink="/produto-novo">Adicionar NOVO produto</IonButton>

  <IonList>
    {produtos.map((comida) => (
      <IonItem className="cardapio-list" key={comida.id}>
        <IonLabel>
          <h2 className="cardapio-fonte">{comida.nome}</h2>
          <p>{comida.ingredientes?.join(', ')}</p>
          <p>{formatarPreco(comida.preco)}</p>
        </IonLabel>

        <IonButton onClick={() => abrirModal(comida)}>Adicionar</IonButton>
      </IonItem>
    ))}
  </IonList>
</IonContent>
```

A lista deve ser recarregada assim:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

### `src/pages/TabsPrincipal/TabsPrincipal.tsx`

O import da tela de formulário fica assim:

```tsx
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
```

Dentro de `IonRouterOutlet`, a rota do formulário fica assim:

```tsx
<Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
```

## 8. Erros comuns

### 8.1 Esquecer que campo de formulário é texto

Mesmo o preço sendo número no tipo `Produto`, ele sai do campo como texto.

Por isso o service usa:

```ts
const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
```

### 8.2 Esquecer de cadastrar a rota

Se clicar em `Adicionar NOVO produto` e a tela não abrir, confira se a rota existe:

```tsx
<Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
```

### 8.3 Esquecer de importar a tela

No `TabsPrincipal.tsx`, precisa existir:

```tsx
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
```

### 8.4 O produto não aparece depois de salvar

Confira se `TelaComes` recarrega a lista quando aparece:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

### 8.5 Usar a rota antiga

Neste projeto, a rota de cadastro é:

```text
/produto-novo
```

Se usar:

```text
/produtos/novo
```

a tela não vai abrir, porque essa rota não existe neste app.

### 8.6 Importar `useHistory` do lugar errado

Neste projeto, o import usado é:

```tsx
import { useHistory } from 'react-router';
```

### 8.7 Importar o formulário como default

Neste projeto, a tela é exportada assim:

```tsx
export const TelaProdutoForm = () => {
```

Então o import correto é:

```tsx
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
```

## 9. Resumo

Nesta aula, você criou um cadastro simples de produto.

Você aprendeu a:

- criar uma tela nova;
- criar uma rota para essa tela;
- usar `useState` para guardar campos;
- usar `IonInput`;
- validar nome, ingredientes, preço e código de barras;
- converter preço de texto para número;
- transformar ingredientes em lista com `split`;
- criar um objeto de produto;
- adicionar esse produto na lista com `push`;
- voltar para a tela `Comidas`.

O mais importante aqui não é fazer um cadastro perfeito.

O mais importante é entender o caminho:

```text
input
  estado
  funcao salvar
  service
  lista na tela
```

## 10. Próximo passo

Na próxima aula, vamos evoluir com calma.

A Aula 6 será sobre edição de produtos.

Mas agora que o cadastro simples já existe, editar fica mais fácil de entender, porque reaproveita a mesma ideia do formulário.
