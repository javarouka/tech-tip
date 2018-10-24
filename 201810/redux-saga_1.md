# Redux Saga 프로젝트 적용기

이 글은 Redux 의 사용 경험이 없다면 읽기 어려울 수 있습니다.
Redux 에 관한 글은 [공식 문서](https://lunit.gitbook.io/redux-in-korean/)를 참고하세요.

## Redux Saga?

### 의미
>Redux Saga 는 애플리케이션의 부수효과(side effect. 비동기 데이터 로딩, 브라우저 캐시 엑세스같은 다소 흐름상 껄끄러운 작업) 을 보다 쉽게 관리하고, 실행하기 쉽고 테스트하기 쉽고, 오류 처리가 용이하도록 해 주는 라이브러리입니다 - [Redux-Saga 공식 문서](https://redux-saga.js.org/)

라고 공식사이트에서 설명하고 있다.

이것으로 Saga 에 대해 알아챈 사람도 있겠지만, 나는 이 글을 이해하지 못해서 몇번이고 다시 읽었었다. 지금은 전부는 아니어도 어느 정도 만큼은 이해할 수 있게 됐다고 생각하기에 이렇게 글을 써 보려고 한다.

<!-- 막 Redux-Thunk 로도 좋구나 하던 내가 Redux-Saga 를 하려고 한 것은, 새 환경의 프로젝트를 시작하는 시기에다가 웬지모를 허세도 한몫하여 팀원들의 동의를 구하는둥 마는둥 하며 신규 프로젝트에 덜컥 Redux-saga 를 자신있게 도입해버렸고, 그 결과는 다행히도 매우 만족스럽게 돌아왔다. -->

### 부수효과

Redux-Saga 의 관련 포스트나 문서를 읽다보면 `부수효과` 라는 것을 많이 언급한다. 하지만 개인적으로 어느 문서도 부수효과에 대해 나를 정확하게 이해시켜주는 글이 없었다. (내 이해력이 부족한 것일수도 있다...😭)

내가 이해한 부수효과라는건 일반적인 동기 방식으로 동작하는 액션 -> 상태 변경 형식의 애플리케이션에서 `액션과 상태 변경 사이에 발생하는 모든 작업` 을 뜻한다.

#### 일반적인 액션 -> 상태

일반적인 액션 -> 상태 변경에서는 부수효과가 발생할 일이 없다.

예를 들어보자. 일반적인 Redux 의 흐름도는 다음과 같다.

1. 액션을 디스패치한다. 별도의 payload 가 있을 수 있다
2. reducer 는 액션을 처리한다. 상태가 갱신된다. 

이 액션에서는 따로 부수효과가 없다.
다이렉트로 action => reducer => update state 로 이어진다.

간단한 예제로 써보자.

action creator 이다. dispatch 는 레퍼런스가 있다고 가정한다.
```javascript
const showMemberDetail = () => {
    dispatch({ type: 'member/show-detail', payload: { actedAt: new Date() } });
}
```

reducer 이다.
```javascript
const reduceMember = (state = {}, action) => {
    const { type } = action;
    switch(type) {
        const { payload: { actedAt } } = action;
        case 'member/show-detail': {
            return {
                ...state,
                actedAt
            }
        }
        default: return state;
    }
}
```

Redux 를 사용한 애플리케이션에서 (React.js 든 Vue.js 든... 다른것이든) `showMemberDetail` 이 수행되면 액션 `user/show-detail` 이 dispatch 되고 `reduceMember` 는 액션을 처리하면서 새 상태를 반환할 것이다.

#### 부수효과가 있는 액션 -> 부수효과 -> 상태

하지만 이 과정의 중간에 부수효과가 발생한다면 상황은 복잡해진다.
앞서 부수효과라는 것은 액션과 상태 변경 사이의 어떤 일이라고 표현했다. 대표적인 부수효과인 ajax 나 fetch 등의 Asynchronous 서버 요청을 예로 들어보자.

1. 어떤 액션이 인자와 함께 dispatch 된다
1. 인자를 가지고 서버에 요청 (Ajax, Fetch ...) 한다
1. 서버 응답은 어떤 것이 올지 모른다. 운이 좋게도 성공적으로 응답될수 있지만, 때로는 에러가 발생한다. 
1. 전달된 인자가 불량일수도 있어서 체크하는 로직이 들어가야 함은 물론, 지나치게 긴 대기로 타임아웃이 발생할 수도 있다.
1. 이 케이스마다 전부 다른 액션 혹은 action payload 가 전달된다
1. 서버 요청은 비동기이기 때문에 요청 중간에 다른 작업도 처리하다가 요청이 끝나면 (오류든 성공이든) 다시 액션을 dispatch 하는 임무도 맡고 있다.
1. 이 과정에서 액션을 dispatch 하는 코드가 장황해진다.

이 과정을 다음 코드에 써보았다.

action creator 이다. dispatch, fetch 는 레퍼런스가 있다고 가정한다.

비동기 처리에 강점이 많은 [async - await](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function) 를 사용하였다.

action creator 이다.

데이터 요청 전/후로 액션이 dispatch 되며, 오류나 맞을 수 없을 떄에도 특별한 액션을 dispatch 하고 있다.

```javascript
const requestMemberData = async memberId => await fetch(`/api/member/${memberId}`);

const showMemberDetail = ({ memberId }) => {
    dispatch({ type: 'member/request-detail' });
    requestMemberData(memberId)
        .then(member => {
            if(!member) {
                dispatch({ type: 'member/response-error', payload: { message: 'not found' } } });
            }
            dispatch({ type: 'member/response-detail', payload: { member } });
        })
        .catch(error => {
            dispatch({ type: 'member/response-error', payload: { message: error.message } } });
        });
}
```

reducer 이다.

요청 진행중을 나타내는 상태와 요청 오류를 나타내는 상태 두개가 추가되었다.

```javascript
// defaultState
const defaultState = {
    memberLoading: false,
    member: null,
    errorMessage: null,
};

const reduceMember = (state = defaultState, action) => {
    const { type, payload = {} } = action;
    switch(type) {
        const { payload: { actedAt } } = action;
        case 'member/request-detail': {
            return {
                ...state,
                memberLoading: true,
                errorMessage: null,
            }
        }
        case 'member/response-detail': {
            const { member = {} } = payload;
            return {
                ...state,
                memberLoading: false,
                member,
            }
        }
        case 'member/response-error': {
            const { message = 'unknown error' } = payload;
            return {
                ...state,
                memberLoading: false,
                errorMessage: message,
            }
        }
        default: return state;
    }
}
```

위 코드에서는 부수효과로 비동기 요청을 예로 들었지만, 특정 액션이 들어올 때 화면 스크롤을 움직인다든지, 팝업을 오픈한다든지, 로깅을 하는 등의 특정 상태에 관련이 없는 것들 모두를 뜻한다고 볼 수 있다.

Redux 쓰면서 일반적으로 하는거 아닌가? 여기서 더 뭔가가 필요한걸까? 라고 느낄수도 있다. 

"이상태로도 괜찮다. 굳이 뭔가를 더 써야하나?"

라고 생각한다면 실무에서 일어나는 좀더 더티하고 실전적인 예제를 들어보겠다.

(사실 이정도 일은 redux-thunk 가 더 단순하고 편하다) 

## 그래서 뭐가 좋은데

### 단순한 Redux + 비동기 작업

특정 회원을 조회하는 액션이 있다고 해보자
그럼 보통 다음과 같은 플로우를 탈 것이다.

위 부수효과가 있는 액션 예제 코드에 추가 요구사항이 필요해졌다.

회원 정보를 로딩한 뒤 로딩이 끝나면 이 회원의 주문정보를 추가 로딩해야 한다고 가정한다.

물론 주문정보 요청시에도 로딩중 상태와, 요청 실패, 데이터 없음 등의 처리가 필요하다.

코드로 써보자

### 추가사항이 들어온다면...

### 이젠 감당할 수 없다... 💣

## 생소한 Effects

## 구조잡기

## 사람잡는 yield군과 디버그양

## 콜백은 어떻게 하지?!?!

## Refactoring ...

## 마치며