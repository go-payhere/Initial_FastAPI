# Pydantic으로 요청 Payload 가져오기(FastAPI에서 HTTP 본문을 가져오는 방법 정리)

이번 포스팅에서는 FastAPI에서 HTTP 본문을 가져오는 방법을 정리하려고 합니다.

우선 정리하기 전에 Pydantic, Payload에 대한 개념을 정리하고 실습과 방법을 정리하겠습니다.

## 🤔 Pydantic이란?

Pydantic은 파이썬 타입 어노테이션을 사용해서 데이터 유효성 검사와 설정 관리를 하는 라이브러리이다.  
런타임때 타입 힌트를 강제하고, 데이터가 유효하지 않을때 유저 친화적인 에러를 제공한다.  
즉, **pydantic을 사용하여 데이터가 어때야 하는지 정의하고, 유효한지 확인할 수 있다.**

pydantic은 validation이 아닌 parsing 라이브러리이기 때문에 input data를  
정의된 타입으로 변환하여 output model의 타입과 제약 조건을 보장한다.

```python
from pydantic import BaseModel
class Model(BaseModel):
    a: int
    b: float
    c: str
print(Model(a=3.1415, b=' 2.72 ', c=123).dict())
#> {'a': 3, 'b': 2.72, 'c': '123'}
```

예를 들어, a의 값으로 float이 들어와도 int로 parsing을 해서 output model의 타입을 보장해준다.  
단, parsing이 불가능한 데이터 타입이 들어오면 validation error가 발생한다.

다른 파이썬 라이브러리와 동일하게 pip install pydantic 명령어로 설치할 수 있다.

### 장점

- IDE plugin으로 제공되고 파이썬 타입 어노테이션과 유사하기 때문에 쉽게 사용할 수 있다.
- 비슷한 라이브러리 중에서 가장 빠르다.
- recursive model, typing 의 타입, validator 를 통해 복잡한 데이터 스키마를 정의하고, validation과 parsing을 할 수 있다.

## 🤔 Payload란?

페이로드(payload)는 전송되는 데이터를 의미합니다.  
데이터를 전송할 때, 헤더와 <a href ="https://github.com/ohyuchan123/TIL_V2/blob/master/What%20did%20you%20study%20today/%EB%8F%99%EA%B8%B0%2C%20%EB%B9%84%EB%8F%99%EA%B8%B0.md#%EB%8F%99%EA%B8%B0-%EB%B9%84%EB%8F%99%EA%B8%B0-%EC%B2%98%EB%A6%AC">메타데이터</a>, 에러 체크 비트 등과 같은 다양한 요소들을 함께 보내어,  
데이터 전송의 효율과 안정성을 높히게 됩니다. 이 때, 보내고자 하는 데이터 자체를 의미하는 것이 바로 페이로드입니다.  
우리가 택배 배송을 보내고 받을 때, 택배 물건이 페이로드이고, 송장이나 박스, 뾱뾱이와 같은 완충재 등등은 부가적인 것이기 때문에 페이로드가 아닙니다.

## 👉 FastAPI에서 HTTP 본문 가져오는 방법 정리

### Pydantic 가져오기 `BaseModel`

`BaseModel` 먼저 다음에 가져와야 합니다. `pydantic`

```python
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
```

### 데이터 모델 만들기

그런 다음 데이터 모델에서 상속하는 클래스로 선언합니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

쿼리 매개변수를 선언할 때와 마찬가지로 모델 속성에 기본값이 있으면 필요하지 않습니다.  
`None` 선택 사항으로 만들기 위해 사용합니다.

예를 들어 위의 이 모델은 다음과 같이 JSON "`object`" (또는 Python `dict`)를 선언합니다.

```python
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```

...as `description` 및 `tax` 선택 사항(기본값 None)인 경우 이 JSON "`object`"도 유효합니다.

```python
{
    "name": "Foo",
    "price": 45.2
}
```

### 매개변수로 선언

*경로 작업*에 추가하려면 경로 및 쿼리 매개변수를 선언한 것과 동일한 방식으로 선언합니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item

```

### 결과

해당 Python 유형 선언만으로 FastAPI는 다음을 수행합니다.

- 요청 본문을 JSON으로 읽습니다.
- 해당 유형을 반환합니다(필요한 경우).
- 데이터를 검증합니다.
  - 데이터가 유효하지 않은 경우 정확하고 명확한 오류를 반환하여 잘못된 데이터의 위치와 내용을 정확하게 나타냅니다.
- 매개변수에서 받은 데이터를 제공합니다. `item`
  - 함수에서 유형으로 선언했으므로 `item` 모든 속성과 해당 유형에 대한 모든 편집기 지원(완성 등)도 갖게 됩니다.
- 생성하는 JSON 스키마 모델에 대한 정의는 프로젝트에 적합한 경우 원하는 다른 곳에서 사용할 수도 있습니다.

### 자동문서

모델의 JSON 스키마는 OpenAPI 생성 스키마의 일부이며 대화형 API 문서에 표시됩니다.
![Alt text](/pydantic/img/image.png)

또한 이를 필요로 하는 각 _경로 작업 내부의 API 문서에서도 사용됩니다._
![Alt text](/pydantic/img/image2.png)

### 모델 사용

함수 내에서 모델 객체의 모든 속성에 직접 액세스할 수 있습니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict

```

### 요청 본문 + 경로 매개변수

경로 매개변수와 요청 본문을 동시에 선언할 수 있습니다.

**FastAPI**는 경로 매개변수와 일치하는 함수 매개변수를 **경로에서 가져와야**하고 Pydantic 모델로 선언된 함수 매개변수를 **요청 본문에서 가져와야** 함을 인식합니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}

```

### 요청 본문 + 경로 + 쿼리 매개변수

**body, path** 및 **쿼리** 매개변수를 모두 동시에 선언할 수도 있습니다.
**FastAPI**는 각각을 인식하고 올바른 위치에서 데이터를 가져옵니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result

```
