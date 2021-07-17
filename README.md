<p align="center">
    <img width="200px;" src="https://raw.githubusercontent.com/woowacourse/atdd-subway-admin-frontend/master/images/main_logo.png"/>
</p>
<p align="center">
  <img alt="npm" src="https://img.shields.io/badge/npm-%3E%3D%205.5.0-blue">
  <img alt="node" src="https://img.shields.io/badge/node-%3E%3D%209.3.0-blue">
  <a href="https://edu.nextstep.camp/c/R89PYi5H" alt="nextstep atdd">
    <img alt="Website" src="https://img.shields.io/website?url=https%3A%2F%2Fedu.nextstep.camp%2Fc%2FR89PYi5H">
  </a>
  <img alt="GitHub" src="https://img.shields.io/github/license/next-step/atdd-subway-service">
</p>

<br>

# 인프라공방 샘플 서비스 - 지하철 노선도

<br>

## 🚀 Getting Started

### Install

#### npm 설치

```
cd frontend
npm install
```

> `frontend` 디렉토리에서 수행해야 합니다.

### Usage

#### webpack server 구동

```
npm run dev
```

#### application 구동

```
./gradlew clean build
```

<br>

## 미션

* 미션 진행 후에 아래 질문의 답을 작성하여 PR을 보내주세요.

### 1단계 - 화면 응답 개선하기

1. 성능 개선 결과를 공유해주세요 (Smoke, Load, Stress 테스트 결과)
   Smoke.md, Load.md, Stress.md 에 작성
2. 어떤 부분을 개선해보셨나요? 과정을 설명해주세요
    - 인터넷 구간 성능 개선을 위한 작업을 했습니다.

    1. 프론트쪽 정적 파일 경량화 작업
    2. Rerverse Proxy 개선

    - http 블록 수준에서 gzip 압축 활성화
    - proxy 캐시 설정
    - http1.1 -> http2 로 변경

    3. was 성능 개선하기

    - redis cache 설정
    - ~~비동기 처리~~
    - ~~적절한 thread pool 설정~~

---

### 2단계 - 조회 성능 개선하기

1. 인덱스 적용해보기 실습을 진행해본 과정을 공유해주세요

* 요구사항 주어진 데이터셋을 활용하여 아래 조회 결과를 100ms 이하로 반환

      use subway;

  Coding as a Hobby 와 같은 결과를 반환하세요.

    create index idx_hobby on programmer (hobby);
    select hobby, (count(hobby) / (select count(id) from subway.programmer)) * 100 as percent
    from subway.programmer group by hobby order by hobby desc;

프로그래머별로 해당하는 병원 이름을 반환하세요. (covid.id, hospital.name)

    select covid.id, hospital.name from subway.covid join subway.hospital on covid.hospital_id = hospital_id;

프로그래밍이 취미인 학생 혹은 주니어(0-2년)들이 다닌 병원 이름을 반환하고 user.id 기준으로 정렬하세요. (covid.id, hospital.name, user.Hobby, user.DevType,
user.YearsCoding)

    alter table programmer change column id id bigint(20) not null,
    add primary key (id);
    create index idx_hospital on covid (hospital_id);  
    create index idx_programmer on covid (programmer_id);
    
    select b.id, c.name
    from covid a   
    join (
    select id
    from programmer where hobby = 'yes' and (student != 'no' or years_coding <= 2)
    ) b
    on b.id = a.programmer_id
    join hospital c
    on a.hospital_id = c.id
    order by b.id;

서울대병원에 다닌 20대 India 환자들을 병원에 머문 기간별로 집계하세요. (covid.Stay)

    create index idx_age on member (age);
    create index idx_country on programmer (country);
    create index idx_member_id on programmer (member_id);
    create index idx_member_id on covid (member_id);
    create index idx_hospital_id on covid (hospital_id);
    create index idx_stay on covid (stay);

    select d.stay, count(p.member_id)
    from (
    select id from member where age between 20 and 29) m
    join (
    select member_id from programmer where country ='India') p
    on m.id = p.member_id
    join (
    select c.member_id, c.hospital_id, c.stay from covid c
    join (select id from hospital where name = '서울대병원') h on c.hospital_id = h.id) d
    on m.id = d.member_id
    group by d.stay;

서울대병원에 다닌 30대 환자들을 운동 횟수별로 집계하세요. (user.Exercise)

    select p.exercise, count(p.member_id) as count
	from programmer p 
    join 
    (select hospital_id, programmer_id from covid) c on p.id = c.programmer_id
    join
    (select id, name from hospital where name = '서울대병원') h on h.id = c.hospital_id
    join
    (select id from member where age between 30 and 39) m on m.id = p.member_id
    group by exercise order by count ;




2. 페이징 쿼리를 적용한 API endpoint를 알려주세요


    /favorites/pages



