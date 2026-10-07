# Flix API

API REST em desenvolvimento para o projeto Flix. A API utiliza Django REST
Framework e implementa operações CRUD para gêneros, atores, filmes e avaliações.
As rotas da API estão versionadas sob o prefixo `/api/v1/`. Ainda não há
autenticação própria da API.

## Tecnologias

- **Python** — linguagem da aplicação.
- **Django 6.1.1** — framework web e ORM.
- **Django REST Framework 3.18.1** — implementação das views genéricas,
  serializers e respostas da API.
- **SQLite** — banco configurado para desenvolvimento (`db.sqlite3`).
- **Django Admin** — interface administrativa para gerenciar gêneros.

## Funcionalidades

- Operações CRUD para gêneros, atores, filmes e avaliações.
- Gerenciar gêneros pelo painel administrativo do Django.

## Endpoints da API

Todas as rotas da API usam o prefixo `/api/v1/` e aceitam/retornam JSON.
Substitua `<id>` pelo identificador numérico do recurso. As rotas de detalhe
terminam com a barra `/`.

| Método | Caminho | Descrição | Status de sucesso |
| --- | --- | --- | --- |
| `GET` | `/api/v1/genres/` | Lista todos os gêneros. | `200 OK` |
| `POST` | `/api/v1/genres/` | Cria um gênero. | `201 Created` |
| `GET` | `/api/v1/genres/<id>/` | Consulta um gênero. | `200 OK` |
| `PUT` | `/api/v1/genres/<id>/` | Atualiza um gênero. | `200 OK` |
| `PATCH` | `/api/v1/genres/<id>/` | Atualiza parcialmente um gênero. | `200 OK` |
| `DELETE` | `/api/v1/genres/<id>/` | Exclui um gênero. | `204 No Content` |

Os mesmos métodos estão disponíveis para atores, filmes e avaliações:

| Recurso | Lista e criação | Consulta, atualização e exclusão |
| --- | --- | --- |
| Atores | `/api/v1/actors/` | `/api/v1/actors/<id>/` |
| Filmes | `/api/v1/movies/` | `/api/v1/movies/<id>/` |
| Avaliações | `/api/v1/reviews/` | `/api/v1/reviews/<id>/` |

### Listar gêneros

`GET /api/v1/genres/`

Exemplo de resposta:

```json
[
  {
    "id": 1,
    "name": "Drama"
  }
]
```

### Criar gênero

`POST /api/v1/genres/`

Envie um objeto JSON com o cabeçalho `Content-Type: application/json`:

```json
{
  "name": "Drama"
}
```

Exemplo de resposta (`201 Created`):

```json
{
  "id": 1,
  "name": "Drama"
}
```

### Consultar gênero

`GET /api/v1/genres/1/`

Exemplo de resposta:

```json
{
  "id": 1,
  "name": "Drama"
}
```

### Atualizar gênero

`PUT /api/v1/genres/1/`

Envie o nome atualizado em um objeto JSON:

```json
{
  "name": "Aventura"
}
```

Exemplo de resposta:

```json
{
  "id": 1,
  "name": "Aventura"
}
```

### Excluir gênero

`DELETE /api/v1/genres/1/`

A implementação retorna `204 No Content` sem body. Esse status indica que a
exclusão foi concluída e, por definição, não inclui conteúdo ou mensagem na
resposta.

As operações de detalhe retornam `404 Not Found` quando o identificador não
corresponde a um gênero existente.

## Estrutura do projeto

```text
.
├── app/
│   ├── settings.py       # Configurações do Django e do SQLite
│   ├── urls.py           # Prefixo /api/v1/ e inclusão das rotas das apps
│   ├── asgi.py           # Entrada ASGI
│   └── wsgi.py           # Entrada WSGI
├── .gitignore            # Arquivos locais e gerados ignorados pelo Git
├── actors/               # App de atores: modelos, serializers, views e rotas
├── genres/               # App de gêneros: modelos, serializers, views e rotas
├── movies/               # App de filmes: modelos, serializers, views e rotas
├── reviews/              # App de avaliações: modelos, serializers, views e rotas
├── requirements.txt      # Dependências Python fixadas
├── manage.py             # Comandos administrativos do Django
└── db.sqlite3            # Banco de dados local
```

O `.gitignore` exclui do controle de versão o ambiente virtual, caches e
arquivos compilados do Python, o banco SQLite local, arquivos de ambiente
(`.env`), pastas de saída locais e arquivos de editores/sistemas operacionais.
Também ignora os arquivos `.py` dentro das pastas `migrations/`. O arquivo
`.env.example` permanece versionável como modelo sem segredos. Arquivos que já
estavam sendo acompanhados pelo Git continuam versionados até serem removidos
explicitamente do índice.

## Como executar localmente

Requer Python compatível com Django 6.1.1.

No PowerShell, na pasta do projeto:

```powershell
py -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

Como os arquivos de migração não são versionados neste projeto, execute
`makemigrations` ao preparar uma cópia local nova para gerar as migrações a
partir dos modelos. Em seguida, `migrate` aplica essas migrações ao banco de
dados. Quando os modelos forem alterados, execute novamente `makemigrations`
e depois `migrate`.

O `requirements.txt` fixa as versões das dependências diretas e transitivas
registradas para o ambiente atual:

```text
asgiref==3.12.1
Django==6.1.1
djangorestframework==3.18.1
sqlparse==0.6.0
tzdata==2026.4
```

A API ficará disponível em `http://127.0.0.1:8000/`.

Para acessar o painel administrativo, crie um usuário:

```powershell
python manage.py createsuperuser
```

Depois, acesse `http://127.0.0.1:8000/admin/`.

## Testes

Execute os testes do projeto com:

```powershell
python manage.py test
```

## Observações

- As configurações atuais são voltadas ao desenvolvimento local. Antes de
  publicar em produção, revise configurações como `DEBUG`, `SECRET_KEY` e
  `ALLOWED_HOSTS`.
- O arquivo `requirements.txt` foi gerado a partir dos pacotes instalados no
  ambiente (`pip freeze`), então pode incluir dependências transitivas ou
  instaladas que ainda não são utilizadas diretamente no código.
