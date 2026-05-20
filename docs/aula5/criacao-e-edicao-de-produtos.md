# Aula 5: Cadastrando um novo produto

## 1. Objetivo da aula

Nesta aula, você vai criar um formulário simples para cadastrar um novo produto no cardápio.

Nas aulas anteriores, os produtos já apareciam na tela `Comes`. Agora vamos permitir que um novo produto seja adicionado pela interface do app.

Ao final da aula, o app terá:

- Um botão `Novo produto` na tela `Comes`.
- Uma tela simples de cadastro.
- Campos para nome, ingredientes, preço e código de barras.
- Uma validação básica.
- Um produto novo aparecendo na lista de produtos da tela `Comes`.

Vamos fazer só cadastro de produto.

Edição, bebidas e API ficam para depois. Se misturar tudo agora, a aula vira mais difícil do que precisa.

## 2. Resultado final

Na tela `Comes`, o aluno verá:

```text
Comes

[Novo produto]

Mc feliz
Pao, Carne, Queijo
R$ 36,98

[Adicionar]
```

Ao tocar em `Novo produto`, o app abre uma tela assim:

```text
Novo produto

Nome
[________________]

Ingredientes
[________________]

Preco
[________________]

Codigo de barras
[________________]

[Salvar produto]
```

Depois de salvar, o app volta para `Comes` e o produto aparece na lista.

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
  onIonInput={(event) => setNome(String(event.detail.value ?? ''))}
/>
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
src/pages/Cardapio.css
```

Para rodar:

```bash
cd /caminho/para/a/pasta-do-projeto
```

Depois:

```bash
ionic serve
```

## 6. Passo a passo

### 6.1 Conferir o tipo `Produto`

Abra:

```text
src/types/Produto.ts
```

Antes de mexer no service, confira se o tipo `Produto` tem o campo `categoria`:

```ts
export type CategoriaProduto = 'comida' | 'bebida';

export type Produto = {
  id: string;
  codBarras: string;
  ingredientes: string[] | null;
  ativo: boolean;
  nome: string;
  preco: number;
  categoria: CategoriaProduto;
  imagem?: string;
};
```

A categoria diz em qual parte do cardápio o produto entra.

Nesta aula, o aluno não vai escolher a categoria no formulário.

O próprio service vai colocar:

```ts
categoria: 'comida'
```

Assim, o produto novo entra como comida e aparece na tela `Comes`.

### 6.2 Começar pelo service

Abra:

```text
src/services/produtoService.ts
```

O service é onde as listas de produtos ficam guardadas.

Vamos deixar esse arquivo com três funções simples:

- `buscarComidas`: mostra as comidas na tela `Comes`.
- `buscarBebidas`: deixa a lista de bebidas preparada para outra tela.
- `cadastrarProduto`: adiciona um produto novo.

```ts
import type { Produto } from '../types/Produto';

const comidas: Produto[] = [
  {
    ativo: true,
    codBarras: '123',
    ingredientes: ['Pao, Carne, Queijo'],
    nome: 'Mc feliz',
    id: '1',
    preco: 36.98,
    categoria: 'comida',
  },
  {
    ativo: true,
    codBarras: '321',
    ingredientes: ['Pao, Carne, Molho especial'],
    nome: 'Mc melt',
    id: '2',
    preco: 38.02,
    categoria: 'comida',
  },
];

const bebidas: Produto[] = [
  {
    ativo: true,
    codBarras: '456',
    ingredientes: ['Lata 350ml'],
    nome: 'Refrigerante',
    id: '3',
    preco: 6.5,
    categoria: 'bebida',
  },
];

export const buscarComidas = (): Produto[] => {
  return comidas;
};

export const buscarBebidas = (): Produto[] => {
  return bebidas;
};
```

Por enquanto, a lista é um array simples.

O aluno já viu array nas aulas anteriores. Então vamos ficar nesse terreno conhecido.

### 6.3 Criar uma função para gerar `id`

Ainda no `produtoService.ts`, adicione:

```ts
const gerarProximoId = () => {
  const ultimoProduto = comidas[comidas.length - 1];

  if (!ultimoProduto) {
    return '1';
  }

  return String(Number(ultimoProduto.id) + 1);
};
```

Essa função pega o último produto da lista e cria o próximo número.

Se o último produto tem `id` igual a `'2'`, o próximo será `'3'`.

### 6.4 Criar a função `cadastrarProduto`

Agora adicione:

```ts
export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: number,
  codBarras: string,
) => {
  const novoProduto: Produto = {
    id: gerarProximoId(),
    nome: nome,
    ingredientes: [ingredientes],
    preco: preco,
    codBarras: codBarras,
    ativo: true,
    categoria: 'comida',
  };

  comidas.push(novoProduto);
};
```

Aqui estamos fazendo o caminho mais simples:

1. Recebemos os dados do formulário.
2. Criamos um objeto `Produto`.
3. Definimos `categoria: 'comida'`.
4. Colocamos esse produto dentro da lista `comidas` com `push`.

`push` adiciona um item no final do array.

Nesta aula, isso é suficiente.

### 6.5 Criar a pasta da tela de formulário

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

### 6.6 Criar a tela vazia

Comece com:

```tsx
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import '../Cardapio.css';

const TelaProdutoForm = () => {
  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">Novo produto</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-container">
          <p>Formulario aqui</p>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaProdutoForm;
```

Até aqui, só criamos a tela.

Ela ainda não salva nada.

### 6.7 Criar os estados dos campos

Dentro do componente, antes do `return`, adicione:

```tsx
const history = useHistory();

const [nome, setNome] = useState('');
const [ingredientes, setIngredientes] = useState('');
const [preco, setPreco] = useState('');
const [codBarras, setCodBarras] = useState('');
const [erro, setErro] = useState('');
```

Cada campo tem um estado.

`preco` começa como texto porque tudo que vem de um campo de formulário chega como texto.

Depois vamos transformar esse texto em número.

### 6.8 Criar a função de salvar

Ainda dentro do componente, adicione:

```tsx
const salvarProduto = () => {
  setErro('');

  if (nome === '') {
    setErro('Informe o nome do produto.');
    return;
  }

  if (preco === '') {
    setErro('Informe o preco.');
    return;
  }

  const precoNumerico = Number(preco.replace(',', '.'));

  if (Number.isNaN(precoNumerico)) {
    setErro('Informe um preco valido.');
    return;
  }

  cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
  history.push('/comes');
};
```

Aqui a função faz pouco, de propósito:

- limpa erro antigo;
- confere se tem nome;
- confere se tem preço;
- transforma preço em número;
- cadastra o produto;
- volta para `Comes`.

Essa linha merece atenção:

```ts
const precoNumerico = Number(preco.replace(',', '.'));
```

O usuário pode digitar:

```text
20,50
```

O JavaScript entende melhor:

```text
20.50
```

Por isso trocamos vírgula por ponto antes de converter.

### 6.9 Montar o formulário

Agora substitua o conteúdo do `IonContent` por:

```tsx
<IonContent className="cardapio-content">
  <div className="cardapio-container">
    {erro !== '' && <p className="mensagem-erro">{erro}</p>}

    <IonList className="formulario-produto">
      <IonItem>
        <IonInput
          label="Nome"
          labelPlacement="stacked"
          value={nome}
          placeholder="Ex: Xis salada"
          onIonInput={(event) => setNome(String(event.detail.value ?? ''))}
        />
      </IonItem>

      <IonItem>
        <IonInput
          label="Ingredientes"
          labelPlacement="stacked"
          value={ingredientes}
          placeholder="Ex: Pao, queijo, alface"
          onIonInput={(event) =>
            setIngredientes(String(event.detail.value ?? ''))
          }
        />
      </IonItem>

      <IonItem>
        <IonInput
          label="Preco"
          labelPlacement="stacked"
          value={preco}
          placeholder="Ex: 28,50"
          inputMode="decimal"
          onIonInput={(event) => setPreco(String(event.detail.value ?? ''))}
        />
      </IonItem>

      <IonItem>
        <IonInput
          label="Codigo de barras"
          labelPlacement="stacked"
          value={codBarras}
          placeholder="Ex: 7890000000000"
          onIonInput={(event) =>
            setCodBarras(String(event.detail.value ?? ''))
          }
        />
      </IonItem>
    </IonList>

    <IonButton expand="block" onClick={salvarProduto}>
      Salvar produto
    </IonButton>
  </div>
</IonContent>
```

Esse formulário usa só `IonInput`.

Nada de componente diferente agora. A ideia é repetir o mesmo padrão até ficar claro.

### 6.10 Registrar a rota da tela

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Importe a tela:

```tsx
import TelaProdutoForm from '../TelaProdutoForm/TelaProdutoForm';
```

Dentro de `IonRouterOutlet`, adicione:

```tsx
<Route exact path="/produtos/novo" component={TelaProdutoForm} />
```

A rota fica simples:

```text
/produtos/novo
```

### 6.11 Adicionar o botão na tela `Comes`

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Acima da lista de produtos, adicione:

```tsx
<div className="acoes-cardapio">
  <IonButton routerLink="/produtos/novo">Novo produto</IonButton>
</div>
```

Agora a tela tem um caminho para abrir o formulário.

### 6.12 Recarregar a lista quando voltar

Ainda em `TelaComes`, vamos usar um recurso do Ionic:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarComidas());
});
```

O nome parece grande, mas a ideia é simples:

```text
quando a tela Comes for aparecer
  buscar a lista de produtos de novo
```

Isso faz o produto recém-cadastrado aparecer quando voltamos do formulário.

## 7. Código completo

### `src/types/Produto.ts`

```ts
export type CategoriaProduto = 'comida' | 'bebida';

export type Produto = {
  id: string;
  codBarras: string;
  ingredientes: string[] | null;
  ativo: boolean;
  nome: string;
  preco: number;
  categoria: CategoriaProduto;
  imagem?: string;
};
```

### `src/services/produtoService.ts`

```ts
import type { Produto } from '../types/Produto';

const comidas: Produto[] = [
  {
    ativo: true,
    codBarras: '123',
    ingredientes: ['Pao, Carne, Queijo'],
    nome: 'Mc feliz',
    id: '1',
    preco: 36.98,
    categoria: 'comida',
  },
  {
    ativo: true,
    codBarras: '321',
    ingredientes: ['Pao, Carne, Molho especial'],
    nome: 'Mc melt',
    id: '2',
    preco: 38.02,
    categoria: 'comida',
  },
];

const bebidas: Produto[] = [
  {
    ativo: true,
    codBarras: '456',
    ingredientes: ['Lata 350ml'],
    nome: 'Refrigerante',
    id: '3',
    preco: 6.5,
    categoria: 'bebida',
  },
];

const gerarProximoId = () => {
  const ultimoProduto = comidas[comidas.length - 1];

  if (!ultimoProduto) {
    return '1';
  }

  return String(Number(ultimoProduto.id) + 1);
};

export const buscarComidas = (): Produto[] => {
  return comidas;
};

export const buscarBebidas = (): Produto[] => {
  return bebidas;
};

export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: number,
  codBarras: string,
) => {
  const novoProduto: Produto = {
    id: gerarProximoId(),
    nome: nome,
    ingredientes: [ingredientes],
    preco: preco,
    codBarras: codBarras,
    ativo: true,
    categoria: 'comida',
  };

  comidas.push(novoProduto);
};
```

### `src/pages/TelaProdutoForm/TelaProdutoForm.tsx`

```tsx
import { useState } from 'react';
import {
  IonButton,
  IonContent,
  IonHeader,
  IonInput,
  IonItem,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useHistory } from 'react-router-dom';

import { cadastrarProduto } from '../../services/produtoService';
import '../Cardapio.css';

const TelaProdutoForm = () => {
  const history = useHistory();

  const [nome, setNome] = useState('');
  const [ingredientes, setIngredientes] = useState('');
  const [preco, setPreco] = useState('');
  const [codBarras, setCodBarras] = useState('');
  const [erro, setErro] = useState('');

  const salvarProduto = () => {
    setErro('');

    if (nome === '') {
      setErro('Informe o nome do produto.');
      return;
    }

    if (preco === '') {
      setErro('Informe o preco.');
      return;
    }

    const precoNumerico = Number(preco.replace(',', '.'));

    if (Number.isNaN(precoNumerico)) {
      setErro('Informe um preco valido.');
      return;
    }

    cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
    history.push('/comes');
  };

  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">Novo produto</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-container">
          {erro !== '' && <p className="mensagem-erro">{erro}</p>}

          <IonList className="formulario-produto">
            <IonItem>
              <IonInput
                label="Nome"
                labelPlacement="stacked"
                value={nome}
                placeholder="Ex: Xis salada"
                onIonInput={(event) =>
                  setNome(String(event.detail.value ?? ''))
                }
              />
            </IonItem>

            <IonItem>
              <IonInput
                label="Ingredientes"
                labelPlacement="stacked"
                value={ingredientes}
                placeholder="Ex: Pao, queijo, alface"
                onIonInput={(event) =>
                  setIngredientes(String(event.detail.value ?? ''))
                }
              />
            </IonItem>

            <IonItem>
              <IonInput
                label="Preco"
                labelPlacement="stacked"
                value={preco}
                placeholder="Ex: 28,50"
                inputMode="decimal"
                onIonInput={(event) =>
                  setPreco(String(event.detail.value ?? ''))
                }
              />
            </IonItem>

            <IonItem>
              <IonInput
                label="Codigo de barras"
                labelPlacement="stacked"
                value={codBarras}
                placeholder="Ex: 7890000000000"
                onIonInput={(event) =>
                  setCodBarras(String(event.detail.value ?? ''))
                }
              />
            </IonItem>
          </IonList>

          <IonButton expand="block" onClick={salvarProduto}>
            Salvar produto
          </IonButton>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaProdutoForm;
```

## 8. Erros comuns

### 8.1 Esquecer que campo de formulário é texto

Mesmo o preço sendo número, ele sai do campo como texto.

Por isso usamos:

```ts
const precoNumerico = Number(preco.replace(',', '.'));
```

### 8.2 Esquecer de cadastrar a rota

Se clicar em `Novo produto` e a tela não abrir, confira se a rota existe:

```tsx
<Route exact path="/produtos/novo" component={TelaProdutoForm} />
```

### 8.3 Esquecer de importar a tela

No `TabsPrincipal.tsx`, precisa existir:

```tsx
import TelaProdutoForm from '../TelaProdutoForm/TelaProdutoForm';
```

### 8.4 O produto não aparece depois de salvar

Confira se `TelaComes` recarrega a lista quando aparece:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarComidas());
});
```

### 8.5 Digitar preço com letra

Se o aluno digitar:

```text
abc
```

O app mostra erro, porque isso não vira número.

## 9. Resumo

Nesta aula, você criou um cadastro simples de produto.

Você aprendeu a:

- criar uma tela nova;
- criar uma rota para essa tela;
- usar `useState` para guardar campos;
- usar `IonInput`;
- validar nome e preço;
- converter preço de texto para número;
- criar um objeto `Produto`;
- preencher `categoria: 'comida'` automaticamente no service;
- adicionar esse produto na lista com `push`;
- voltar para a tela `Comes`.

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

Na próxima aula, podemos evoluir com calma.

Um bom próximo passo é editar um produto existente.

Mas agora que o cadastro simples já existe, editar fica mais fácil de entender, porque reaproveita a mesma ideia do formulário.
