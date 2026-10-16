# Flix API

API REST em desenvolvimento para gerenciar gêneros, atores, filmes e
avaliações. O projeto usa Django e Django REST Framework, com rotas sob
`/api/v1/` e autenticação JWT.

## Tecnologias

- Python (o projeto não fixa uma versão mínima).
- Django 6.1.1.
- Django REST Framework 3.18.1.
- djangorestframework-simplejwt 5.5.1.
- SQLite no arquivo local `db.sqlite3`.

As dependências Python estão fixadas em `requirements.txt`.

## Recursos e modelo de dados

| Recurso | Campos e comportamento |
| --- | --- |
| Gênero | `id`, `name` (até 200 caracteres). |
| Ator | `id`, `name` (até 200 caracteres), `birthday` (opcional) e `nationality` (opcional: `USA` ou `BRAZIL`). |
| Filme | `id`, `title` (até 255 caracteres), `genre` (gênero obrigatório), `release_date` (opcional), `actor` (lista de atores) e `resume` (opcional; máximo de 500 caracteres validado pelo serializer). |
| Avaliação | `id`, `movie` (filme obrigatório), `stars` (inteiro entre 0 e 5) e `comment` (opcional). |

Filmes pertencem a um gênero e podem se relacionar com vários atores.
Avaliações pertencem a um filme. A resposta de filme inclui ainda `rate`,
calculado como a média das estrelas e arredondado para uma casa decimal; o
campo é somente leitura. Atualmente, o serializer também retorna `null` quando
a média é `0.0`, além de quando o filme não tem avaliações.

## Autenticação e permissões

Os endpoints de recursos exigem autenticação. A API usa JWT do
`djangorestframework-simplejwt`; envie o access token no cabeçalho:

```http
Authorization: Bearer <access-token>
```

| Endpoint | Método | Finalidade |
| --- | --- | --- |
| `POST /api/v1/authentication/token/` | POST | Obter access e refresh tokens usando as credenciais de um usuário existente. |
| `POST /api/v1/authentication/token/refresh/` | POST | Obter um novo access token usando um refresh token. |
| `POST /api/v1/authentication/token/verify/` | POST | Verificar um token. |

O access token expira em 5 minutos e o refresh token em 1 dia. O projeto não
implementa cadastro de usuários.

Todos os recursos exigem um usuário autenticado, mas as permissões de escrita
e leitura não são uniformes:

- **Atores e filmes:** qualquer usuário autenticado pode listar, consultar,
  criar, atualizar e excluir.
- **Gêneros e avaliações:** além de estar autenticado, o usuário precisa da
  permissão Django correspondente à ação e ao modelo: `view`, `add`, `change`
  ou `delete` (por exemplo, `genres.view_genre`). Os superusuários têm essas
  permissões.

Não há filtragem de objetos por usuário: as listagens retornam os registros
acessíveis pelo queryset completo de cada recurso.

## Endpoints de recursos

Todos os endpoints abaixo aceitam e retornam JSON. Substitua `<id>` pelo
identificador do registro; as rotas terminam com `/`.

| Recurso | Listar / criar | Consultar / atualizar / excluir |
| --- | --- | --- |
| Gêneros | `GET`, `POST /api/v1/genres/` | `GET`, `PUT`, `PATCH`, `DELETE /api/v1/genres/<id>/` |
| Atores | `GET`, `POST /api/v1/actors/` | `GET`, `PUT`, `PATCH`, `DELETE /api/v1/actors/<id>/` |
| Filmes | `GET`, `POST /api/v1/movies/` | `GET`, `PUT`, `PATCH`, `DELETE /api/v1/movies/<id>/` |
| Avaliações | `GET`, `POST /api/v1/reviews/` | `GET`, `PUT`, `PATCH`, `DELETE /api/v1/reviews/<id>/` |

As rotas usam as views genéricas do Django REST Framework. As respostas usuais
são `200 OK` para listagem, consulta e atualização, `201 Created` para criação
e `204 No Content` para exclusão; dados inválidos normalmente retornam
`400 Bad Request`, e IDs inexistentes retornam `404 Not Found`.

### Exemplos de criação

Os exemplos de recursos pressupõem um access token válido. Para criar relações,
envie os IDs dos registros relacionados.

Gênero — `POST /api/v1/genres/`:

```json
{
  "name": "Drama"
}
```

Ator — `POST /api/v1/actors/`:

```json
{
  "name": "Nome do ator",
  "birthday": "1990-04-20",
  "nationality": "BRAZIL"
}
```

Filme — `POST /api/v1/movies/`:

```json
{
  "title": "Título do filme",
  "genre": 1,
  "release_date": "2024-05-10",
  "actor": [1],
  "resume": "Resumo do filme."
}
```

Avaliação — `POST /api/v1/reviews/`:

```json
{
  "movie": 1,
  "stars": 5,
  "comment": "Excelente filme."
}
```

Para obter tokens, envie as credenciais de um usuário cadastrado para
`POST /api/v1/authentication/token/`:

```json
{
  "username": "usuario",
  "password": "senha"
}
```

Use `Content-Type: application/json` nos requests JSON. Os serializers expõem
os campos dos modelos; `rate` é a exceção e é somente leitura.

## Validações e exclusões

- A data de lançamento não pode ser anterior a 1900.
- O resumo do filme tem limite de 500 caracteres no serializer.
- A nota da avaliação deve estar entre 0 e 5, inclusive.
- Gênero e filme são obrigatórios para criar filme e avaliação,
  respectivamente.
- O gênero de um filme usa `PROTECT`: um gênero associado a filmes não pode
  ser excluído.
- A avaliação também usa `PROTECT` para o filme: um filme com avaliações não
  pode ser excluído.
- Uma exclusão bem-sucedida retorna `204 No Content`, sem corpo.

## Administração

Os modelos `Genre`, `Actor`, `Movie` e `Review` estão registrados no Django
Admin. Crie um superusuário e acesse `/admin/` para administrar os registros e
as permissões dos usuários.

```powershell
python manage.py createsuperuser
```

## Estrutura

```text
.
├── actors/          # Modelo, serializer, views, rotas e admin de atores
├── app/             # Configurações, permissões e roteamento raiz do Django
├── authentication/  # Rotas JWT
├── genres/          # Modelo, serializer, views, rotas e admin de gêneros
├── movies/          # Modelo, serializer, views, rotas e admin de filmes
├── reviews/         # Modelo, serializer, views, rotas e admin de avaliações
├── manage.py
├── requirements.txt
└── README.md
```

## Executar localmente

No PowerShell, na raiz do repositório:

```powershell
py -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

A API fica disponível em `http://127.0.0.1:8000/api/v1/`; por exemplo,
`http://127.0.0.1:8000/api/v1/movies/`. O Django REST Framework disponibiliza
uma interface navegável, mas os endpoints continuam sujeitos à autenticação e
às permissões descritas acima.

## Verificações

```powershell
python manage.py check
python manage.py test
python manage.py makemigrations --check --dry-run
```

Os arquivos `tests.py` ainda são apenas placeholders: atualmente
`manage.py test` não executa casos de teste. A verificação de migrações pode
apontar mudanças pendentes.

## Limitações e configuração de produção

- O `.gitignore` ignora arquivos Python sob `*/migrations/`. Portanto, o
  histórico de migrações não é compartilhado pelo Git; `makemigrations` gera
  as migrações locais. Versionar as migrações é necessário para manter o
  histórico do esquema consistente entre ambientes.
- No checkout analisado, `makemigrations --check --dry-run` detecta uma
  alteração pendente no campo `stars` de `Review`, relacionada aos validadores
  do modelo atual e às migrações locais ignoradas.
- A configuração é voltada a desenvolvimento: `DEBUG` está habilitado,
  `ALLOWED_HOSTS` está vazio e `SECRET_KEY` está definida diretamente em
  `app/settings.py`. Não use esses valores em produção. Configure segredos por
  variáveis de ambiente e siga a checklist de implantação do Django.
- O banco configurado é SQLite local.
