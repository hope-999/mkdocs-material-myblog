# FastApi中的安全防护-接口安全

接口的安全性、身份认证和授权等问题，在fastapi中也提供了非常方便的解决方案。

安全认证的一些解决方案：

- OpenID Connect
- OAuth2
- OAuth1
- OpenID
- OpenAPI

在`fastapi`的`fastapi.security`模块中提供了几种安全认证的封装，使其用起来更加简化。

想像一个场景：

你有一个前端系统想要通过`username` `password`进行认证登录；

你有一个后端系统来实现这个逻辑；

这两个系统又不在同一个域名下，怎么办呢？  

FastApi给出了简单的实现方式。

## 先体验下最精简的一段认证代码

```python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/79")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

测试：  
```
GET  http://127.0.0.1:9999/items/79
Accept: application/json
Authorization: Bearer 123456
```
结果：
```
HTTP/1.1 200 OK
date: Thu, 12 Oct 2023 06:34:55 GMT
server: uvicorn
content-length: 18
content-type: application/json

{
  "token": "123456"
}
```

## 获取当前用户的例子

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    return user


@app.get("/users/me")
async def read_users_me(current_user: Annotated[User, Depends(get_current_user)]):
    return current_user
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
