npm install
nodemon index.js
localhost:4000 -> apollo 라이브러리에서 제공하는 프로그램 뜬다.
(restAPI 사용 시 포스트맨을 활용한 것과 같은 맥락.)

===

# GraphQL은 왜 탄생했고, 어떤 특성을 가지고 있는가?

query 라고 친다. rest api로 치면 get 같은 거다.
우리는 팀을 받아올 거기 때문에 일단 teams에 모든 항목을 받아와본다.

```
query {
  teams {
    id
    manager
    office
    extension_number
    mascot
    cleaning_duty
    project
  }
}
```

재생버튼을 누르면 모든 팀의 정보를 받아온다.

1. overfetching을 해결
   manager와 office만 받아오고 싶다면 이렇게 teams를 요청하면 된다.

```
query {
  teams {
    manager
    office
  }
}
```

id가 2인 팀을 받고 싶다면 team(id: 2)로 요청한다.

```
query {
  team(id: 2) {
    manager
    office
  }
}
```

2. underfetching 해결
   rest api에서는 team과 people은 다른 api로 받아와야 했다.
   graphql에서는 그냥 팀에 대한 쿼리에 members를 같이 끼워넣으면 된다.

```
query {
  team(id: 2) {
    manager
    office
    members {
      team
      first_name
    }
  }
}
```

이로서 team이라는 계층과 people이라는 정보를 한꺼번에 받아올 수 있는 것이다.
여기서 왜 people 이 아니라 members냐 하면 typedefs 에서 members라는 키로 people을 조회하도록 설정했기 때문이다.

```
const typeDefs = gql`
    type Team {
        id: ID!
        manager: String!
        office: String
        extension_number: String
        mascot: String,
        cleaning_duty: String!
        project: String
        members: [People]
    }
}

const resolvers = {
    Query: {
        teams: (parent, args) => dbWorks.getTeams(args),
        team: (parent, args) => dbWorks.getTeams(args)[0],
    },
}
```

다시 이제 team(id: 1)이 아니라, 모든 팀에 대한 정보를 받아오기 위해선 teams로 쿼리를 날리면 된다.

```
query {
  teams {
    manager
    office
    members {
      team
      first_name
    }
  }
}
```

현재 디비에는 roles라는 데이터도 있다.

```
query {
  roles {
    id
    job
  }
}
```

그러면 아까 요청했던 teams와 roles을 같이 받아오고 싶다면 둘 다 쓰면 된다.

```
query {
  teams {
    manager
    project
  }
  roles {
    id
    job
  }
}
```

이런 식으로 하면 overfetching, underfetching 모두 해결이 된다.

===

정보를 넣을 때는 어떻게 할까?
GraphQL로 정보를 바꿀 때는 query가 아니라 mutation으로 요청한다.

```
mutation {
  postTeam (input: {
    manager: "John Smith"
    office: "104B"
    extension_number: "#9982"
    mascot: "Dragon"
    cleaning_duty: "Monday"
    project: "Lordaeron"
  }) {
    manager
    office
    extension_number
    mascot
    cleaning_duty
    project
  }
}
```

postTeam이라는 명령어에 새 정보를 실어보낸다. 그리고 다시 query로 팀을 요청해 본다.

```
query {
  teams {
    id
    office
  }
}
```

추가했던 대로 6팀까지 조회되는 것을 알 수 있다.
teams.js에 postTeam 명령어가 정의돼 있다.

```
const resolvers = {
    Mutation: {
        postTeam: (parent, args) => dbWorks.postTeam(args),
        editTeam: (parent, args) => dbWorks.editTeam(args),
        deleteTeam: (parent, args) => dbWorks.deleteItem('teams', args)
    }
}
```

팀 정보를 수정하거나 삭제할 때 위에 미리 정의한 명령어를 활용하면 된다.

===

GraphQL의 장점을 정리하면 다음과 같다.

1. 필요한 정보들만 선택하여 받아올 수 있다.

- overfetching 문제 해결
- 데이터 전송량 감소

2. 여러 계층의 정보들을 한 번에 받아올 수 있음

- underfetching 문제 해결
- 요청 횟수 감소

rest api를 사용하는 서비스에서도 기술적으로 위와 같은 해결 방법이 없는 것은 아니다. 그러나 rest api에 정해진 원칙을 지켜야 하기 때문에 데이터의 소통을 설계하는 데 있어서 제약이 생긴다. GraphQL은 이처럼 다른 형식을 사용하기 때문에 이와 같은 해결책이 가능한 것이다.

3. 하나의 endpoint에서 모든 요청을 처리

- 하나의 URI에서 POST로 모든 요청 가능.

대신에 명령어를 정해서 어떤 요청을 할지 상대방에게 알려주는 것이다.

서비스가 어떤 데이터를 어떻게 주고받냐에 따라서 rest api가 더 적합할 때가 있다. 둘의 차이를 이해해야 효율적으로 설계를 할 수 있을 것이다.

===

# Apollo는 무엇인가, 왜 쓰는가?

RestAPI와 마찬가지로, graphql은 한 형식일 뿐이다.
누군가가 정보를 받을 수 있도록 뭔가 만들어둔 게 아니라는 것이다.

graphql을 구현할 솔루션(라이브러리)이 필요하다.

- 백엔드에서 정보를 제공 및 처리
- 프론트엔드에서 요청 전송
- GraphQL.js, GraphQL Yoga, AWS Amplify, Relay, ...

백엔드 단에서 요청이 들어오면 Graphql이 요청을 해석해서 데이터베이스로부터 정보를 꺼내다가 가공 및 처리해서 다시 보내주는 것이다.
또 프론트 단에서도 해당 형식에 맞게 graphql 요청을 보낼 수 있어야 한다.
백엔드, 프론트 모두에서 graphql 명세대로 정보를 요청하고 답신해주는 솔루션이 필요하다.

graphql에서 정식으로 제공하는 graphql.js가 있고, graphql yoga도 많이 사용되는 백엔드단 라이브러리 중 하나다. 프론트단에서는 aws amplify, relay 등이 있다.

https://graphql.org/code/

각 언어나 환경, 클라이언트, 서버단 마다 프로젝트에서 사용할 수 있는 graphql 라이브러리를 찾아서 쓸 수 있다.

이 많은 라이브러리들 중에서 Apollo를 사용하는 이유는

- 백엔드와 프론트엔드 모두 제공
- 간편하고 쉬운 설정
- 풍성한 기능들 제공

https://www.apollographql.com/docs/apollo-server/
https://www.apollographql.com/docs/react/

아폴로 프론트단 문서에는 리액트가 나와 있지만, Vue에서도 Apollo를 사용할 수 있다.
먼저 apollo server를 활용해서 백엔드 서버를 제작하고, 이를 심화한 다음 apollo client와 react를 활용해서 프론트엔드 웹을 제작할 것이다.
