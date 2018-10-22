# Redux Saga 프로젝트 적용기

이 글은 Redux 의 사용 경험이 없다면 읽기 어려울 수 있습니다.
Redux 에 관한 글은 [공식 문서](https://lunit.gitbook.io/redux-in-korean/)를 참고하세요.

## Redux Saga?

### 의미
>Redux Saga 는 애플리케이션의 부수효과(side effect. 비동기 데이터 로딩, 브라우저 캐시 엑세스같은 다소 흐름상 껄끄러운 작업) 을 보다 쉽게 관리하고, 실행하기 쉽고 테스트하기 쉽고, 오류 처리가 용이하도록 해 주는 라이브러리입니다 - [Redux-Saga 공식 문서](https://redux-saga.js.org/)

라고 공식사이트에서 설명하고 있다.

이것으로 Saga 에 대해 알아챈 사람도 있겠지만, 나는 이 글을 이해하지 못해서 몇번이고 다시 읽었었다.

지금와서는 이 글이 무엇인지 전부는 아니어도 대충은 알게 되었고, 덕분에 이 글을 쓸 자신이 생겨 이렇게 글을 쓰게 되었다.

때마침 새 환경의 프로젝트를 시작하는 시기였고, 웬지모를 허세도 한몫하여 그때의 나에게 익숙하지도 않은 ES6의 [Generator](https://www.google.co.kr/search?q=bsidesoft+Generator&oq=bsidesoft+Generator&aqs=chrome..69i57j69i60.1017j0j9&sourceid=chrome&ie=UTF-8) 문법으로 구성된 Redux-saga 를 자신있게 도입해버렸고, 그 결과는 다행히도 매우 만족스럽게 돌아왔다.

### 부수효과

Redux-Saga 의 관련 포스트나 문서를 읽다보면 `부수효과` 라는 것을 많이 언급한다. 하지만 개인적으로 어느 문서도 부소효과에 대해 나를 정확하게 이해시키는 글이 없었다. (내 이해력이 부족한 것일수도 있다)

내가 이해한 부수효과라는건 일반적인 동기 방식으로 동작하는 액션 -> 상태 변경 형식의 애플리케이션에서 `액션과 상태 변경 사이에 발생하는 모든 작업` 을 뜻한다.

일반적인 액션 -> 상태 변경에서는 부수효과가 발생할 일이 없다.

일반적인 redux 의 흐름도는 다음과 같다

```javascript
const showMemberDetail = () => {
    dispatch({ type: 'user/show-detail', payload: { actedAt: new Date() } });
}

const reduceMember = (state = {}, action) => {
    const { type } = action;
    switch(type) {
        const { payload: { actedAt } } = action;
        case 'user/show-detail': {
            return {
                ...state,
                actedAt
            }
        }
        default: return state;
    }
}
```
이 액션에서는 따로 부수효과가 없다.

Redux 를 사용한 애플리케이션에서 (React.js 든 Vue.js 든... 다른것이든) `showMemberDetail` 이 수행되면 액션 `user/show-detail` 이 dispatch 되고 `reduceMember` 는 액션을 처리하면서 새 상태를 반환할 것이다.

하지만 이 과정의 중간에 부수효과가 발생한다면 상황은 복잡해진다.
대표적인 부수효과인 ajax 나 fetch 등의 바동기 서버 요청을 예로 들어보자.

1. 어떤 액션이 인자와 함께 dispatch 된다
2. 인자를 가지고 서버에 요청 (Ajax, Fetch ...) 한다
3. 서버 응답은 어떤 것이 올지 모른다. 운이 좋게도 성공적으로 응답될수 있지만, 때로는 에러가 발생한다. 전달된 인자가 불량일수도 있어서 체크하는 로직이 들어가야 함은 물론, 지나치게 긴 대기로 타임아웃이 발생할 수도 있다.
4. 게다가 서버 요청은 비동기이기 때문에 요청 중간에 다른 작업도 처리하다가 요청이 끝나면 (오류든 성공이든) 다시 액션을 dispatch 하는 임무도 맡고 있다.
5. 이 과정에서 발생하는 다양한 분기 및 비동기, 여러 케이스를 처리하려면 액션을 dispatch 하는 코드가 장황해질수밖에 없다.

이 과정을 다음 코드에 써보았다.

```javascript
```

뭔가 장황해졌다.
이 장황함을 불러온 부분을 나는 `부수효과` 라고 부르고 있다. 여기서는 비동기 요청을 예로 들었지만, 특정 액션이 들어올 때 화면 스크롤을 움직인다든지, 팝업을 오픈한다든지, 로깅을 하는 등의 특정 상태에 관련이 없는 것들 모두를 뜻한다고 볼 수 있다.

여기까지 읽다 보면 몇몇 분들은 이상태로도 괜찮지 않나. 굳이 뭔가를 더 써야하나? 라는 생각이 들지도 모르겠다. 그동안 내가 Redux 쓰면서 일반적으로 했던 일인데 뭐가 특별한지 라고 느낄수도 있다.

그렇다면 실무에서 일어나는 좀더 더티하고 실전적인 예제를 들어보겠다.

## 그래서 뭐가 좋은데

## 생소한 Task 와 Take

## 구조잡기

## 사람잡는 yield군과 디버그양

## 콜백은 어떻게 하지?!?!

## Refactoring ...

## 마치며