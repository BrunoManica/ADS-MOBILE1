# Aula 6: Edição de produtos

## 1. Objetivo da aula

Nesta aula, você vai evoluir o formulário da Aula 5 para editar um produto existente.

Na aula anterior, o app aprendeu a cadastrar um novo produto. Agora vamos reaproveitar quase a mesma tela para alterar nome, ingredientes, preço e código de barras de um produto que já está na lista.

Ao final da aula, o app terá:

- Um botão `Editar` em cada produto da tela `Comidas`.
- Uma rota de edição com `id` do produto.
- O formulário carregando os dados do produto escolhido.
- Uma função no service para buscar produto por `id`.
- Uma função no service para alterar o produto.
- A lista atualizada depois que o aluno salvar.

Vamos fazer só edição de produto.

Login e controle de permissões ficam para a Aula 7.

Comunicação com API fica para a Aula 8.

## 2. Resultado final

Na tela `Comidas`, cada produto terá um botão de edição:

```text
Comidas

[Adicionar NOVO produto]

Mc feliz
Pao, Carne, Queijo
R$ 36,98

[Adicionar] [Editar]
```

Ao tocar em `Editar`, o app abre a tela do formulário com os dados preenchidos:

```text
Novo Produto

Nome
[Mc feliz________]

Ingredientes
[Pao, Carne, Queijo]

Preco
[36.98__________]

Cod Barras
[123____________]

[salvar produto]
```

Depois de salvar, o app volta para `Comidas` e mostra o produto com os dados atualizados.

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
  service altera o Produto
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

Na Aula 5, a rota de cadastro era:

```text
/produto-novo
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
const { id } = useParams<{ id?: string }>();
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
cd /caminho/para/react-burguer
```

Depois:

```bash
npm run dev
```

## 6. Passo a passo

### 6.1 Começar pelo service

Abra:

```text
src/services/produtoService.ts
```

No topo do arquivo do projeto pode aparecer este import:

```ts
import { map } from 'ionicons/icons';
```

Ele não será usado nesta aula.

Se ele ficar no arquivo, o app continua funcionando. Se quiser limpar, pode remover.

Na Aula 5, o service já tinha:

- `buscarTodosOsProdutos`;
- `cadastrarProduto`.

Agora vamos adicionar mais duas funções:

- `buscarProdutoPorId`;
- `alterarProduto`.

Também vamos deixar a lista como `let`, porque a função `alterarProduto` vai recriar a lista com `map`.

Troque:

```ts
const comida: Produto[] = [
```

por:

```ts
let comidas: Produto[] = [
```

Depois disso, troque os outros usos do array `comida` por `comidas` dentro do service.

O parâmetro usado dentro do `map` pode continuar se chamando `comida`:

```ts
comidas = comidas.map((comida) => {
```

E ajuste a busca:

```ts
export const buscarTodosOsProdutos = (): Produto[] => {
  return comidas;
};
```

### 6.2 Ajustar a geração de `id`

Na Aula 5, o cadastro gerava o `id` com a lista `comida`:

```ts
let id = Math.max(...comida.map(c => Number(c.id)));
id = id + 1;
```

Agora, como a lista se chama `comidas`, ajuste para:

```ts
let id = Math.max(...comidas.map(c => Number(c.id)));
id = id + 1;
```

Na criação do produto, salve o `id` como texto:

```ts
id: id.toString(),
```

Isso evita repetir o mesmo `id` de um produto que já existe.

A função `cadastrarProduto` fica assim depois desse ajuste:

```ts
export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  let id = Math.max(...comidas.map(c => Number(c.id)));
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

  comidas.push(novoProduto);
};
```

### 6.3 Criar a função `buscarProdutoPorId`

Ainda no `produtoService.ts`, adicione:

```ts
export const buscarProdutoPorId = (id: string): Produto | null => {
  const todasAsComidas = comidas;

  const produtoEncontrado = todasAsComidas.find((comida) => comida.id == id);

  if (!produtoEncontrado) return null;

  return produtoEncontrado;
};
```

Aqui fazemos três coisas:

1. Pegamos a lista de comidas.
2. Procuramos o produto com o mesmo `id`.
3. Retornamos o produto encontrado ou `null`.

Essa parte:

```ts
comida.id == id
```

é a comparação.

Se o produto tem o mesmo `id` que veio da rota, ele é o produto que queremos editar.

### 6.4 Criar a função `alterarProduto`

Agora adicione:

```ts
export const alterarProduto = (
  id: string,
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  const produto = buscarProdutoPorId(id);
  if (!produto) {
    return;
  }

  comidas = comidas.map((comida) => {
    if (comida.id == id) {
      const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
      comida.id = id;
      comida.nome = nome;
      comida.ingredientes = ingredientes.split(',');
      comida.preco = precoTratado;
      comida.codBarras = codBarras;
    }
    return comida;
  });
};
```

Essa função recebe os dados do formulário e altera o produto encontrado.

O `map` percorre a lista. Quando encontra o produto com o mesmo `id`, troca os campos.

Nesta aula, ainda não vamos trabalhar com API ou banco de dados.

O foco é entender o fluxo:

```text
achar produto
  trocar campos
  voltar para a lista
```

### 6.5 Registrar a rota de edição

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Confira se o arquivo já importa o formulário:

```tsx
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
```

Na Aula 5, você adicionou esta rota:

```tsx
<Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
```

Agora adicione também:

```tsx
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm}></Route>
```

As duas rotas usam a mesma tela.

Isso é importante.

O formulário será reaproveitado:

- se não tiver `id`, é cadastro;
- se tiver `id`, é edição.

### 6.6 Adicionar o botão `Editar` na tela `Comidas`

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Confira se `IonButton` está no import do Ionic. Ele já deve estar, porque a tela usa o botão `Adicionar`:

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
```

Na listagem de produtos, perto do botão `Adicionar`, coloque:

```tsx
<IonButton routerLink={`/produtos/editar/${comida.id}`}>
  Editar
</IonButton>
```

Essa linha monta a URL usando o `id` do produto.

Se o produto tem `id` igual a `'1'`, o botão abre:

```text
/produtos/editar/1
```

### 6.7 Preparar o formulário para cadastro e edição

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
import { useHistory, useParams } from 'react-router';
```

E atualize o import do service:

```tsx
import {
  cadastrarProduto,
  buscarProdutoPorId,
  alterarProduto,
} from '../../services/produtoService';
```

Se o arquivo ainda tiver só este import:

```tsx
import { cadastrarProduto } from '../../services/produtoService';
```

troque pelo import completo acima.

### 6.8 Pegar o `id` da rota

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

Mantenha os estados da Aula 5 no componente:

```tsx
const [codBarras, setCodBarras] = useState('');
const [ingredientes, setIngredientes] = useState('');
const [nome, setNome] = useState('');
const [preco, setPreco] = useState('');
const [erro, setErro] = useState('');
```

### 6.9 Carregar os dados do produto

Depois dos estados, adicione:

```tsx
useEffect(() => {
  if (!id) {
    return;
  }

  const produtoPorId = buscarProdutoPorId(id);
  if (!produtoPorId) {
    return;
  }

  setCodBarras(produtoPorId?.codBarras);
  setPreco(produtoPorId?.preco.toString());
  setNome(produtoPorId?.nome);
  setIngredientes(produtoPorId?.ingredientes?.join(', ') ?? '');
}, [id]);
```

Esse código roda quando a tela abre.

Ele faz o seguinte:

- se não tem `id`, não precisa carregar nada;
- se tem `id`, busca o produto;
- se não encontrar, para a execução;
- se encontrar, preenche os campos.

Essa linha:

```ts
setIngredientes(produtoPorId?.ingredientes?.join(', ') ?? '');
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

### 6.10 Atualizar a função `salvarProduto`

Na Aula 5, a função sempre chamava `cadastrarProduto`.

Agora ela precisa decidir:

- cadastrar produto novo;
- ou alterar produto existente.

Depois de limpar os campos, troque a chamada direta de `cadastrarProduto` por:

```tsx
if (estaEditando) {
  alterarProduto(id, nome, ingredientes, preco, codBarras);
} else {
  cadastrarProduto(nome, ingredientes, preco, codBarras);
}

history.push('/comes');
```

A função completa fica assim:

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

  if (estaEditando) {
    alterarProduto(id, nome, ingredientes, preco, codBarras);
  } else {
    cadastrarProduto(nome, ingredientes, preco, codBarras);
  }

  history.push('/comes');
};
```

Repare que a validação continua igual.

Os campos são obrigatórios tanto para cadastrar quanto para editar.

### 6.11 Recarregar a lista quando voltar

Na Aula 5, `TelaComes` já passou a usar:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

Mantenha esse trecho.

Ele é o que faz a lista atualizar quando o formulário volta para `Comidas`.

## 7. Código completo

### `src/services/produtoService.ts`

```ts
import { map } from 'ionicons/icons';
import { Produto } from '../types/Produto';

let comidas: Produto[] = [
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
  return comidas;
};

export const cadastrarProduto = (
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  let id = Math.max(...comidas.map(c => Number(c.id)));
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

  comidas.push(novoProduto);
};

export const alterarProduto = (
  id: string,
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
) => {
  const produto = buscarProdutoPorId(id);
  if (!produto) {
    return;
  }

  comidas = comidas.map((comida) => {
    if (comida.id == id) {
      const precoTratado = Number(preco.replace('R$', '').replace(',', '.'));
      comida.id = id;
      comida.nome = nome;
      comida.ingredientes = ingredientes.split(',');
      comida.preco = precoTratado;
      comida.codBarras = codBarras;
    }
    return comida;
  });
};

export const buscarProdutoPorId = (id: string): Produto | null => {
  const todasAsComidas = comidas;

  const produtoEncontrado = todasAsComidas.find((comida) => comida.id == id);

  if (!produtoEncontrado) return null;

  return produtoEncontrado;
};
```

### `src/pages/TelaProdutoForm/TelaProdutoForm.tsx`

```tsx
import { IonButton, IonContent, IonHeader, IonInput, IonItem, IonList, IonPage, IonTitle, IonToolbar } from '@ionic/react';
import { useEffect, useState } from 'react';
import '../Cardapio.css';
import { useHistory, useParams } from 'react-router';

import {
  cadastrarProduto,
  buscarProdutoPorId,
  alterarProduto,
} from '../../services/produtoService';

export const TelaProdutoForm = () => {
  const history = useHistory();

  const { id } = useParams<{ id?: string }>();
  const estaEditando = id !== undefined;

  const [codBarras, setCodBarras] = useState('');
  const [ingredientes, setIngredientes] = useState('');
  const [nome, setNome] = useState('');
  const [preco, setPreco] = useState('');
  const [erro, setErro] = useState('');

  useEffect(() => {
    if (!id) {
      return;
    }

    const produtoPorId = buscarProdutoPorId(id);
    if (!produtoPorId) {
      return;
    }

    setCodBarras(produtoPorId?.codBarras);
    setPreco(produtoPorId?.preco.toString());
    setNome(produtoPorId?.nome);
    setIngredientes(produtoPorId?.ingredientes?.join(', ') ?? '');
  }, [id]);

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

    if (estaEditando) {
      alterarProduto(id, nome, ingredientes, preco, codBarras);
    } else {
      cadastrarProduto(nome, ingredientes, preco, codBarras);
    }

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

### `src/pages/TabsPrincipal/TabsPrincipal.tsx`

Dentro de `IonRouterOutlet`, as rotas do formulário ficam assim:

```tsx
<Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm}></Route>
```

O trecho completo do `IonRouterOutlet` fica assim:

```tsx
<IonRouterOutlet>
  <Route exact path="/bebes" component={TelaBebes}></Route>
  <Route exact path="/comes" component={TelaComes}></Route>
  <Route exact path="/pedido" component={TelaPedido}></Route>
  <Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
  <Route exact path="/produtos/editar/:id" component={TelaProdutoForm}></Route>
  <Route exact path="/">
    <Redirect to="/bebes"></Redirect>
  </Route>
</IonRouterOutlet>
```

### `src/pages/TelaComes/TelaComes.tsx`

Na lista de produtos, os botões ficam assim:

```tsx
<IonButton onClick={() => abrirModal(comida)}>Adicionar</IonButton>
<IonButton routerLink={`/produtos/editar/${comida.id}`}>
  Editar
</IonButton>
```

Eles ficam dentro do `produtos.map`, no mesmo `IonItem` que mostra nome, ingredientes e preço:

```tsx
{produtos.map((comida) => (
  <IonItem className="cardapio-list" key={comida.id}>
    <IonLabel>
      <h2 className="cardapio-fonte">{comida.nome}</h2>
      <p>{comida.ingredientes?.join(', ')}</p>
      <p>{formatarPreco(comida.preco)}</p>
    </IonLabel>

    <IonButton onClick={() => abrirModal(comida)}>Adicionar</IonButton>
    <IonButton routerLink={`/produtos/editar/${comida.id}`}>
      Editar
    </IonButton>
  </IonItem>
))}
```

E a tela deve continuar recarregando a lista quando aparecer:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

## 8. Erros comuns

### 8.1 Esquecer o `:id` na rota

Se a rota ficar assim:

```tsx
<Route exact path="/produtos/editar" component={TelaProdutoForm}></Route>
```

o formulário não recebe o `id`.

O correto é:

```tsx
<Route exact path="/produtos/editar/:id" component={TelaProdutoForm}></Route>
```

### 8.2 Montar o link sem o `id`

Se o botão estiver assim:

```tsx
<IonButton routerLink="/produtos/editar">Editar</IonButton>
```

a tela abre sem saber qual produto editar.

O correto é:

```tsx
<IonButton routerLink={`/produtos/editar/${comida.id}`}>
  Editar
</IonButton>
```

### 8.3 Usar o nome errado da função

Neste projeto, a função de edição no service se chama:

```ts
alterarProduto
```

Então o import correto é:

```tsx
import {
  cadastrarProduto,
  buscarProdutoPorId,
  alterarProduto,
} from '../../services/produtoService';
```

Se chamar `editarProduto`, o código não compila porque essa função não existe neste service.

### 8.4 Esquecer de carregar os campos

Se o formulário abre vazio na edição, confira se o `useEffect` está buscando o produto e chamando:

```tsx
setCodBarras(produtoPorId?.codBarras);
setPreco(produtoPorId?.preco.toString());
setNome(produtoPorId?.nome);
setIngredientes(produtoPorId?.ingredientes?.join(', ') ?? '');
```

### 8.5 Chamar sempre `cadastrarProduto`

Se o app cria outro produto em vez de editar, confira a decisão dentro de `salvarProduto`.

Errado:

```tsx
cadastrarProduto(nome, ingredientes, preco, codBarras);
```

Correto:

```tsx
if (estaEditando) {
  alterarProduto(id, nome, ingredientes, preco, codBarras);
} else {
  cadastrarProduto(nome, ingredientes, preco, codBarras);
}
```

### 8.6 Usar `buscarComidas`

Neste projeto, a função de listagem se chama:

```ts
buscarTodosOsProdutos
```

Então a tela `Comidas` deve recarregar a lista assim:

```tsx
useIonViewWillEnter(() => {
  setProdutos(buscarTodosOsProdutos());
});
```

### 8.7 Importar `useParams` de outro lugar

Neste projeto, o import usado é:

```tsx
import { useHistory, useParams } from 'react-router';
```

## 9. Resumo

Nesta aula, você transformou o formulário de cadastro em um formulário que também edita produtos.

Você aprendeu a:

- criar uma rota com parâmetro;
- pegar o `id` da URL com `useParams`;
- buscar um produto pelo `id`;
- preencher os estados do formulário com dados existentes;
- reaproveitar a mesma tela para cadastro e edição;
- decidir entre `cadastrarProduto` e `alterarProduto`;
- alterar os dados de um produto no service;
- voltar para a tela `Comidas` após salvar.

O mais importante aqui é entender que edição precisa de identificação.

Cadastro cria um produto novo.

Edição procura um produto existente e altera seus campos.
