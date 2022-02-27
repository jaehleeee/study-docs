# 기초

## Data Type
```python
# 데이터 타입
v_str1 = "string type" #str
v_bool = True #bool
v_float = 10.3 #float
v_int = 7 #int
v_complex = 2 + 3j
v_dict = { #dict
    "name" : "Lee",
    "age" : 31,
    "city" : "Seoul"
} 
v_list = [3,5,7] #list
v_tuple = 3,5,7 #tuple
v_set = {7,8,9} #set
```
각 타입별 메서드는 아래 블로그를 참조
[jewon119.log](https://velog.io/@jewon119/01.-Python-%EA%B8%B0%EC%B4%88-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%83%80%EC%9E%85data-type)

## Class
### 1. 구조
```python
class SomeClass: 
    def __init__(self,something): 
        self.something = something 
    def some_function(self): 
        print(self.something)
```

### 2. __init__의 이해
 * 초기화 함수
 * 객체 생성시 가장 처음 호출되어, 객체 초기값 설
 * `self`를 반드시 첫 인수로 선언해야함
    * `self`는 인스턴스 본인을 의미
