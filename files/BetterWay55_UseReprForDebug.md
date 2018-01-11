# Better way 55. 디버깅 출력용으로는 repr 문자열을 사용하자


#### 285쪽
#### 2017/08/20 작성

파이썬 프로그램을 디버깅할 때 우리가 자주 쓰는 print는 생각보다 많은 일을 한다.  
보통 파이썬 내부를 일반 속성으로 쉽게 접근할 수 있다.  
print로 프로그램이 실행 중에 상태가 어떻게 변하는지 출력하여 어떤 부분이 제대로  
동작하지 않는지 보기만 하면 된다.  

print 함수는 사람이 읽기 쉬운 **문자열** 버전으로 결과를 출력한다.  
예를 들어 기본 문자열을 출력하면 따옴표 문자로 감싸지 않은 문자열의 내용을 출력한다.


```python
print('foo bar')

>>>
foo bar
```


출력 내용은 '%s' 포맷 문자열과 % 연산자로 출력하거나,  
또는 '{}'.format 과 같은 연산자로 출력한 내용과 동일하다.  

```python
print('foo bar')  # 전통적인 방법
print('{}'.format('foo bar'))  # 파이썬 3문법

>>>
foo bar
foo bar
```

<br>

문제는 사람이 읽기 쉬운 문자열로는 값의 실제 타입이 무엇인지 명확하게 파악하기 힘들다는 것이다.  
예를 들어 print의 기본 출력으로는 숫자 5와 문자열 '5'를 구분할 수 없다.

```python
print(5)
print('5')

>>>
5
5
```

<br>

print로 디버깅한다면 이런 종류의 차이는 문제가 된다. 디버깅 중에는 대부분 객체의 repr 버전을  
보려고 한다. 내장 함수 **repr은 객체의 출력가능한 표현을 반환**하며, 이 표현은 객체를 가장  
명확하게 이해할 수 있는 문자열 표현이어야 한다. 

```python
a = '\x07'
print(a)
print(repr(a))

>>>
# 아무 것도 출력되지 않음.(ascii code 07 - bell)
'\x07'
```

<Br>

repr이 반환한 값을 내장 함수 **eval에 전달하면 원래의 파이썬 객체와 동일한 결과**가 나와야 한다.

```python
b = eval(repr(a))
assert a == b
```

<br>

print로 디버깅할 때는 타입의 차이가 명확히 드러나도록 값을 출력하기 전에 repr을 사용해야 한다.

```python
print(repr(5))
print(repr('5'))

>>>
5
'5'
```

<br>

위의 결과는 '%r' 포맷 문자열과 % 연산자로 출력한 결과와 같다.

```python
# %r은 파이썬 한정으로 객체의 repr 표현을 표현해준다.
print('%r' % 5)
print('%r' % '5')

>>>
5
'5'
```

<br>
<br>

**동적 파이썬 객체의 경우 기본으로 사람이 이해하기 쉬운 문자열 값이 repr값과 같다.**  
이는 print에 동적 객체를 넘기면 제대로 동작하므로 명시적으로 repr을 호출하지 않아도 됨을 의미한다.  
불행하게도 object 인스턴스에 대한 repr 기본값은 특별히 도움이 되지 않는다.
(object는 모든 파이썬 클래스의 부모 크래스)  

예를 들어 간단한 클래스를 정의하고 그 값을 출력해보자.

```python
class OpaqueClass:
    def __init__(self, x, y):
        self.x = x
	self.y = y

obj = OpaqueClass(1, 2)
print(obj)

>>>
<__main__.OpaqueClass object at 0x102b1c28>
```

<br>

이 결과는 eval 함수에 넘길 수 없으며 객체의 인스턴스 필들에 대한 정보를 전혀 알려주지 않는다.  

문제를 해결하는 방법은 두 가지가 있다.

1. 클래스를 제어할 수 있다면, 직접 \_\_repr\_\_ 메서드를 정의해서 객체 표현 문자열을 반환하면 된다.

```python
class BetterClass:
    def __init__(self, x, y):
        self.x = x
	self.y = y

    def __repr__(self):
        return 'BetterClass(%d, %d)' % (self.x, self.y)

obj = BetterClass(1, 2)
print(obj)

>>>
BetterClass(1, 2)
```

<br>

2. 클래스 정의를 제어할 수 없을 때는 \_\_dict\_\_ 속성에 저장된 객체의 인스턴스 딕셔너리를 사용한다.  
다음은 OpaqueClass 인스턴스의 내용을 출력하는 예다.

```python
obj = OpaqueClass(4, 5)
print(obj.__dict__)

>>>
{'y': 5, 'x': 4}
```

<br><br>

### 핵심정리
* 내장 타입에 print를 호출하면 사람이 이해하기는 쉽지만 타입 정보는 숨은 문자열 버전의 값이 나온다.
* 내장 타입에 repr을 호출하면 출력할 수 있는 문자열 버전의 값이 나온다.
이 repr 문자열을 eval 내장 함수에 전달하면 원래 값으로 되돌릴 수 있다.
* 포맷 문자열에서 %s는 str처럼 사람이 이해하기 쉬운 문자열을 만들며,
%r은 repr처럼 출력하기 쉬운 문자열을 만들어낸다.
* \_\_repr\_\_ 메서드를 정의하면 클래스의 출력 가능한 표현을 사용자화하고 더 자세한 디버깅 정보를 제공할 수 있다.
* 객체의 \_\_dict\_\_ 속성에 접근하면 객체의 내부를 볼 수 있다.