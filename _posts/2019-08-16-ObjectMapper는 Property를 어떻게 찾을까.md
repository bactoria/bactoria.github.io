---
layout: post
title: "ObjectMapper는 Property를 어떻게 찾을까 ?"
categories: Java
author: bactoria
---

최근에 Restful API 에 POST Method로 요청 시에 Body의 Json을 requestDto로 매핑하는데 있어서  @Setter가 필요없다는 것을 알게 되었다.

[@Reuqest Body에서는 Setter가 필요없다? - jojoldu](https://jojoldu.tistory.com/407)

스프링에서 POST Method로 요청을 받을 시에, `Jackson2HttpMessageConverter`가 Http Message의 Body(Json)를 객체(ReuqestDto)로 매핑시켜주는데, 이 과정에서 `ObjectMapper.class` 의 `readValue(String content, Class<T> valueType)` 를 사용한다. ObjectMapper를 사용하니까 dto에 setter가 없어도 매핑 된다는 것이었다.

&nbsp;

그래서 requestDto에 Getter/Setter 없이 쓰려고 했는데, 아래와 같이 실패했다.

```java
@NoArgsConstructor
class RequestDto {
    private String name;
    private long value;
}

public class ObjectMapperTest {
    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    public void ObjectMapper를_이용하여_Json을_객체로_매핑한다() throws IOException {
        String json = "{\"name\":\"name\", \"value\": 1}";
        mapper.readValue(json, RequestDto.class);
    }
}
```

**Error**

```java
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException:  
Unrecognized field "name" (class RequestDto), not marked as ignorable (0 known properties: ])  
at [Source: (String)"{"name":"name", "value": 1}"; line: 1, column: 10] (through reference chain: RequestDto["name"])
```

&nbsp;

**UnrecognizedPropertyException** 이 발생했다.


>#### Property ?
>  
>자바스크립트의 객체는 키(Key)와 값(Value)으로 구성된 프로퍼티(Property)들의 집합이다.

json에는 `name`, `value` 키를 가진 Property가 있는데, 객체에는 `name`, `value`를 찾지 못하는 상황이다

&nbsp;

그런데 여기서 **@Getter**를 추가했더니 매핑이 되더라. 

**아니... 이게 왜 되지?**

---

&nbsp;
&nbsp;

### ObjectMapper는 Property를 어떻게 찾는가?

&nbsp;

#### ObjectMapper가 객체의 프로퍼티를 찾는 기본 정책을 찾아봤다. 

(Version: jackson-databind 2.9.9)

&nbsp;

**ObjectMapper.class**
```java
public class ObjectMapper extends ObjectCodec implements Versioned, Serializable {
    protected final ConfigOverrides _configOverrides;
    //...

    public ObjectMapper() {
        this((JsonFactory)null, (DefaultSerializerProvider)null, (DefaultDeserializationContext)null);
    }

    public ObjectMapper(JsonFactory jf, DefaultSerializerProvider sp, DefaultDeserializationContext dc) {
       //...
        this._configOverrides = new ConfigOverrides();
        //...
    }
}
```

`new ObjectMapper();` 를 할 때 `new ConfigOverrides();`를 호출한다.

&nbsp;

**ConfigOverrides.class**

```java
public class ConfigOverrides implements Serializable {
    public ConfigOverrides() {
        this((Map)null, 
            Value.empty(),
            com.fasterxml.jackson.annotation.JsonSetter.Value.empty(),
            Std.defaultInstance(), 
            (Boolean)null);
    }
}
```
`Std.defaultInstance()` 를 호출한다.

&nbsp;

**VisibilityChecker.class**
```java
public interface VisibilityChecker<T extends VisibilityChecker<T>> {
    public static class Std implements VisibilityChecker<VisibilityChecker.Std>, Serializable {
        protected static final VisibilityChecker.Std DEFAULT;
        //...

        public static VisibilityChecker.Std defaultInstance() {
            return DEFAULT;
        }

        static {
            DEFAULT = new VisibilityChecker.Std(
                Visibility.PUBLIC_ONLY, 
                Visibility.PUBLIC_ONLY, 
                Visibility.ANY, 
                Visibility.ANY, 
                Visibility.PUBLIC_ONLY);
        }

        public Std(Visibility getter, Visibility isGetter, Visibility setter, Visibility creator, Visibility field) {
            this._getterMinLevel = getter;
            this._isGetterMinLevel = isGetter;
            this._setterMinLevel = setter;
            this._creatorMinLevel = creator;
            this._fieldMinLevel = field;
        }        
    }
}
```

&nbsp;

|타입|private|default|protected|public|
|--|--|--|--|--|
|Creator|o|o|o|o|
|Getter|x|x|x|o|
|isGetter|x|x|x|x|
|Setter|o|o|o|o|
|Field|x|x|x|o|

&nbsp;

Json를 객체에 파싱할 때 객체의 프로퍼티 정보를 알기 위해서는 

객체 생성을 위한 `기본 생성자`(접근 제한자 무관)가 반드시 존재해야 하고, 

`Getter`/`Setter`/`Field` 중에 접근 제한자가 위 조건을 만족하는 것이 하나라도 있어야 한다.

(Dto에서 Builder 패턴 사용할 때 `AllArgsConstructor` 쓰는데, ObjectMapper를 위해서 `NoArgsConstructor` 를 꼭 추가해줘야 함.)

&nbsp;
&nbsp;

### Getter/Setter 없이 Property를 읽도록 하는 방법들

#### 1. @JsonProperty

```java
@NoArgsConstructor
class RequestDto {
    @JsonProperty("name")
    private String name;
    @JsonProperty("value")
    private long value;
}
```

필드에 어노테이션만 추가해주어도 프로퍼티를 읽을 수 있다.

다만, 모든 필드에 추가하기에는 부담이 있다.

&nbsp;

#### 2. @JsonAutoDetect

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)
@NoArgsConstructor
class RequestDto {
    private String name;
    private long value;
}
```

클래스에 어노테이션을 추가하는 방법도 있다.

클래스단위로 Visibility를 설정할 수 있다.

&nbsp;

#### 3. ObjectMapper 설정 변경

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
``` 

objectMapper로 매핑할 Dto에게 모두 동일한 Visibility를 부여하고자 한다면 이 방법이 깔끔한 것 같다.

필드의 접근제한자가 private라도 읽을 수 있기에, 기본생성자와 필드만 있으면 읽을 수 있다. 이 방법이 성능저하를 일으키는지는 잘 모르겠다.

---

&nbsp;

RequestDto에서 Entity로 변환할 때 `ModelMapper`, `Setter`, `Builder(생성자) `, `toEntity()` 로 할 수 있는데, toEntity()를 사용한다면 dto에는 더이상 setter/getter가 필요가 없어진다.

(`ModelMapper`의 경우에도 아래처럼 설정한다면 `Getter/Setter` 없이 매핑 할 수 있다.)

```java
ModelMapper modelMapper = new ModelMapper();
modelMapper.getConfiguration()
        .setFieldMatchingEnabled(true)
        .setFieldAccessLevel(Configuration.AccessLevel.PRIVATE);
```

다만, 테스트할 때에는 불편할 것 같다.

@Controller 테스트 할 때 dto에 대한 검증을 @toString으로 해야 한다거나, 

@Service 테스트 할 때 dto를 만들기 위해서 objectMapper를 써야할 것 같다.

그렇다고 테스트의 편의를 위해 getter/setter를 추가하는 것이 좋은 방법일까는 모르겠다. 어떤 것이 더 나은 방법일까. 이것도 트레이드 오프인가

&nbsp;

---

&nbsp;

### (Gson)

**Gson**은 기본생성자와 getter/setter 없이, 필드가 private 이어도 매핑이 가능함. 

오와우~

```java
@ToString
class RequestDto {
    private String name;
    private long value;

    // 생성자 접근시 RuntimeException 발생!!
    public RequestDto(String name, long value) {
        throw new RuntimeException();
    }
}

public class GsonTests {
    private static final Gson gson = new Gson();

    @Test
    public void Gson은_객체에_기본생성자_setter_getter가_없어도_매핑된다() {
        String json = "{\"name\":\"name\", \"value\": 1}";
        String result = "RequestDto(name=name, value=1)";

        RequestDto requestDto = gson.fromJson(json, RequestDto.class);

        assertThat(requestDto.toString()).isEqualTo(result); // Test passed!
    }
}
```

