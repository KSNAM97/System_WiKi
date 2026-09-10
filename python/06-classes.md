# Python 06 — 클래스

## 절차 지향 vs 객체 지향

지금까지 다룬 변수·함수 중심의 코드 작성 방식을 **절차 지향(procedural)** 프로그래밍이라고 한다. 데이터(변수)와 그 데이터를 다루는 로직(함수)이 서로 분리되어 있는 방식이다.

```python
# 절차 지향 방식: 데이터와 함수가 분리되어 있다
student_name = "홍길동"
student_score = 85

def print_info(name, score):
    print(f"{name}의 점수는 {score}점입니다.")

print_info(student_name, student_score)
```

```text
(base) C:\Users\guest\project> python procedural.py
홍길동의 점수는 85점입니다.
```

- 학생이 여러 명이 되면 `student_name`, `student_score` 같은 변수를 학생 수만큼 따로 만들어야 하고, 각 학생의 데이터와 함수 호출을 짝지어 관리하는 부담이 커진다.

**객체 지향(object-oriented)** 프로그래밍은 데이터(속성)와 그 데이터를 다루는 로직(메서드)을 **클래스(class)**라는 하나의 틀로 묶어서 관리하는 방식이다.

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_info(self):
        print(f"{self.name}의 점수는 {self.score}점입니다.")

s1 = Student("홍길동", 85)
s2 = Student("김철수", 92)

s1.print_info()
s2.print_info()
```

```text
(base) C:\Users\guest\project> python oop_intro.py
홍길동의 점수는 85점입니다.
김철수의 점수는 92점입니다.
```

- 학생이 몇 명이 되더라도 `Student(이름, 점수)`로 객체를 하나씩 만들기만 하면 되며, 각 객체는 자신의 데이터와 동작을 스스로 가지고 있다.

**정리**: 절차 지향은 데이터와 함수를 분리해서 다루는 방식이라 같은 형태의 데이터가 여러 개로 늘어나면 관리가 번거로워지지만, 객체 지향은 데이터(속성)와 동작(메서드)을 클래스라는 하나의 틀로 묶어, 같은 형태를 가진 대상이 여러 개라도 각각을 독립된 객체로 일관되게 다룰 수 있게 해준다.

## 클래스 선언과 __init__ 생성자

**클래스(class)**는 객체를 찍어내는 틀(설계도)이며, `class` 키워드로 정의한다.

```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        print(f"{self.name}: 멍멍!")
```

- `class 클래스이름:` 뒤에 콜론을 붙이고, 그 아래 들여쓰기된 부분이 클래스 본문이다. 클래스 이름은 관용적으로 각 단어의 첫 글자를 대문자로 쓰는 **파스칼 케이스(PascalCase)**를 사용한다.
- `__init__`은 객체가 생성되는 순간 자동으로 호출되는 특수한 메서드로, **생성자(constructor)**라고 부른다. 객체가 처음 만들어질 때 필요한 초기 설정(속성 값 지정 등)을 이 안에서 수행한다.
- 메서드 이름 앞뒤로 밑줄 두 개(`__`)가 붙은 `__init__`처럼 특수한 이름의 메서드를 **매직 메서드(magic method, dunder method)**라고 하며, 파이썬이 특정 상황에서 자동으로 호출해준다.

```python
dog = Dog("초코", 3)
dog.bark()
```

```text
(base) C:\Users\guest\project> python dog_init.py
초코: 멍멍!
```

- `Dog("초코", 3)`을 호출하면 파이썬이 자동으로 `__init__(self, "초코", 3)`을 실행하여 `name`과 `age` 속성을 설정한 새 객체를 만들어준다.

**정리**: 클래스는 `class 클래스이름:`으로 선언하는 객체의 설계도이며, `__init__`은 객체가 생성되는 시점에 자동으로 호출되어 초기 속성 값을 설정하는 생성자 역할을 하고, 이런 `__이름__` 형태의 특수 메서드를 매직 메서드라고 부른다.

## self 키워드와 인스턴스 변수

클래스 안에 정의된 메서드는 첫 번째 매개변수로 반드시 **self**를 받는다. `self`는 그 메서드를 호출한 객체 자신을 가리키는 참조다.

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increase(self):
        self.count += 1

    def show(self):
        print(f"현재 카운트: {self.count}")
```

```python
c1 = Counter()
c2 = Counter()

c1.increase()
c1.increase()
c2.increase()

c1.show()
c2.show()
```

```text
(base) C:\Users\guest\project> python self_test.py
현재 카운트: 2
현재 카운트: 1
```

- `c1.increase()`를 호출하면 파이썬은 내부적으로 `Counter.increase(c1)`처럼 `c1`을 첫 번째 인자로 자동 전달한다. 그래서 메서드를 호출할 때는 `self`에 해당하는 인자를 직접 넘기지 않아도 된다.
- `self.count`처럼 `self.`으로 시작하는 변수를 **인스턴스 변수(instance variable)**라고 하며, 각 객체(인스턴스)마다 독립적으로 값을 가진다. `c1`의 `count`를 증가시켜도 `c2`의 `count`에는 아무 영향이 없다는 것이 위 결과로 확인된다.

**self를 빠뜨리면 오류가 발생한다**

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increase():   # self를 빠뜨림
        pass
```

```python
>>> c = Counter()
>>> c.increase()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Counter.increase() takes 0 positional arguments but 1 was given
```

- `c.increase()`를 호출하면 파이썬은 여전히 `c`를 자동으로 첫 번째 인자로 넘기려 하지만, `increase()`는 매개변수를 하나도 받지 않도록 정의되어 있어 인자 개수가 맞지 않는다는 오류가 발생한다.

**정리**: 클래스의 메서드는 첫 번째 매개변수로 호출한 객체 자신을 가리키는 `self`를 받아야 하며, 호출할 때는 파이썬이 자동으로 넘겨주므로 별도로 전달할 필요가 없고, `self.변수명` 형태로 선언한 인스턴스 변수는 객체마다 독립적인 값을 가지므로 한 객체의 상태 변경이 다른 객체에 영향을 주지 않는다.

## 객체 생성

클래스로부터 실제 데이터를 가진 개별 대상을 만드는 것을 **객체(object)** 또는 **인스턴스(instance)**를 생성한다고 한다.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

p1 = Person("홍길동", 20)
p2 = Person("김철수", 25)

print(p1.name, p1.age)
print(p2.name, p2.age)
print(type(p1))
```

```text
(base) C:\Users\guest\project> python create_object.py
홍길동 20
김철수 25
<class '__main__.Person'>
```

- `클래스이름(인자, ...)` 형태로 클래스를 호출하면 `__init__`이 실행되며 새 객체가 만들어져 변수에 대입된다.
- `p1`과 `p2`는 같은 `Person` 클래스로 만들어졌지만 서로 다른 독립된 객체이며, `type()`으로 확인하면 `Person` 클래스의 인스턴스임을 알 수 있다.

**속성에 직접 접근하고 수정하기**

```python
p1.age = 21
print(p1.age)
```

```text
21
```

- `객체.속성명 = 값` 형태로 객체 바깥에서도 속성 값을 직접 수정할 수 있다. 다만 무분별한 외부 수정은 데이터 일관성을 해칠 수 있으므로, 실무에서는 속성을 검증하며 바꾸는 전용 메서드를 통해 수정하는 방식을 권장하는 경우가 많다.

**정리**: 클래스를 함수처럼 호출하면 `__init__`이 실행되며 새로운 독립된 객체(인스턴스)가 만들어지고, 각 객체는 `객체.속성명`으로 자신의 데이터에 접근·수정할 수 있으며, 같은 클래스로 만든 객체라도 서로 다른 메모리 공간에 독립적으로 존재한다.

## 상속 (부모-자식 클래스)

**상속(inheritance)**은 기존 클래스(부모 클래스)의 속성과 메서드를 새 클래스(자식 클래스)가 물려받아 재사용하는 문법이다.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def eat(self):
        print(f"{self.name}이(가) 먹이를 먹습니다.")


class Dog(Animal):
    def bark(self):
        print(f"{self.name}이(가) 짖습니다: 멍멍!")


dog = Dog("초코")
dog.eat()
dog.bark()
```

```text
(base) C:\Users\guest\project> python inheritance.py
초코이(가) 먹이를 먹습니다.
초코이(가) 짖습니다: 멍멍!
```

- `class Dog(Animal):`처럼 클래스 이름 뒤 괄호 안에 부모 클래스를 지정하면 상속 관계가 만들어진다. `Animal`이 **부모 클래스(parent class, superclass)**, `Dog`이 **자식 클래스(child class, subclass)**다.
- `Dog` 클래스는 `bark()` 메서드만 새로 정의했을 뿐인데도, 부모 클래스 `Animal`의 `__init__`과 `eat()`을 그대로 물려받아 사용할 수 있다.
- 상속을 사용하면 여러 클래스가 공통으로 가지는 속성·동작을 부모 클래스에 한 번만 작성하고, 자식 클래스마다 달라지는 부분만 추가로 정의하면 되어 중복을 줄일 수 있다.

**여러 자식 클래스가 같은 부모를 상속받기**

```python
class Cat(Animal):
    def meow(self):
        print(f"{self.name}이(가) 웁니다: 야옹!")

cat = Cat("나비")
cat.eat()
cat.meow()
```

```text
(base) C:\Users\guest\project> python cat_inherit.py
나비이(가) 먹이를 먹습니다.
나비이(가) 웁니다: 야옹!
```

**정리**: 상속은 `class 자식클래스(부모클래스):` 형태로 선언하며, 자식 클래스는 부모 클래스의 속성과 메서드를 그대로 물려받아 사용할 수 있고 자신만의 새 메서드도 추가로 정의할 수 있어, 여러 클래스가 공유하는 공통 로직을 부모 클래스 한 곳에 모아 코드 중복을 줄이는 데 사용된다.

## super()

자식 클래스에서 `__init__`을 새로 정의하면서도 부모 클래스의 초기화 로직을 그대로 활용하고 싶을 때는 **super()**를 사용한다.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def eat(self):
        print(f"{self.name}이(가) 먹이를 먹습니다.")


class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed

    def bark(self):
        print(f"{self.name}({self.breed})이(가) 짖습니다: 멍멍!")


dog = Dog("초코", "말티즈")
dog.eat()
dog.bark()
```

```text
(base) C:\Users\guest\project> python super_test.py
초코이(가) 먹이를 먹습니다.
초코(말티즈)이(가) 짖습니다: 멍멍!
```

- `super().__init__(name)`은 부모 클래스 `Animal`의 `__init__`을 호출하여 `self.name = name`을 그대로 처리하게 해준다. 이렇게 하면 부모 클래스의 초기화 로직을 자식 클래스에서 다시 작성할 필요가 없다.
- 자식 클래스의 `__init__`은 부모에게 없는 새로운 속성(`breed`)만 추가로 처리하면 되므로, 부모-자식 간 역할이 명확히 분리된다.

**super()를 쓰지 않고 직접 다시 작성하면 중복이 생긴다**

```python
class Dog(Animal):
    def __init__(self, name, breed):
        self.name = name   # Animal.__init__과 똑같은 코드가 중복됨
        self.breed = breed
```

- 위처럼 `self.name = name`을 자식 클래스에서 그대로 다시 쓰는 것도 동작은 하지만, 부모 클래스의 초기화 로직이 나중에 복잡해지면 자식 클래스마다 일일이 따라 고쳐야 하므로 `super()`를 사용하는 것이 유지보수 측면에서 더 바람직하다.

**정리**: `super().__init__(...)`은 자식 클래스의 `__init__` 안에서 부모 클래스의 `__init__`을 호출하는 문법으로, 부모가 이미 처리하는 초기화 로직을 자식 클래스에서 중복 작성하지 않고 그대로 재사용하면서 자식 클래스만의 추가 속성을 이어서 처리할 수 있게 해준다.

## 메서드 오버라이딩

자식 클래스에서 부모 클래스와 **같은 이름의 메서드**를 새로 정의하면, 자식 클래스의 메서드가 부모 클래스의 메서드를 덮어써서 사용된다. 이를 **메서드 오버라이딩(method overriding)**이라고 한다.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def make_sound(self):
        print(f"{self.name}이(가) 소리를 냅니다.")


class Dog(Animal):
    def make_sound(self):
        print(f"{self.name}이(가) 짖습니다: 멍멍!")


class Cat(Animal):
    def make_sound(self):
        print(f"{self.name}이(가) 웁니다: 야옹!")


animals = [Dog("초코"), Cat("나비"), Animal("이름모를동물")]

for animal in animals:
    animal.make_sound()
```

```text
(base) C:\Users\guest\project> python override_test.py
초코이(가) 짖습니다: 멍멍!
나비이(가) 웁니다: 야옹!
이름모를동물이(가) 소리를 냅니다.
```

- `Dog`과 `Cat`은 `Animal`의 `make_sound()`를 각자의 방식으로 다시 정의했으므로, 부모 클래스의 기본 동작 대신 자식 클래스의 동작이 실행된다.
- 같은 `make_sound()`라는 이름의 메서드를 호출했는데도 객체의 실제 클래스(`Dog`인지 `Cat`인지 `Animal`인지)에 따라 서로 다른 동작이 일어나는 것을 **다형성(polymorphism)**이라고 부르며, 위 예제의 `for` 반복문처럼 서로 다른 자식 클래스의 객체들을 동일한 방식으로 다룰 수 있게 해주는 객체 지향의 핵심 특징이다.

**오버라이딩된 메서드 안에서 부모의 동작도 함께 쓰고 싶다면**

```python
class Dog(Animal):
    def make_sound(self):
        super().make_sound()
        print("(멍멍 소리 추가)")

dog = Dog("초코")
dog.make_sound()
```

```text
(base) C:\Users\guest\project> python override_super.py
초코이(가) 소리를 냅니다.
(멍멍 소리 추가)
```

- `super().make_sound()`를 호출하면 오버라이딩으로 가려진 부모 클래스의 원래 메서드도 명시적으로 실행할 수 있다. `__init__`뿐 아니라 어떤 메서드에서든 `super()`로 부모 버전을 호출할 수 있다.

**정리**: 메서드 오버라이딩은 자식 클래스가 부모 클래스와 같은 이름의 메서드를 새로 정의해 동작을 덮어쓰는 것이며, 같은 이름의 메서드 호출이 객체의 실제 클래스에 따라 다르게 동작하는 다형성으로 이어지고, `super().메서드이름()`을 호출하면 오버라이딩된 상태에서도 부모 클래스의 원래 동작을 함께 실행할 수 있다.

## 실전 예제: 학생 출석·성적 관리

앞서 다룬 개념들(생성자, 인스턴스 변수, 상속, `super()`, 오버라이딩)을 하나로 묶어, 학생의 출석과 성적을 관리하는 예제를 작성해본다.

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.attendance = 0
        self.scores = []

    def check_in(self):
        self.attendance += 1
        print(f"{self.name}: 출석 처리 (총 {self.attendance}회)")

    def add_score(self, score):
        self.scores.append(score)

    def average(self):
        if not self.scores:
            return 0
        return sum(self.scores) / len(self.scores)

    def report(self):
        print(f"[{self.name}] 출석 {self.attendance}회, 평균 {self.average():.1f}점")


class HonorStudent(Student):
    def __init__(self, name, scholarship):
        super().__init__(name)
        self.scholarship = scholarship

    def report(self):
        super().report()
        print(f"  -> 장학금: {self.scholarship}원")


s1 = Student("홍길동")
s1.check_in()
s1.check_in()
s1.add_score(88)
s1.add_score(92)

s2 = HonorStudent("김철수", 500000)
s2.check_in()
s2.add_score(98)
s2.add_score(100)

for student in [s1, s2]:
    student.report()
```

```text
(base) C:\Users\guest\project> python student_manager.py
홍길동: 출석 처리 (총 1회)
홍길동: 출석 처리 (총 2회)
김철수: 출석 처리 (총 1회)
[홍길동] 출석 2회, 평균 90.0점
[김철수] 출석 1회, 평균 99.0점
  -> 장학금: 500000원
```

- `Student` 클래스는 이름·출석 횟수·성적 리스트를 인스턴스 변수로 관리하며, `check_in()`으로 출석을 누적하고 `average()`로 평균을 계산한다.
- `HonorStudent`는 `Student`를 상속받아 장학금 정보를 추가로 관리하며, `__init__`에서 `super().__init__(name)`으로 부모의 초기화를 재사용하고, `report()`를 오버라이딩해 부모의 출력(`super().report()`)에 장학금 정보를 덧붙인다.
- 서로 다른 클래스(`Student`, `HonorStudent`)의 객체를 같은 `for` 반복문에서 동일하게 `student.report()`로 호출했지만, 각 객체의 실제 클래스에 맞는 `report()`가 실행되는 다형성을 확인할 수 있다.

**정리**: 학생 관리 예제는 인스턴스 변수로 개별 학생의 상태(출석·성적)를 독립적으로 관리하고, 상속과 `super()`로 일반 학생과 장학생의 공통 로직을 재사용하며, 메서드 오버라이딩으로 장학생만의 추가 출력을 구현하는 흐름을 통해 이 문서에서 다룬 클래스의 핵심 개념이 실제로 어떻게 조합되어 쓰이는지 보여준다.

## 실습 예제 (EX1~EX4)

**EX1) 계좌 클래스 만들기 (인스턴스 변수 활용)**
- 잔액을 관리하는 `Account` 클래스를 만들고 `deposit()`(입금), `withdraw()`(출금) 메서드를 구현한다. 출금 시 잔액이 부족하면 처리하지 않고 메시지를 출력한다.

```python
# ex01_account.py
class Account:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount
        print(f"{amount}원 입금, 잔액: {self.balance}원")

    def withdraw(self, amount):
        if amount > self.balance:
            print("잔액이 부족합니다.")
            return
        self.balance -= amount
        print(f"{amount}원 출금, 잔액: {self.balance}원")

acc = Account("홍길동", 10000)
acc.deposit(5000)
acc.withdraw(20000)
acc.withdraw(8000)
```

```text
(base) C:\Users\guest\project> python ex01_account.py
5000원 입금, 잔액: 15000원
잔액이 부족합니다.
8000원 출금, 잔액: 7000원
```

**EX2) 도형 클래스 상속으로 넓이 계산하기**
- 공통 부모 클래스 `Shape`을 만들고, `Rectangle`과 `Circle`이 각각 `area()`를 오버라이딩해 넓이를 계산하도록 한다.

```python
# ex02_shapes.py
class Shape:
    def area(self):
        return 0


class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

shapes = [Rectangle(4, 5), Circle(3)]
for shape in shapes:
    print(f"{type(shape).__name__} 넓이: {shape.area()}")
```

```text
(base) C:\Users\guest\project> python ex02_shapes.py
Rectangle 넓이: 20
Circle 넓이: 28.259999999999998
```

**EX3) super()로 초기화 재사용하는 직원 클래스**
- `Employee` 클래스를 상속받은 `Manager` 클래스를 만들고, `super().__init__()`으로 이름·급여를 초기화한 뒤 관리하는 팀원 수를 추가 속성으로 구현한다.

```python
# ex03_employee.py
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def info(self):
        print(f"{self.name}, 급여 {self.salary}원")


class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)
        self.team_size = team_size

    def info(self):
        super().info()
        print(f"  -> 관리 팀원 수: {self.team_size}명")

m = Manager("이영희", 6000000, 8)
m.info()
```

```text
(base) C:\Users\guest\project> python ex03_employee.py
이영희, 급여 6000000원
  -> 관리 팀원 수: 8명
```

**EX4) 여러 객체를 리스트에 담아 순회하며 다형성 확인하기**
- EX2에서 만든 `Rectangle`, `Circle` 객체 여러 개를 리스트에 담고, 순회하며 각 도형의 넓이 총합을 계산한다.

```python
# ex04_total_area.py
shapes = [Rectangle(2, 3), Rectangle(4, 4), Circle(2)]

total = 0
for shape in shapes:
    total += shape.area()

print(f"전체 넓이 합계: {total:.2f}")
```

```text
(base) C:\Users\guest\project> python ex04_total_area.py
전체 넓이 합계: 34.56
```

- 서로 다른 클래스(`Rectangle`, `Circle`)의 객체가 섞여 있어도 동일한 `shape.area()` 호출만으로 각자에 맞는 계산이 이루어진다는 점에서 오버라이딩과 다형성이 실전에서 어떻게 코드를 단순하게 만들어주는지 확인할 수 있다.

**정리**: EX1~EX4는 인스턴스 변수로 계좌 잔액을 독립적으로 관리하고, 상속과 오버라이딩으로 도형마다 다른 넓이 계산 로직을 구현하고, `super()`로 부모 초기화를 재사용하며, 서로 다른 클래스의 객체를 하나의 리스트로 다루는 다형성까지 이 문서의 핵심 개념을 코드로 직접 확인해보는 예제이며, 다음 문서에서는 이런 클래스와 함수를 재사용 가능한 단위로 묶는 모듈과 라이브러리를 다룬다.

[Python 05 — 조건문과 반복문](05-conditions-loops.md) · [Python 07 — 모듈과 라이브러리](07-modules-libraries.md)
