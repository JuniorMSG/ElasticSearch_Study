# ElasticSearch

## 특징

    엘라스틱서치는 모든 요청과 응답을 Rest API 형태로 제공한다. 

## document

    엘라스틱서치에서 데이터가 저장되는 기본 단위 JSON 형태이다. 
    여러 필드(field)와 값(value)를 갖는다. 
    실제 데이터를 저장하는 단위
    도큐먼트 삭제와 수정은 비용이 많이 들어가는 작업이다.

| MySQL | ElasticSearch |
|-------|---------------|
| 테이블   | 인덱스           |
| 레코드   | 도큐먼트          |
| 칼럼    | 필드            |
| 스키마   | 매핑            |

## Index

    도큐먼트를 저장하는 논리적 단위 테이블과 유사한 개념임
    하나의 인덱스에 다수의 도큐먼트가 포함되며 동일 인덱스에 있는 도큐먼트는 동일한 스키마를 갖는다.
    모든 도큐먼트는 반드시 특정 인덱스에 속해야한다.

### 그룹핑 - 스키마

    일반적으로 스키마에 따라 인덱스를 구분한다.

### 그룹핑 - 관리 목적

    인덱스는 용량이나 숫자 제한 없이 무한대의 도큐먼트를 포함 가능하다
    인덱스 용량 / 일|주|월|년 단위등 날짜 시간 단위 분리 /

## Index CRUD

### Create / Update

    PUT index1 or POST index1
    PUT index2/_doc/1
    {
        "name" : "mike",
        "age" : 25,
        "gender" : "mail"
    }

    PUT index2/_doc/2
    {
        "name" : "jane",
        "country": "france"
    }

    
    PUT index2/_doc/3
    {
        "name" : "kim",
        "age" : 20,
        "gender" : "femail"
    }

#### update

    PUT index2/_doc/1
    {
        "name" : "mike",
        "age" : 45,
        "gender" : "mail",
        "country": "Korea"
    }

#### Dynamic mapping

    도큐먼트의 필드와 값을 보고 자동으로 타입을 지정해줌

### Read

    GET index1 | 인덱스 정보 읽기
    GET index2/_doc/1 | 개별 doc 읽기
    GET index2/_search | 해당 인덱스 전체 읽기

### DELETE

    도큐먼트 삭제와 수정은 비용이 많이 들어가는 작업이다.
    DELETE index1
    DELETE index2/_doc/2

## Bulk Data

    Bulk API는 생성/수정/삭제만 지원한다.
    NDJSON (New-line Delimited JSON (ndjson.org)) 형태
    각 줄 사이에는 별도의 구분자가 없고 라인 사이 공백을 허용하지 않는다.

### Example

    POST _bulk
    {"index": {"_index": "index2", "_id": "6"}}
    {"name":"hong", "age": 10, "gender": "female"}
    {"index": {"_index": "index2", "_id": "7"}}
    {"name":"choi", "age": 90, "gender": "male"}

    curl -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "bulk_test_01"

## 3.6 매핑

    스키마는 테이블을 구성하는 구성요소 간의 논리적인 관계와 정의를 의미함.
    스키마와 비슷한 역할을 하는 것이 매핑임.
    JSON 형태의 데이터를 루씬이 이해할 수 있또록 바꿔주는 작업 
    자동으로 하면 다이내믹 매핑
    사용자가 직접 명시적 매핑
    GET index2/_mapping 으로 확인 가능 

### 3.6.1 다이나믹 매핑

    엘라스틱서치의 모든 인덱스는 매핑 정보를 갖고 있지만 유연한 활용을 위해 인덱스 생성 시 매핑 정의를 강제하지 않는다. 
    특별히 데이터 타입이나 스키마에 고민하지 않아도 JSON 도큐먼트의 데이터에 맞춰 엘라스틱서치가 자동으로 인덱스 매핑을 해주는 것

#### 다이내믹 매핑 기준

| 원본 소스 데이터 타입 | 다이내믹 매핑으로 변환된 데이터 타입                    |                
|--------------|-----------------------------------------|
| null         | 필드를 추가하지 않음                             |                             
| boolean      | boolean                                 |                                 
| float        | float                                   |                                   
| integer      | long                                    |                                    
| object       | object                                  |                                  
| string       | string 데이터 형태에 따라 date, text/keyword 필드 | 

![image](https://user-images.githubusercontent.com/22822369/209065624-982d3449-2011-462c-b56a-b7e30124a232.png)

### 3.6.2 명시적 매핑 (explicit mapping)

    인덱스 매핑을 직접 정의하는 것. 

```
PUT index3
{
  "mappings":{
    "properties": {
      "age": {"type": "short"},
      "name": {"type": "text"},
      "gender": {"type": "keyword"}
    }
  }
}

```

![image](https://user-images.githubusercontent.com/22822369/209065541-a725b72d-7a02-41fd-875e-2376656f8000.png)

```
GET index3/_mapping

{
  "index3" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "short"
        },
        "gender" : {
          "type" : "keyword"
        },
        "name" : {
          "type" : "text"
        }
      }
    }
  }
}
```

### 3.6.3 매핑 타입

    매핑을 잘 활용하면 엘라스틱서치의 인덱스 성능을 올릴 수 있다 

| 데이터 형태 | 데이터 타입                                                                     | 설명                                                                                                                                                                                                |
|--------|----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 텍스트    | text                                                                       | 전문 검색이 필요한 데이터로 텍스트 분석기가 텍스트를 작은 단위로 분리한다.                                                                                                                                                        |
| 텍스트    | keyword                                                                    | 정렬이나 집계에 사용되는 텍스트 데이터로 분석을 하지 않고 원문을 통째로 인덱싱 한다.                                                                                                                                                  |
| 날짜     | date                                                                       | 날짜/시간 데이터                                                                                                                                                                                         |
| 정수     | byte,short,intger,long                                                     | * byte: 부호있는 8비트 데이터(-128~127) <br> * short: 부호 있는 16비트 (-32,768~32,767) <br> * integer: 부호 있는 32 비트 데이터<br> * long: 부호 있는 64 비트 데이터<br>                                                          |
| 실수     | scaled_float, half_float, dobule, float                                    | * scaled_float: float 데이터에 특정 값을 곱해서 정수형으로 바꾼 데이터, 정확도는 떨어지나 필요에 따라 집계 등에서 효율적으로 사용 가능하다.<br> * half_float: 16비트 부동소수점 실수 데이터<br> * dobule: 32비트 부동소수점 실수 데이터<br> * float: 64 비트 부동소수점 실수 데이터<br> |
| 불린     | boolean                                                                    | true/false                                                                                                                                                                                        |
| IP 주소  | ip                                                                         | ipv4, ipv6 타입 IP 주소                                                                                                                                                                               |
| 위치 정보  | geo-point, geo-shape                                                       | * geo-point: 위도, 경도 값<br> * geo-shape: 하나의 위치 포인트가 아닌 임의의 지형<br>                                                                                                                                  |
| 범위 값   | integer_range, long_range, float_range, double_range, ip_range, date_ragne | 범위를 설정할 수 있는 데이터, 범위 값을 저장하고 검색할 수 있게한다.                                                                                                                                                          |
| 객체형    | object                                                                     | 계층형 구조를 갖는 형태로 필드 안에 다른 필드들이 들어갈 수 있다. name: {"first": "kim", "last": "tony"}로 타입을 정의하면 name.fist, name.last 형태로 접근할 수 있다.                                                                        |
| 배열형    | nested                                                                     | 배열형 객체를 저아한다. 객체를 따로 인덱싱하여 객체가 하나로 합쳐지는 것을 막고, 배열 내부에의 객체에서 쿼리로 접근할 수 있다.                                                                                                                         |
| 배열형    | join                                                                       | 부모/자식 관계를 표현할 수 있다.                                                                                                                                                                               |

### 3.6.4 멀티 필드를 활용한 문자열 처리

> 엘라스틱 서치 5.x 버전부터 문자열 타입이 텍스트와 키워드 두 가지 타입으로 분리되어 있다.`
>

#### 3.6.4.1 텍스트 타입

> 텍스트 타입으로 지정된 문자열 분석기에 의해 토큰으로 분리되고, <br>
> 이렇게 분리된 토큰들은 인덱싱되는데 이를 역인덱싱이라고 한다. 이때 역인덱싱에 저장된 토큰들을 용어 라고한다.

> 엘리스틱서치에 텍스트 타입은 일반적으로 문장을 저장하는 매핑 타입으로 사용된다.<br/>
> 강제성은 없지만 일반적으로 문장이나 여러 단어가 나열된 문자열 텍스트 타입으로 지정한다.

> Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cum sociis
> natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Donec quam felis, ultricies nec,
> pellentesque
> eu, pretium quis, sem. Nulla consequat massa quis enim. Donec pede justo, fringilla vel, aliquet nec, vulputate eget,
> arcu. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium.
> Integer tincidunt. Cras dapibus. Vivamus elementum semper nisi. Aenean vulputate eleifend tellus. Aenean leo ligula,
> porttitor eu, consequat vitae, eleifend ac, enim. Aliquam lorem ante, dapibus in, viverra quis, feugiat a, tellus.
> Phasellus viverra nulla ut metus varius laoreet. Quisque rutrum. Aenean imperdiet. Etiam ultricies nisi vel augue.
> Curabitur ullamcorper ultricies nisi. Nam eget dui. Etiam rhoncus. Maecenas tempus, tellus eget condimentum rhoncus,
> sem
> quam semper libero, sit amet adipiscing sem neque sed ipsum. Nam quam nunc, blandit vel, luctus pulvinar, hendrerit
> id,
> lorem. Maecenas nec odio et ante tincidunt tempus. Donec vitae sapien ut libero venenatis faucibus. Nullam quis ante.
> Etiam sit amet orci eget eros faucibus tincidunt. Duis leo. Sed fringilla mauris sit amet nibh. Donec sodales sagittis
> magna. Sed consequat, leo eget bibendum sodales, augue velit cursus nunc,

``` 
POST _analyze
{
"analyzer": "standard",
"text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Donec quam felis, ultricies nec, pellentesque eu, pretium quis, sem. Nulla consequat massa quis enim. Donec pede justo, fringilla vel, aliquet nec, vulputate eget, arcu. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium. Integer tincidunt. Cras dapibus. Vivamus elementum semper nisi. Aenean vulputate eleifend tellus. Aenean leo ligula, porttitor eu, consequat vitae, eleifend ac, enim. Aliquam lorem ante, dapibus in, viverra quis, feugiat a, tellus. Phasellus viverra nulla ut metus varius laoreet. Quisque rutrum. Aenean imperdiet. Etiam ultricies nisi vel augue. Curabitur ullamcorper ultricies nisi. Nam eget dui. Etiam rhoncus. Maecenas tempus, tellus eget condimentum rhoncus, sem quam semper libero, sit amet adipiscing sem neque sed ipsum. Nam quam nunc, blandit vel, luctus pulvinar, hendrerit id, lorem. Maecenas nec odio et ante tincidunt tempus. Donec vitae sapien ut libero venenatis faucibus. Nullam quis ante. Etiam sit amet orci eget eros faucibus tincidunt. Duis leo. Sed fringilla mauris sit amet nibh. Donec sodales sagittis magna. Sed consequat, leo eget bibendum sodales, augue velit cursus nunc,"
}
```

![image](https://user-images.githubusercontent.com/22822369/209067814-cf556fe8-80b8-4c08-8490-43fd45ecc1e4.png)


> [this, documentation, is]와 같이 토큰으로 분리되고, 불필요한 토큰을 걸러내고 대소문자를 통일하는 등 가공 과정을 거쳐 용어가 된다. 이러한 용어들은 역인덱스에 저장되어 전문 검색을 할 수
> 있게 한다.
> <br> "documentation"를 검색하면 전체 문장을 검색할 수 있다. 일반적인 관계형 데이터베이스에 익숙할 경우 문자열 부분 검색으로 LIKE이 있지만 LIKE 검색은 인덱싱 되지 않아 엘라스틱서치처럼
> 많은 문서를 처리하기엔 무리가 있다.

```
PUT text_index
{
  "mappings": {
    "properties": {
      "contents": {
        "type": "text"
      }
    }
  }
}
```

```
PUT text_index/_doc/1
{
  "contents": "beatiful day"
}
```

```
GET text_index/_search
{
  "query": {
    "match": {
      "contents": "beatiful day"
    }
  }
}
```

![image](https://user-images.githubusercontent.com/22822369/209068351-10ea2027-43a7-45be-b06b-851b545753af.png)

    DSL 쿼리를 이용해 검색도 가능하다. 
    match는 전문 검색을 할 수 있는 쿼리이며, 
    contents 필드에 있는 역 인덱싱된 용어 중 일치하는 용어가 있는 도큐먼트를 찾는 쿼리이다.
    텍스트 타입의 경우 기본적으로 집계나 정렬을 지원하지 않으며, 
    매핑 파라미터로 집계나 정렬을 지원 할 수는 있으나 메모리를 많이 사용한다는 단점이 있다.

#### 3.6.4.2 키워드 타입

> 키워드 타입은 카테고리나 사람 이름, 브랜드 등 규칙성이 있거나 유의미한 값들의 집학, 즉 범주형 데이터에 주로 사용된다.  
> 키워드 타입은 텍스트 타입과 다르게 분석기를 거치지 않고 문자열 전체가 하나의 용어로 인덱싱된다.
> 키워드 타입은 "beautiful day"라는 문자열을 [beautiful day] 라는 1개의 용어로 만든다.
> 따라서 키워드 타입으로 매핑된 데이터는 부분 일치 검색은 어렵지만 대신 완전 일치 검색을 위해 사용할 수 있으며 집계나 정렬에 사용할 수 있다.

```
PUT keyword_index
{
  "mappings": {
    "properties": {
      "contents": {
        "type": "keyword"
      }
    }
  }
}

PUT keyword_index/_doc/1
{
  "contents": "beatiful day"
}

GET keyword_index/_search
{
  "query": {
    "match": {
      "contents": "beatiful day"
    }
  }
}
```

#### 3.6.4.3 멀티 필드

> 멀티 필드는 단일 필드 입력에 대해 여러 하위 필드를 정의하는 기능으로,   
> 이를 위해 fields라는 매핑 파라미터가 사용된다. fields는 하나의 필드를 여러 용도로 사용할 수 있게 만들어 준다.   
> 처음 매핑할 때 텍스트와 키워드를 동시에 지원이 가능하다.

```
PUT multifield_index 
{
  "mappings": {
    "properties": {
      "message": { "type": "text" },
      "contents": {
        "type": "text",
        "fields": {
            "keyword": { "type": "keyword" }
        }
      }
    }
  }
}

PUT multifield_index/_doc/1
{
  "message": "1 document",
  "contents": "beatiful day"
}
PUT multifield_index/_doc/2
{
  "message": "2 document",
  "contents": "beatiful day"
}
PUT multifield_index/_doc/3
{
  "message": "3 document",
  "contents": "wonderful day"
}

```

#### 인덱스 전문 쿼리

```
GET multifield_index/_search
{
    "query": {
        "match": {
            "contents": "day"
        }
    }
}
```

#### 인덱스 용어 쿼리

```
GET multifield_index/_search
{
    "query": {
        "term": {
            "contents.keyword": "wonderful day"
        }
    }
}
```

#### 인덱스 집계 쿼리

```
GET multifield_index/_search
{
    "size": 0,
    "aggs": {
        "contents":{
            "terms": {
                "field": "contents.keyword"
            }
            
        }
    }
}
```

## 3.7 인덱스 템플릿

> 인덱스 템플릿은 주로 설정이 동일한 복수의 인덱스를 만들 때 사용한다.  
> 관리 편의성, 성능 등을 위해 인덱스를 파티셔닝하는 일이 많은데 이때 파티셔닝되는 인덱스를은 설정이 같아야 한다.  
> 설정이 동일한 인덱스를 직접 인덱스 템플릿을 만들어 편리하게 사용할 수 있다.

### 3.7.1 템플릿 확인

```
GET _template
GET _template/ilm-history
GET _template/ilm*
```

특정 인덱스 템플릿만 확인할 수 있과 와일드카드 표현식을 이용해 특정 인덱스 템플릿을 확인할 수 있다.

### 3.7.2 템플릿 설정

#### 템플릿 생성

    test_template이라는 이름의 템플릿을 생성한다.

```
PUT /_template/test_template
{
  "index_patterns": ["test_*"],

  "settings":{
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings":{
    "properties":{
      "name": {"type": "text"},
      "age": {"type": "short"},
      "genger": {"type": "keyword"}
    }
  }
}
```

#### 템플릿 적용

> 템플릿을 만들기 전에 이미 존재하던 인덱스는 템플릿 패턴과 일치하더라도 템플릿에 적용되지 않는다.  
> test_template 템플릿이 적용되어 다이내믹 매핑이 아닌 test_template에 정의되었던 매핑값이 적용되어있습니다.  
> 템플릿이 없다면 다이내믹 매핑으로 동작하게 된다.

```
PUT test_index/_doc/1
{
  "name": "kim",
  "age": 10,
  "genger": "male"
}

GET test_index/_mapping
```

#### 템플릿 삭제

> 템플릿을 지워저도 기존 인덱스들은 영향을 받지 않는다.   
> 이미 만들어진 인덱스를 변경 되는 것이 아니고 단순히 템플릿이 지워지는 것 뿐이다.

```
DELETE _template/test_template
```

### 3.7.3 템플릿 우선순위

> 인덱스 템플릿 파라미터중 priority를 이용해 복수의 템플릿이 매칭될 경우 우선순위를 정할 수 있다. 숫자가 클수록 우선순위가 높다.

```
GET _template/test_*
PUT _template/test_template1
{
  "index_patterns": ["test_*"],
  "order": 2,
  "settings":{
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings":{
    "properties":{
      "name": {"type": "text"},
      "age": {"type": "short"},
      "genger": {"type": "keyword"}
    }
  }
}

PUT /_template/test_template2
{
  "index_patterns": ["test_*"],
  "order": 1,
  "settings":{
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings":{
    "properties":{
      "name": {"type": "text"},
      "age": {"type": "long"},
      "genger": {"type": "keyword"}
    }
  }
}

DELETE test_templateindex
PUT test_templateindex
GET test_templateindex
```

### 3.7.4 다이내믹 템플릿

> 다이내믹 템플릿은 매핑을 다이내믹하게 지정하는 템플릿 기술이다.  
> 매핑은 인덱스 내부의 데이터 저장과 검색 등의 기초가 되기 때문에 매핑은 신중하게 작업 해야한다.  
> 하지만 로그 시스템이나 비정형화된 데이터를 인덱싱하는 경우 로그 시스템 구조를 알지 못하기 때문에
> 필드 타입을 정확하게 정의하기 힘들고, 필드 개수를 정할 수 없는 경우도 있다.   
> 다이내믹 템플릿은 이처럼 매핑을 정확하게 정할 수 없거나 대략적인 데이터 구조만 알고있는 경우 사용할수 있는 방법이다.


> dynamic_index1 인덱스는 다이내믹 템플릿을 사용한다. my_string_fields는 임의로 정의한 다이내믹 템플릿의 이름이다.   
> match_mapping_type은 조건문 혹은 매핑 트리거다. 조건에 만족할 경우 트러거링이 된다.   
> 문자열 타입의 데이터가 들어오면 키워드 타입으로 매핑한다.

    다이내믹 템플릿에 의해 문자열을 가진 데이터는 모두 키워드가 타입으로 변경된다. 
    name 필드가 이에 해당한다.(기본적으로 다이내믹 매핑을하게되면 문자열 타입은 keyword가 아닌 text로 결정됨)

```
PUT dynamic_index1
{
  "mappings": {
    "dynamic_templates": [
      {
        "my_string_fields":{
          "match_mapping_type": "string",
          "mapping": {"type": "keyword"}
        }
        
      }
    ]
  }
}

PUT dynamic_index1/_doc/1
{
  "name": "mr. kim",
  "age": 40
}

GET dynamic_index1/_mapping
```

![image](https://user-images.githubusercontent.com/22822369/209103754-0d87a622-89af-41af-abf8-1f60d9d93148.png)

match 라는 정규표현식을 이용하여 필드명을 검사할 수 있다. match 조건에 맞는 경우 mapping에 의해 필드들은 모두 숫자 타입을 갖는다.

```
PUT dynamic_index2
{
  "mappings": {
    "dynamic_templates": [
      {
        "my_long_fields":{
          "match": "long_*",
          "unmatch": "*_text",
          "mapping": {"type": "long"}
        }
        
      }
    ]
  }
}
PUT dynamic_index2/_doc/1
{
  "long_num": "5",
  "long_text": "170"
}
GET dynamic_index2/_mapping
```

long_num 필드는 match 조건에 의해 문자열 숫자 타입으로 매핑되었지만 long_text는 match 조건에 부합하지만  
unmatch 조건에 부합하여 다이내믹 템플릿에서 제외되어 다이내믹 매핑에 의해 테스트/키워드를 갖는 멀티 필드 타입이 되었다.
![image](https://user-images.githubusercontent.com/22822369/209104491-ee566356-b2d1-4636-a7a1-5824d6919f6c.png)

| 조건문                      | 설명                                                                                     |
|--------------------------|----------------------------------------------------------------------------------------|
| match_mapping_type       | 데이터 타입을 확인하고 타입들 중 일부를 지정한 매핑 타입으로 변경한다.                                               |
| match, unmatch           | match: 필드명이 패턴과 일치하는 경우 매핑 타입으로 변경한다.<br> unmatch: match 패턴과 일치하는 경우 제외할 패턴을 지정할 수 있다. |
| match_pattern            | match 패턴에서 사용할 수 있는 파라미터를 조정한다. 정규식, 와일드 패턴 등을 지정할 수 있다.                               |
| path_match, path_unmatch | match,unmatch와 비슷하지만 `.`이 들어가는 필드명에서 사용한다.                                             |

## 3.8 분석기

    전문 검색을 지원하기 위해 역인덱싱 기술을 사용한다.
    전문 검색은 장문의 문자열에서 부분 검색을 수행하는 것
    역인덱싱은 장문의 문자열을 분석해 작은 단위로 쪼개어 인덱싱 하는 기술.
    양질의 결과를 얻기 위해서는 문자열을 나누는 기준이 중요하고 엘라스틱 서치는 캐릭터 필터, 토크나이저, 토큰 필터로 구성된 분석기 모듈을 갖고 있다.
    essential 분석기 1 - 토크나이저 1 / option 캐릭터 필터 | 토큰 필터

토큰과 용어 (token, term)

* 분석기가 토크나이저를 이용해 문자열을 자르면 그 잘린 단위가 토큰이며 정제하면서 최종으로 역인덱스에 저장되는 상태의 토큰들을 용어라고함.

* 토큰
  분석기 내부에서 일시적으로 존재하는 상태
* 용어
    * 인덱싱되어 있는 단위, 검색에 사용되는 단위는 모두 용어

### 3.8.1 분석기 구성

| 구성요소   | 설명                                               |
|--------|--------------------------------------------------|
| 캐릭터 필터 | 문자열의 전처리 작업  => 입력받은 문자열을 변경하거나 불필요한 문자들을 제거한다.  |
| 토크나이저  | 문자열을 토큰으로 분리한다. 분리할 때 토큰 순서나 시작, 끝 위치도 기록한다.     |
| 토큰 필터  | 분리된 토큰들의 필터 작업을 한다. 대소문자 구분, 형태소 분석 등의 작업이 가능하다. |

#### 역인덱싱

    분식기는 문자열을 토큰화하고 이를 인덱싱하는데 이를 역인덱싱이라고 한다. 
    책 뒷면에 있던 색인 처럼 가장 많이 쓰는 단어들을 선벌해 그 단어가 몇 페이지에 나와 있는지 알려주는 것을 색인(인덱스)라고 한다. 
    엘레스틱서치는 이와 비슷한 방법으로 단어들을 역인덱싱하여 도큐먼트를 손쉽게 찾을 수 있다.

#### 분석기 API

    필터와 토크나이저를 테스트 해볼 수 있는 analyze REST API 
    analyzer, text, tokenizer, explain, attributes 

[analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/indices-analyze.html)

```
POST _analyze
{
  "analyzer": "stop",
  "text": "The 10 most loving dog breeds."
}
```

#### 분석기 종류

| 분석기        | 설명                                                                                                                                |
|------------|-----------------------------------------------------------------------------------------------------------------------------------|
| standard   | 특별한 설정이 없다면 엘라스틱서치가 기본적으로 사용하는 분석기다. 영문법을 기준으로 한 스탠다드 토크나이저와 소문자 변경 필터, 스톱 필터가 포함되어 있다. <br> [The, 10, most, loving, log, breeds] |
| simple     | 문자만 토큰화한다. 공백, 숫자, 하이픈이나 작은 따음표 같은 무자는 토큰화하지 않는다. <br> [The, most, loving, dog, breeds]                                           |
| whitespace | 공백을 기준으로 구분하여 토큰화한다. <br> [The, 10, most, loving, log, breeds]                                                                    |
| stop       | simple 분석기와 비슷하지만 스톱 필터가 포함되어 있다. 스톱 필터에 의해 `the`가 제거되었다. <br> [most, loving, dog, breeds]                                        |

### 3.8.2 토크나이저

    토크나이저는 반드시 하나를 포함해야 한다.
    문자열을 분리해 토큰화 하는 역할을 하며 형태에 맞는 토크나이저 선택이 중요하다.

[Reference Tokenizers](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-tokenizers.html)

| 토크나이저         | 설명                                                                                                                                                                                                                                                 |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| standard      | 스탠다드 분석기가 사용하는 토크나이저로, 특별한 설정이 없으면 기본 토크나이저로 사용된다. 쉼표(,)나 점(.) 같은 기호를 제거하며 텍스트 기반으로 토큰화한다                                                                                                                                                          |
| lowercase     | 텍스트 기반으로 토큰화하여 모든 문자를 소문자로 변경해 토큰화한다.                                                                                                                                                                                                              || 
| ngram         | 원문으로부터 N개의 연속된 글자 단위를 모두 토큰화한다. `엘라스틱서치`라는 원문을 2gram으로 토큰화한다면 [엘라, 라스, 스틱, 틱서, 서치]와 같이 연속된 두 글자를 모두 추출한다. 사실상 원본으로부터 검색할 수 있는 거의 모든 조합을 얻어낼 수 있기때문에 정밀한 부분 검색에 장점이 있지만, 토크나이징을 수행한 N개의 이하의 글자 수로느 검색이 불가능하며 모든 조합을 추출하기 때문에 저장곤강을 많이 차지한다는 단점이 있다. |
| uax_url_email | 스탠다드 분석기와 비슷하지만 URL,이나 이메일을 토큰화 하는데 장점이 있다.                                                                                                                                                                                                        |

```
POST _analyze
{
  "tokenizer": "uax_url_email",
  "text": "elastic@elk-compay.com"
}
```

### 3.8.3 필터

    분석기는 하나의 토크나이저 + 다수의 필터로 조합된다. 없어도 된다.
    엘라스틱에서 제공하는 분석기들은 하나 이상의 필터를 포함하고 있다.
    필터는 단독 사용이 불가능하며 반드시 토크나이저가 있어야한다.

```
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["uppercase"],
  "text": "The 10 most loving dog breeds."
}
```

#### 캐릭터 필터

    캐릭터 필터는 토크나이저 전에 위치하며 문자들을 전처리하는 역할을 한다. 
    HTML 문법을 제거/변경하거나 다른 문자로 대체하는 일들을 한다. &npsp;

#### 토큰 필터

    토큰화되어 있는 문자들에 필터를 적용한다.

[Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-tokenfilters.html)

| 필터        | 설명                                                                                       |
|-----------|------------------------------------------------------------------------------------------|
| lowercase | 모든 문자를 소문자로 변환한다. 반대로 대문자로 변환하는 uppercase 필터가 있다.                                        |
| stemmer   | 영어 문법을 분석하는 필터 이다. 언어마다 고유한 문법이 있어서 필터 하나로 모든 언어에 대응하기는 힘들다. 한글의 경우 아리랑, 노리 같은 오픈소스가 있다. |
| stop      | 기본 필터에서 제거하지 못하는 특정한 단어를 제거할 수 있다.                                                       |

### 커스텀 분석기
    직접 토크나이저, 필터 등을 조합해 사용 할 수 있는 분석기 

#### 커스텀 분석기 설정

```
PUT customer_analyzer
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": ["lions"]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": [],
          "tokenizer": "standard",
          "filter": ["lowercase","my_stopwords"]
        }
      }
    }
  }
}

GET customer_analyzer/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Cats Lions Dogs"
}
```

#### 필터 적용 순서
    커스텀 분석기에 필터를 적용할 때는 필터들의 순서에 유의해야 한다.
    가능하면 모든 문자를 소문자로 변환한 후에 필터를 적용하는 것이 실수를 줄인다.


```
GET customer_analyzer/_analyze
{
  "tokenizer": "standard",
  "filter": [ "lowercase", "my_stopwords" ],
  "text": "Cats Lions Dogs"
}

GET customer_analyzer/_analyze
{
  "tokenizer": "standard",
  "filter": [ "my_stopwords", "lowercase" ],
  "text": "Cats Lions Dogs"
}
```