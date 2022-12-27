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
  "filter": [ "my_stopwords",ㅊ "lowercase" ],
  "text": "Cats Lions Dogs"
}
```

# 4. 엘라스틱서치 검색

## 4.1. 쿼리 컨텍스트와 필터 컨텍스트

* 쿼리 컨텍스트: 연관성을 계산하여 최대한 비슷한 도큐먼트들을 찾아 준다.
* 필터 컨텍스트: 결과가 맞는지 아닌지만 확인 하여 도큐먼트를 찾아 준다.
    * 스코어 계산하지 않기 때문에 전체적인 쿼리 속도를 올릴 수 있다.
    * 결과에 대한 업데이트를 매번 수행할 필요가 없어서 캐시를 이용할 수 있다.
        * 힙 메모리의 10%를 캐시에 이용하고 있으며 캐시를 이용한 빠른 검색을 하려면 필터 컨택스트를 이용해야 한다.

_search는 검색 쿼리를 위해 엘라스틱에서 제공하는 REST API다.
match는 전문 검색을 위한 쿼리로, 역인덱싱된 용어를 검색할 때 사용한다.   
kinans_sample_date_ecommerce 인덱스에 있는 category 필드의 역인덱스 테이블에 'clothing'
도큐먼트를 찾아달라는 요청이다.

### 쿼리 컨텍스트

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "category": "clothing"
    }
  }
}
```

hits.total.value은 인덱스에 3927개의 도큐먼트를 찾았음을 의미한다.
hits.total에는 엘라스틱서치가 찾는 3927개의 도큐먼트가 높은 스코어 순으로 정렬됨, 쿼리 컨텍스트는 유사도 계산 알고리즘 의해 가장 연관성 높은 도큐먼트를 찾는다.
_socre 값은 요청한 검색과 유사도를 나타내는 지표로, 일반적으로 값이 클수록 찾고자 하는 확율이 높다.

### 필터 컨텍스트

엘라스틱 1.x에서는 용어 검색, 용어 필터처럼 쿼리컨텍스트와 필터 컨텍스트가 명확히 구분되어 문법상 쿼리와 필터 컨텍스트를 구분할 수 있었는데
필터 컨텍스트는 모두 논리 쿼리에 포함되었다. 필터 컨텍스트를 단독으로 사용하기보다는 쿼리/필터 컨텍스트를 조합해 사용하는 방향으로 가는 추세라고 생각하면 된다.

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "day_of_week": "Friday"
          }
        }
      ]
    }
  }
}
```

## 4.2. 쿼리 스트링과 쿼리 DSL

엘라스틱서치에서 쿼리를 사용하는 방법은 쿼리 스트리밍, 쿼리 DSL 두 가지가 있다.
쿼리 스트리밍은 한 줄 정도로 간단한 쿼리에 사용하고 쿼리 DSL은 한 줄에 넣기 복잡한 쿼리에 사용한다.   
쿼리 DSL은 엘라스틱서치에서 제공하는 쿼리 전용 언어로, JSON 기반의 직관적인 언어다.
간단한 조건에 대한 검색 => 쿼리스트링
복잡한 논리조건 or 코드 수준에서 쿼리 제어 => 쿼리 DSL

### 4.2.1 쿼리 스트링

쿼리 스트리밍은 REST API의 URI 주소에 쿼리문을 작성하는 방식으로 실행해 볼 수 있어 사용하기 쉽다.   
하지만 복잡한 논리 조건에 해당 하는 경우 괄호를 이용해야 하는데, 이는 조건이 복잡해지면 가독성도 좋지 않고 오류를 범하기 쉽다.

```
GET kibana_sample_data_ecommerce/_search?q=customer_full_name:Mary
```

### 4.2.2 쿼리 DSL

쿼리 DSL은 REST API 요청을 본문 안에 JSON 형태로 쿼리를 작성한다.   
엘라스틱서치의 모든 쿼리 스팩을 지원하기 때문에 매우 강력하며 복잡한 쿼리를 구현할 수 있다.

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "Mary"
    }
  }
}
```

## 4.3 유사도 스코어

쿼리 컨텍스트는 엘라스틱에서 지원하는 다양한 스코어 알고리즘을 사용할 수 있는데 기본적으로 BM25 알고리즘을 이용해 유사도 스코어를 계산한다.
유사도 스코어는 질의문과 도큐먼트의 유사도를 표현하는 값으로, 스코어가 높을수록 찾고자 하는 도큐먼트에 가깝다는 사실을 의미한다.
스코어 계산을 위한 알고리즘의 동작 방식을 이해하고 있다면 더 똑똑한 쿼리 및 인덱스 디자인이 가능하다.

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "products.product_name": "Pants"
    }
  },
  "explain": true
}
```

### 4.3.1 스코어 알고리즘(BM25)

    히트된 도큐먼트는 모두 스코어 값을 갖고 있다.
    스코어는 도큐먼트와 쿼리 간의 연광성 수치로, 값이 클수록 연관성이 높다는 것을 의미한다.
    5.x 이전 버전 TF-IDF (Term Frequency Inverse Document Frequency) / 이후 버전 BM25

#### TF-IDF (Term Frequency Inverse Document Frequency)

    개념에 문서 길이를 고려한 알고리즘 검색어가 문서에서 얼마나 자주 나타나는지, 검색어가 문서 내에서 중요한 용어인지 등을 판단하는 근거를 제공함.

### 4.3.2 IDF 계산

    Document Frequency는 특정 용어가 얼마나 자주 등장했는지를 의미하는 지표 
    관사나 접속부사처럼 자주 등장하는 용어는 실제 큰 의미가 없어서 발생 빈도가 적을수록 가중치를 높게 주는데 
    이를 문서 빈도의 역수 Inverse Document Frequency (IDF)라고 한다. 
    n : 검색한 용어(term)
    3개의 도큐먼트에서 products.product_name 필드에 'Pants' 라는 용어를 포함하고 있고
    N은 인덱스의 전체 도큐먼트 수 (4675)

```
{
  "value" : 7.1974354,
  "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
  "details" : [
    {
      "value" : 3,
      "description" : "n, number of documents containing term",
      "details" : [ ]
    },
    {
      "value" : 4675,
      "description" : "N, total number of documents with field",
      "details" : [ ]
    }
  ]
},
```

### TF 계산

    용어 빈도(Term Frequency)는 특정 용어가 하나의 토큐먼트에 얼마나 많이 등장했는지를 의미하는 지표
    특정 용어가 도큐먼트에서 많이 반복되었다면 주제와 연관되어 있을 확률이 높다.
    freq : 도큐먼트 내에서 용어가 나온 횟수

```
{
  "value" : 0.52217203,
  "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
  "details" : [
    {
      "value" : 1.0,
      "description" : "freq, occurrences of term within document",
      "details" : [ ]
    },
    {
      "value" : 1.2,
      "description" : "k1, term saturation parameter",
      "details" : [ ]
    },
    {
      "value" : 0.75,
      "description" : "b, length normalization parameter",
      "details" : [ ]
    },
    {
      "value" : 5.0,
      "description" : "dl, length of field",
      "details" : [ ]
    },
    {
      "value" : 7.3161497,
      "description" : "avgdl, average length of field",
      "details" : [ ]
    }
  ]
}
```

## 4.4 쿼리

엘라스틱서치는 검색을 위해 쿼리를 지원하는데, 크게 리프 쿼리와 복합 쿼리로 나눌 수 있다.  
리프 쿼리는 특정 필드에서 용어를 찾는 쿼리로, 매치, 용어, 범위 쿼리 등이 있다.
반면 복합 쿼리는 쿼리를 조합해서 사용되는 쿼리로, 대표적으로 논리 쿼리 등이 있다.

### 4.4.1 전문 쿼리와 용어 수준 쿼리

전문 쿼리는 전문 검색을 하기 위해 사용되며, 전문 검색을 할 필드는 인덱스 매핑 시 텍스트 타입으로 매핑해야 한다.
반면 용어 수준 쿼리는 정확히 일치하는 용어를 찾기 위해 사용되며, 인덱스 매핑 시 필드를 키워드 타입으로 매핑 해야 한다.
강제는 아니지만 정확한 결과를 얻기 위한 권장사항이다.

### 4.4.2 매치 쿼리

    매치 쿼리는 대표적인 전문 쿼리다. 가장 기본이 되는 쿼리로, 전체 텍스트 중에서 특정 용어나 용어(단어)들을 검색할 떄 사용한다
    GET kibana_sample_data_ecommerce/_mapping
    전문 검색의 경우 검색어도 토큰화되기 때문에 Mary => mary로 토근화된다.

#### 하나의 용어를 검색하는 매치 쿼리

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match": {
      "customer_full_name": "Mary"
    }
  }
}
```

#### 복수 개의 용어를 검색하는 매치 쿼리

    공백은 OR로 인식한다. mary, bailey로 토큰화되고 mary, 
    bailey가 하나라도 포함된 도큐먼트가 있다면 매칭되었다고 판단한다.

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match": {
      "customer_full_name": "mary bailey"
    }
  }
}
```

#### 둘다 포함된 도큐먼트를 찾는방법

    operator를 and로 설정한 매치 검색

```
# operator를 and로 설정한 매치 검색
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match": {
      "customer_full_name": {
        "query": "mary bailey",
        "operator": "and"
      }
    }
  }
}
```

### 4.4.3 매치 프레이즈 쿼리

    전문 쿼리의 한 종류로 매치 프레이즈 쿼리는 구를 검색할 때 사용 한다. 구는 동사가 아닌 2개 이상의 단어가 연결되어 만들어진 단어이다. 
    '빨간 바지', '65 인치 텔레비전' 같이 여러 단어가 모여서 뜻을 이루는 단어다. 단어의 순서도 중요하다.
    'mary bailey'가 [mary, bailey]로 토큰화되는 것까지는 매치 쿼리와 같지만, 매치 프레이즈 쿼리는 검색어에 사용된 용어들 모두 포함되어 용어의 순서까지 맞아야 한다. 
    용어 순서가 바뀐 'bailey mary'나 중간에 다른 단어가 포함된 경우에는 매칭되지 않는다

#### 매치 프레이즈 쿼리

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match_phrase": {
      "customer_full_name": "mary bailey"
    }
  }
}
```

#### 용어 쿼리

    용어 수준의 대표적인 쿼리다. 용어 검색은 용어 수준 쿼리에 속하기 때문에 검색어인 'mary bailey'가 분석기에 의해 토큰화되지 않는다. 
    즉 'mary bailey'와 정확하게 일치하는 경우에만 매핑된다 
    customer_full_name은 분석기에 의해 대문자가 소문자로 변경되어 [mary, bailey]로 매핑되어 있지만 용어 쿼리는 [Mary]를 찾기 때문에 
    매칭이 되지 않는다.

```
# 텍스트 타입 필드에 대한 용어 쿼리
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "term": {
      "customer_full_name": "Mary Bailey"
    }
  }
}

# 키워드 타입 필드에 대한 용어 쿼리
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "term": {
      "customer_full_name.keyword": "Mary Bailey"
    }
  }
}
```

### 4.4.5 용어들 쿼리

    용어들 쿼리는 용어 수준 쿼리의 일종이며 여러 용어들을 검색 해준다. 
    키워드 타입으로 매핑된 필드에서 사용해야 하며 분석기를 거치지 않았기 때문에 대소문자도 신경써야한다.

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week"],
  "query": {
    "terms": {
      "day_of_week": ["Monday", "Sunday"]
    }
  }
}
```

### 4.4.6 멀티 매치 쿼리

    검색하고자 하는 용어나 구절이 정확히 어떤 필드에 있는지 모르는 경우가 있다. 예를 들어 트럼프를 검색할 때 트럼프가 어떤 필드에 저장되어 있는지 정확히 알 수 있을까?   
    여러 개의 필드에서 검색하기 위한 멀티 매치 쿼리는 전문 검색 쿼리의 일종으로, 텍스트 타입으로 매핑된 필드에서 사용 하는 것이 좋다.  

#### 여러 필드에 쿼리 요청하기

```
GET kibana_sample_data_ecommerce/_search?explain=true
{
  "_source": ["customer_first_name", "customer_last_name", "customer_full_name"],
  "query": {
    "multi_match": {
      "query": "mary",
      "fields": [
        "customer_first_name",
        "customer_last_name",
        "customer_full_name"       
        ]
    }
  }
}
```

멀티 매치 쿼리는 1개 이상의 필드에 쿼리를 요청할 수 있다.   
매치 쿼리를 하고 3개의 필드에서 개별 스코어를 구한 다음에 그중 가장 큰 값을 대표 스코어로 구분한다.

#### 와일드카드를 이용한 멀티 필드에 쿼리 요청하기

```
GET kibana_sample_data_ecommerce/_search?explain=true
{
  "_source": ["customer_first_name", "customer_last_name", "customer_full_name"],
  "query": {
    "multi_match": {
      "query": "mary",
      "fields": 
        "customer_*_name"
      }
  }
}
```

#### 필드에 가중치 두기 (가중치를 이용한 검색)

```
GET kibana_sample_data_ecommerce/_search?explain=true
{
  "_source": ["customer_first_name", "customer_last_name", "customer_full_name"],
  "query": {
    "multi_match": {
      "query": "mary",
      "fields": [
        "customer_first_name^2",
        "customer_last_name",
        "customer_full_name"
        
        ]
    }
  }
}
```

### 4.4.7 범위 쿼리

    특정 날짜나 숫자의 범위를 지정해 범위 안에 포함된 데이터들을 검색할 때 사용된다.
    날짜 / 숫자 / IP타입의 데이터는 범위 쿼리 O 
    문자형, 키워드 타입의 데이터에는 범위 쿼리를 사용할 수 없다.

#### 날짜/시간 범위 쿼리

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "range": {
      "tiemstamp": {
        "gte": "2020-12-15",
        "lt": "2020-12-16"
      }
    }
  }
}
```

tiemstamp라는 필드에서 2020-12-15 00:00:00 ~ 2020-12-16 :23:59:59인 데이터를 찾는다.

| 파라미터 | 설명                                                          |
|------|-------------------------------------------------------------|
| gte  | * gte: 10: 10과 같거나 보다 큰 값 <br> * gte: 2021-01-21 같거나 그이후 날짜 |
| gt   | * gte: 10: 10 보다 큰 값 <br> * gt: 2021-01-21 그 이후 날짜          |
| lte  | * lte: 20: 20과 같거나 보다                                       | 큰값 <br> * lte: 2021-01-21 같거나 그 이전의 날짜|
| lt   | * lt: 20: 20 보다 큰 값 <br> * lt: 2021-01-21 이전의 날짜            |

#### 날짜/시간 데이터 타입

    날짜/시간 검색은 현재 시간을 기준으로 하는 경우가 많다. 

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "range": {
      "tiemstamp": {
        "gte": "now-1M"
      }
    }
  }
}
```

tiemstamp 필드에서 현재 시각 기준으로 한 달 전까지의 모든 데이터를 가져오는 데, 현재 시각을 기준으로 날짜/시간 범위를 직관적으로 이해할 수 있다.

| now            | 현지 시각                |
|----------------|----------------------|
| now            | 현재 시각 +1일            |
| now+1h+30m+10s | 현재 시각 + 1시, 30분, 10초 |
| 2021-01-21     | +M                   | 2021-01-21 + 1달|

#### 날짜/시간 단위 표기법

| 시간 단위        | 의미  |
|--------------|-----|
| y(year)      | 연   |
| M(month)     | 월   |
| w(weeks)     | 주   |
| d(days)      | 일   |
| H, h (hours) | 시   |
| m(minutes)   | 분   |
| s(seconds)   | 초   |

### 범위 데이터 타입

    integer_range, float_range, long_range, double_range, date_range, ip_ragne 타입을 지원한다.

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "range": {
      "tiemstamp": {
        "gte": "now-1M"
      }
    }
  }
}
```

### 4.4.8 논리 쿼리

    논리 쿼리는 복합 쿼리로 앞에서 배웠던 쿼리를 조합할 수 있따.
    논리 쿼리는 쿼리를 조합할 수 있도록 4개의 타입을 지원한다.

| 타입       | 설명                                                                                        |
|----------|-------------------------------------------------------------------------------------------|
| must     | 쿼리를 실행하여 참인 도큐먼트를 찾는다 / 복수의 쿼리를 실행하면 AND 연산을 한다.                                          |
| must_not | 쿼리를 실행하여 거짓인 도큐먼트를 찾는다 / 다른 타입과 같이 사용할 경우 도큐먼트에서 제외한다.                                    |
| should   | 단독으로 사용 시 쿼리를 실행하여 참인 도큐먼트를 찾는다. / 복수의 쿼리를 실행하면 OR 연산을 한다. / 다른 타입과 같이 사용할 경우 스코어에만 활용된다. |
| filter   | 쿼리를 실행하여 예/아니오 형식의 필터 컨텍스트를 수행한다.                                                         |

`

#### 복수 개의 쿼리를 사용하는 Must 타입

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": 
        { "match": { "customer_full_name": "mary" }}
      
    }
  }
}

GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week", "customer_full_name"], 
  "query": {
    "bool": {
      "must": [
        { "term": { "day_of_week": "Sunday" }},
        { "match": { "customer_full_name": "mary" }}
      ]
    }
  }
}
```

#### must_not 타입

    도큐먼트에서 제외할 쿼리를 실행한다.

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must_not": 
        { "match": { "customer_full_name": "mary" }}
      
    }
  }
}
```

#### 다른 타입 + must_not 타입을 함께 사용하는 경우

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "bool": {
      "must":  {
        "match" :{"customer_first_name": "mary"}
      },
      "must_not": {
        "term": {"customer_last_name": "bailey"}
      }
    }
  }
}
```

#### 4.4.8.3 should 타입

    must 타입에 하나의 쿼리를 사용하는 경우와 같다.
    복수 개의 쿼리를 사용하면 OR 효과를 얻을 수 있다.
    should 를 사용해 도큐먼트의 검색 순위를 최적화 할 수 있다.

##### 하나의 쿼리를 사용하는 should 타입

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "should": [
          { "match": { "customer_first_name": "mary"}}
      ]
    }
  }
}
```

##### 복수 개의 쿼리를 사용하는 should type

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week", "customer_full_name"], 
  "query": {
    "bool": {
      "should": [
          { "match": { "customer_first_name": "mary"}},
          { "term": { "day_of_week": "Sunday" }}
      ]
    }
  }
}
```

##### must only

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week", "customer_full_name"], 
  "query": {
    "bool": {
      "must": [
          { "match": { "customer_full_name": "mary"}}
      ]
    }
  }
}
```

##### must + should

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week", "customer_full_name"], 
  "query": {
    "bool": {
      "must": [
          { "match": { "customer_full_name": "mary"}}
      ],
      "should": [
        { "term": { "day_of_week": "Sunday" }}
      ]
    }
  }
}
```

#### 4.4.8.4 filter 타입

    filter는 must와 같은 동작을 하지만 필터 컨텍스트로 동작하기 때문에 유사도 스코어에 영향을 미치지 않는다.

##### 하나의 쿼리를 사용하는 filter 타입

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["products.base_price"], 
  "query": {
    "bool": {
      "filter": {
        "range": {
          "products.base_price": {
            "gte": 30,
            "lte": 60
          }
        }
      }
    }
  }
}
```

##### filter + must

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week", "customer_full_name"],
  "query": {
    "bool": {
      "filter": [
        {"term": { "day_of_week": "Sunday" }}
      ],
      "must": [
        {"match": { "customer_full_name": "mary" }}
      ]
    }
  }
}
```

### 4.4.9 패턴 검색

    검색어를 정확하게 알지 못하는 경우에 사용한다.
    와일드카드 쿼리와 정규식 쿼리 두 가지 방법이 존재한다. 
    용어 수준 쿼리에 해당하므로 분석기에 의해 분리된 용어를 찾기 위한 쿼리라는 점을 주의하자.
    SQL Like 검색처럼 원문에서 특정 문자열을 검색하는 용도로는 적합하지 않다.
    많은 리소스를 사용하기 때문에 권장하지는 않지만 필요할때 효율적으로 사용하면 된다.

#### 4.4.9.1 와일드카드 쿼리

    * : 공백까지 포함하여 글자 수에 상관없이 모든 문자를 매칭
    ? : 오직 한 문자만 매칭
    ⭐️ 용어의 맨 앞에 사용하면 속도가 매우 느려지기 때문에 주의해야 한다.

##### 와일드카드 패턴 검색

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "wildcard": {
      "customer_full_name.keyword": {
        "value": "M?r*"
      }
    }
  }
}
```

#### 4.4.9.2 정규식 쿼리

    정규식은 특정한 패턴을 가진 문자열을 표현하기 위한 형식 언어

##### 정규식을 이용한 패턴 검색

```
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_first_name"],
  "query": {
    "regexp": {
      "customer_first_name.keyword": {
        "value": "Mar."
      }
    }
  }
}
```

# 5. 엘라스틱 서치: 집계

    데이터를 그룹핑하고 통곗값을 얻는 기능으로 SQLdml Group by와 통계 함수를 포함하는 개념 
    집계 기능은 강력한 검색 성능과 맞물려 엘라스틱서치를 고성능 집계 엔진으로 활용할 수 있게 해준다. 
    대표적으로 키바나를 들 수 있다. 키바나의 주 기능인 데이터 시각화와 대시보드는 대부분 집계 기능을 기반으로 동작한다.

## 5.1 집계의 요청 - 응답 형태

    집계 search API의 요청 본문에 aggs 파라미터를 이용하면 쿼리 결과에 대한 집계를 생성 가능하다.
    엘라스틱 서치에는 크게 메트릭 집계 (Matric aggregations)와 버킷 집계 (bucket_aggregations)라는 두 가지 타입의 집계가 있다.
    메트릭 집계 => 통계나 계산
    버킷 집계 => 도큐먼트를 그룹핑 
    my_aggs : 집계 이름으로 사용자가 지정한 이름
    value : 실제 집계 결과 

## 5.2 메트릭 집계 (metric_ aggregations)

    통계나 계산에 활용한다. 최소/최대/합계/평균/중간값 같은 통계 결과를 보여준다. 텍스트 타입 필드는 수치 연산이 불가능하다.

| 매트릭 집계       | 설명                                          |
|--------------|---------------------------------------------|
| avg          | 필드의 평균값을 계산한다.                              |
| min          | 필드의 최솟값을 계산한다                               |
| max          | 필드의 최댓값을 계산한다                               |
| sum          | 필드의 총합을 계싼한다.                               |
| percentiles  | 필드의 백분위 값을 계산한다.                            |
| stats        | 필드의 min, max, sum, avg, count를 한 번에 볼 수 있다. |
| cardinality  | 필드의 유니크한 값 개수를 보여준다.                        |
| geo-centrold | 필드 내부의 위치 정보의 중심점을 계산한다.                    |

### 5.2.1 평균값/중간값 구하기.

    평균 집계 avg를 이용해 products.base_price 필드의 평균 값을 구하는 요청이다. 집계 이름은 각자 원하는
    형태로 정의할 수 있는데 여기서는 stats_aggs라는 이름을 사용 했다. 
    size=0 설정을 하면 집계에 사용한 도큐먼트를 결과에 포함하지 않음으로써 비용을 절약할 수 있다.

#### 평균값을 구하는 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "avg": {
        "field": "products.base_price"
      }
    }
  }
}
```

#### 백분위를 구하는 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "percentiles": {
        "field": "products.base_price",
        "percents": [25, 50]
      }
    }
  }
}
```

### 5.2.2. 카디널리티 집계 (cardinality_aggregation)

    필드의 유니크한 값들의 개수를 확인하는 집계 -

#### 카디널리티 집계 요청

    precision_threshold : 정확도 수치 값이 크면 정확도가 올라가는 대신 시스템 리소스를 많이 소모한다.
    정확도를 너무 낮게하면 다 나온다. 

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "cardi_aggs":{
      "cardinality": {
        "field": "day_of_week",
        "precision_threshold": 100
      }
    }
  }
}
```

#### 용어 집계 요청

    용어 집계를 사용하면 유니크한 필드 개수와 함께 필드값들을 확인할 수 있다.

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "cardi_aggs":{
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```

### 5.2.3 검색 결과 내에서의 집계

#### 쿼리를 이용해 집계 범위 지정

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "term": {
      "day_of_week": "Monday"
    }
  },
  "aggs": {
    "quey_aggs": {
      "sum": {
        "field": "product.base_price"
      }
    }
  }
}
```

## 5.3 버킷 집계

    메트릭 집계 - 특정 필드를 기준으로 통곗값을 계산하려는 목적
    버킷 집계 - 특정 기준에 맞춰서 도큐먼트를 그룹핑 하는 역할
    버킷 - 도큐먼트가 분할되는 단위로 나뉜 각 그룹을 의미함
    모든데이터 -> 월/화/수/목/금/토/일 
    버킷으로 도큐먼트를 구분한 후에 메트릭 집계와 연계해 분석을 할 수 있다.

[버킷 집계 Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/search-aggregations-bucket.html)

| 버킷 집계            | 설명                                                                                  |
|------------------|-------------------------------------------------------------------------------------|
| histogram        | 숫자 타입 필드를 일정 간격으로 분류한다.                                                             |
| data_histogram   | 날짜/시간 타입 필드를 일정 날짜, 시간 간격으로 분류한다.                                                   |
| rage             | 숫자 카입 필드를 사용하기 지정하는 범위 간격으로 분류된다.                                                   |
| terms            | 필드에서 많이 나타나는 용어들을 기준으로 분류한다.                                                        |
| sighftcaht_terms | terms 버킷 유사하거나 모든 값을 대상으로 하지 않고 인덱스 내 전체 문서 대비 현재 조건조건에서 통걔적으로 유의미한 값을 대상자로 하지 않는다. |
| fillters         | 각 그룹애 포함시킬 문서위 조건을 직접 지정한다. 이때 조선은 일반적으로 검색에 사용되는 쿼리와 같다.                           |

### 히스토그램 집계

    숫자 타입 필드를 일정 간격으로 분류한다.   

#### 히스토그램 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_aggs": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      }
    }
  }
}
```

### 범위 집계

    히스토그램 집계는 설정이 간단하지만 각 버킷의 범위를 동일하게 지정할 수밖에 없다는 단점이 있기에 
    범위를 직접 설정할 수 있는 집계가 범위 집계이다.

#### 범위 집계 요청

    products.base_price 는 배열형태로 되어있어서 위의 값과는 다를 수 있다.

```GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "rage_aggs": {
      "range": {
        "field": "products.base_price",
        "ranges": [
          {  "from": 0,"to": 50  },
          {  "from": 50,"to": 100  },
          {  "from": 100,"to": 200  },
          {  "from": 200,"to": 1000  }
        ]
      }
    }
  }
}
```

### 용어 집계 (terms aggregation)

    유니크한 기준으로 값을 나눌 떄 사용된다.

##### 용어 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 6
      }
    }
  }
}
```

#### 용어 집계가 정확하지 않은 이유

    분산 시스템의 집계 과정에서 발생하는 잠재적인 오류 가능성이 있다.
    분산 시스템에서는 데이터를 여러 노드에서 분산하고 취합하는 과정에서 오류가 발생할 수 있다. 
    엘라스틱서치는 샤드에 도큐먼트를 저장하고 이를 분산하는데, size 설정값과 샤드 개수 등에 의해 집계에 오류가 발생할 수 있다.

#### 용어 집계 정확성 높이기

    리소스 소비량을 늘리면 정확도를 높일 수 있다

##### 용어 집계 오류 확인 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 6,
        "show_term_doc_count_error": true
      }
    }
  }
}
```

##### 용어 집계 시 샤드 크기를 늘린 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 6,
        "shard_size": 100
      }
    }
  }
}
```

## 집계의 조합

    버킷 집계 + 메르릭 집계 동시에 요청하기

### 버킷 집계 후 메트릭 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 5
      },
      "aggs": {
        "avg_aggs":{
          "avg": {
            "field": "products.base_price"
          }
        }
      }
    }
  }
}
```

### 버킷 집계 후 다수의 메트릭 집계 요청

    버킷 집계 내부에서 2개의 메트릭 집계가 동작한다. 먼저 용어 집계를 요일별로 상위 5개 버킷을 만든다. 
    메트릭스 집계 평균과 총합 집계는 용어 집계 내부에서 사용되고 내부에서는 products.base_price 필드의 평균 값과 총합을 구하는 두 번의 메트릭 집계를 각각 수행한다.

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 5
      },
      "aggs": {
        "avg_aggs":{
          "avg": {
            "field": "products.base_price"
          }
        },
        "sum_aggs":{
          "sum": {
            "field": "products.base_price"
          }
        }
      }
    }
  }
}
```

![image](https://user-images.githubusercontent.com/22822369/209643659-f15d3104-2595-4e32-b706-ec98b22ce337.png)

### 서브 버킷 집계

    버킷 안에서 다시 버킷 집계를 요청하는 집계 
    트리구조를 떠올리면 된다.

#### 서브 버킷 생성 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_aggs": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      },
      "aggs": {
        "term_aggs":{
          "terms": {
            "field": "day_of_week",
            "size": 2
          }
        }
      }
    }
  }
}
```

![image](https://user-images.githubusercontent.com/22822369/209644079-8b91a3f9-9d86-4234-8c8b-d1292c988b34.png)

## 파이프라인 집계 (pipeline_aggregation)

    이전 결과를 다음 단계에서 이용하는 개념을 차용한다.
    이전 집계로 만들어진 결과를 입력으로 삼아 다시 집계하는 방식.
    parent aggregation / sibling aggregation 두가지 유형이 있따.
    parent aggregation은 기존 집계 내부에서
    sibling aggregation은 기존 집계 외부에서 새로 작성한다.

[파이프라인 reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/search-aggregations-pipeline.html)

| 집계종류  | json                                               | 설명                                              |
|-------|----------------------------------------------------|-------------------------------------------------|
| 부모 집계 | ```{ "aggs": { ... "aggs": { "부모 집계": ... } } }``` | 기존 집게 경과를 이용해 새로운 집계를 생성한다. 결과는 기존 집계 내부에서 나온다. |
| 형제 집계 | ```{ "aggs": { ... "aggs": { "형제 집계": ... } } }``` | 기존 집계를 참고해 집계를 수행한다. 결과는 기존 집계와 동일 선상에서 나온다.    |

| 집계    | 집계 종류             | 설명                                    |
|-------|-------------------|---------------------------------------|
| 형제 집계 | min_bucket        | 기존 집계중 최솟값을 구한다.                      |
| 형제 집계 | max_bucket        | 기존 집계중 최댓값을 구한다.                      |
| 형제 집계 | avg_bucket        | 기존 집계의 평균값을 구한다.                      |
| 형제 집계 | sum_bucket        | 기존 집계의 총합을 구한다.                       |
| 형제 집계 | stat_bucket       | 기존집계의 min, max, sum, count, avg를 구한다. |
| 형제 집계 | percentile_bucket | 기존 집계의 백분윗값을 구한다.                     |
| 형제 집계 | moving_avg        | 기존 집계의 이동 평균을 구한다                     |
| 부모 집계 | derivative        | 기존 집계의 미분을 구한다.                       |
| 부모 집계 | cumulative_sum    | 기존 집계의 누적합을 구한다.                      |

### parent aggregation

    단독으로 사용이 불가능하며 이전 집계 내부에서 실행한다. 결괏값도 기존 집계 내부에서 나타난다.

#### 누적합을 구하는 부모 집계 요청

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_aggs": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      },
      "aggs": {
        "sum_aggs":{
          "sum": {
            "field": "taxful_total_price"
          }
        },
        "cum_sum": {
          "cumulative_sum": {
            "buckets_path": "sum_aggs"
          }
        }
      }
    }
  }
}
```

### sibling_aggregation

    형제 집계는 기존 집계 내부가 아닌 외부에서 기존 집계를 이용해 집계 작업을 한다.

    term_aggs는 용어 집계로 day_of_week 필드를 기준으로 요일별 버킷을 나눈다. 
    size 2로 상위 2개의 버킷을 생성하고 sum_aggs에서 products.base_price 필드의 총합을 구한다. 
    다음으로 sum_bucket 형제 집계를 이용해 기존 버킷별 합을 구한 집계를 다시 합친다.
    버킷 경로에서 >는 하위 집계 경로를 나타낼 때 사용된다.

```
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 2
      },
      "aggs": {
        "sum_aggs": {
          "sum": {
            "field": "products.base_price"
          }
        }
      }
    },
    "sum_total_price": {
      "sum_bucket": {
        "buckets_path": "term_aggs>sum_aggs"
      }
    }
  }
}
```

# 로그스태시 (Logstash)

    로그를 저장한다. 
    로그 형태를 분석하고 시스템에서 인식할 수 있도록 로그를 정제하는 과정을 쉽고 편하게 할 수 있도록 지원한다.

## Logstash 소개

* 플러그인 기반의 오픈소스 데이터 처리 파이프라인 도구
* 데이터 전처리 과정을 별도의 애플리케이션 작성 없이 비교적 간단한 설정만으로 수행할 수 있다.
* 데이터를 저장하기 전에 사용자가 원하는 형태로 변경할 수 있는 강력한 기능을 제공한다.
* 장애 대응 로직이나 성능 저하 요인을 쉽게 파악할 수 있는 모니터링 API, 간단한 조정으로 성능을 튜닝할 수 있는 파라미터들도 제공한다.

비츠, 로그스태시, 엘라스틱서치, 키바나를 이용하여 데이터 수집 - 변환 - 저장 - 시각화 하는 서비스를 구성할 때  
로그스태시는 데이터를 저장하기 전에 원하는 형태로 가공하는 역할을 한다.  
로그스태시는 비츠를 포함한 여러 소스 파일을 입력으로 받을 수 있고 데이터를 수정/삭제/추가해 엘라스틱 서치나 다른 시스템으로 전송할 수 있다.

### 로그스태시 특징

* 플러그인 기반
    * 로그스태시는 파이프라인을 구성하는 각 요소들은 전부 플러그인 형태로 만들어져 있다.
    * 확장성이 뛰어나다.
* 모든 형태의 데이처 처리
    * JSON, XML 등의 구조화된 텍스트뿐만 아니라 다양한 형태의 데이터를 입력받아 가공한 다음에 저장할 수 있다.
    * 이벤트 데이터 (시간에 따라 발생하는 데이터)를 처리하는데 최적화되어 있다.
    * 관계형 데이터베이스에서 마이그레이션, 엘라스틱서치 도큐먼트 리인덱싱등 데이터를 처리르하는 모든 작업에서 유연하게 사용이 가능하다.
* 성능
    * 자체적으로 내장되어 있는 메모리와 파일 기반의 큐를 사용하므로 처리 속도와 안전성이 높다.
    * 인덱싱할 도큐먼트의 수와 용량을 종합적으로 고려해 벌크 인덱싱을 수행할 뿐만 아니라 파이프라인 배치 크기 조정을 통해 병목현상을 방지하고 성능을 최적화할 수 있다.
* 안전성
    * 장애에 대응하기 위한 재시도 로직이나 오류가 발생한 도큐먼트를 따로 보관하는 데드 레터 큐를 내장하고 있다.
    * 파일 기반의 큐를 사용할 경우 뜻하지 않은 로그스태시의 장애 상황에서도 도큐먼트 유실을 최소화 할 수 있다.


