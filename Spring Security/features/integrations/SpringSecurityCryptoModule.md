# Spring Security Crypto Module

## 😀 Introduction

Spring Security Crypto module은 대칭 암호화 및 키 생성, 암호 인코딩을 지원합니다.  코드는 코어 모듈과 함께 배포되지만, 그 어떤 Spring Security 코드와의 의존성은 갖고 있지 않습니다.

## 😀 Encryptors

Encryptors 클래스는 대칭 암호화를 위한 symmetric encryptor를 생성하는 팩토리 메서드를 제공합니다. 클래스를 사용하면, raw byte[] 형태로 암호화하는 ByteEncryptors를 만들 수 있습니다. 또한 text string을 암호화하는 TextEncryptors를 만들 수도 있습니다. 이 Encrytor는 Thread-safe 합니다.

### BytesEncryptor

`Encryptors.stronger` 팩토리 메서드를 사용하여 BytesEncryptor를 생성합니다:

**Bytes Encryptor**

```java
Encryptors.stronger("password", "salt");
```

“strong” 암호화 메서드는 256 bit 암호 강도의 AES 암호화 방식이며, Galois Counter Mode (GCM) encryptor를 생성합니다. 이는 PKCS #5의 PBKDF2를 사용하여 secret key를 얻습니다. 이 메서드는 Java 6버전 이상을 요구합니다. 패스워드는 비밀키를 만들 때 사용되며, 이는 안전한 장소에 보관해야하며 공유해선 안됩니다. salt는 암호화한 데이터가 손상되었을 때 키에 대한 사전 공격을 방지하기 위해 사용됩니다. 16-byte 난수 초기화 백터는 각각의 암호화된 메시지에 적용되며, 이는 유니크 합니다.

제공되는 salt는 hex로 인코딩 된 String 형태이어야 하며, 난수이며, 최소 8byte 이상의 길이어야합니다. 아래는 KeyGenerator를 사용하여 salt를 만드는 방법의 예시입니다:

```java
String salt = KeyGenerators.string().generateKey(); // generates a random 8-byte salt that is then hex-encoded
```

사용자는 또한 256-bit 강도의 AES 암호화, CBC 모드에 해당하는 `standard` 암호화 메서드를 사용할 수 있습니다. 이 모드는 [인증](https://en.wikipedia.org/wiki/Authenticated_encryption) 되지 않았으며 데이터의 진위 여부를 보장하지 않습니다. 가급적이면, `Encryptors.stronger` 를 사용하는 것이 좋습니다.

### TextEncryptor

`Encryptors.text` 팩토리 메서드를 사용하여 TextEncryptor를 생성합니다:

**TextEncryptor**

```java
Encryptors.text("password", "salt");
```

TextEncryptor는 표준 BytesEncryptor를 사용하여 text 데이터를 암호화합니다. 암호화된 결과는 데이터베이스 또는 파일시스템에 쉽게 저장할 수 있도록 hex 인코딩된 String 값으로 반환됩니다.

“queryable”한 TExtEncryptor를 생성할 땐 Encryptors.queryableText 팩터리 메서드를 사용합니다.

```java
Encryptors.queryableText("password","salt");
```

queryable TextEncryptor와 standard TextEncryptor의 차이점은 initialization vector(IV) 를 다루는지에 대한 여부입니다. IV는 queryable TextEncryptor#encrypt 연산에서 사용하는데, 이를 공유하거나, 고정된 값이며, 무작위로 생성하지 않습니다. 그 말인즉슨, 동일한 텍스트를 여러 번 암호화했을 때, 매번 동일한 암호화된 결과를 준다는 의미입니다. 이는 덜 안전하지만, 암호화 데이터를 질의해야할 때는 필요합니다. OAuth API KEY 같은 경우 질의가능한 암호화된 텍스트를 사용합니다.

## 😀 Key Generators

KeyGenerators 클래스는 서로 다른 타입의 key generator를 생성하는데 유용한 팩터리 메서드 몇가지를 제공합니다. 이 클래스를 사용하면, byte[] key를 생성하는 `BytesKeyGenerator`, string 형태의 key를 생성하는 `StringKeyGenerator` 를 생성할 수 있습니다. Key Generators는 thread-safe 합니다.

### BytesKeyGenerator

SecureRandom 인스턴스를 뒷받침하는 BytesKeyGenerator를 생성할 때에는 BytesKeyGenerator KeyGenerators.secureRandom 팩토리 메서드를 사용합니다:

**BytesKeyGenerator**

```java
BytesKeyGenerator generator = KeyGenerators.secureRandom();
byte[] key = generator.generateKey();
```

기본 키 길이는 8bytes 입니다. 이는 또한 `KeyGenerators.secureRandom` 메서드에 키 길이를 파라미터로 작성하여, 키 길이를 변경할 수도 있습니다.

```java
KeyGenerators.secureRandom(16);
```

항상 동일한 키를 리턴하는 BytesKeyGenerator를 만들때는 `KeyGenerators.shared` 팩터리 메서드를 사용합니다.

```java
KeyGenerators.shared(16);
```

### StringKeyGenerator

각 키를 hex-인코딩하는 8 bytes SecureRandom KeyGenerator를 만들 때는 `KeyGenerators.string` 팩토리 메서드를 사용합니다:

```java
KeyGenerators.string();
```

## 😀 Password Encoding

spring-security-crypto 모듈의 password 패키지는 패스워드 인코딩 지원을 제공합니다. `PasswordEncoder` 는 핵심 서비스 인터페이스이며 시그니처는 아래와 같습니다.

```java
public interface PasswordEncoder {
	String encode(String rawPassword);
	boolean matches(String rawPassword, String encodedPassword);
}
```

`matches` 메서드는 rawPassword와 암호화된 Password의 비교 결과가 일치하는지는 반환해줍니다. 이 메서드는 Password-based 인증 스키마를 지원하기 위해 만들어졌습니다.

`BCryptPasswordEncoder` 구현체는 “bcrypt”를 광범위하게 지원하는 비밀번호 해시 알고리즘입니다.

Bcrypt는 16 byte 길이의 랜덤한 salt 값을 사용하며, Password cracker로부터 cracking(i.e. BRute Force Attack, Rainbow Table Attack 등 패스워드를 알아내기 위해 공격하는 것을 일컬음)할 수 없도록 의도적으로 느리게 동작합니다. 작업량은 “strength” 파라미터를 조정하여 4~31 값으로 튜닝할 수 있습니다. “strength”의 값이 높을수록, 해시 값을 계산하기 위해 더 많은 양의 작업을 수행합니다. 기본값은 10입니다. 이 값은 인코딩한 해시에 함꼐 저장되므로 기존에 배포한 시스템에서도 비밀번호에 영향 없이 이 값을 변경할 수 있습니다.

```java
// Create an encoder with strength 16
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(16);
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

`Pbkdf2PasswordEncoder` 구현체는 PBKDF2 알고리즘으로 비밀번호를 해싱할 때 사용합니다. PBKDF2는 Password cracker로부터 cracking 할 수 없도록 의도적으로 느리게 동작합니다.시스템에서 비밀번호 하나를 검증하는 데에는 0.5초 정도가 소요되도록 설정해야 합니다.