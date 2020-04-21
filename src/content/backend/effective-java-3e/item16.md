# public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라
- 가끔 인스턴스를 모아두는 퇴보한 클래스를 작성하는 경우가 있다.
- 이런 클래스는 public이어서는 안된다.
  - 이 클래스는 데이터 필드에 직접 접근이 가능해 캡슐화의 이점을 제공하지 못한다.
  - API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
  - 불변식을 보장할 수 없다.
  - 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
```java
class Point {
    public double x;
    public double y;
}
```
- 객체지향 프로그래머라면 아래와 같이 작성한다.
```java
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }    

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }    
}
```
- 패키지 바깥에서 접근 가능한 클래스라면 접근자를 제공하여 클래스 내부 표현 방식을 언제든 바꿀수 있다.
- package-private 클래스 혹은 private 중접 클래스라면 데이터 필드를 노출해도 문제가 없다.
  - 어차피 패키지 내부에서만 동작하고 패키지 외부엔 영향을 미치지 않기 때문이다.
  - private 중첩 클래스는 더 제한적
- 자바 플랫폼에서 Point, Dimension 클래스는 이러한 제약을 어긴 대표적인 사례로 따라하지 않도록 하자.