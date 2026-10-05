# 🛒 Lista do Mercado

Aplicativo web para usar **durante as compras no supermercado**. Você escreve a lista do que precisa comprar e, conforme vai pegando cada item, digita o preço. O app marca o item como pego, soma tudo automaticamente e, no final, mostra o valor total da compra.

**Acesse:** https://ivancampos517.github.io/lista-mercado/

Funciona no navegador do celular ou do computador, sem instalar nada e sem criar conta.

---

## Sumário

- [Funcionalidades](#funcionalidades)
- [Como usar](#como-usar)
- [Como foi criado](#como-foi-criado)
- [Como funciona por dentro](#como-funciona-por-dentro)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Rodando localmente](#rodando-localmente)
- [Publicação no GitHub Pages](#publicação-no-github-pages)
- [Privacidade](#privacidade)
- [Ideias para próximas versões](#ideias-para-próximas-versões)

---

## Funcionalidades

| Recurso | O que faz |
|---|---|
| **Lista de compras** | Adicione itens um por um ou cole vários de uma vez (um por linha). |
| **Marcar item pego** | Toque no item, digite o preço e a quantidade; ele vai para "No carrinho". |
| **Soma automática** | O total fica fixo no rodapé e é atualizado a cada item pego. |
| **Quantidade** | Aceita unidades e valores com vírgula (ex.: `1,5` kg). O subtotal é preço × quantidade. |
| **Progresso** | Mostra quantos itens da lista já foram pegos (ex.: `4 de 7`) com uma barra de progresso. |
| **Itens fora da lista** | Botão para registrar algo que você pegou e não estava na lista; ele entra na soma e recebe a etiqueta "fora da lista". |
| **Orçamento** | Campo opcional com o limite de gastos. Mostra quanto ainda resta, avisa em laranja ao chegar em 90% e mostra quanto passou se estourar. |
| **Resumo final** | Quando todos os itens da lista são pegos, aparece um resumo em formato de cupom com cada item, o total e a situação do orçamento. |
| **Editar e desfazer** | Toque em um item do carrinho para corrigir o preço, tirá-lo do carrinho ou excluí-lo. |
| **Reutilizar a lista** | "Usar a mesma lista de novo" mantém os itens e zera preços e marcações para a próxima compra. |
| **Salva sozinho** | A lista fica salva no aparelho, então fechar o navegador não apaga nada. |
| **Tema claro e escuro** | Segue automaticamente o tema do celular ou computador. |

---

## Como usar

1. **Monte a lista** antes de sair de casa: digite cada item e toque em **Adicionar**, ou use **Colar vários itens de uma vez**.
2. *(Opcional)* Informe o **orçamento**, por exemplo `300,00`.
3. **No mercado**, ao pegar um produto, toque nele na lista:
   - digite o **preço** (ex.: `4,99`);
   - ajuste a **quantidade**, se precisar;
   - toque em **Pegar item**.
4. Acompanhe o **total** e o **orçamento** no rodapé.
5. Quando pegar tudo, confira o **resumo final** antes de passar no caixa.
6. Na próxima compra, use **Usar a mesma lista de novo** ou **Nova lista**.

> 💡 **Dica:** no celular, abra o link e escolha **"Adicionar à tela inicial"** no menu do navegador. O app ganha um ícone e abre como se fosse um aplicativo instalado.

---

## Como foi criado

O projeto nasceu de uma necessidade simples: saber quanto a compra vai dar **antes** de chegar ao caixa, sem precisar de calculadora nem papel.

Ele foi desenvolvido por **Ivan Campos** com a ajuda do **Claude**, assistente de IA da Anthropic. A ideia foi descrita em português numa conversa, e o app foi construído e ajustado em etapas:

1. **Primeira versão:** lista de itens, marcação com preço e quantidade, soma automática e resumo final.
2. **Orçamento:** limite opcional de gastos com avisos de "quase no limite" e "passou do orçamento".
3. **Publicação:** o app virou uma página independente e foi publicado no GitHub Pages para poder ser compartilhado com qualquer pessoa.

### Escolhas técnicas

- **HTML, CSS e JavaScript puros**, sem frameworks e sem etapa de build. O app inteiro está em um único arquivo (`index.html`), fácil de entender, editar e hospedar.
- **Pensado primeiro para o celular**, porque o uso real é com o celular na mão, no corredor do mercado: botões grandes, teclado numérico no campo de preço e total sempre visível no rodapé.
- **Sem servidor e sem banco de dados.** Os dados ficam no próprio aparelho, então não há custo de hospedagem nem dados pessoais trafegando pela internet.
- **Formato brasileiro de dinheiro:** valores exibidos como `R$ 1.234,56` e digitação aceita tanto `4,99` quanto `4.99`.

---

## Como funciona por dentro

### Dados

Toda a informação do app fica em um único objeto JavaScript:

```js
state = {
  items: [
    {
      id: "k3j9x2a",      // identificador aleatório
      name: "Leite",      // nome do item
      qty: 6,             // quantidade
      price: 4.79,        // preço unitário (null enquanto não foi pego)
      done: true,         // true = está no carrinho
      extra: false        // true = item que não estava na lista original
    }
  ],
  budget: 300,            // orçamento (null se não definido)
  sample: false           // true enquanto a lista de exemplo está na tela
}
```

### Salvamento

A cada alteração, o `state` é convertido para JSON e gravado no **`localStorage`** do navegador com a chave `lista-mercado-v1`. Ao abrir o app, ele lê esses dados de volta. Se não houver nada salvo, mostra uma lista de exemplo.

As leituras e gravações ficam dentro de `try/catch`, então o app continua funcionando mesmo em modo anônimo ou com o armazenamento bloqueado; nesse caso ele só não guarda a lista.

### Cálculos

- **Subtotal de um item:** `preço × quantidade`
- **Total:** soma dos subtotais dos itens com `done: true`
- **Progresso:** itens da lista original já pegos ÷ total de itens da lista original (itens extras não contam)
- **Orçamento:**
  - restante = `orçamento − total`
  - aviso "quase no limite" quando o total chega a 90% do orçamento
  - aviso "passou do orçamento" quando `total > orçamento`

### Leitura de valores digitados

A função `parseNum` aceita os formatos que as pessoas costumam digitar:

| Digitado | Interpretado como |
|---|---|
| `4,99` | 4.99 |
| `4.99` | 4.99 |
| `1.234,56` | 1234.56 |
| `R$ 10` | 10 |

### Exibição de valores

Os valores são formatados com a API nativa do navegador:

```js
new Intl.NumberFormat("pt-BR", { style: "currency", currency: "BRL" })
```

### Interface

- A tela é redesenhada pela função `render()` sempre que algo muda, a partir do `state`.
- Os cliques são tratados por **delegação de eventos**: um único ouvinte no `document` lê o atributo `data-act` do botão (`open`, `confirm`, `undo`, `remove` etc.).
- Os textos digitados pelo usuário passam pela função `esc()` antes de ir para a tela, o que evita injeção de HTML.
- As cores são definidas como variáveis CSS (`--bg`, `--accent`...) com uma versão para o tema claro e outra para o escuro.
- Fontes: **Bricolage Grotesque** (títulos), **Figtree** (textos) e **JetBrains Mono** (valores em dinheiro, com números alinhados), carregadas do Google Fonts.

---

## Estrutura do projeto

```
lista-mercado/
├── index.html   # o app completo: estrutura (HTML), visual (CSS) e lógica (JavaScript)
└── README.md    # este arquivo
```

Dentro do `index.html`:

| Parte | Conteúdo |
|---|---|
| `<head>` | título, metatags para celular e fontes |
| `<style>` | cores, tipografia e layout, incluindo o tema escuro |
| `<body>` | cabeçalho, campo de orçamento, formulário, seções "Falta pegar" e "No carrinho" e rodapé com o total |
| `<script>` | estado, salvamento, cálculos, renderização e eventos |

---

## Rodando localmente

Como é um único arquivo estático, não precisa instalar nada:

1. Clone o repositório:
   ```bash
   git clone https://github.com/Ivancampos517/lista-mercado.git
   ```
2. Abra o arquivo `index.html` no navegador.

Se preferir servir por um servidor local:

```bash
cd lista-mercado
python3 -m http.server 8000
# acesse http://localhost:8000
```

---

## Publicação no GitHub Pages

O site é publicado automaticamente pelo **GitHub Pages** a partir da branch `main`, pasta raiz (`/`).

Para atualizar o app:

1. Edite o `index.html`.
2. Faça commit e push para a branch `main`.
3. Em um ou dois minutos a nova versão aparece no link público.

---

## Privacidade

- O app **não tem servidor**, não usa cadastro e não coleta dados.
- A lista, os preços e o orçamento ficam salvos **somente no navegador do aparelho** de quem usa.
- Cada pessoa que abre o link tem a sua própria lista; ninguém vê a lista de outra pessoa.
- Limpar os dados do navegador apaga a lista salva.

---

## Ideias para próximas versões

- Histórico das compras anteriores com o total de cada uma
- Categorias por setor do mercado (hortifrúti, limpeza, frios...)
- Compartilhar a lista com outra pessoa por link
- Comparar o preço pago com o da última compra
- Funcionar totalmente offline (PWA)

---

Feito por [Ivan Campos](https://github.com/Ivancampos517).
