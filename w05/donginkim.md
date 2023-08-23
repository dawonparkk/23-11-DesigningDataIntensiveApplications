# 트랜잭션 처리나 분석?

- 초창기 비즈니스 데이터 처리는 커머셜 트랜잭션(commercial transaction)에 해당했다.
- 데이터베이스가 확장됐어도 트랜잭션이란 용어는 변하지 않고 논리 단위 형태로서 읽기와 쓰기 그룹을 나타내고 있다.

> 트랜잭션이 반드시 ACID 속성을 가질 필요는 없다. 트랜잭션은 주기적으로 수행되는 일괄 처리 작업과 달리 클라이언트가 지연 시간(low-latency)이 낮은 읽기와 쓰기를 가능하게 한다는 의미다.

- 보통 애플리케이션은 대화식이기 때문에 이 접근 패턴을 온라인 트랜잭션 처리(online transaction processing, OLTP)
- 데이터베이스를 데이터 분석(data anaytic) 사용이 증가했다.
- 분석 질의는 비즈니스에 대한 의사결정을 돕는 보고서를 제공한다.(비즈니스 인텔리전스, business intelligence)
- 이런 데이터베이스 사용 패턴을 트랜잭션 처리와 구별하기 위해 온라인 분석 처리(online analytic processing, OLAP)라고 부른다.

| 특성 | 트랜잭션 처리 시스템(OLTP) | 분석 시스템(OLAP) |
| --- | --- | --- |
| 주요 읽기 패턴 | 질의당 적은 수의 레코드, 키 기준으로 가져옴 | 많은 레코드에 대한 집계 |
| 주요 쓰기 패턴 | 임의 접근, 사용자 입력을 낮은 지연 시간으로 기록 | 대규모 불러오기(bulk import, ETL) 또는 이벤트 스트림 |
| 주요 사용처 | 웹 애플리케이션을 통한 최종 사용자/소비자 | 의사결정 지원을 위한 내부 분석가 |
| 데이터 표현 | 데이터의 최신 상태(현재 시점) | 시간이 지나며 일어난 이벤트 이력 |
| 데이터셋 크기 | 기가바이트에서 테라바이트 | 테라바이트에서 페타바이트 |
- OLAP를 개별 데이터베스에서 분석을 수행하도록 한 데이터베이스를 데이터 웨어하우스(data warehouse)라고 부른다.

## 데이터 웨어하우징

- OLTP 시스템은 사업 운영에 중요하기 때문에 일반적으로 높은 가용성과 낮은 지연 시간의 트랜잭션 처리를 기대
- 데이터베이스 관리자는 OLTP 데이터베이스를 철저하게 보려한다. 데이터베이스 관리자는 비즈니스 분석가가 OLTP 데이터베이스에 즉석 분석 질의(ad hoc analytic query)를 실행하는 것을 꺼려한다.
    - 즉석 분석 질의는 대게 비용이 비싸다.
    - 분석 질의가 트랜잭션의 성능을 저하할 가능성이 있다.
- 데이터 웨어하우스는 분석가들이 OLTP 작업에 영향을 주지 않고 마음껏 질의할 수 있는 개별 데이터베이스이다.
    - OLTP 시스템에 있는 데이터의 읽기 전용 복사본이다.
    - OLTP 데이터베이스에서 주기적으로 데이터를 추출(extract)하고 분석 친화적인 스키마로 변환(transform)하고 깨끗하게 정리한다음 데이터 웨어하우스에 적재(load)한다. 이 과정을 ETL(Extract-Transform-Load)이라 한다.

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/0d8ac831-7845-4fed-96ce-e51977477654)

- 대기업에는 많이 있지만 소규모 기업에는 많이 없다.
- 데이터 웨어하우스를 사용하면 분석 접근 패턴에 맞게 최적화할 수 있다.
- 색인 알고리즘은 OLTP에서 잘 동작하지만 분석 질의의 응답에서는 별로 좋지 않은 편이다.

## OLTP 데이터베이스와 데이터 웨어하우스의 차이점

- SQL 질의를 생성하고 결과를 시각화하고 분석가가 (드릴 다운(drill-down), 슬라이싱(slicing), 다이싱(dicing) 같은 작업을 통해) 데이터 탐색할 수 있게 해주는 여러 그래픽 데이터 분석 도구가 있다.
- 다수의 데이터베이스 벤더는 트랜잭션 처리와 분석 작업부하 양쪽 모두 지원하기보다는 둘 중 하나를 지원하는 데 중점을 둔다.
- 아마존 레드시프트(Amazon RedShift), 다수의 오픈소스 SQL 온 하둡(SQL-on-Hadoop) 프로젝트가 생겨났다.
- 아파치 하이브(Apache Hive), 스파크 SQL(Spark SQL) 등등도 데이터 웨어하우스 시스템과의 경쟁을 목표로 한다.

## 분석용 스키마: 별 모양 스키마와 눈꽃송이 모양 스키마

- 분석에서는 데이터모델의 다양성이 적다.
- 데이터 웨어하우스는 별 모양 스키마(star schema)(차원 모델링(dimensional modeling)이라고도 함)로 알려진 상당히 정형화된 방식을 사용한다.
- 스키마 중심에 소위 사실 테이블(fact table)이 있다. 사실 테이블의 각 로우는 특정 시간에 발생한 이벤트에 해당한다.

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/a759872f-57f8-4c84-b073-10a9c0d87434)

- 보통 사실은 개별 이벤트를 담는다.
- 애플(Apple),  월마트(Walmart), 이베이(eBay) 같은 대기업은 데이터 웨어하우스에 수십 페타바이트의 트랜잭션이 있고 이 중 대부분이 사실 테이블이다.
- 사실 테이블의 일부 컬럼은 비용 같은 속성이고 다른 컬럼은 차원 테이블(dimension table)이라 부르는 다른 테이블을 가리키는 외래키 참조다.
- 사실 테이블은 각 로우는 이벤트를 나타내고 차원은 이벤트의 속성인 누가(who), 언제(when), 어디서(where), 무엇을(what), 어떻게(how), 왜(why)를 나타낸다.
- 사실 테이블의 각 로우는 특정 트랜잭션에서 제품이 판매됐는지를 나타내기 위해 외래 키를 사용한다. 날짜와 시간도 보통 차원 테이블을 사용해 표현한다. 차원 테이블을 사용하면 날짜에 대해 추가적인 정보를 부호화할 수 있고 휴일과 평일의 판매 차이를 질의할 수 있다.

별 모양 스키마란 테이블 관계가 시각화될 때 테이블이 가운데에 있고 차원 테이블로 둘러싸고 있다는 사실에서 비롯됐다.

이 템플릿의 변형은 눈꽃송이 모양 스키마(snowflake schema)이다.

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/94dad9b1-ba75-450f-87b7-898c022c8062)

- 차원이 하위 차원으로 더 세분화 된다.
- 스타 스키마만큼 스노우플레이크 스키마가 많이 사용되지 않는다.

# 컬럼 지향 저장소

- 테이블에 엄청난 개수의 로우와 페타바이트 데이터가 있다면 효율적으로 저장하고 질의하기는 어려운 문제가 된다.
- 일반적으로 데이터 웨어하우스 질의는 한 번에 4개 또는 5개 컬럼만 접근한다.
- 대부분의 OLTP 데이터베이스에서 저장소는 로우 지향 방식으로 데이터를 배치한다.
    - 테이블에서 한 로우의 모든 값은 서로 인접하게 저장된다.
    - 모든 로우를 메모리로 적재한 다음 쿼리를 해석해 필요한 조건을 충족하지 않은 로우를 필터링한다.
- **컬럼 지향 저장소의 기본 개념**
    - 모든 값을 하나의 로우에 함께 저장하지 않는 대신 각 컬럼별로 모든 값을 함께 저장한다.
    - 각 컬럼을 개별 파일에 저장하면 질의에 사용되는 컬럼만 읽고 구분 분석하면 된다.
    - 이 방식을 사용하면 작업량이 많이 줄어든다.

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/46e6b08a-4e9d-43b7-b460-585cb1247354)

- 컬럼 지향 저장소 배치는 각 컬럼 파일에 포함된 로우가 모두 같은 순서인 점에 의존한다.

## 컬럼 압축

- 질의에 필요한 컬럼을 디스크에서 읽어 적재하는 작업 외에도 데이터를 압축하면 디스크 처리량을 더 줄일 수 있다.
- 다행히 컬럼 지향 저장소는 대개 압축에 적합하다.
- 데이터 웨어하우스에서 특히 효과적인 **비트맵 부호화(bitmap encoding)**로서 확인할 수 있다.

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/9b69c1ec-44ed-447c-ade8-a94cf5df3539)

- 보통 컬럼에서 고유 값의 수는 로우 수에 비해 적다. n개의 고유 값을 가진 컬럼을 가져와 n개의 개별 비트맵으로 변환할 수 있다. 고유 값 하나가 하나의 비트맵이고 각 로우는 한 비트를 가진다.
- n이 매우 작으면 이 비트맵은 로우당 하나의 비트로 저장할 수 있다. 하지만 n이 더 크면 대부분의 비트맵은 0이 더 많다.(이런 상황을 희소(sparse) 하다고 말한다).

> **컬럼 지향 저장소와 컬럼 패밀리**
> 카산드라와 HBase는 빅테이블로부터 내려오는 컬럼 패밀리 개념이 있다. 하지만 이를 컬럼 지향적이라고 부르기에는 오해의 소지가 많다. 각 컬럼 패밀리 안에는 로우 키에 따라 로우와 모든 컬럼을 함께 저장하며 컬럼 압축을 사용하지 않는다. 따라서 빅테이블 모델은 여전히 대부분 로우 지향이다.

### 메모리 대역폭과 백터화 처리

- 데이터 웨어하우스 질의는 디스크로부터 메모리로 데이터를 가져오는 대역폭이 큰 병목이다.
- 분석용 데이터베이스 개발자는 메인 메모리에서 CPU 캐시로 가는 대역폭을 효율적으로 사용하고 CPU 명령 처리 파이프라인에서 분기 예측 실패(branch misprediction)와 버블(bubble)을 피하며 최신 CPU에서 단일 명령 다중 데이터(single-instruction-multi-data, SIMD) 명령을 신경써야 한다.
- 디스크로부터 적재할 데이터 양 줄이기 외에도 컬럼 저장소 배치는 CPU 주기를 효율적으로 사용하기 적합하다.
- 컬럼 압축을 사용하면 같은 양의 L1 캐시에 칼럼의 더 많은 로우를 저장할 수 잇다.
- 앞에서 설명한 비트 AND와 OR 같은 연산자는 압축된 컬럼 데이터 덩어리를 바로 연산할 수 있다.
- 이런 기법을 **벡터화 처리(vetorized processing)**이라 한다.

## 컬럼 저장소의 순서 정렬

- 로우가 저자되는 순서가 반드시 중요하지 않다.
- 각 컬럼을 독립적으로 정렬할 수는 업삳.
- 컬럼별로 저장됐을지라도 데이터는 한 번에 전체 로우를 정렬해야 한다.
- 정렬된 순서의 장점으로 컬럼 압축에 도움이 된다.

## 집계: 데이터 큐브와 구체화 뷰

- 모든 데이터 웨어하우스가 컬럼 저장이 필수는 아니다.
    - 로우 지향 데이터베이스와 기타 아키텍처도 사용된다.
    - 컬럼 저장소는 즉석 분석 질의에 대해 상당히 빠르기 때문에 급속하게 인기를 얻고 있다.
- 데이터 웨어하우스의 다른 측면으로 **구체화 집계(materialized aggregate)**가 있다.
- 질의가 자주 사용하는 일부 카운트(count)나 합(sum)을 캐시하는 건 어떨까?
    - 이런 캐시를 만드는 한 가지 방법이 구체화 뷰(materialized view)다. 관계형 데이터 모델에서는 이런 캐시를 대개 표준 (가상) 뷰로 정의한다. 표준 뷰는 테이블 같은 객체로 일부 질의의 결과가 내용이다.
    - 차이점으로 구체화 뷰는 디스크에 기록된 질의 결과의 실제 복사본이지만 가상 뷰(virtual view)는 단지 질의를 작성하는 단축키일 뿐이다.
- 구체화 뷰는 원본 데이터의 비정규화된 복사본이기 때문이다.
- OLTP 데이터베이스에서는 구체화 뷰를 자주 사용하지 않는다.