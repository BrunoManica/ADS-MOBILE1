# Aula 2: Carregando listas de comidas e bebidas

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

Nesta aula, você vai transformar as telas `Comes` e `Bebes` em telas que exibem listas de produtos.

Ao final, o app terá:

- Um tipo `Produto` para representar comidas e bebidas.
- Um arquivo separado para guardar esse tipo.
- Produtos com `id`, `nome`, `ingredientes`, `preco`, `codBarras`, `imagem` e `ativo`.
- Funções de service para listar comidas e bebidas.
- Uma renderização dinâmica usando `.map()`.
- Um CSS compartilhado para as duas telas do cardápio.
- Nome, ingredientes e preço sendo exibidos para cada produto.

As telas deixam de mostrar apenas `Comida 1` e `Bebida 1` e passam a trabalhar com dados estruturados.

## 2. Resultado final

Ao acessar a rota de comidas:

```text
/comes
```

A tela vai exibir uma lista parecida com esta:

```text
Cardapio
Tela de Comes
3 itens disponiveis

Torrada
Torrada tradicional da casa
R$ 20,00

Xis Salada
Pao, Hamburguer, Queijo, Alface, Tomate, Maionese
R$ 28,00

Batata Frita
Porcao media de batata frita
R$ 18,00
```

Ao acessar a rota de bebidas:

```text
/bebes
```

A tela vai exibir uma lista parecida com esta:

```text
Cardapio
Tela de Bebes
3 itens disponiveis

Refrigerante
Lata 350ml
R$ 6,00

Suco Natural
Suco natural da fruta
R$ 9,00

Agua Mineral
Garrafa 500ml
R$ 4,00
```

O importante aqui não é a aparência final. O foco é entender como uma lista de dados vira conteúdo na tela.

## 3. Contexto

Um app de comanda precisa mostrar produtos disponíveis para venda.

No mundo real, esses produtos normalmente vêm de uma API ou de um banco de dados. Mas antes de buscar dados externos, é melhor começar com uma lista fixa no próprio código.

Isso ajuda a entender três coisas importantes:

- Como representar um produto.
- Como guardar comidas e bebidas em listas.
- Como transformar essas listas em elementos visuais na tela.

Depois que isso estiver claro, buscar produtos de uma API fica muito mais simples.

## 4. Explicação conceitual

Antes de mexer nas telas, precisamos organizar os dados do cardápio.

Uma comida e uma bebida têm praticamente o mesmo formato para o app: identificador, nome, ingredientes, preço, código de barras, imagem e situação. Por isso, vamos criar um tipo chamado `Produto`. Ele serve como o formato combinado que todo item do cardápio deve seguir.

Nesta aula, alguns campos aparecem na tela e outros ainda ficam guardados para uso futuro:

- `id`: identifica o produto dentro da lista.
- `nome`: nome que aparece no cardápio.
- `ingredientes`: lista de textos com os ingredientes ou detalhes do item.
- `preco`: valor numérico do produto.
- `codBarras`: código que pode ser usado em leitura, busca ou integração.
- `imagem`: endereço de uma imagem do produto.
- `ativo`: indica se o produto está disponível.

Depois disso, vamos colocar as listas em um arquivo de service. A tela não precisa saber se os dados vieram de um array local, de um arquivo ou de uma API. Ela só precisa chamar uma função e exibir.

Com os dados chegando na tela, usamos `.map()` para transformar cada produto em um item visual do Ionic.

Também vamos aplicar classes CSS nas telas. Isso deixa o resultado mais próximo do app real: fundo claro, cards para os produtos, preço em destaque e um topo mostrando quantos itens existem naquela categoria.

## 5. Setup inicial

Esta aula continua a partir da Aula 1.

Os arquivos usados nesta aula são:

```text
src/types/produto.ts
src/services/produtoService.ts
src/pages/cardapio.css
src/pages/TelaComes/TelaComes.tsx
src/pages/TelaBebes/TelaBebes.tsx
```

Para rodar o projeto:

No Linux, entre na pasta do projeto com um caminho parecido com:

```bash
cd /caminho/para/a/pasta-do-projeto
```

No Windows, entre na pasta do projeto com um caminho parecido com:

```bash
cd C:\caminho\para\a\pasta-do-projeto
```

Depois execute:

```bash
ionic serve
```

Depois, acesse a tela de comidas:

```text
http://localhost:8100/comes
```

E também a tela de bebidas:

```text
http://localhost:8100/bebes
```

## 6. Passo a passo

### 6.1 Começar pelo caminho dos dados

Vamos começar pelo que vem antes da tela: o formato dos dados.

Primeiro criamos o `Produto`, porque comidas e bebidas precisam seguir o mesmo padrão. Em seguida criamos funções de service para entregar esses dados para o app. Só depois disso entramos em `TelaComes` e `TelaBebes`.

Assim fica mais fácil entender o papel de cada arquivo: tipo define o formato, service fornece os dados, tela exibe o resultado.

### 6.2 Criar a pasta de tipos

Dentro de `src`, crie a pasta:

```text
src/types
```

Essa pasta guarda tipos que podem ser usados em várias partes do projeto.

Por que fazer isso?

- A tela fica responsável por renderizar interface.
- O tipo fica responsável por descrever o formato dos dados.
- Outros arquivos podem reutilizar `Produto` depois, sem copiar e colar código.

### 6.3 Criar o tipo `Produto`

Dentro de `src/types`, crie o arquivo:

```text
src/types/produto.ts
```

Antes de escrever o código, veja os tipos usados:

- `number`: número, usado em `id` e `preco`.
- `string`: texto, usado em `nome`, `codBarras` e `imagem`.
- `string[]`: lista de textos, usada em `ingredientes`.
- `boolean`: verdadeiro ou falso, usado em `ativo`.
- `type`: cria um formato próprio para um objeto.

Depois escreva:

```ts
export type Produto = {
  id: number;
  nome: string;
  ingredientes: string[];
  preco: number;
  codBarras: string;
  imagem: string;
  ativo: boolean;
};
```

O que esse código faz:

- Cria um formato chamado `Produto`.
- Define quais campos todo produto precisa ter.
- Ajuda o editor a sugerir propriedades como `produto.nome`, `produto.ingredientes` e `produto.preco`.
- Evita erros simples, como escrever `produto.descricao` quando o campo correto é `produto.ingredientes`.
- Usa `export` para permitir que esse tipo seja importado em outros arquivos.

### 6.4 Entender o campo `ingredientes`

O campo `ingredientes` não é um texto único. Ele é uma lista de textos:

```ts
ingredientes: ['Pao', 'Hamburguer', 'Queijo']
```

Isso é diferente de:

```ts
ingredientes: 'Pao, Hamburguer, Queijo'
```

Usar lista é melhor porque, mais tarde, o app pode trabalhar com cada ingrediente separadamente. Por exemplo: remover um ingrediente, exibir tags ou montar uma tela de detalhes.

Na hora de mostrar na tela, vamos transformar essa lista em texto usando `.join(', ')`.

### 6.5 Criar a pasta de services

Dentro de `src`, crie a pasta:

```text
src/services
```

Essa pasta guarda arquivos responsáveis por fornecer dados para o app.

Nesta aula, o service ainda vai usar arrays locais. Mesmo assim, essa separação já deixa o projeto preparado para trocar os arrays por uma chamada de API depois.

### 6.6 Criar o arquivo de service

Dentro de `src/services`, crie o arquivo:

```text
src/services/produtoService.ts
```

No topo do arquivo, importe o tipo `Produto`:

```ts
import type { Produto } from '../types/produto';
```

O `import type` deixa claro que esse import existe apenas para tipagem. Ele não representa um componente visual nem uma função que roda no navegador.

### 6.7 Criar a lista de comidas

No mesmo arquivo `produtoService.ts`, crie a lista de comidas:

```ts
const comidas: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    ingredientes: ['Torrada tradicional da casa'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 20,
  },
  {
    id: 2,
    nome: 'Xis Salada',
    ingredientes: ['Pao', 'Hamburguer', 'Queijo', 'Alface', 'Tomate', 'Maionese'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 28,
  },
  {
    id: 3,
    nome: 'Batata Frita',
    ingredientes: ['Porcao media de batata frita'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 18,
  },
];
```

O trecho `Produto[]` significa:

```text
uma lista de Produto
```

Então, cada item dentro do array precisa respeitar o formato definido no tipo `Produto`.

### 6.8 Criar a lista de bebidas

Abaixo da lista de comidas, crie a lista de bebidas:

```ts
const bebidas: Produto[] = [
  {
    id: 1,
    nome: 'Refrigerante',
    ingredientes: ['Lata 350ml'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 6,
  },
  {
    id: 2,
    nome: 'Suco Natural',
    ingredientes: ['Suco natural da fruta'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 9,
  },
  {
    id: 3,
    nome: 'Agua Mineral',
    ingredientes: ['Garrafa 500ml'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 4,
  },
];
```

Repare que comida e bebida usam o mesmo tipo. Isso simplifica o app, porque as telas podem trabalhar com produtos sem precisar de dois formatos diferentes.

### 6.9 Exportar as funções do service

Ainda em `produtoService.ts`, exporte uma função para cada lista:

```ts
export const listarComidas = () => {
  return comidas;
};

export const listarBebidas = () => {
  return bebidas;
};
```

O que esse código faz:

- `listarComidas` entrega a lista de comidas.
- `listarBebidas` entrega a lista de bebidas.
- As telas não precisam saber onde os dados estão guardados.
- Mais tarde, essas funções podem trocar o array por uma chamada `fetch`.

### 6.10 Usar o service na tela `TelaComes`

Volte para:

```text
src/pages/TelaComes/TelaComes.tsx
```

Abaixo dos imports do Ionic, importe o service:

```tsx
import { listarComidas } from '../../services/produtoService';
```

Depois busque as comidas:

```tsx
const comidas = listarComidas();
```

Neste momento, a chamada ainda é local. Mesmo assim, a tela já passa a depender de uma função do service, não de um array escrito dentro dela.

### 6.11 Criar uma função para formatar o preço

O preço está salvo como número:

```tsx
preco: 20
```

Mas na tela queremos exibir como moeda:

```text
R$ 20,00
```

Para isso, crie a função:

```tsx
const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};
```

O que acontece aqui:

- `preco` entra como número.
- `toLocaleString` formata esse número de acordo com o padrão brasileiro.
- `style: 'currency'` indica que queremos formato de moeda.
- `currency: 'BRL'` indica Real brasileiro.

Essa função evita repetir a mesma lógica em vários lugares.

### 6.12 Transformar ingredientes em texto

Como `ingredientes` é uma lista, não vamos usar:

```tsx
{produto.ingredientes}
```

Vamos usar:

```tsx
{produto.ingredientes.join(', ')}
```

O `.join(', ')` pega todos os textos da lista e junta em uma única frase, separando cada item por vírgula e espaço.

Exemplo:

```ts
['Pao', 'Hamburguer', 'Queijo'].join(', ')
```

Resultado:

```text
Pao, Hamburguer, Queijo
```

### 6.13 Renderizar a lista de comidas com `.map()`

Dentro do `IonList`, troque o item fixo por:

```tsx
{comidas.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>
      <h2>{produto.nome}</h2>
      <p>{produto.ingredientes.join(', ')}</p>
      <p>{formatarPreco(produto.preco)}</p>
    </IonLabel>
  </IonItem>
))}
```

O que cada parte faz:

- `comidas.map(...)`: percorre todos os produtos da lista de comidas.
- `(produto) => (...)`: recebe um produto por vez.
- `IonItem`: cria uma linha visual para o produto.
- `key={produto.id}`: identifica cada item para o React.
- `{produto.nome}`: mostra o nome.
- `{produto.ingredientes.join(', ')}`: mostra os ingredientes em formato de texto.
- `{formatarPreco(produto.preco)}`: mostra o preço formatado.

### 6.14 Usar o service na tela `TelaBebes`

Agora abra:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Importe o mesmo service:

```tsx
import { listarBebidas } from '../../services/produtoService';
```

Depois busque as bebidas:

```tsx
const bebidas = listarBebidas();
```

Repare que `Produto` serve para comida e bebida. O tipo fica no arquivo `produto.ts`, os dados ficam no service e as telas apenas renderizam.

### 6.15 Renderizar a lista de bebidas com `.map()`

Na `TelaBebes`, troque o item fixo:

```tsx
<IonItem>
  <IonLabel>Bebida 1</IonLabel>
</IonItem>
```

Por:

```tsx
{bebidas.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>
      <h2>{produto.nome}</h2>
      <p>{produto.ingredientes.join(', ')}</p>
      <p>{formatarPreco(produto.preco)}</p>
    </IonLabel>
  </IonItem>
))}
```

A lógica é a mesma da tela `Comes`:

- `bebidas.map(...)`: percorre a lista de bebidas.
- `produto`: representa uma bebida por vez.
- `IonItem`: cria uma linha para cada bebida.
- `produto.ingredientes.join(', ')`: transforma a lista de ingredientes em texto.
- `formatarPreco(produto.preco)`: exibe o preço em Real.

### 6.16 Criar o CSS compartilhado

Como `TelaComes` e `TelaBebes` mostram o mesmo tipo de informação, podemos usar um único arquivo CSS para as duas telas.

Crie o arquivo:

```text
src/pages/cardapio.css
```

Esse arquivo vai guardar as classes visuais do cardápio.

Comece com:

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
```

Alguns componentes do Ionic usam variáveis CSS próprias. Elas começam com `--`, como `--background` e `--border-color`. Essa é a forma correta de ajustar partes internas de componentes como `IonToolbar`, `IonContent` e `IonItem`.

### 6.17 Criar o container e o topo da tela

Ainda em `cardapio.css`, adicione:

```css
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
```

Essas classes criam uma área centralizada, um título para a tela e uma linha informando a quantidade de itens disponíveis.

### 6.18 Estilizar a lista e os produtos

No mesmo arquivo, adicione:

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

Aqui a lista deixa de parecer uma listagem simples e passa a ter aparência de cardápio: cada produto fica em um card branco, com nome, ingredientes e preço separados visualmente.

### 6.19 Importar o CSS nas telas

Em `TelaComes.tsx`, importe o CSS logo abaixo do service:

```tsx
import { listarComidas } from '../../services/produtoService';
import '../cardapio.css';
```

Faça o mesmo em `TelaBebes.tsx`:

```tsx
import { listarBebidas } from '../../services/produtoService';
import '../cardapio.css';
```

Sem esse import, o arquivo CSS existe, mas as classes não são carregadas pela tela.

### 6.20 Aplicar as classes em `TelaComes`

Agora ajuste a estrutura da página para usar as classes:

```tsx
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
            </IonLabel>
          </IonItem>
        ))}
      </IonList>
    </div>
  </IonContent>
</IonPage>
```

O trecho `{comidas.length}` conta quantos itens existem na lista de comidas.

### 6.21 Aplicar as classes em `TelaBebes`

A tela de bebidas usa a mesma estrutura. A diferença é que ela chama `bebidas.length` e percorre a lista `bebidas`:

```tsx
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
            </IonLabel>
          </IonItem>
        ))}
      </IonList>
    </div>
  </IonContent>
</IonPage>
```

Esse reaproveitamento é importante: as duas telas têm conteúdos diferentes, mas usam o mesmo padrão visual.

### 6.22 Entender os campos que ainda não aparecem na tela

Nesta aula, a tela mostra `nome`, `ingredientes` e `preco`.

Mesmo assim, o produto também guarda:

- `codBarras`: pode ser usado depois para busca, leitura de código ou integração.
- `imagem`: pode ser usada depois para mostrar uma foto do produto.
- `ativo`: pode ser usado depois para esconder produtos indisponíveis.

Esses campos já ficam no tipo e no service porque fazem parte do modelo real de um produto de cardápio. A interface ainda não precisa exibir tudo.

### 6.23 Entender o fluxo completo

O fluxo da tela fica assim:

```text
src/types/produto.ts
    define o formato completo do produto

produtoService.ts
    guarda as listas e expõe funções para buscar dados

cardapio.css
    define a aparência compartilhada das telas

TelaComes.tsx
    chama listarComidas()

comidas.map()
    transforma cada comida em um IonItem

TelaBebes.tsx
    chama listarBebidas()

bebidas.map()
    transforma cada bebida em um IonItem
```

Essa é uma base muito comum em React: criar uma lista de dados e transformar essa lista em interface.

## 7. Código completo

Arquivo:

```text
src/types/produto.ts
```

Código completo:

```ts
export type Produto = {
  id: number;
  nome: string;
  ingredientes: string[];
  preco: number;
  codBarras: string;
  imagem: string;
  ativo: boolean;
};
```

Arquivo:

```text
src/services/produtoService.ts
```

Código completo:

```ts
import type { Produto } from '../types/produto';

const comidas: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    ingredientes: ['Torrada tradicional da casa'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 20,
  },
  {
    id: 2,
    nome: 'Xis Salada',
    ingredientes: ['Pao', 'Hamburguer', 'Queijo', 'Alface', 'Tomate', 'Maionese'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 28,
  },
  {
    id: 3,
    nome: 'Batata Frita',
    ingredientes: ['Porcao media de batata frita'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 18,
  },
];

const bebidas: Produto[] = [
  {
    id: 1,
    nome: 'Refrigerante',
    ingredientes: ['Lata 350ml'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 6,
  },
  {
    id: 2,
    nome: 'Suco Natural',
    ingredientes: ['Suco natural da fruta'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 9,
  },
  {
    id: 3,
    nome: 'Agua Mineral',
    ingredientes: ['Garrafa 500ml'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 4,
  },
];

export const listarComidas = () => {
  return comidas;
};

export const listarBebidas = () => {
  return bebidas;
};
```

Arquivo:

```text
src/pages/cardapio.css
```

Código completo:

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

Arquivo:

```text
src/pages/TelaComes/TelaComes.tsx
```

Código completo:

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
                  <p className="cardapio-descricao">{produto.ingredientes.join(', ')}</p>
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

Arquivo:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Código completo:

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
                  <p className="cardapio-descricao">{produto.ingredientes.join(', ')}</p>
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

## 8. Erros comuns

### 8.1 Esquecer a `key`

Errado:

```tsx
{produtos.map((produto) => (
  <IonItem>
    <IonLabel>{produto.nome}</IonLabel>
  </IonItem>
))}
```

Correto:

```tsx
{produtos.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>{produto.nome}</IonLabel>
  </IonItem>
))}
```

Sem `key`, o React costuma mostrar um aviso no console.

### 8.2 Criar produto fora do formato definido

Errado:

```tsx
const produtos: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    valor: 20,
  },
];
```

O campo `valor` não existe no tipo `Produto`. O campo correto é `preco`.

Correto:

```tsx
const produtos: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    ingredientes: ['Torrada tradicional da casa'],
    codBarras: '1234567890',
    imagem: 'https://via.placeholder.com/150',
    ativo: true,
    preco: 20,
  },
];
```

### 8.3 Usar `descricao` quando o tipo usa `ingredientes`

Errado:

```tsx
<p>{produto.descricao}</p>
```

O tipo `Produto` não tem um campo chamado `descricao`.

Correto:

```tsx
<p>{produto.ingredientes.join(', ')}</p>
```

### 8.4 Esquecer que `ingredientes` é uma lista

Evite tentar tratar `ingredientes` como se fosse um texto simples:

```tsx
ingredientes: 'Lata 350ml'
```

Prefira:

```tsx
ingredientes: ['Lata 350ml']
```

Assim o campo respeita o tipo `string[]`.

### 8.5 Salvar preço como texto

Evite:

```tsx
preco: 'R$ 20,00'
```

Prefira:

```tsx
preco: 20
```

O preço deve ficar como número para permitir cálculos depois. A formatação para `R$ 20,00` deve acontecer apenas na hora de exibir.

### 8.6 Esquecer as chaves ao escrever JavaScript dentro do JSX

Errado:

```tsx
<h2>produto.nome</h2>
```

Isso mostra o texto literal `produto.nome`.

Correto:

```tsx
<h2>{produto.nome}</h2>
```

As chaves `{}` permitem escrever JavaScript dentro do JSX.

### 8.7 Esquecer de exportar ou importar o tipo

Errado em `src/types/produto.ts`:

```ts
type Produto = {
  id: number;
  nome: string;
  ingredientes: string[];
  preco: number;
  codBarras: string;
  imagem: string;
  ativo: boolean;
};
```

Sem `export`, outro arquivo não consegue usar esse tipo.

Correto:

```ts
export type Produto = {
  id: number;
  nome: string;
  ingredientes: string[];
  preco: number;
  codBarras: string;
  imagem: string;
  ativo: boolean;
};
```

Errado em `produtoService.ts`:

```ts
import { Produto } from '../types/produto';
```

Para tipos, prefira:

```ts
import type { Produto } from '../types/produto';
```

O `import type` deixa o código mais explícito: esse import existe só para o TypeScript entender o formato dos dados.

### 8.8 Esquecer de importar o CSS da tela

Errado:

```tsx
import { listarComidas } from '../../services/produtoService';
```

Se o arquivo `cardapio.css` não for importado, as classes aparecem no JSX, mas o visual não muda.

Correto:

```tsx
import { listarComidas } from '../../services/produtoService';
import '../cardapio.css';
```

## 9. Resumo

Nesta aula, as telas `Comes` e `Bebes` passaram a trabalhar com listas de produtos.

O ponto principal é entender que:

- `src/types/produto.ts` guarda o tipo `Produto`.
- `export type Produto` permite reutilizar o tipo em outros arquivos.
- `Produto` tem `id`, `nome`, `ingredientes`, `preco`, `codBarras`, `imagem` e `ativo`.
- `src/services/produtoService.ts` concentra as listas de comidas e bebidas.
- `src/pages/cardapio.css` concentra o visual compartilhado das telas.
- `import type` importa um tipo sem misturar com código visual.
- `Produto[]` representa uma lista de produtos.
- `.map()` transforma cada produto em um item visual.
- `key` ajuda o React a controlar listas.
- `ingredientes.join(', ')` transforma a lista de ingredientes em texto.
- `className` aplica as classes CSS nos componentes React e Ionic.
- Preço deve ser armazenado como número e formatado apenas para exibição.

Com isso, o app começa a sair de telas fixas e passa a representar dados reais de uma comanda.

## 10. Próximo passo

Na próxima aula, o projeto pode evoluir usando mais campos do produto na interface.

As listas já estão funcionando e as telas já têm uma aparência inicial. Agora faz sentido usar campos como `imagem`, `ativo` e `codBarras` para deixar o cardápio mais completo.
