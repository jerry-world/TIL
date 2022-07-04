# 단일 책임 원칙(Single Responsibility Principle)

본 내용은 `개발자가 반드시 정보해야할 객체 지향과 디자인 패턴(저자: 최범균)` 를 읽고 내용을 정리하며 저만의 이해 방식으로 다시 기술한 내용입니다.

>💡 클래스는 단 하나의 책임을 가져야 한다.

`SOLID`의 `S`는 클래스가 단 하나의 책임을 갖도록하는 원칙이다. 여기서 말하는 책임이라는 것은 무엇을 의미하는 것일까?

책임이란 맡아서 해야 할 임무나 의무를 말한다. 그래서 클래스의 책임이라는 것은 클래스가 맡아서 수행해야할 임무 즉, 클래스가 하나의 책임으로 갖는 필드들이나 메서드들을 의미하는 것이다. 여기서 말하는 하나의 책임은 그 클래스가 갖는 고유한 역할 이라고 생각하면 될 것 같다.

그러면 왜 클래스는 이 단일 책임을 준수해야하는 것일까? 여러 책임을 갖으면 안되는 것일까?

아래의 예시를 한번 살펴보자.

### 단일 책임을 위반하여 작성된 아주 범용적인 클래스

```java
public class DataViewer {
	public void display() {
		String data = loadHtml();
		updateGui(data);
	}

	public String loadHtml(){
		HttpClient client = new Httpclient();
		client.connect(url);
		return client.getResponse();
	}
		
	
	private void updateGui(String data){
		GuiData guiModel = parseDataToGuiData(data);
		tableUI.changeData(guiModel);
	}

	private GuiData parseDataToGuiData(String data){
		// data를 파싱하여 GuiData타입으로 변환
	}
}

//예시 출처 : 개발자가 반드시 정보해야할 객체 지향과 디자인 패턴(저자: 최범균)
```

위 DataViewer 클래스는 display 메서드를 통해, 요청에 따른 결과 정보를 GuiData로 변환하여 UI에 출력해주는 식의 클래스이다.

위의 클래스를 딱보면 왜 단일 책임 원칙을 위반했는지 알 수 없을 수도 있다. 보통 객체 지향에서 말하는 이 SOLID원칙은 변경이 일어나면 준수 여부가 명확히 들어나기도 한다.

### 변경의 시작

loadHtml 호출시 접근하는 대상 URL에 해당하는 데이터 제공 서버가 HTTP 프로토콜 기반에서 소켓 기반의 프로토콜로 변경되었다고 가정해보자.

소켓 기반 프로토콜로 변경되면서, Response의 반환 형태가 byte[] 로 변경되었다.

이에 따라 소스코드를 한번 변경해보자.

```java
public class DataViewer {
	public void display() {
		byte[] data = loadHtml();
		updateGui(data);
	}

	public byte[] loadHtml(){
		SocketClient client = new Socketclient();
		client.connect(server, port);
		return client.read();
	}
		
	
	private void updateGui(byte[] data){
		GuiData guiModel = parseDataToGuiData(byte[]);
		tableUI.changeData(guiModel);
	}

	private GuiData parseDataToGuiData(byte[] data){
		// data를 파싱하여 GuiData타입으로 변환
	}
}

//예시 출처 : 개발자가 반드시 정보해야할 객체 지향과 디자인 패턴(저자: 최범균)
```

loadHtml 메서드에서 대상 서버와의 연결 방식을 SocketClient를 통해 소켓 통신하도록 변경하였고, 반환타입을 byte[]로 변경하였다.

그렇다보니, 연쇄적으로 updateGui, parseDataToGuiData 메서드들도 전부 byte[] 타입을 받도록 변경해야했다.

그리고 또 이것 말고도 문제점이 있다.

지금 예시에서는 크게 어떤 라이브러리를 의존하는지 보여지지 않지만, 

- loadHtml 메서드에서 SocketClient 라이브러리를 의존하고,
- updateGui 메서드에서는 GuiData와 TableUI
- 그리고 parseDataToGuiData 메서드에서는 파싱을 위한 특정 라이브러리를

의존한다고 생각해보자. loadHtml 메서드는 파싱이나 화면 출력을 위한 라이브러리가 전혀 필요가 없는데도, 여러 책임을 갖는 클래스 안에 함께 있다보니, 어쩔 수 없이 이 라이브러리를 함께 필요해져버렸다.

이렇게 한번 생각해보자. 만약 이 클래스가 데이터를 읽는 클래스와 데이터를 화면에 보내주는 클래스로 분리했다면 어땠을까? 그리고 서로 공통적으로 사용하는 data에 대해서는 추상화하여 주고받게 하는 거지.

한번 그렇게 바꿔보자!!

```java
import solid.srp.nonsrp.SocketClient;

//특정 서버로 데이터를 요청하는 책임만 수행
public class DataLoader {
    private static final String SERVER = "192.168.0.1";
    private static final int PORT = 8086;

    public Data loadHtml(){
        SocketClient client = new SocketClient();
        client.connect(SERVER, PORT);
        return new ByteData(client.read());
    }
}
```

```java
//데이터를 화면으로 변경하여 출력해주는 책임만 수행
public class DataDisplayer{
	private TableUI tableUI;

  public void display(){
      DataLoader dataLoader = new DataLoader();
      Data data = dataLoader.loadHtml();
      updateGui(data);
  }

  private void updateGui(Data data){
      GuiData guiModel = parseDataToGuiData(data);
      tableUI.changeData(guiModel);
  }

  private GuiData parseDataToGuiData(Data data){
      // data를 파싱하여 GuiData타입으로 변환
      return new GuiData(data);
  }
}
```

```java
//DataLoader와 DataDisplayer가 함께 사용하는 데이터를 추상화
public abstract class Data{
}

public class StringData extends Data{
    public StringData(String data) {
    }
}

public class ByteData extends Data{
    public ByteData(byte[] data) {
    }
}
```

먼저 공통적으로 사용하는 Data를 추상화 하여, String이든 Byte배열이든 어떠한 정보가 오더라도 Data로 받을 수 있도록 추상화 하였다.

DataLoader 클래스에서는 HttpClient에서 SocketClient로 변경하였다.

DataDisplayer 클래스에서는 기존과 동일하게 코드 변화가 없었다.

여기서 말하고싶은바는 책임을 분리하여 다른 곳에 영향이 없게되었고, 확장도 보다 용이해졌다는 것이다.

이처럼 단일 책임 원칙은 변경하는 것에서 확실히 들어난다. 그러면 변경이 일어나기 전에 이 단일 책임 원칙을 지키려면 우리는 어떻게 사고하고 책임 분리를 계획해야할까?

### 변경을 예측하는 사고방법

앞선 내용을 통해, 단일 책임 원칙이 지켜지지 않을때 변경을 통해 겪게되는 어려움을 몸소 이해하였다.

이 단일 책임 원칙을 처음부터 지키려면 결국엔 변경에 대한 대응을 미리 해야한다는 것을 알 수 있다. 개발자는 어느 시점에서 어떻게 변경될지를 미리 예측해야하고 그럴 토대로 책임을 분리하고 객체 지향적으로 개발해야 한다.

그럼 `어떻게 이게 한개의 책임일까? 라는 생각에 대한 결론`을 스스로 낼 수 있어야한다는 것이다.

가장 쉬운 방법은 해당 객체를 사용하는 대상의 입장에서 이해해보면 된다.

우리가 위에서 책임을 분리했을 때에도, SocketClient로 변경되면서, 변경되지 않아도 되는 메서드에까지 영향을 미친다는 것을 알 수 있었고, 그를 통해 클래스를 분리해야겠다고 판단했다.

DataViewer를 사용하는 누군가는 Data를 요청하는 자와 UI를 요청하는 자로 분리될 수 있었고, 그리고 그 요청자들은 서로 각기 다른 메서드를 사용하고 있었다.(만약 Data 요청자가 UI도 요청했다면 이야기가 달라졌겠지..) 그럼에도 불구하고 서로 같은 클래스에 존재하다보니 영향을 미칠 수 밖에 없었고, UI와 관련없는 Socket이나 Http Client 라이브러리도 같이 가지고 있어야해

이처럼 클래스를 사용하는 대상의 입장에서 서로 각기 다른 메서드를 사용한다면 책임 분리를 고민해보아야한다. 물론 클래스가 작성되기 이전에 생각할 수 있으면 더 좋겠지…