## 인터페이스 이야기
- 이종간 경계면.
- 인터페이스가 책임과 역할을 규정한다.
- 자바는 컴파일 규칙으로 인터페이스를 달성한다.

## Favor Composition over inheritance

### 관련 정보
- Adapter
- Decorator
- Facade

구성이란
```java
public class Origin {
    public void parent() {
        ...
    }
}

// inheritance
public class Detail extends Origin {
    public void child() {
        ...
    }
}

//composition
public class Detail {
    private Origin origin = new Origin();

    public void parent() {
        this.origin.parent()
    }

    public void child() {
        ...
    }
}
```

### Adapter
1. 이미 존재하는 인터페이스가 있다.
2. 인터페이스에 해당하지 않는 클래스가 있다.
3. 인터페이스로 클래스를 사용하고 싶다.
4. 해당 클래스가 직접 인터페이스를 구현하지 않고, 어댑터를 통해 구현하도록 한다.

```java
interface IPlug {
    public Voltage plug();
}

// 타겟
public class 110V {
    private Voltage voltage = 110;
    public void eletron(){
        return voltage;
    }
}

// 어댑터
public class Adapter implements IPlug {
    private 110V old = new 110V();
    private Voltage extraVoltage = 110;

    public void plug(){
        return extraVoltage + this.old.electron();
    }
}
```

### Decorator
장식자. 코어 기능을 재사용하면서 책임(기능)을 덧붙인다.

```java
interface Shape {
    public void area();
}

public abstract class ShapeDecorator implements Shape {
    protected Shape shape;
    public ShapeDecorator (Shape shape) {
        this.shape = shape;
    }

    public void area() {
        shape.area();
    }
}

public class Rectangle extends ShapeDecorator {
    public Rectangle (Shape shape) {
        super(shape);
    }

    public void area() {
        shape.area();
        printName();
    }
    
    private void printName() {
        System.out.prinln("Rectangle");
    }
    ...
}
```

### Facade
입면이란 뜻. 내부를 숨기고 대리인을 내세우는 형태.

```java
public class Grinder {
    public void grind() {
        ...
    }
}

public class WaterPump {
    public void water() {
        ...
    }
}

public class CoffeeCapsule {
    public void coffee() {
        ...
    }
}
// FACADE
public class Nespresso {
    private Grinder grinder;
    private WaterPump water;
    private CoffeeCapsule coffee;

    public void dripCoffee() {
        grinder.grinder();
        water.water();
        coffee.coffee();
        ...
    }
    ...
}
```
