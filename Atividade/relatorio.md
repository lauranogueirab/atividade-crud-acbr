# Relatório: CRUD de vagas com SQLite puro

## 1. Como o projeto funciona

### Objetivo e o que é um CRUD

O projeto é um programa de terminal para cadastrar vagas de estágio. Ele usa só Python e SQL, com o módulo `sqlite3` que já vem com o Python, então não precisa de framework nem de instalar nada.

Ele implementa um CRUD, que são as quatro operações básicas com dados:

- **Create** (criar): cadastrar uma vaga nova, com `INSERT`.
- **Read** (ler): listar ou detalhar vagas, com `SELECT`.
- **Update** (atualizar): mudar uma vaga que já existe, com `UPDATE`.
- **Delete** (remover): apagar uma vaga, com `DELETE`.

### O caminho de um cadastro, do menu até o `vagas.db`

1. Ao abrir o programa, o `main()` do `app.py` chama `crud.conectar()`, que abre o arquivo `vagas.db` (ou cria o arquivo, se ele ainda não existe). Depois chama `crud.criar_tabela()`, que executa o `sql/schema.sql` para garantir que a tabela `vaga` existe.
2. No menu, digito `1` (Cadastrar). O `main()` procura a opção no dicionário `OPCOES` e chama a função `cadastrar`.
3. A função `ler_dados()` pergunta título, empresa e telefone. Cada resposta passa por `.strip()`, que tira os espaços do começo e do fim.
4. O `cadastrar` manda esses três valores para `crud.criar_vaga()`.
5. A `criar_vaga` deixa o título em maiúsculas com `upper()` e executa o `INSERT` usando `?` no lugar dos valores.
6. Antes de gravar, o SQLite confere as regras da tabela (`NOT NULL` e `CHECK`). Se estiver tudo certo, o `commit()` salva a vaga no `vagas.db`. Se alguma regra for violada, o banco recusa e o erro `sqlite3.IntegrityError` é tratado no `main()`.
7. O banco gera o id da vaga, a função o devolve e o `app.py` mostra "Vaga cadastrada com o id X".

### O papel de cada arquivo

- **`src/app.py`**: é a parte que conversa com o usuário. Mostra o menu, lê o que eu digito, chama o `crud.py` e exibe os resultados e os erros.
- **`src/crud.py`**: é a parte que conversa com o banco. Tem a conexão e as funções com o SQL de cada operação: `criar_vaga`, `listar_vagas`, `buscar_vaga`, `atualizar_vaga` e `remover_vaga`.
- **`sql/schema.sql`**: define a tabela `vaga` e as regras de validação. O programa o executa toda vez que inicia.
- **`sql/dados_exemplo.sql`**: tem `INSERT`s para encher o banco com vagas de teste.
- **`sql/exercicios.sql`**: tem consultas para praticar SQL.
- **`src/test_crud.py`**: testes automáticos do `crud.py`, que rodam num banco em memória e não mexem no `vagas.db`.

## 2. Exemplo de execução

Rodei `python src/app.py` e fiz as operações abaixo, na ordem.

### Passo 1: cadastrar a vaga 1

```
Opção: 1
Título: Estágio em Desenvolvimento de Software
Empresa: Magazine Luiza
Telefone (DDD 19):      19912345678      
Vaga cadastrada com o id 1.
```

Cadastrei a primeira vaga pela opção 1. O banco gravou o registro e devolveu o id 1. Repare que digitei o telefone com espaços nas pontas e ele foi aceito, porque o `strip()` os remove antes de enviar ao banco.

### Passo 2: cadastrar a vaga 2

```
Opção: 1
Título: Estágio em Análise de Dados
Empresa: Itaú Unibanco
Telefone (DDD 19): 19987654321
Vaga cadastrada com o id 2.
```

Cadastrei a segunda vaga do mesmo jeito. Ela recebeu o id 2, o que mostra que o banco numera as vagas sozinho, de um em um.

### Passo 3: listar as vagas

```
Opção: 2
[1] ESTÁGIO EM DESENVOLVIMENTO DE SOFTWARE | Magazine Luiza | 19912345678
[2] ESTÁGIO EM ANÁLISE DE DADOS | Itaú Unibanco | 19987654321
```

A opção 2 leu as vagas do banco e mostrou as duas, cada uma com id, título, empresa e telefone. Os títulos aparecem em maiúsculas porque a `criar_vaga` aplica `upper()` antes de gravar.

### Passo 4: atualizar a vaga 1

```
Opção: 4
ID da vaga: 1
Título: Mercado Livre
Empresa: Mercado Livre
Telefone (DDD 19):     19912345678 
Vaga atualizada.
```

Usei a opção 4 para alterar a vaga de id 1. O programa pede todos os campos de novo, mesmo que eu só queira mudar um, e grava tudo por cima. Nesse teste coloquei "Mercado Livre" no título e na empresa.

### Passo 5: listar depois de atualizar

```
Opção: 2
[1] MERCADO LIVRE | Mercado Livre | 19912345678
[2] ESTÁGIO EM ANÁLISE DE DADOS | Itaú Unibanco | 19987654321
```

Listei de novo para conferir. A vaga 1 mudou e a vaga 2 continua igual, então o `UPDATE` afetou só o registro do id informado.

### Passo 6: remover a vaga 2

```
Opção: 5
ID da vaga: 2
Vaga removida.
```

Com a opção 5, apaguei a vaga de id 2.

### Passo 7: listar depois de remover

```
Opção: 2
[1] MERCADO LIVRE | Mercado Livre | 19912345678
```

Só sobrou a vaga 1, então o `DELETE` apagou apenas o registro que eu pedi.

## 3. Explicação de uma função

Escolhi a `criar_vaga`, do `crud.py`, que faz a parte **Create** do CRUD.

```python
def criar_vaga(conexao, titulo, empresa, telefone):
    """INSERT: grava uma nova vaga e devolve o id gerado."""
    sql = 'INSERT INTO vaga (titulo, empresa, telefone) VALUES (?, ?, ?)'
    cursor = conexao.execute(sql, (titulo.upper(), empresa, telefone))
    conexao.commit()
    return cursor.lastrowid
```

### O comando SQL

```sql
INSERT INTO vaga (titulo, empresa, telefone) VALUES (?, ?, ?)
```

- `INSERT INTO vaga` manda o banco inserir uma linha nova na tabela `vaga`.
- `(titulo, empresa, telefone)` diz quais colunas vão ser preenchidas. O `id` não entra na lista porque o banco gera sozinho (`INTEGER PRIMARY KEY AUTOINCREMENT`).
- `VALUES (?, ?, ?)` reserva três espaços para os valores, na mesma ordem das colunas.

Além do SQL, a função faz duas coisas que merecem ser citadas. O `titulo.upper()` transforma o título em maiúsculas antes de gravar, e é por isso que a listagem mostra tudo em caixa alta. O `conexao.commit()` confirma a gravação no arquivo; sem ele, a vaga não ficaria salva no `vagas.db`.

### Por que usar `?` e não colar os valores no texto

Os valores vão separados do comando, na tupla `(titulo.upper(), empresa, telefone)`, e cada `?` é preenchido na ordem. Isso evita **SQL injection**. Se eu montasse o comando juntando os textos (com uma f-string, por exemplo), alguém poderia digitar um pedaço de SQL num campo e o banco o executaria como comando. Com o `?`, o banco sempre trata o que veio do usuário como dado, nunca como código. O README do projeto tem um teste sobre isso (`test_sql_injection_e_gravado_como_texto`), que mostra um texto malicioso sendo gravado como texto comum.

### O que a função devolve e como o `app.py` usa

A função devolve `cursor.lastrowid`, o id que o banco criou para a vaga nova. No `app.py`, a função `cadastrar` faz assim:

```python
def cadastrar(conexao):
    novo_id = crud.criar_vaga(conexao, *ler_dados())
    print(f'Vaga cadastrada com o id {novo_id}.')
```

O `ler_dados()` devolve título, empresa e telefone, e o `*` entrega cada um como argumento separado para a `criar_vaga`. O id que volta fica em `novo_id` e aparece na mensagem de confirmação, como o "Vaga cadastrada com o id 1" do meu teste.

## 4. Easter egg 🥚

### Como provoquei

Cadastrei uma vaga digitando o telefone com parênteses, `(19) 91234-5678`. O banco recusou:

```
O banco recusou os dados: CHECK constraint failed: telefone LIKE '19%'
```

Depois testei um nome de empresa curto e apareceu outra recusa:

```
O banco recusou os dados: CHECK constraint failed: length(empresa) >= 3
```

### O que é exigido de cada campo

- **Telefone:** precisa começar com `19`. A regra `telefone LIKE '19%'` compara o texto com um padrão: o `19` é fixo e o `%` vale para qualquer coisa que venha depois. Por isso `(19) 91234-5678` falhou (começa com `(`) e `19912345678` passou.
- **Empresa:** precisa ter no mínimo 3 caracteres. A função `length()` conta os caracteres e a condição `>= 3` exige três ou mais. Nomes como "A" ou "AB" são recusados, e um campo vazio também, já que o tamanho seria 0.
- **Título:** não tem regra além de `NOT NULL`, por isso "Mercado Livre" foi aceito como título.

### Onde a regra está e como é escrita

Está no `sql/schema.sql`, dentro do `CREATE TABLE` da tabela `vaga`, como restrições nas colunas:

```sql
empresa  TEXT NOT NULL CHECK (length(empresa) >= 3),
telefone TEXT NOT NULL CHECK (telefone LIKE '19%')
```

O `NOT NULL` obriga o campo a existir, e o `CHECK` define uma condição que todo registro precisa cumprir. Como a `criar_tabela` executa esse arquivo toda vez que o programa abre, as regras sempre estão valendo.

### O que acontece com a aplicação quando a regra é violada

O SQLite se recusa a gravar e levanta o erro `sqlite3.IntegrityError`. Esse erro nasce dentro do `execute` da `criar_vaga`, passa pela função `cadastrar` (que não trata nada) e é capturado no `try/except` do `main()`, que imprime "O banco recusou os dados:" seguido da mensagem original do banco. Como o erro acontece no `execute`, o `commit()` nem chega a rodar e nada é gravado. O programa não fecha: volta para o menu.

### Por que vale a pena ter a regra no banco

Se a regra existisse só no código Python, qualquer outro jeito de gravar dados (outro script, um editor de banco, um arquivo `.sql`) poderia passar por cima dela. Com a regra no banco, ela vale para todo mundo, sempre, e fica escrita num só lugar. No projeto, o Python faz a limpeza (o `strip()` tira espaços e o `upper()` deixa o título em maiúsculas), e a validação final fica a cargo do banco.
