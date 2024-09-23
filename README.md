# sensei
<a href="https://pypi.org/project/sensei/">
<h1 align="center">
Cha<img alt="Logo Banner" src="https://raw.githubusercontent.com/CrocoFactory/.github/main/branding/sensei/logo/bookmark_transparent.png" width="300">
</h1><br>
</a>

[![PyPi Version](https://img.shields.io/pypi/v/sensei)](https://pypi.org/project/sensei/)
[![PyPI Downloads](https://img.shields.io/pypi/dm/sensei?label=downloads)](https://pypi.org/project/sensei/)
[![License](https://img.shields.io/github/license/CrocoFactory/sensei.svg)](https://pypi.org/project/sensei/)
[![Last Commit](https://img.shields.io/github/last-commit/CrocoFactory/sensei.svg)](https://pypi.org/project/sensei/)
[![Development Status](https://img.shields.io/pypi/status/sensei)](https://pypi.org/project/sensei/)

The python framework, providing fast and robust way to build client-side API wrappers.
                           
- **[Maintain](https://www.patreon.com/user/membership?u=142083211)**
- **[Bug reports](https://github.com/CrocoFactory/sensei/issues)**

Source code is made available under the [MIT License](LICENSE).  
                   
# Quick Overview

Here is example of OOP style.

```python
from typing import Annotated, Any, Self
from sensei import Router, Query, Path, APIModel, Header, Args, pascal_case, fill_path_params, RateLimit

router = Router('https://reqres.in/api', rate_limit=RateLimit(5, 1))


@router.model()
class BaseModel(APIModel):
    def __finalize_json__(self, json: dict[str, Any]) -> dict[str, Any]:
        return json['data']

    def __prepare_args__(self, args: Args) -> Args:
        args.headers['X-Token'] = 'secret_token'
        return args

    def __header_case__(self, s: str) -> str:
        return pascal_case(s)


class User(BaseModel):
    email: str
    id: int
    first_name: str
    last_name: str
    avatar: str

    @classmethod
    @router.get('/users')
    def query(
            cls,
            page: Annotated[int, Query(1)],
            per_page: Annotated[int, Query(3, le=7)],
            bearer_token: Annotated[str, Header('secret', le=10)],
    ) -> list[Self]:
        ...

    @classmethod
    @router.get('/users/{id_}')
    def get(cls, id_: Annotated[int, Path(alias='id')]) -> Self:
        ...

    @router.delete('/users/{id_}')
    def delete(self) -> Self:
        ...

    @delete.prepare
    def _delete_in(self, args: Args) -> Args:
        url = args.url
        url = fill_path_params(url, {'id_': self.id})
        args.url = url
        return args


users = User.query(per_page=7)
user_id = users[0].id
user = User.get(user_id)
print(user == users[0])
```

Example of functional style.

```python
from typing import Annotated
from sensei import Router, Path, APIModel

router = Router('https://pokeapi.co/api/v2/')


class Pokemon(APIModel):
    name: str
    id: int
    height: int
    weight: int
    types: list


@router.get('/pokemon/{pokemon_name}')
def get_pokemon(
        pokemon_name: Annotated[str, Path()],
) -> Pokemon:
    ...


pokemon = get_pokemon(pokemon_name="pikachu")
print(pokemon)
```

# Installing sensei
To install `sensei` from PyPi, you can use that:

```shell
pip install sensei
```

To install `sensei` from GitHub, use that:

```shell
pip install git+https://github.com/CrocoFactory/sensei.git
```