# Flix API

API REST em desenvolvimento para gerenciar gêneros, atores, filmes e avaliações.
O backend utiliza Django e Django REST Framework, com endpoints versionados sob
o prefixo `/api/v1/`.

## Tecnologias

- **Python**
- **Django 6.1.1** — framework web, ORM e administração.
- **Django REST Framework 3.18.1** — serializers, views genéricas e API navegável.
- **SQLite** — banco de dados local (`db.sqlite3`).

As versões das dependências Python estão fixadas em `requirements.txt`.

## Recursos e modelo de dados

| Recurso | Campos |
| --- | --- |
| Gênero | `id`, `name` (até 200 caracteres) |
| Ator | `id`, `name` (até 200 caracteres), `birthday` (opcional), `nationality` (opcional: `USA` ou `BRAZIL`) |
| Filme | `id`, `title` (até 255 caracteres), `genre` (gênero), `release_date` (opcional), `actor` (lista de atores), `resume` (opcional; até 500 caracteres) e `rate` (média calculada, somente leitura) |
| Avaliação | `id`, `movie` (filme), `stars` (inteiro de 0 a 5), `comment` (opcional) |

Filmes estão associados a um gênero e a zero ou mais atores. Avaliações estão
associadas a um filme. A média (`rate`) é calculada a partir das estrelas das
avaliações e arredondada a uma casa decimal; filmes sem avaliações retornam
`null` para esse campo.

## Endpoints

Todos os endpoints de recursos aceitam e retornam JSON. As rotas de detalhe
terminam com `/`; substitua `<id>` por um identificador existente.

| Recurso | Listar | Criar | Consultar | Atualizar | Atualizar parcialmente | Excluir |
| --- | --- | --- | --- | --- | --- | --- |
| Gêneros | `GET /api/v1/genres/` | `POST /api/v1/genres/` | `GET /api/v1/genres/<id>/` | `PUT /api/v1/genres/<id>/` | `PATCH /api/v1/genres/<id>/` | `DELETE /api/v1/genres/<id>/` |
| Atores | `GET /api/v1/actors/` | `POST /api/v1/actors/` | `GET /api/v1/actors/<id>/` | `PUT /api/v1/actors/<id>/` | `PATCH /api/v1/actors/<id>/` | `DELETE /api/v1/actors/<id>/` |
| Filmes | `GET /api/v1/movies/` | `POST /api/v1/movies/` | `GET /api/v1/movies/<id>/` | `PUT /api/v1/movies/<id>/` | `PATCH /api/v1/movies/<id>/` | `DELETE /api/v1/movies/<id>/` |
| Avaliações | `GET /api/v1/reviews/` | `POST /api/v1/reviews/` | `GET /api/v1/reviews/<id>/` | `PUT /api/v1/reviews/<id>/` | `PATCH /api/v1/reviews/<id>/` | `DELETE /api/v1/reviews/<id>/` |

As rotas usam as views genéricas `ListCreateAPIView` e
`RetrieveUpdateDestroyAPIView` do Django REST Framework. Por padrão, elas
retornam `200 OK` para listagem, consulta e atualização, `201 Created` para
criação, `204 No Content` para exclusão, `400 Bad Request` para dados inválidos
e `404 Not Found` quando o recurso não existe.

### Exemplos de criação

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

Para criar ou atualizar relacionamentos, use os IDs dos registros relacionados.
Envie JSON com `Content-Type: application/json`. Os serializers incluem todos
os campos do respectivo modelo (`fields = '__all__'`); `rate` é somente leitura.

### Validações e exclusões

- A data de lançamento do filme não pode ser anterior a 1900.
- O resumo do filme tem limite de 500 caracteres.
- A nota de uma avaliação deve estar entre 0 e 5.
- O gênero de um filme é obrigatório e usa `PROTECT`: gêneros associados a
  filmes não podem ser excluídos.
- A avaliação exige um filme. O relacionamento também usa `PROTECT`, então um
  filme com avaliações associadas não pode ser excluído.
- Exclusões bem-sucedidas respondem `204 No Content`, sem corpo de resposta.

## Administração

Os modelos `Genre`, `Actor`, `Movie` e `Review` estão registrados no Django
Admin. Crie um usuário administrador e acesse `/admin/` para gerenciar os dados.

## Estrutura

```text
.
├── app/                  # Configurações e roteamento raiz
├── actors/               # Modelo, serializer, views, rotas e admin de atores
├── genres/               # Modelo, serializer, views, rotas e admin de gêneros
├── movies/               # Modelo, serializer, views, rotas e admin de filmes
├── reviews/              # Modelo, serializer, views, rotas e admin de avaliações
├── .gitignore
├── manage.py
├── requirements.txt
└── db.sqlite3            # Banco local, ignorado pelo Git
```

O `.gitignore` exclui ambientes virtuais, caches Python, banco SQLite local,
arquivos `.env`, arquivos de editores e outros arquivos gerados. Também ignora
arquivos Python sob `*/migrations/`. Portanto, migrações locais já existentes
não são enviadas ao repositório; arquivos que já estavam rastreados pelo Git
continuam rastreados.

## Executar localmente

Requer uma versão de Python compatível com as dependências do projeto. No
PowerShell, na raiz do repositório:

```powershell
py -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

Como as migrações estão ignoradas pelo Git, `makemigrations` precisa gerar os
arquivos localmente a partir dos modelos antes de `migrate` criar/atualizar o
banco. Repita os dois comandos depois de mudanças nos modelos.

A base das rotas de recursos é `http://127.0.0.1:8000/api/v1/`. Por exemplo,
acesse `http://127.0.0.1:8000/api/v1/movies/` para listar filmes. As views do
Django REST Framework também podem ser exploradas no navegador por meio da API
navegável.

Para acessar o painel administrativo:

```powershell
python manage.py createsuperuser
```

Em seguida, abra `http://127.0.0.1:8000/admin/`.

## Verificações

Execute as verificações de configuração e testes com:

```powershell
python manage.py check
python manage.py test
```

Os arquivos `tests.py` das apps ainda não contêm casos de teste. Para verificar
se há mudanças de modelos que precisam de migração:

```powershell
python manage.py makemigrations --check --dry-run
```

## Configuração e limitações conhecidas

- A configuração atual é para desenvolvimento: `DEBUG` está habilitado,
  `ALLOWED_HOSTS` está vazio e a `SECRET_KEY` está definida diretamente em
  `app/settings.py`. Não use essa configuração em produção; mova segredos para
  variáveis de ambiente e siga a checklist de segurança do Django.
- O projeto não define uma política própria de autenticação ou permissões da
  API. A interface administrativa do Django possui seu próprio mecanismo de
  autenticação.
- O SQLite é usado como banco local de desenvolvimento.
- Como os arquivos de migração são ignorados, ambientes diferentes podem
  gerar migrações distintas. Versionar migrações é recomendável para compartilhar
  um histórico consistente do esquema do banco em equipe.
