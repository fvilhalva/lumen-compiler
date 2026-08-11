# lumen-compiler

Analisador léxico para a linguagem **Lumen**, desenvolvido para a disciplina de **Compiladores** (`<instituição>`, `<semestre/ano>`).

Esta é a **Fase 1 (Análise Léxica)** de um compilador construído incrementalmente ao longo da disciplina. O scanner lê o código-fonte, reconhece os *tokens* da linguagem e produz um fluxo de tokens (com tipo, lexema, linha e coluna) que alimentará as fases seguintes (sintática e semântica).

> Implementação em Python, com o scanner escrito manualmente como um **autômato finito determinístico (AFD)** — cada rotina de reconhecimento (`_number`, `_identifier`, `_string`, ...) corresponde diretamente a um AFD, permitindo mapear código ↔ estados de transição.

---

## Sumário

- [Visão geral](#visão-geral)
- [Especificação léxica](#especificação-léxica)
- [Arquitetura](#arquitetura)
- [Como executar](#como-executar)
- [Formato de saída](#formato-de-saída)
- [Tratamento de erros](#tratamento-de-erros)
- [Testes](#testes)
- [Estrutura de diretórios](#estrutura-de-diretórios)
- [Roadmap](#roadmap)
- [Autor](#autor)

---

## Visão geral

| Item | Descrição |
|------|-----------|
| **Fase** | Análise léxica (scanner / tokenizer) |
| **Linguagem de implementação** | Python 3.10+ |
| **Linguagem-alvo** | Lumen (`<descreva: subset de C / linguagem própria>`) |
| **Abordagem** | AFD implementado manualmente (loop caractere-a-caractere com *lookahead*) |
| **Saída** | Fluxo de tokens `⟨tipo, lexema, linha, coluna⟩` + tabela de símbolos |

---

## Especificação léxica

### Palavras reservadas

```
if   else   while   for   return   int   float   bool   true   false   void
```
> `<ajuste conforme a gramática da sua linguagem>`

### Categorias de tokens

| Categoria | Descrição | Exemplos |
|-----------|-----------|----------|
| `KEYWORD` | Palavras reservadas | `if`, `while`, `return` |
| `IDENT` | Identificadores | `contador`, `_x`, `soma2` |
| `NUMBER` | Literais numéricos | `42`, `3.14`, `1e-9` |
| `STRING` | Literais de string | `"olá"` |
| `OP` | Operadores | `+  -  *  /  ==  !=  <=  >=  =` |
| `DELIM` | Delimitadores | `(  )  {  }  ;  ,` |

### Regras (informais)

- **Identificador:** `[A-Za-z_][A-Za-z0-9_]*`
- **Número:** parte inteira obrigatória, parte fracionária e expoente opcionais — `\d+(\.\d+)?([eE][+-]?\d+)?`
- **String:** delimitada por aspas duplas, com escapes `\n`, `\t`, `\"`, `\\`
- **Comentários:** de linha `// ...` e de bloco `/* ... */` (ignorados pelo scanner)
- **Espaços em branco:** ignorados, mas usados para contagem de linha/coluna

---

## Arquitetura

O scanner é um AFD codificado manualmente. O laço principal despacha para uma sub-rotina conforme o caractere corrente, e cada sub-rotina percorre os estados do autômato daquele token:

```
código-fonte ──► [ Lexer ] ──► fluxo de Tokens ──► (Fase 2: Parser)
                     │
                     └──► Tabela de Símbolos
                     └──► Relatório de erros léxicos
```

- **`Token`** — estrutura imutável com `tipo`, `lexema`, `linha`, `coluna`.
- **`Lexer`** — mantém `pos`, `linha`, `coluna`; expõe `tokenize()` que retorna a lista/gerador de tokens.
- **`SymbolTable`** — registra identificadores encontrados (base para a fase semântica).
- **`LexError`** — exceção com posição precisa do erro.

---

## Como executar

**Requisitos:** Python 3.10 ou superior (sem dependências externas).

```bash
# clonar
git clone https://github.com/<usuário>/lumen-compiler.git
cd lumen-compiler

# executar o léxico sobre um arquivo-fonte
python -m lexer examples/hello.lm

# ou, se preferir script direto
python lexer/main.py examples/hello.lm
```

Exemplo de entrada (`examples/hello.lm`):

```c
int soma(int a, int b) {
    return a + b;   // retorna a soma
}
```

---

## Formato de saída

Cada token é impresso como `⟨TIPO, lexema, linha, coluna⟩`:

```
⟨KEYWORD, 'int',   1,  1⟩
⟨IDENT,   'soma',  1,  5⟩
⟨DELIM,   '(',     1,  9⟩
⟨KEYWORD, 'int',   1, 10⟩
⟨IDENT,   'a',     1, 14⟩
⟨DELIM,   ',',     1, 15⟩
...
⟨EOF,     '',      3,  2⟩
```

Tabela de símbolos (identificadores):

| # | Lexema | 1ª ocorrência (linha, col) |
|---|--------|----------------------------|
| 1 | `soma` | (1, 5)  |
| 2 | `a`    | (1, 14) |
| 3 | `b`    | (1, 21) |

---

## Tratamento de erros

O scanner reporta a posição exata do erro e continua ou aborta conforme configurado:

```
erro léxico: caractere inesperado '@' na linha 4, coluna 12
erro léxico: número malformado '3.' na linha 7, coluna 5
erro léxico: string não terminada iniciada na linha 9, coluna 3
```

Casos de borda cobertos:

- Número com ponto sem dígito fracionário (`3.`)
- String sem fechamento até o fim do arquivo
- Comentário de bloco não terminado (`/* ...`)
- Caractere fora do alfabeto da linguagem

---

## Testes

```bash
python -m pytest tests/ -v
```

A suíte cobre: identificadores vs. palavras reservadas, números (inteiro/real/notação científica), operadores compostos (`==`, `<=`), strings com escape, comentários, e entradas malformadas.

---

## Estrutura de diretórios

```
lumen-compiler/
├── lexer/
│   ├── __init__.py
│   ├── main.py          # ponto de entrada (CLI)
│   ├── token.py         # TokenType, Token
│   ├── lexer.py         # o AFD / scanner
│   └── symbol_table.py
├── examples/
│   └── hello.lm
├── tests/
│   └── test_lexer.py
├── docs/
│   └── especificacao.md # gramática léxica formal
└── README.md
```

---

## Roadmap

- [x] **Fase 1 — Análise léxica** (este repositório)
- [ ] **Fase 2 — Análise sintática** (parser / AST)
- [ ] **Fase 3 — Análise semântica** (verificação de tipos, escopo)
- [ ] **Fase 4 — Geração de código intermediário**

---

## Autor

**`<seu nome>`** — `<curso>`, `<instituição>`
Disciplina de Compiladores — `<professor(a)>`, `<semestre/ano>`
