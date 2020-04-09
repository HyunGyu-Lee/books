# equals는 일반 규약을 지켜 재정의하라
#### \#1. equals 개요
- 기본적으로 equals 메소드를 재정의하지 않으면 인스턴스는 자기 자신과만 같다.
- 다음 케이스에서는 equals를 재정의하지 않는것이 낫다.
  - Thread와 같이 동작하는 개체를 표현한 경우
  - 논리적 동치성을 검사할 일이 없는 경우
  - 상위에서 재정의한 equals를 하위 클래스에서 그대로 재사용할 수 있는 경우
    - Map, Set
  - 클래스가 비공개 (private, package-private)이고 equals가 호출될 일이 없는 경우
- 논리적 동치성을 확인해야하는 경우라면 equals를 재정의 해야한다.
  - 주로 값 객체가 이에 해당한다.

#### \#2. equals 재정의 일반 규약
- 반사성 (reflexivity) : null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true
- 대칭성 (symmetry) : null이 아닌 모든 참조 값 x, y에 대해 x.eqauls(y)가 true면 y.equals(x)도 true
- 추이성 (transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true
- 일관성 (consistency) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 참조값 x에 대해 x.equals(null)은 false다.

#### \#3. 위 규약을 지키며 좋은 equals 작성하기
1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인
2. `instanceof` 연산자를 통해 입력이 올바른 값인지 확인
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 **핵심 필드**들이 모두 일치하는지 하나씩 검사
  - 기본형은 `==`, 참조형은 equals로 비교
  - **float**, **double**의 경우 부동소수점 이슈등의 문제로 `Float.compare`, `Double.compare` 메소드를 사용

#### \#4. 주의사항
- equals를 재정의할 땐 hashCode도 재정의하자
- 너무 복잡하게 해결하려 들지 말자 (핵심 필드들의 동치성 검사만으로도 규약을 지킬수 있다)
- equals 재정의시 입력타입은 반드시 Object 형이여야한다.
  - 하위 클래스에서 재정의 시 긍정오류 발생
  - 잘못된 정보를 제공
  
 #### \#5. 참고
 - AutoValue 라이브러리를 사용하면 자동으로 양질의 equals 메소드를 생성
 - IDE의 자동완성 기능 사용 (AutoValue 보다는 퀄리티가 떨어지지만, 직접 작성해서 실수하는 것 보단 나음)