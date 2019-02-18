# TypeScript Module Alias 잡담

typescript 에서의 module alias 는 생각보다 까다롭다.

개발에서야 tsconfig.json 의 path 를 잘 적용할 수 있지만, 실제 배포시에는 빌드된 js 파일이 package.json 혹은 node 실행하는 실제 경로와 alias 된 매치가 안되어 실제 모듈을 엉뚱한 곳에서 로딩할 수 있다.

예를 들면 개발시의 엔트리 포인트가 src/index.ts이고 빌드 후 배포 상태에서의 엔트리 포인트는 dist/index.js 라고 한다면 모듈 alias 도 root: 'src' 에서 root: 'dist' 로 바뀌어야 제대로 alias 가 적용된다는 것이다.

한참 삽질하다가 모듈 전처리기를 넣는걸로 타협했다.
코드 런타임에 모듈 루트를 재정의하는 방식이다.

이 링크를 참고했다.

https://gist.github.com/branneman/8048520

장점만 보였던 코드 전처리들이 최근에는 뭔가 억지스러운 느낌도 여러 부분에서 받는다.

전처리류(바벨 등)들이 주는 편의성이 매우 좋지만, 가끔은 바닐라가 그립다.
