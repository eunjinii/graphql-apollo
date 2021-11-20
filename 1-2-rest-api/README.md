https://www.yalco.kr/@graphql-apollo/1-2/

nodemon 설치하고 nodemon index.js 입력했더니 localhost:3000 이라는 주소로 data-in-csv 에 있는 데이터들이 REST API 형식으로 제공되고 있다.
localhost:3000 들어가 보면 현재는 아무것도 뜨지 않는다.

index.js에 있는 /api/team을 요청해 보자. 데이터가 온다.

현재 우리 데이터베이스에는 어떤 회사에 대한 팀, 인원, 역할, 장비, 소프트웨어 등이 데이터베이스로 제공이 되고 있다.
이 데이터를 REST API 형식으로 받아볼 것이다.

임의로 설정한 주소 /api
뒤에다가 /team 을 붙이면 -> 나는 이 팀에 대한 정보들을 get으로 받아오겠다.
포스트맨으로 쏴 보자. http://localhost:3000/api/team

이 중에서 1팀만 받아오고 싶다?
/team 뒤에 /:id 를 붙여서 받아온다.
뒤에 1을 붙이면 1팀만 오고, 2를 붙이면 2팀에 대한 정보만 온다.

전직원에 대한 정보를 받아오고 싶으면 /people 이라고 요청한다.
여기서 특정 키에 대한 값으로만 걸러내고 싶다면 ?${key}=${value} 로 요청한다.
http://localhost:3000/api/people?role=developer&sex=female 로 여성 개발자만 요청해 보자.

==
그럼 새로운 팀을 넣고 싶다면?
body에 post로 정보를 실어 보낸다.
http://localhost:3000/api/people 에
{
"id": 6,
"manager": "Olivia Smith",
"office": "106A",
"extension_number": "#3509",
"mascot": "Dolphin",
"cleaning_duty": "Monday",
"project": "Apple"
}
보낸 다음, get으로 다시 /people 을 조회해 보면, 추가돼 있다.

수정과 삭제는 put, delete로 각각 요청을 해 본다.

==
합리적이고 체계적인 방법인데 왜 GraphQL이 등장했을까?

1. overfetching
   나는 해당 팀의 매니저와 오피스만을 알고 싶다. 그런데 restAPI로 요청을 하면, 내가 원하는 정보 외에도 많은 것이 한꺼번에 넘어온다.
   주고받는 팀의 갯수가 많아지면 많아질 수록 네트워크 비용이 많아질 수밖에 없다.
   이걸 overfetching이라고 한다.
   딱 필요한 정보들만 알아왔으면 좋겠다는 요구가 생겨났다.

2. underfetching
   내가 원하는 정보들이 여러 계층에 걸쳐 있을 수 있다.
   팀들을 받아오는 요청과 사람들을 받아오는 요청이 따로 있는데,
   팀장과 그에 대한 팀원을 받아오려면 두 개의 요청을 따로 보내서 받아와야 한다는 것이다.
   한 번의 요청에 정보가 충분하지 않다.

최소한의 요청으로 내가 원하는 데이터만 받아와서, 네트워크 낭비를 방지할 수 있다면?

그렇게 해서 graphql이 등장했다.
