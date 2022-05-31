# equals() 와 hashCode()

equals와 hashCode는 모든 Java 객체의 부모 객체인 Object 클래스에 정의되어 있다. 따라서 Java의 모든 객체는 Object 클래스에 정의돈 equals와 hashCode를 상속받고 있다.

## equals()

boolean equals(Object obj)로 정의돈 equals 메서드는 2개의 객체가 동일한지 검사하기 위해 사용된다. 방법은 2개의 객체가 참조하고 있는 대상이 동일한지를 확인하는 것이다.

만약에 우리가 서로 같은 값을 가지는 다른 String 타입의 변수 2개를 생성하면, 각 변수는 서로 다른 참조 값을 가진다. 

```java
String s1 = new String("ABC");
String s2 = new String("ABC");

s1 == s2 // false
s1.equals(s2) // true
```

따라서, s1과 s2가 참조하는 값인 각 ABC는 서로 다른 주소 값을 가지고 있다.

하지만 String의 equals 메서드를 사용하여 s1과 s2를 확인하면 결과는 true가 반환된다.

그 이유는 String Class에서 equals 재정의하는 오버라이드 메서드를 가지고 있기 때문이다.

```java
/**
* String Class의 equals 메서드
*/
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                                  : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }
//equals로 호출된 대상이 참조하는 주소값이 동일하면 true
//그렇지 않으면 호출된 대상이 String인지 확인 후, 
// 각 String의 자리마다 값이 일치하는지 확인하여 일치 여부를 반환한다
```

## hashCode

hashCode 메서드는 실행 중에 객체의 유일한 integer값을 반환한다. Object 클래스에서는 heap에 저장된 객체의 메모리 주소를 반환하도록 되어있다.

만약 equals를 재정의할 경우, hashCode도 반드시 재정의 해야한다.