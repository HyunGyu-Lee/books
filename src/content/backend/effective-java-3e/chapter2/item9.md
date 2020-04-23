# try-finally보다는 try-with-resources를 사용하라
#### \#1. 자바의 자원 처리와 전통적인 try-finally
- InputStream, OutputStream, Connection 등 자원을 닫아주어야한다.
- 제대로 처리하지 않으면 성능문제로 직결되어 신경써주어야한다.
- 전통적인 try-finally 패턴
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
- 제대로 처리해두지 않으면 예외가 집어삼켜질 수 있고 디버깅이 어려워진다.
- 이런 방식은 자원이 2개이상 될 경우 너무 지저분해지고, 마찬가지로 제대로 처리해두지 않으면 예외가 집어삼켜질 수 있고 디버깅이 어려워진다.

#### \#2. try-with-resources를 통한 처리
- 자바7에서는 `try-with-resources`로 완전이 해결되었다.
- 이 방식은 매우 간결해 읽기 쉬우며 디버깅도 쉽다.
- `try-with-resources` 기능을 이용하려면 닫아야하는 자원을 구현한다면 `AutoCloseable`을 구현해주어야한다.
- try-with-resources를 이용한 방법
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
- 만약 readLine메소드와 자동으로 처리된 close 두 곳에서 오류가 발생한다면, 오류는 readLine 호출의 오류가 노출된다.
  - try-finally에서는 close 오류에 readLine오류가 먹히게되는 점이 개선됨
  - 게다가 close오류도 소멸되는게 아니라 숨겨졌다가(suppressed) 나중에 출력된다. (getSuppressed() 메소드 통해 프로그램 상에서 가져올 수도 있다.)
- try-with-resources를 활용한 다중 자원 처리
```java
static void copy(String src, String desc) throws IOException {
    try (InputStream in = new FileInputStream(src));
         OutputStream out = new FileOutputStream(dest)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) {
          out.write(buf, 0, n);
        }
    }
}
```
- try-with-resources 구문에서도 catch절을 쓸 수 있다. 