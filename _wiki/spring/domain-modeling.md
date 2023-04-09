---
layout  : wiki
title   : 도메인 모델 중심으로 REST API 설계하기
toc     : true
date    : 2023-04-09 16:09:00 +0900
updated : 2023-04-09 16:09:00 +0900
toc     : true
public  : true
parent  : spring
comment : true
regenerate: true
---

* TOC
{:toc}

기존에 개발이 진행되어왔던 프로젝트를 유지보수 하면서, 도메인 모델을 중심으로 설계를 하는 것에 대해 고민이 늘어났다.  

사용하는 사람이 기업의 인사 담당자라고 가정을 하고, 지원자의 인적사항, 학력사항, 경력사항을 받아서 서버 어딘가에 저장을 하는 API가 필요한 상황이라고 했을 때,
유지보수가 가능하도록 어떻게 유연한 설계를 할 수 있을까?  

우선, API 설계부터 시작을 하고, 지원자로 부터 받아오고 싶은 정보들을 정리해보자.  

인적사항
- 이름/이메일/생년월일
- 연락처
- 링크드인과 같은 SNS
- 블로그 주소

학력사항
- 학교명
- 대학인 경우, 전공명을 함께 기술
- 어떤 논문을 제출 했는지, 무엇을 공부했는지 요약할 수 있는 필드
- 기간

경력사항
- 회사명
- 역할
- 포지션
- 어떤 일을 했는지 요약할 수 있는 필드
- 기간

이 정보들을 취합해서, 아래와 같이 API의 그림을 대략적으로 그려본다.

```json
{
    "name": "테스터",
    "email": "tester@example.com",
    "birthDate": "1991-01-01",
    "phoneNumber": "01011119999",
    "linkedIn": "https://www.linkedIn.com/...",
    "blog": "...",
    "careers": [{
        "company": "kakaobank",
        "role": "Backend Developer",
        "position": "Junior",
        "description": "무엇을 했나?",
        "period":{
            "DayOfBeginning": "2023-01-01",
            "DayOfEnd": null
        }
    }],
    "schools": [{
        "name": "학교이름",
        "fieldOfStudy": "전공명이 들어갑니다",
        "description": "무엇을 공부했는지 요약이 들어갑니다",
        "period":{
            "DayOfBeginning": "2013-03-02",
            "DayOfEnd": "2016-02-10"
        }
    }]
}
```

대략적으로 이런 모양이 나올 것 같은데, 실제로 구현할 때는 데이터 수정이 일일이 번거로울 수 있고 `의미 없이 데이터가 나열된 것 같은 느낌`이 든다. 위의 데이터가 `무얼 말하고 싶은지` 와닿지 않는다.  

인사담당자는 지원자의 평소 습관, 활동 이력을 알 수 있도록 웹 사이트 정보를 필요로 한다. 그렇다면 블로그 주소, 링크드인 계정과 같은 웹 사이트 정보를 urls 라는 데이터로 모을 수 있을 것 같다.  

학력사항의 학교도 보자. 단순히 학교를 적을 수 있지만 요즘은 부트캠프, 온라인 강의로 수료증을 제공하는 교육기관도 점차 늘어나고 있다. 비단 학교 뿐 일까? 요새는 기업에서도 교육 프로그램을 운영하고 있다. 이 부분까지 포함하려면 school 이라는 이름은 지엽적일 수 있다. 포괄하는 educations로 변경할 필요성이 있을테고, school 대신에 institution으로 잡아보자. 데이터 목록에 추가된 기관들은 선택이 가능하도록 하고, 그 이외의 교육 코스들은 수기로 작성하도록 오픈해두자.  

```json
{
    "name": "테스터",
    "email": "tester@example.com",
    "birthDate": "1991-01-01",
    "phoneNumber": "01011119999",
    "urls": [{
        "label": "LinkedIn", "url":"https://www.linkedIn.com/...",
        "label": "GitHub", "url": "https://bloomspes.github.io/"
    }],
    "careers": [{
        "company": "kakaobank",
        "role": "Backend Developer",
        "position": "Junior",
        "description": "무엇을 했나?",
        "period":{
            "DayOfBeginning": "2023-01-01",
            "DayOfEnd": null
        }
    }],
    "educations": [{
        "institution": {
            "id": "존재하는 경우 선택 가능하도록",
            "name": "coursera"
        },
        "fieldOfStudy": "관련 전공에 대한 설명",
        "description": "",
        "period":{
            "DayOfBeginning": "2013-03-02",
            "DayOfEnd": "2016-02-10"
        }
    }]
}
```

이렇게 데이터 스키마를 잡았으면, 지원자의 Aggregate 안에 포함되는 도메인 클래스들을 모델링을 통해 사용할 수 있는 도메인 객체로 변환해야 한다.  

여기서, 지원자의 인적사항에 대해서는 `PersonalInformation` 이라는 Value Object로 Hierachy를 나타낼 수 있고, PersonalInformation 안에는 지원자의 이름, 생년월일, 연락처, 웹 사이트 정보가 담기게 된다.  

나는 학력사항과 경력사항에 대해서 의미의 유사성을 보게 되었다. 시간 순으로 내가 어디서, 무엇을, 세부사항, 기간 과 같은 정보로 나열이 된 클래스라는 것을 인지하게 됐을 때 이 의미상 구조를 표현하려고 노력했다.  

학력사항, 경력사항은 공통으로 상속 받을 수 있는 `HistoryItems` 라는 상위 추상 클래스를 선언하고, 이 HistoryItems 클래스 안에는 어디서, 무엇을, 세부사항, 기간에 대한 정보가 담기게 된다.

```Java
@MappedSuperClass
public abstract class HistroyItems {
    @EmbeddedId
    protected Id id;

    @Embedded
    protected Institution institution;

    @Embedded
    protected Description description;

    @Embedded
    protected Period period;
}
```

나는 프로젝트에서 JPA를 사용중이기 때문에, 도메인 객체의 상속 관계 매핑을 구현하는 경우, 공통 매핑 정보를 가져올 수 있는 기능에 대해 찾게 되었다.  

Spring Boot에서는 감사하게도! 클래스 앞에 `@MappedSuperClass`를 붙여주면 자동으로 잡아준다.  

이 작업을 하면서 한 번더 깨닫게 된 사실은, API 설계와 도메인 모델, 데이터 테이블 설계는 서로 무관하고 일대일로 매칭이 될 이유가 없다는 점 이다.  

이런 유연한 접근이 `도메인 주도 설계`라는 책을 이해하는데 조금 더 도움이 되었다.

