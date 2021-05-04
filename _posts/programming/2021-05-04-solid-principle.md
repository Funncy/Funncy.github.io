---
layout: post
title: '[Programming] - SOLID 원칙 '
subtitle: 'design pattern delegate'
categories: programming
tags: designpattern solid
comments: true
---

### SOLID란?

- S - Single responsibility Principle (단일 책임 원칙)
- O - Open closed Principle (개방/폐쇄의 원칙)
- L - Liskov Substitution Principle (리스코프 치환 원칙)
- I - Interface Segregation Principle (인터페이스 분리 원칙)
- D - Dependency Inversion Principle (의존관계 역전 원칙)

[참고] : [https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)

## 단일 책임 원칙 (Single Responsibility Principle)

---

> 클래스는 한 가지 이유만으로 변경되어야 하며, 한 가지 작업만 가져야 한다.

도형별로 면적을 계산해주는 프로그램을 예시로 들어보자.

```dart
class Square {
	final double length;
	Square(this.length);
}
```

```dart
class Circle {
	final double radius;
	Circle(this.radius);
}
```

AreaCalculator는 여러 종류의 도형을 받아서 면적을 계산하고 더해서 결과를 반환해준다.

```dart
class AreaCalculator
{
    List<dynamic> _shapes;

    AreaCalculator(List<dynamic> shapes) {
      _shapes = shapes;
    }

    List<double> sum()
    {
			List<double> result = [];
      _shapes.forEach((shape) {
         if(shape is Square) {
           result.add(pow(shape.length , 2));
         } else if(shape is Circle) {
           result.add(pow(shape.radius, 2) * pi);
         }
      });

      return result;
    }

   String output()
    {
        return
              'Sum of the areas of provided shapes: '
              + sum().toString();
    }
}
```

```dart
void main() {
  var shapes = [
    Circle(2),
    Square(5),
    Square(6),
  ];

  var areas = AreaCalculator(shapes).output();

  print(areas);
}
```

### 문제 상황

만약 여기서 Json형태 출력도 요구사항에 추가가 된다면 어떻게 할까?

지금 모든 로직이 AreaCalculator에 모여있다. 이것은 단일책임원칙 위반이다.

AreaCalculator가 면적 계산 로직만 책임지고 결과를 반환해주는 부분을 새로운 클래스로 만들자.

```dart
class SumCalculatorOutputter {
  AreaCalculator calculator;
  SumCalculatorOutputter(this.calculator);

  Map<String, Object> toJson() {
    return {
      'sum' : calculator.sum(),
    };
  }

  String toString() {
    return 'Sum of the areas of provided shapes: '
       + calculator.sum().toString();
  }
}
```

```dart
void main() {
  var shapes = [
    Circle(2),
    Sqaure(5),
    Sqaure(6),
  ];

  var areas = AreaCalculator(shapes);
  var output = SumCalculatorOutputter(areas);

  print(output.toJson());
  print(output.toString());
}
```

단일 책임 원칙이 지켜졌다.

## 개방 폐쇠 원칙 (Open closed Principle)

---

> 확장에는 열려있지만 수정에는 닫혀있어야 한다.

클래스를 수정하지 않고 클래스를 확장할 수 있어야 한다.

여기서 sum 메소드를 다시 확인해보자.

```dart
List<double> sum()
    {
			List<double> result = [];
      _shapes.forEach((shape) {
         if(shape is Square) {
           result.add(pow(shape.length , 2));
         } else if(shape is Circle) {
           result.add(pow(shape.radius, 2) * pi);
         }
      });

      return result;
    }
```

여기서 새로운 도형이 추가된다면 AreaCalculator를 계속해서 if-else문을 하나씩 더 추가해주어야 한다.

개방-폐쇄 원칙 위반이다.

더 나은 방법은 area 계산 로직을 각 도형에게 넘겨주는 것이다.

```dart
class Square {
  final double length;
  Sqaure(this.length);

  double area() {
    return pow(length, 2);
  }
}

class Circle {
  final double radius;
  Circle(this.radius);

  double area() {
    return pow(radius, 2) * pi;
  }
}
```

```dart
class AreaCalculator
{
    List<dynamic> _shapes;

    AreaCalculator(List<dynamic> shapes) {
      _shapes = shapes;
    }

    double sum()
    {
      double result = 0.0;
      _shapes.forEach((shape) {
        result += shape.area();
      });

      return result;
    }

   String output()
    {
        return
              'Sum of the areas of provided shapes: '
              + sum().toString();
    }
}
```

이제 코드 변경없이 도형의 확장이 가능해졌다.

하지만 현재 dynamic 타입을 쓰고 있는 문제가 있다.

이것을 Interface로 해결해주자.

```dart
abstract class ShapeInterface {
  double area();
}
```

```dart
class Sqaure implements ShapeInterface{...
}

class Circle implements ShapeInterface{ ...
}
```

```dart
class AreaCalculator
{
    List<ShapeInterface> _shapes;

    AreaCalculator(List<ShapeInterface> shapes) {
      _shapes = shapes;
    }
...
}
```

이제 개방-폐쇄 원칙이 지켜졌다.

## 리스코프 치환 원칙 (Liskov Substitution Principle)

---

모든 하위 클래스 또는 파생 클래스가 기본 또는 상위 클래스로 대체 가능해야 함을 의미한다.

문장이 이해가 가지 않으니 아래 예시를 봐보자.

[출처] : [https://velog.io/@y_dragonrise/소프트웨어-설계-리스코프-치환-원칙LSP-Liskov-Substitution-Principle](https://velog.io/@y_dragonrise/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EC%84%A4%EA%B3%84-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99LSP-Liskov-Substitution-Principle)

### 일반화 관계에서의 LSP

원숭이는 포유류이다. 이 문장에서 출발해보자.

### 포유류

- 포유류는 알을 낳지 않고 새끼를 낳아 번식한다.
- 포유류는 젖을 먹여서 새끼를 키우고 폐를 통해 호흡한다.
- 포유류는 체온이 일정한 정온 동물이며 털이나 두꺼운 피부로 덮여 있다.

### 원숭이

- 원숭이는 알을 낳지 않고 새끼를 낳아 번식한다.
- 원숭이는 젖을 먹여서 새끼를 키우고 폐를 통해 호흡한다.
- 원숭이는 체온이 일정한 정온 동물이며 털이나 두꺼운 피부로 덮여 있다.
  → 원숭이와 포유류는 행위(번식 방식, 양육 방식, 호흡 방식, 체온 조절 방식, 피부 보호 방식 등)에 일관성이 있음

### 오리너구리 \*\*

- ~~오리너구리는 알을 낳지 않고 새끼를 낳아 번식한다.~~
- 오리너구리는 젖을 먹여서 새끼를 키우고 폐를 통해 호흡한다.
- 오리너구리는 체온이 일정한 정온 동물이며 털이나 두꺼운 피부로 덮여 있다.
  → 오리너구리는 알을 낳아 번식하는 동물임
  → LSP를 만족하려면 부모 클래스의 인스턴스를 자식 클래스의 인스턴스로 대신할 수 있어야 함

여기서 알을 낳지 않는다는 부분은 자식 클래스에 의해서 대체되게 된다.

가방으로 예시를 들어보자.

### Bag

```dart
class Bag {
	int _price;
	int get price => _price;
	set price(int price) => _price = price;
}
```

여기서 Bag 클래스는 가격의 설정과 조회의 역할을 맡고 있다.

아래 서브 클래스 DiscountedBag 을 만들어보자.

### DiscountedBag

```jsx
class DiscountedBag extends Bag {
	double _discountedRate = 0;
	set discountedRate(double discountedRate) => _discountedRate = discountedRate;

	void applyDiscount(int price) {
		super.setPrice(price - (int)(discountedRate * price));
	}
}
```

discountedBag은 할인율을 설정해서 할인된 가격을 계산해주는 기능이 추가되었다.

```dart
void main() {
	Bag b1 = new Bag();
	Bag b2 = new Bag();
	b1.price = 50000;
	print(b1.price);
	b2.price = b1.price;
	print(b2.price);

	DiscountedBag b3 = DiscountedBag();
	DiscountedBag b4 = DiscountedBag();
	b3.price = 50000;
	print(b3.price);
	b4.price = b1.price;
	print(b4.price);
}
```

Bag클래스의 매서드를 DiscountedBag클래스에서 재정의되지 않았으므로 위의 코드의 결과는 동일하다!

⇒ price setter를 override했을 경우 같은 결과가 나오지 않는다.

다시 넓이 계산기로 돌아와보자.

넓이 말고 부피 계산기를 만들어보자.

```dart
class VolumeCalculator extends AreaCalculator {
    List<ShapeInterface> _shapes;

    VolumeCalculator(List<ShapeInterface> shapes) : super(shapes);

    @override
    List<double> sum() {
      List<double> result = [0,0,0,0];
      // volume 계산을 위한 로직...
      return result;
    }

}
```

```dart
void main() {
  List<ShapeInterface> shapes = [
    Circle(2),
    Square(5),
    Square(6),
  ];

  var areas = AreaCalculator(shapes);
  var volumes = VolumeCalculator(shapes);

  var output = SumCalculatorOutputter(areas);
  var output2 = SumCalculatorOutputter(volumes);

  print(output.toJson());
  print(output.toString());
  print(output2.toJson());
  print(output2.toString());
}
```

SumCalculatorOutputter에 AreaCalculator(부모 클래스) 혹은 VolumeCalculator(서브클래스)를 넣어도 문제없이 동작한다.

> 추가적으로 다음 내용을 만족해야 한다.

1. 하위클래스에서 선행 조건(파라미터 체크 등)을 추가할 수 없다.
2. 하위클래스에서 후행 조건(결과갑 반환전 체크 등)을 약화(느슨)할 수 없다.
3. 하위클래스에서 부모클래스의 불변값, 조건은 반드시 유지되어야 한다.
4. 하위클래스에서 부모클래스가 던진 예외 이외 새로운 예외를 던지면 안된다.

## 인터페이스 분리 원칙 (Interface Segregation Principle)

---

클라이언트는 사용하지 않는 인터페이스를 강제로 구현해서는 안되며, 사용하지 않는 메서드에 강제로 의존해서는 안된다.

ShapeInterface 코드를 살펴보자.

이제 Volume 계산을 위해서 새로운 함수를 추가해줘보자.

```dart
abstract class ShapeInterface {
  double area();
	double volume();
}
```

하지만 여기서 문제가 발생한다.

평면 사각형과 같은 도형의 경우 volume을 계산할 수 없다.

여기서 인터페이스 분리가 필요하다.

```dart
abstract class ManageShapeInterface {
    double calculate();
}

class Square implements ShapeInterface, ManageShapeInterface {
    double area(){
        // calculate the area of the square
    }

    double calculate() {
        return $this->area();
    }
}

class Cuboid implements ShapeInterface, ThreeDimensionalShapeInterface, ManageShapeInterface {
    double area() {
        // calculate the surface area of the cuboid
    }

    double volume() {
        // calculate the volume of the cuboid
    }

    double calculate() {
        return $this->area();
    }
}
```

## 의존성 역전 원칙 (Dependency Inversion Principle)

---

인터페이스를 통해서 의존성을 역전해야한다. decupling

다음 예시 코드를 보며 살펴보자.

```dart
class MySQLConnection {
	void connect() {
		print("connection");
	}
}

class PasswordReminder {
	var _dbConnection;
	PasswordReminder(MySQLConnection dbConnection) : _deConnection = dbConnection;
}
```

위 코드는 high-level module인 PasswordReminder는 low-level module인 MySQLConnection에 의존하고 있다.

만약 DataBase Engine이 변경된다면 PasswordReminder 코드를 변경해주어야 한다.

이것은 개방-폐쇄 원칙을 위반이다.

인터페이스로 수정해보자.

```dart
abstract class DBConnectionInterface {
	void connect();
}

class MySQLConnection implements DBConnectionInterface {
	@override
	void connect() {
		print("connection");
	}
}

class PasswordReminder {
	var _dbConnection;
	PasswordReminder(DBConnectionInterface dbConnection) : _deConnection = dbConnection;
}
```

## 결론

---

이렇게 SOLID 원칙들을 정리해보았다.

사실 아직 실무적으로는 완벽히 쓸 자신은 없지만 계속해서 고민하면서 나아가봐야겠다.
