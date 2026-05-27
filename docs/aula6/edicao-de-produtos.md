# Aula 6: Edição de produtos

## 1. Objetivo da aula

Nesta aula, você vai evoluir o formulário da Aula 5 para editar um produto existente.

Na aula anterior, o app aprendeu a cadastrar um novo produto. Agora vamos usar quase a mesma tela para alterar nome, ingredientes, preço e código de barras de um produto que já está na lista.

Ao final da aula, o app terá:

- Um botão `Editar` em cada produto da tela `Comes`.
- Uma rota de edição com `id` do produto.
- O formulário carregando os dados do produto escolhido.
- Uma função no service para buscar produto por `id`.
- Uma função no service para salvar as alterações.
- A lista atualizada depois que o aluno salvar.

Vamos fazer só edição de produto.

Login e controle de permissões ficam para a Aula 7.

Comunicação com API fica para a Aula 8.

## 2. Resultado final

Na tela `Comes`, cada produto terá um botão de edição:

```text
Comes

[Novo produto]

Mc feliz
Pao, Carne, Queijo
R$ 36,98

[Adicionar] [Editar]
```

Ao tocar em `Editar`, o app abre a tela do formulário já preenchida:

```text
Editar produto

Nome
[Mc feliz________]

Ingredientes
[Pao, Carne, Queijo]

Preco
[36.98__________]

Codigo de barras
[123____________]

[Salvar alterações]
```

Depois de salvar, o app volta para `Comes` e mostra o produto com os dados atualizados.

## 3. Contexto

Editar parece uma coisa muito diferente de cadastrar, mas o fluxo é parecido.

No cadastro, o caminho foi:

```text
campo digitado
  vira estado no React
  vira um objeto Produto
  entra na lista do service
  aparece na tela
```

Na edição, o caminho muda um pouco:

```text
id vem da rota
  service encontra o Produto
  campos recebem os dados atuais
  aluno altera os campos
  service atualiza o Produto
  lista mostra os dados novos
```

A diferença principal é que agora precisamos saber qual produto será editado.

Para isso, vamos usar o `id` na URL.

Exemplo:

```text
/produtos/editar/1
```

Esse `1` é o `id` do produto.

## 4. Explicação conceitual

Uma rota pode carregar informação.

Na Aula 5, a rota era fixa:

```text
/produtos/novo
```

Ela sempre abria um formulário vazio.

Agora vamos criar uma rota com parâmetro:

```text
/produtos/editar/:id
```

O trecho `:id` significa:

```text
essa parte da URL muda
```

Então estas URLs usam a mesma tela:

```text
/produtos/editar/1
/produtos/editar/2
/produtos/editar/3
```

Dentro do React, pegamos esse valor com `useParams`.

Exemplo:

```tsx
const { id } = useParams<{ id: string }>();
```

Se a URL for:

```text
/produtos/editar/2
```

Então `id` será:

```text
2
```

Com esse `id`, o service consegue procurar o produto correto.

## 5. Setup inicial

Esta aula continua a partir da Aula 5.

Arquivos que vamos mexer:

```text
src/services/produtoService.ts
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
src/pages/TelaComes/TelaComes.tsx
src/pages/TabsPrincipal/TabsPrincipal.tsx
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

### 6.1 Começar pelo service

Abra:

```text
src/services/produtoService.ts
```

Na Aula 5, o service já tinha:

- `buscarComidas`;
- `buscarBebidas`;
- `cadastrarProduto`.

Agora vamos adicionar mais duas funções:

- `buscarProdutoPorId`;
- `editarProduto`.

### 6.2 Criar a função `buscarProdutoPorId`

Ainda no `produtoService.ts`, adicione:

```ts
export const buscarProdutoPorId = (id: string): Produto | undefined => {
  const todosProdutos = [...comidas, ...bebidas];

  return todosProdutos.find((produto) => produto.id === id);
};
```

Aqui fazemos três coisas:

1. Juntamos comidas e bebidas em uma lista só.
2. Procuramos o produto com o mesmo `id`.
3. Retornamos o produto encontrado.

Essa parte:

```ts
const todosProdutos = [...comidas, ...bebidas];
```

cria uma nova lista com os itens de `comidas` e `bebidas`.

Essa parte:

```ts
produto.id === id
```

é a comparação.

Se o produto tem o mesmo `id` que veio da rota, ele é o produto que queremos editar.

### 6.3 Criar a função `editarProduto`

Agora adicione:

```ts
export const editarProduto = (
  id: string,
  nome: string,
  ingredientes: string,
  preco: number,
  codBarras: string,
) => {
  const produto = buscarProdutoPorId(id);

  if (!produto) {
    return;
  }

  produto.nome = nome;
  produto.ingredientes = [ingredientes];
  produto.preco = preco;
  produto.codBarras = codBarras;
};
```

Essa função recebe os dados do formulário e altera o produto encontrado.

Nesta aula, vamos atualizar o objeto direto.

Ainda não vamos trabalhar com API, banco de dados ou atualização imutável.

O foco é entender o fluxo:

```text
achar produto
  trocar campos
  voltar para a lista
```

### 6.4 Registrar a rota de edição

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Na Aula 5, você adicionou esta rota:

```tsx
<Route exact path="/produtos/novo" component={TelaProdutoForm} />
```

Agora adicione também:

```tsx
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm} />
```

As duas rotas usam a mesma tela.

Isso é importante.

O formulário será reaproveitado:

- se não tiver `id`, é cadastro;
- se tiver `id`, é edição.

### 6.5 Adicionar o botão `Editar` na tela `Comes`

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Na listagem de produtos, adicione um botão para editar.

Perto do botão `Adicionar`, coloque:

```tsx
<IonButton routerLink={`/produtos/editar/${produto.id}`}>
  Editar
</IonButton>
```

Essa linha monta a URL usando o `id` do produto.

Se o produto tem `id` igual a `'1'`, o botão abre:

```text
/produtos/editar/1
```

### 6.6 Preparar o formulário para cadastro e edição

Abra:

```text
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
```

Na Aula 5, esse formulário só cadastrava.

Agora ele precisa descobrir se está em modo cadastro ou em modo edição.

Importe `useEffect` junto com `useState`:

```tsx
import { useEffect, useState } from 'react';
```

Importe também `useParams`:

```tsx
import { useHistory, useParams } from 'react-router-dom';
```

E atualize o import do service:

```tsx
import {
  buscarProdutoPorId,
  cadastrarProduto,
  editarProduto,
} from '../../services/produtoService';
```

### 6.7 Pegar o `id` da rota

Dentro do componente, logo depois do `history`, adicione:

```tsx
const { id } = useParams<{ id?: string }>();
const estaEditando = id !== undefined;
```

Agora o componente sabe em qual modo está.

Se `id` existe:

```text
modo edição
```

Se `id` não existe:

```text
modo cadastro
```

### 6.8 Carregar os dados do produto

Depois dos estados, adicione:

```tsx
useEffect(() => {
  if (!id) {
    return;
  }

  const produto = buscarProdutoPorId(id);

  if (!produto) {
    setErro('Produto nao encontrado.');
    return;
  }

  setNome(produto.nome);
  setIngredientes(produto.ingredientes?.join(', ') ?? '');
  setPreco(String(produto.preco));
  setCodBarras(produto.codBarras);
}, [id]);
```

Esse código roda quando a tela abre.

Ele faz o seguinte:

- se não tem `id`, não precisa carregar nada;
- se tem `id`, busca o produto;
- se não encontrar, mostra erro;
- se encontrar, preenche os campos.

Essa linha:

```ts
setIngredientes(produto.ingredientes?.join(', ') ?? '');
```

transforma a lista de ingredientes em texto.

Exemplo:

```ts
['Pao', 'Carne', 'Queijo']
```

vira:

```text
Pao, Carne, Queijo
```

### 6.9 Atualizar a função `salvarProduto`

Na Aula 5, a função sempre chamava `cadastrarProduto`.

Agora ela precisa decidir:

- cadastrar produto novo;
- ou editar produto existente.

Substitua o final da função por:

```tsx
if (estaEditando && id) {
  editarProduto(id, nome, ingredientes, precoNumerico, codBarras);
} else {
  cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
}

history.push('/comes');
```

A função completa fica assim:

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

  if (estaEditando && id) {
    editarProduto(id, nome, ingredientes, precoNumerico, codBarras);
  } else {
    cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
  }

  history.push('/comes');
};
```

Repare que a validação continua igual.

Nome e preço são obrigatórios tanto para cadastrar quanto para editar.

### 6.10 Alterar título e texto do botão

No `IonTitle`, troque:

```tsx
<IonTitle className="cardapio-toolbar-title">Novo produto</IonTitle>
```

por:

```tsx
<IonTitle className="cardapio-toolbar-title">
  {estaEditando ? 'Editar produto' : 'Novo produto'}
</IonTitle>
```

No botão, troque:

```tsx
Salvar produto
```

por:

```tsx
{estaEditando ? 'Salvar alterações' : 'Salvar produto'}
```

Assim a mesma tela fica clara para os dois casos.

### 6.11 Recarregar a lista quando voltar

Na Aula 5, `TelaComes` já passou a usar:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarComidas());
});
```

Mantenha esse trecho.

Ele é o que faz a lista atualizar quando o formulário volta para `Comes`.

## 7. Código completo

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

export const buscarProdutoPorId = (id: string): Produto | undefined => {
  const todosProdutos = [...comidas, ...bebidas];

  return todosProdutos.find((produto) => produto.id === id);
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

export const editarProduto = (
  id: string,
  nome: string,
  ingredientes: string,
  preco: number,
  codBarras: string,
) => {
  const produto = buscarProdutoPorId(id);

  if (!produto) {
    return;
  }

  produto.nome = nome;
  produto.ingredientes = [ingredientes];
  produto.preco = preco;
  produto.codBarras = codBarras;
};
```

### `src/pages/TelaProdutoForm/TelaProdutoForm.tsx`

```tsx
import { useEffect, useState } from 'react';
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
import { useHistory, useParams } from 'react-router-dom';

import {
  buscarProdutoPorId,
  cadastrarProduto,
  editarProduto,
} from '../../services/produtoService';
import '../Cardapio.css';

const TelaProdutoForm = () => {
  const history = useHistory();
  const { id } = useParams<{ id?: string }>();
  const estaEditando = id !== undefined;

  const [nome, setNome] = useState('');
  const [ingredientes, setIngredientes] = useState('');
  const [preco, setPreco] = useState('');
  const [codBarras, setCodBarras] = useState('');
  const [erro, setErro] = useState('');

  useEffect(() => {
    if (!id) {
      return;
    }

    const produto = buscarProdutoPorId(id);

    if (!produto) {
      setErro('Produto nao encontrado.');
      return;
    }

    setNome(produto.nome);
    setIngredientes(produto.ingredientes?.join(', ') ?? '');
    setPreco(String(produto.preco));
    setCodBarras(produto.codBarras);
  }, [id]);

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

    if (estaEditando && id) {
      editarProduto(id, nome, ingredientes, precoNumerico, codBarras);
    } else {
      cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
    }

    history.push('/comes');
  };

  return (
    <IonPage className="cardapio-page">
      <IonHeader>
        <IonToolbar className="cardapio-toolbar">
          <IonTitle className="cardapio-toolbar-title">
            {estaEditando ? 'Editar produto' : 'Novo produto'}
          </IonTitle>
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
            {estaEditando ? 'Salvar alterações' : 'Salvar produto'}
          </IonButton>
        </div>
      </IonContent>
    </IonPage>
  );
};

export default TelaProdutoForm;
```

### `src/pages/TabsPrincipal/TabsPrincipal.tsx`

Dentro de `IonRouterOutlet`, as rotas do formulário ficam assim:

```tsx
<Route exact path="/produtos/novo" component={TelaProdutoForm} />
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm} />
```

### `src/pages/TelaComes/TelaComes.tsx`

Na lista de produtos, o botão de edição fica assim:

```tsx
<IonButton routerLink={`/produtos/editar/${produto.id}`}>
  Editar
</IonButton>
```

E a tela deve continuar recarregando a lista quando aparecer:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarComidas());
});
```

## 8. Erros comuns

### 8.1 Esquecer o `:id` na rota

Se a rota ficar assim:

```tsx
<Route exact path="/produtos/editar" component={TelaProdutoForm} />
```

o formulário não recebe o `id`.

O correto é:

```tsx
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm} />
```

### 8.2 Montar o link sem o `id`

Se o botão estiver assim:

```tsx
<IonButton routerLink="/produtos/editar">Editar</IonButton>
```

a tela abre sem saber qual produto editar.

O correto é:

```tsx
<IonButton routerLink={`/produtos/editar/${produto.id}`}>
  Editar
</IonButton>
```

### 8.3 Esquecer de carregar os campos

Se o formulário abre vazio na edição, confira se o `useEffect` está buscando o produto e chamando:

```tsx
setNome(produto.nome);
setIngredientes(produto.ingredientes?.join(', ') ?? '');
setPreco(String(produto.preco));
setCodBarras(produto.codBarras);
```

### 8.4 Chamar sempre `cadastrarProduto`

Se o app cria outro produto em vez de editar, confira a decisão dentro de `salvarProduto`:

```tsx
if (estaEditando && id) {
  editarProduto(id, nome, ingredientes, precoNumerico, codBarras);
} else {
  cadastrarProduto(nome, ingredientes, precoNumerico, codBarras);
}
```

### 8.5 O produto editado não aparece atualizado

Confira se `TelaComes` ainda usa:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarComidas());
});
```

Esse trecho recarrega a lista quando a tela aparece novamente.

## 9. Resumo

Nesta aula, você transformou o formulário de cadastro em um formulário que também edita produtos.

Você aprendeu a:

- criar uma rota com parâmetro;
- pegar o `id` da URL com `useParams`;
- buscar um produto pelo `id`;
- preencher os estados do formulário com dados existentes;
- reaproveitar a mesma tela para cadastro e edição;
- decidir entre `cadastrarProduto` e `editarProduto`;
- alterar os dados de um produto no service;
- voltar para a tela `Comes` após salvar.

O mais importante aqui é entender que edição precisa de identificação.

Cadastro cria um produto novo.

Edição procura um produto existente e altera seus campos.

## 10. Próximo passo

Na próxima aula, vamos começar a proteger algumas ações do app.

A Aula 7 será sobre login e controle de permissões.

Depois que existe cadastro e edição de produtos, faz sentido perguntar:

```text
todo usuário pode alterar o cardápio?
```

Na Aula 7, vamos tratar essa pergunta com um login simples e permissões básicas.
