# item 09 try-finally보다는 try-with-resources를 사용하라

# 요약

close를 통해 회수해야 하는 자원을 다룰 때는**반드시**try-with-resources를 사용하자.

가독성이 좋고 정확하게 회수 가능하다.

또한 커스텀 자원을 회수해야 하는 경우 AutoCloseable 인터페이스를 구현하는 것을 잊지 말도록 하자. (아이템 8)

# 주요 내용

자바에는 close를 호출해 직접 닫아줘야 하는 자원들이 있음 (I/O Stream,Connection 등)

만약 **그냥 자원을 다 쓰고 close를 호출하는 식**으로 코드를 짜면 문제가 발생한다.

```java
public static String inputString()throws IOException{
    BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
    String result=br.readLine();
    br.close(); // 위에서 에러나면, 메모리에 br이 상주하는 문제 발생
    return result;
    }
```

BufferedReader는 사용 중 IOException이 발생할 수 있는데, 만약 br.readLine() 메서드에서 IOException이 발생하게 되면 메서드가 종료되므로
close가 호출되지 않고 스트림이 메모리에 남아있게 된다.

따라서 **예외가 발생해도 닫을 수 있게** try-finally를 사용한다.

```java
public static String inputString()throws IOException{
    BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
    try{
    return br.readLine();
    }finally{
    br.close(); // 예외가 발생하더라도 닫음
    }
    }
```

또한 만약 **try**절을 하던 도중, 문제가 생기면, **try**절 뿐만 아니라 **finally**절도 예외를 던지게 되는데 이 때, **finally**절에서 터진 예외가
**try** 블록에서 생긴 예외를 집어 삼켜서 **finally**절의 예외만 체크하게 된다. ( 예를들어 try에서 예외1, finally에서 예외2가 동시에 터지면 예외2만
stackTrace에 출력됨)

⇒ 이는 최초 예외 원인을 체크하지 못해 추후 문제가됨.

또한 만약 close를 여러번 해야하는 한다면 다음과 같은 코드가 나오는데, 보기 지저분해진다. (=유지보수 힘듬)

```java
public static void inputAndWriteString()throws IOException{
    BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
    try{
    BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(System.out));
    try{
    String line=br.readLine();
    bw.write(line);
    }finally{
    bw.close();
    }
    }finally{
    br.close();
    }
    }
```

### 해결책으로 **try-with-resources를 사용하면된다!**

이런 문제를 해결하기 위해 자바7부터 try-with-resources가 도입되어, 사용하는 자원이 `AutoCloseable`인터페이스를 구현하면 사용할 수 있다. (이미 대부분
라이브러리가 구현을 해놨고, close가 필요한 자원 클래스를 커스텀해야한다면, 반드시 구현 추천) (아이템 8)

```java
public interface AutoCloseable {

    void close() throws Exception;
}
```

이제 위에 메서드를 리팩토링 해보겠다.

```java
public static String inputString()throws IOException{
    try(BufferedReader br=new BufferedReader(new InputStream(System.in))){
    return br.readLine();
    }
    }
```

이제 기존에 문제였던 가독성도 좋아졌고, 예외도 집어 삼키는 일어 없어졌다.

(기존에 삼켜졌던 예외는 `Suppressed`: 로 stackTrace에 출력됨)

# 질문 및 추가