---
layout: post
title: "Mybatis에서 ORM으로"
date: 2023-08-20
excerpt: "왜 ORM을 사용해야 하나요? Mybatis로도 구현 가능한데"
tags: [springboot, mybatis, jpa, ORM, backend]
comments: true
---

## 왜 ORM을 사용해야 하나요? Mybatis로도 구현 가능한데

Java/Spring 기반 프레임워크를 사용하여 백엔드 개발을 하면서 Mybatis + DB SQL의 조합에 익숙해진 탓에 ORM의 공부에 대한 필요성을 느끼지 못했다. 

뿐만 아니라 2번의 인턴 경험을 통해 Mybatis를 사용한 Spring 프레임워크를 접할 기회가 많았고  복잡한 쿼리를 처리하기 위해서는 ORM을 사용하는 것은 '현실적으로 어렵다' 라는 나름의 결론을 내리는 탓에 더욱 ORM을 공부할 필요성을 느끼지 못했던 것 같다. 

-----

## 개발자의 자세

개발자는 개발에 대해 꾸준히 공부하는 사람이라고 생각하면서도 ORM에 대해 궁금해 하지도 않는 어느 순간 내 모습에 대해 괴리갑을 느끼게 되었다. 뿐만아니라 Java/Spring을 공부하는 다른 친구들과의 대화에서 ORM을 주로 사용하는 탓에 커뮤니케이션이 잘 되지 않았다. 

때문에 이번글에서는 내가 왜 Mybatis의 익숙함을 버리고 ORM을 공부하게 되었는지, ORM의 단점이라고 생각했던 '복잡한 쿼리를 다루는 것에 어려움'을 어떻게 극복했는지 등 다양한 내용을 포스팅하려고 합니다. 

----

## Mybatis의 장단점

내가 느끼기에 Mybatis는 다음과 같은 장점이 있다고 생각했다. 

1. 직관적이다. 
2. 쿼리에 대해 잘 알고 있다면 식은죽 먹기다. 
3. **복잡한 쿼리를 다루기 편리하다.**
4. 디버깅이 쉽다. (보통 에러가 나는 원인은 쿼리가 잘못되는 경우가 많았다.)

물론 Mybatis의 단점도 분명했다. 

1. **DB에 종속적인 코드를 작성해야한다.** (DB가 변경되면 모든 코드를 변경해야한다.)
2. xml 파일을 사용해야한다. (이게 단점인지는 모르겠으나 대부분 개발자는 xml 파일을 싫어하는 것 같아 보인다. )
3. 디버깅이 어렵다. (컴파일시 에러를 못잡는다.)


내가 Mybatis를 고집했던 이유는 Mybatis의 가장 큰 단점인 DB에 종속적인 코드 작성의 문제는 내가 당장 하는 프로젝트에서는 문제가 되지 않았기 때문이다. (DB가 자주 바뀌는 상황이 자주 발생하는가? 나는 그렇지 않다고 생각했다.)

그리고 Mybatis의 가장 큰 장점인 복잡한 쿼리를 다루는 것이 쉬웠기 때문이다. 어려워 보이는 기능도 쿼리를 다루는게 능숙하다면 내게는 큰 걸림돌이 되지 않았었다. 때문에 쉽게 Mybatis를 포기하지 못했지만, ORM에 도전을 하기로 한 이상 문제를 하나씩 해결해 보기로 했다. 

----

## ORM의 장단점

ORM의 장점은 다음과 같다. 

1. 도메인 주도 설계(객체지향적 설계 가능)
2. **간단한 CRUD를 구현하는데 있어서의 강력함.** 
3. 컴파일시 에러를 잘 잡아낸다. 
4. **DB에 독립적인 코드 작성 가능**

이외에도 여러가지 장점이 있지만 이정도로 하고 단점을 알아보자. 

1. 복잡한 쿼리를 다루는데 있어서 구현이 어렵다. (이후에 설명할요..ㅎ)
2. DB 쿼리 자체를 이해하지 못하는 경우가 있는것 같다. (이건 개인적인 생각인데 ORM만을 경험한 개발자라면 간단한 쿼리 자체를 작성하지 못하는 경우를 생각보다 많이 봐서 굉장히 놀랐다.)

단점이 잘 떠오르지 않는다.(ORM을 괜히 쓰는게 아닌것 같다.)

이 글은 사실 ORM의 단점 '복잡한 쿼리 다루기'를 어떻게 할 것인지에 대한 빌드업이었던것 같기도 하다. 이제부터 어떻게 처리할 수 있는지 내 경험을 작성해보려고한ㄷ.ㅏ 

----

## JPA ORM - 복잡한 쿼리를 다루기 위한 노력

내가 ORM에서 복잡한 쿼리를 다루는 2가지 방법은 다음과 같다. 

1. JPQL
2. QueryDsl

이 글에서는 QueryDsl에 대해 더 자세하게 이야기 하고 싶어서 JPQL에 대해 간단히 정리하고 넘어 가겠다. 

* JPQL
    * @Query 어노테이션을 사용하면서 쿼리를 작성할 수 있지만 보통 DB 테이블이 아니라 엔티티를 기준으로 쿼리를 작성하기에 엔티티 관계에 대해서만 잘 이해하고 있다면 문제 없이 사용할 수 있다. 
    * 컴파일시 에러를 찾기 힘들다 (문자열 형태로 쿼리를 관리하기 때문에 실행전까지 에러를 발견할 수 없다. )
    * 조금 복잡한 쿼리는 JPA Interface를 사용하는 것보다 강력할 수 있다. 


그럼 이제 QueryDsl은 어떻게 복잡한 쿼리를 다룰 수 있었는지에 대해 코드로 확인해보자. 더 자세한 내용은 [Spring JPA와 QueryDsl-JPA](https://freckle-hallway-4b2.notion.site/Spring-JPA-QueryDsl-JPA-7b550a3f63e340d2907744f7627a3135?pvs=4)를 참고해도 좋을 듯 하다. 

먼저 나는 '복잡한 쿼리'에 대한 정의를 다음고 같이 내린다. 

1. Join 절이 많은가?
2. 서브쿼리를 사용할 일이 많은가?
3. 집계함수를 써야하는가?

이외에도 많지만 보통 3가지 경우를 주로 보는것 같다. 아래의 코드는 querydsl을 통해 로직을 설계하는 예시를 가져와봤다. 

```java
import static gdsc.netwalk.domain.activity.entity.QActivity.activity;
import static gdsc.netwalk.domain.club.entity.QParticipant.participant;
import static gdsc.netwalk.domain.user.entity.QUser.user;

public class ClubQueryDslRepositoryImpl implements ClubQueryDslRepository {
    JPAQueryFactory queryFactory;
    @Override
    public List<ClubInUserListResponse> findClubInUsers(Long clubId) {
        return queryFactory
                .select(new QClubInUserListResponse(
                    user.id,
                        user.name,
                        user.email,
                        participant.isActive,
                        queryFactory
                                .select(activity.totalActDistance.sum())
                                .from(activity)
                                .where(activity.user.id.eq(user.id)),
                        queryFactory
                                .select(activity.totalActTime.sum())
                                .from(activity)
                                .where(activity.user.id.eq(user.id))
                ))
                .from(participant)
                .innerJoin(participant.user, user)
                .where(participant.club.id.eq(clubId))
                .fetch();
    }
}
```

이 코드에서 보여주고 싶었던 부분은 join절과 서브쿼리를 사용하는것이 생각보다 단순하다는 것이었다. 무엇보다 **코드 자체가 가독성이 좋다** 는 점이 맘에 들었다. 

QueryDsl은 본래 interface를 설계하는 것으로 '원하는 데이터와 그 데이터를 가져오기 위한 데이터'를 적재적소에 가져다 놓기만한다면 쉽게 사용할 수 있는 것 같다. 

QueryDsl을 사용하는 방법에 대한 글이 아닌만큼 이쯤에서 마무리하겠다. 

-----

## 종합적인 내생각

마지막은 Mybatis와 ORM에 대한 종합적인 내생각을 정리하고 마무리하겠다. 

1. ORM을 잘 다루기 위해서는 쿼리에 대한 이해가 반드시 필요하다. 
2. 무조건적으로 하나의 기술을 고집할게 아니라 프로젝트의 특성에 맞춰 기술을 고르자. 
3. ORM은 생각보다 강력하다.(더 좋은 코드를 작성하기 위해서 배민 기술 블로그를 참고하자. )