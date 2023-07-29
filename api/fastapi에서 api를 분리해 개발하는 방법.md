# fastapi에서 api를 분리해 개발하는 방법

FastAPI에서 API를 분리해 개발하는 방법은 기능별로 API를 여러 개의 파일로 분리하는 방법, 패키지로 분리, 클래스 기반 라우터가 있다고 할 수 있다.

## 1. 기능별로 API를 여러 개의 파일로 분리하는 방법

이 방법은 각 기능 또는 엔드포인트를 개별 파일로 만들어 관리하는 방식입니다.  
이렇게 하면 프로그젝트가 커지더라도 각 엔드포인트를 쉽게 파악하고 유지보수할 수 있습니다.

예를 들어 `main.py` 파일에서 `router1.py`, `router2.py`, `router3.py` 등으로  
라우터를 분리하는 것입니다.

(main.py)

```python
# main.py
from fastapi import FastAPI
from .router1 import router as router1
from .router2 import router as router2
from .router3 import router as router3

app = FastAPI()

app.include_router(router1)
app.include_router(router2)
app.include_router(router3)

```

(router1.py)

```python
# router1.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint1")
def get_endpoint1():
    return {"message": "This is endpoint1"}

```

(router2.py)

```python
# router2.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint2")
def get_endpoint2():
    return {"message": "This is endpoint2"}

```

(router3.py)

```python
# router3.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint3")
def get_endpoint3():
    return {"message": "This is endpoint3"}

```

## 2. 패키지로 분리

API 라우터를 기능별로 여러 개의 패키지로 분리하는 방법입니다.  
각 패키지는 자체적으로 API 라우터들을 포함하고 있으며, 이들을 메인 애플리케이션에서 import하여 사용합니다.

```lua
app/
|-- main.py
|-- api/
|   |-- __init__.py
|   |-- router1.py
|   |-- router2.py
|   |-- router3.py

```

1. `main.py``

```python
from fastapi import FastAPI
from api.router1 import router as router1
from api.router2 import router as router2
from api.router3 import router as router3

app = FastAPI()

app.include_router(router1)
app.include_router(router2)
app.include_router(router3)

```

2. `api/router1.py`

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint1")
def get_endpoint1():
    return {"message": "This is endpoint1"}
```

3. `api/router2.py`

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint2")
def get_endpoint2():
    return {"message": "This is endpoint2"}

```

4. `api/router3.py`

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/endpoint3")
def get_endpoint3():
    return {"message": "This is endpoint3"}
```
