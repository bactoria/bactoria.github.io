---
layout: post
title:  "ModelAttribute와 RedirectAttributes"
categories: Spring
author: bactoria
---

**DB**
![image](https://user-images.githubusercontent.com/25674959/58744822-fc003e00-8482-11e9-9341-24b600fdb00c.png)

**HelloVO**
![image](https://user-images.githubusercontent.com/25674959/58745084-eb51c700-8486-11e9-91e0-7ad1984fe45f.png)

**http://localhost:8080/hello/1**
```json
{
    "id": 1,
    "name: "이름",
    "regDateTime": null
}
```

DB의 `reg_date_time` 과 스프링의 `regDateTime`이 바인딩이 안됨.

snake -> camel 로 변경하는 뭔가가 필요함.

&nbsp;
&nbsp;

### Case 1. Alias ✔️

**HelloMapper.xml**
```xml
    <select id="findAll" resultType="me.bactoria._27_FileUpload.domain.hello.Hello">
        SELECT id, name, reg_date_time as regDateTime
        FROM hello;
    </select>
```

&nbsp;

**http://localhost:8080/hello/1**
```json
{
    "id": 1,
    "name: "이름",
    "regDateTime": "2019-06-01T07:00:46.818145"
}   
```

regDateTime 이 아닌, regdatetime으로 나오지만, 자바에선 필드명을 대소문자 구분하지 않기 때문에 바인딩이 가능한 것 같음.

하지만, 하나하나 저렇게 적기엔 뭔가 꺼림칙함.

&nbsp;
&nbsp;

### Case 2. mybatis 설정 변경 (application.yml) ❌

**application.yml**
```yarm
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

snake를 camel로 바꾸는 설정은 맞지만, 나는 sqlSession 빈을 직접 등록했기 때문에 application.yml에 추가해도 적용이 안됨.

&nbsp;

**http://localhost:8080/hello/1**
```json
{
    "id": 1,
    "name: "이름",
    "regDateTime": null
}
```

&nbsp;
&nbsp;

### Case 3. mybatis 설정 변경 (@Bean) ✔️

**DateBaseConfig**
```java
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource datasource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(datasource);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:me/bactoria/_27_FileUpload/mapper/*.xml"));
        sqlSessionFactory.setConfiguration(getSqlSessionConfig()); // (추가)
        return sqlSessionFactory.getObject();
    }

    // (추가)
    private org.apache.ibatis.session.Configuration getSqlSessionConfig() { 
        org.apache.ibatis.session.Configuration sqlSessionConfig = new org.apache.ibatis.session.Configuration();
        sqlSessionConfig.setMapUnderscoreToCamelCase(true);
        return sqlSessionConfig;
    }
```

&nbsp;

**http://localhost:8080/hello/1**
```json
{
    "id": 1,
    "name: "이름",
    "regDateTime": "2019-06-01T07:00:46.818145"
}   
```

**바인딩 성공!**
