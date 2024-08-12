---
title: (Python) 좋은 코딩 습관 1편
date: 2024-08-07 07:00:00 +0900
categories: [python, habit]
tags: [python, habit, coding, programming]
---

# 좋은 코딩 습관 Python 1편

Python 코딩을 하면서 갖고 있으면 좋은 습관을 하나씩 업로드해보려고 한다. 오늘 소개할 습관은 `Type Hint`이다.

## 들어가며

Type Hint가 무엇인가? Type Hint는 말그대로 Type에 대한 힌트를 주는 것이다. `C/C++`처럼 Data Type이 정적이지 않고 `Python`에서는 계속 바뀔 수 있다. 이러한 특징이 주는 장단점은 너무나도 명확하다. 장점은 코드의 유연성이 좋다는 점이다. 가령, 어떤 프로세싱의 결과가 완전히 자료형이 바뀐다면, 새롭게 변수를 설정하지 않아도 된다(물론, 상황에 따라 원본 데이터가 필요한 경우도 있지만 말이다.). 단점은 코드의 해석이 어려워진다는 점이다. 보통 코드를 추적(traceback)하기위해 PyCharm, Anaconda, VSC Extension등을 통해 Syntax 검사도 하고 등등 여러 백그라운드 프로세스를 돌릴 것이다. 하지만, 이들이 모두 완벽하다고 할 수 없다. 결국 자료형이 계속 바뀌다보면, 또는 여러 자료형이 올 수 있는 가능성이 있다면 이러한 기능들이 정확하게 Traceback할 수 없게 된다(보통 이렇게 추적하는 기능을 두고 Syntax Lint, Resolution이라 칭한다). 이를 해결해줄 수 있는 방식이 Python에서는 `Type Hint`로써 존재한다.



## 초기화 문법 (Syntax-Initialization) 

기본적인 구조는 다음과 같다.

```python
var_name: type = value
```

여기서 주목해야할 부분은 `: [type]`이다. 이를 통해 Type에 대한 Hint를 줄 수 있다. 아래는 직접적인 예시다.

```python
var: int = 10
```

굉장히 문법 자체는 쉽다. 하지만, 익숙해지지 않으면 변수이름과 자료형의 위치를 반대로 적는 불상사가 많이 생기기도 한다. 필자는 주언어가 `C/C++`인데 `int var = 10;`이 생각나서 몇 번 헷갈리기도 했다. 이것만 빼면 굉장히 쉽다.



## 인자 문법 (Syntax-Argument)

Type Hint는 함수 인자로도 사용될 수 있다. 물론 기능은 전부 동일하다.

```python
def __function_name__(var-name: type [= value]) [-> type]:
    # do something
    pass
```

`[]`의 값들은 선택적으로 넣을 수 있다. 함수의 형태로만 확인한다면, 이해하기 힘들 수 있다. 아래의 함수는 Function 형태의 Type Hint의 간단한 형태이다.

```python
def foo(a: int, b: int):
    print(f'a = {a} / b = {b}')
```

또한, 선택적인 부분들을 포함한 예시는 아래와 같이 표현될 수 있다.

```python
def foo(a: int = 5) -> None:
    print(f'a = {a}')

foo()
foo(1)
```

위의 함수를 해석해보면 이렇다. 함수 `foo`의 인자 `a`는 default로 5라는 값을 가지며, `Return Type`으로 `NoneType`을 리턴한다는 의미이다. 



## 타입 힌트 지원 (Type Hint Support)

### 다수의 타입 (Multiple Types Hint)

앞서 표현한 `type`에서는 마치 `type`이 한 가지만 가능한 듯이 표현되어 있다. 하지만, 실질적으로는 다수의 자료형도 올 수 있다. 이는 `|` 표현을 통해서 가능하다. 물론 가장 좋은 것은 타입이 한 가지만 오는 것이다. 그렇지만, 어쩔 수 없이 다수의 타입이 사용되어야 하는 경우도 있다. 다수의 타입이 사용될 수 있는 경우의 문제는 물론 명확하다. 예를 들어 가능한 타입이 `int`와 `list`라고 했을 때, `int`는 `append`함수가 없지만 `list`는 `append`함수가 존재한다. 이러한 경우 대부분의 Code Linter에서는 이 변수들에 대해서 `append`함수가 있다고 표현한다. 그런데 의도치 않게 정수에 대해서 이 함수를 사용하게 되면, 당연하게도 `AttributeError`를 발생시킬 것이다. 아래의 코드는 다수의 타입에 대해 힌트를 주는 예시이다.

```python
def foo(a: int | float | None, 
        b: int | float | None) -> None:
    print(a+b)
```



### [typing] Union

타입 힌트를 보조해주는 기능으로 `|`만 존재하진 않는다. 타입 힌트를 지원하기위해서 Python에서는 `ver >= python 3.5`부터 `typing`이라는 모듈을 지원해오고 있다. 이 모듈 내부에는 `Union`은 `|`을 대체하는 기능을 한다. `Union`을 사용하는 형식은 아래와 같다.

```python
Union[type, ...]
```

`Union` 내부의 타입들을 하나로 묶어 표현해준다. 이 방식은 굉장히 길어질 수 있는 타입 힌트에 대해서 더 높은 가독성을 제공한다. 아래의 코드는 `Union`을 활용한 함수 예시이다.

```python
from typing import Union

def foo(a: Union[int, float, None]) -> None:
    if not a:
      return
    print(a)
```

위의 코드는 `a`가 None이면 아무런 출력없이 함수를 끝냈다. 그리고 그렇지 않으면 `a`를 출력해준다. `typing`에서 `Union`은 꼭 import 해주어야 한다.



### [typing] Optional

위의 코드를 봐왔다면 굉장히 많은 부분에서 타입 힌트의 뒤쪽에 `None`이 계속 있음을 알 수 있다. 이와 같은 표현은 실제 실무에서 굉장히 많이 쓰이는 표현이다. 가령, 예를 들자면, `MySQL`, `SQLite` 등과 같은 DB 프레임워크를 썼을 때, `SELECT` 기능을 사용하는 경우 못 찾으면 None을 리턴하기 때문이다. 이와 같은 상황을 지원하는 클래스도 `typing`에 정의되어 있다. 아래는 그 형식이다.

```python
Optional[type]
```

이 형식은 `type | None`을 의미하게 된다. 

```python
from typing import Optional

def foo(a: Optional[Union[int, float]]) -> None:
    if not a:
      return
    print(a)
```

이전에는 그렇게 많이 쓰이는 느낌은 아니었는데, 요즘 배포되는 Module을 살펴보면 이러한 `typing` Module을 굉장히 많이 사용하는 느낌이 든다. 이렇게 하는 표현이 결국 정확한 표현과 리팩터링에 있어서 장점이 있기 때문이라고 생각한다.



### [typing] Final

`Final`은 굉장히 특별한 특징을 가진 생성자이다. 이 타입 힌트는 type checker에게 최종 변수임을 나타낸다. 이는 `Final`로 선언된 변수가 재할당 또는 overriding될 수 없음을 의미한다. `Final`을 사용하는 형식은 아래와 같다.

```python
Final[type]
```

사용 자체는 굉장히 쉽다. 만약, 자료형이 확실치 않다면 `Final`만 작성해도 상관 없다. 아래는 공식 문서에서 말하는 에러로 여겨질 수 있는 상황 두 가지이다.

```python
from typing import Final

MAX_SIZE: Final = 9000
MAX_SIZE += 1 # Error reported by type checker
```

```python
from typing import Final

class Connection:
    TIMEOUT: Final[int] = 10

class FastConnector(Connection):
    TIMEOUT = 1 # Error reported by type checker
```

전자의 코드는 변수의 재할당을, 후자는 변수의 overriding을 표현했다고 볼 수 있다. 즉, 이는 값 및 자료형이 바뀌지 않는, `C++` 언어와 비교했을 때, 일종의 `const` 키워드와 유사하다.



### [typing] Callable

Callable은 무언가 호출할 수 있는 어떤 대상을 의미하는 특별한 타입 힌트이다. 이는 `Callable`에 대해서는 함수가 될 수도 있고, 클래스 생성자가 될 수도 있고 호출할 수 있는 대상이라면 모두 가능하다. `Callable`을 사용하는 형식은 아래와 같다.

```python
Callable[[arg_type, ...], return_type]
```

앞 쪽의 이중으로 싸여진 타입들은 호출시 넘기는 인자들의 자료형이며, 후자는 리턴 자료형이다. 사용자체는 굉장히 쉽다. 다만, 이 `Callable`을 사용할 때에 주의해야 할 점은 공식문서에서 `typing.Callable`은 `deprecated`, 즉 더 이상 쓸모없어져서 지원되지 않을 예정이라고 표시해놓은 점이다. 공식 문서에서는 이렇게 표현한다: `Deprecated alias to collections.abc.Callable`. 즉, `Callable`의 실제 정의는 `collection.abc`에 있다는 표현이다. 이는 다시 말해, `collections.abc`에서 불러오는 방향이 바람직하다. 아래는 그 예시이다.

```python
from typing import Callable # deprecated
from collections.abc import Callable

def foo(a: int = 1) -> int:
    return a
    
def bar(func: Callable[[int], int], var: int) -> None:
    print(func(var))
```

물론 위의 함수는 약간의 Tricky함이 묻어 있지만, 간단한 코드이므로 해석에 크게 문제 없으리라 생각한다. 이와 같은 형식으로 `javascript`에서 많이 쓰이는 `forEach`같은 함수에 대해서 python에서는 이 기능을 이용해 조금 더 명시적으로 표현해줄 수 있다. 하지만, 모든 타입 힌트 지원 기능들을 typing에서 관리하고 싶다면, 아래와 같이 쓸 수도 있다. 다만, Tricky함이 조금 더 가미된다.

```python
import collections
from typing import Callable

Callable = collections.abc.Callable
```



## 제너릭 타입 (Generic Type)

Generic type을 한국어로 뭐라 부르는지는 모르겠어서 그냥 제너릭 타입이라 썼다. 안다면 알려주기를 바란다.



### List

Python에서는 더 좋은 코드 유연성과 유지보수 용이성, 효율적인 코드 린팅을 확보하기 위해 Type hint로 Generic types를 지원한다. 이름에서 알 수 있듯이 이는 python에 기존부터 존재하는 자료형에 대해서 전반적으로 포괄할 수 있는 자료형을 대체한다. 물론, 많은 사람들이 이렇게 생각할 수 있다: "`list`만 있어도 충분히 표현할 수 있지않은가? (아래와 같이)".
```python
a: list = [1,2,3]
```

맞다. 이렇게도 충분히 가능하다. 그러나 해당 리스트의 내부 원소의 자료형을 명확히 표현해주지 못하기 때문에 이 부분에서 단점이 있다. 이러한 경우 아래와 같이 표현이 가능하다.

```python
from typing import List

a: List[int] = [1,2,3]
```

이와 같이 사용한다면, 더욱 더 복잡한 코드도 유연하고 쉽게 코드 린터들이 분석할 수 있다.



### Dictionary

`typing` 모듈은 `dictionary`형에 대해서도 지원하고 있다. 아래는 사용 문법 형식이다.

```python
Dict[key_type, val_type]
```

위의 형식을 이용해 표현한다면, 아래와 같이 쓸 수 있다.

```python
from typing import Dict, List

scores: Dict[int, List[str]] = {
    101010: ["TOM", "B"],
    987654: ["HARRISON", "C"]
} 
```



### Others

이 외에도 많은 Generic types들이 존재한다. 많이 사용되는 Generic types들은 다음과 같다. `typing.Tuple`, `typing.Set`, `typing.Iterable`, `typing.Literal`, `typing.Any` 등이 있다(물론, Iterable과 Literal을 generic이라고 부르기는 조금 애매하긴 하다). 

# 마치며

이 기능은 말 그대로 Type **Hint**이므로 이 기능을 잘못 썼다고 해서 에러를 발생시키지는 않는다. 물론 Type Hint 구조에 맞게 썼을 때 말이다. 잘못 쓰게 되면, 당연하게도 본인이 코드를 제대로 짜거나 분석하기가 힘들겠지만 말이다. 다만, Code Linter에서는 경고로 표시하기는 한다. 이 글은 여기서 마치지만, 이것이 전부가 아니고 다른 기능들도 많다. 하지만, 가장 필수적으로 알아야 할 몇 가지 기능을 적어놓았다. 이 기능을 잘 활용해 다들 좋은 코드를 만들었으면 좋겠다.