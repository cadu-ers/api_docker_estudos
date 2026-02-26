### Cenário 1

**Tarefa:** Alterar o banco de dados para o MariaDB



#### O que precisou ser alterado?

Para realizar a migração de **PostgreSQL** para **MariaDB**, foram necessárias modificações em três camadas da aplicação: infraestrutura, código da aplicação e dependências.

---

##### 1. Infraestrutura (Docker Compose)

- Substituição da imagem do banco:
  
  - De: `postgres:16-alpine`
  - Para: `mariadb:11`

- Atualização das variáveis de ambiente:
  
  - `POSTGRES_DB` → `MARIADB_DATABASE`
  - `POSTGRES_USER` → `MARIADB_USER`
  - `POSTGRES_PASSWORD` → `MARIADB_PASSWORD`
  - Porta padrão alterada de `5432` para `3306`

- Alteração do diretório de persistência:
  
  - De: `/var/lib/postgresql/data`
  - Para: `/var/lib/mysql`

- Implementação de controle de inicialização do banco:
  
  - Adição de `healthcheck` no serviço `db`
  - Ajuste no `depends_on` para `condition: service_healthy`

Essas alterações garantiram que a API só iniciasse após o banco estar pronto para conexões.

---

##### 2. Aplicação (FastAPI + SQLAlchemy)

- Alteração da **string de conexão**:

  Antes:
  
`postgresql://usuario:senha@db:5432/database`

      Depois:

`mysql+pymysql://usuario:senha@db:3306/database`

---

***Substituição das variáveis `POSTGRES_*` por `MARIADB_*` no arquivo `main.py`.***

##### 3. Dependências do Projeto

- Remoção do driver PostgreSQL:

`psycopg2-binary`


- Inclusão do driver MariaDB/MySQL:

`pymysql`


Após essas alterações, foi executado:


`docker compose down`
`docker compose up --build`


---

#### Qual foi o impacto na aplicação?

A estrutura da aplicação — incluindo modelos ORM, rotas e regras de negócio — permaneceu inalterada, graças à abstração proporcionada pelo **SQLAlchemy**.

No entanto, alguns impactos técnicos foram observados:

- A aplicação passou a depender do tempo de inicialização do MariaDB.
- Foi necessário implementar controle adicional para evitar falhas no startup.
- Os dados anteriormente persistidos no PostgreSQL não são automaticamente reutilizados no MariaDB.
- Houve necessidade de atualizar o driver e a string de conexão.

Após os ajustes estruturais e de inicialização, a aplicação voltou a operar normalmente, utilizando MariaDB como sistema gerenciador de banco de dados.

---



### Cenário 2

Tarefa: Alterar a porta da aplicação de 8000 para 9000

#### Perguntas

- O que precisou ser alterado?
- Qual foi o impacto na aplicação?

### Cenário 3

Tarefa: Alterar a porta padrão do banco de dados

#### Perguntas

- O que precisou ser alterado?
- Qual foi o impacto na aplicação?


### Cenário 4

Tarefa: Delete o volume do container do banco de dados

#### Perguntas

- Qual foi o impacto na aplicação?
- O que isso impacta em ambientes de produção?


### Cenário 5

Tarefa: Parar apenas o container do banco de dados

#### Perguntas

- Qual foi o comportamento da aplicação?
- Retorne o banco de dados. Como foi o comportamento após essa ação?
- Existe retry de conexão?


### Cenário 6

Tarefa: Criar uma nova rede e mover apenas a API para ela. Após isso, testar a comunicação com o banco de dados.

#### Perguntas

- A API conseguiu acessar o banco de dados?
- Qual a importância disso no aspecto de segurança?

### Cenário 7

Atualmente o banco de dados está na mesma rede interna do Docker e não está exposto publicamente.Imagine que um desenvolvedor decide expor a porta do banco no docker compose para facilitar testes externos.


#### Perguntas

- Quais riscos essa decisão pode gerar em um ambiente real?
- Como um invasor poderia descobrir essa porta aberta?
- Como você mitigaria esse risco?
- Que tipo de ataque pode acontecer?
