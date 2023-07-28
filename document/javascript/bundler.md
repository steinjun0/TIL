# 번들러, babel, swc, remix가 무엇인가

[차세대 번들러 비교 및 분석 (feat. webpack, rollup, esbuild, vite)](https://bepyan.github.io/blog/2023/bundlers)

# 번들러

번들러는 여러개의 파일들을 하나로 묶어주는 도구이다.

1. 브라우저는 ES Modules를 지원하지 않았다
    1. ES6 import를 사용할 수 없고, script간 모듈을 보낼 수 없었다.
2. 그래서 전역 객체를 활용해서 스크립트 간 모듈을 공유했다.

# Webpack

1. SPA가 급부상하면서, 번들러의 역할이 커질 때, 사용량이 증가함
2. webpack의 정체성은 **“통합”**
3. 하나의 설정 파일에서 번들이 생성될 수 있도록 컨트롤 할 수 있음
4. entry, output을 명시하고 plugin, loader로 파일을 다룰 건지 명시하면 된다.
5. plugin과 loader를 쉽게 부착할 수 있다
    1. loader로 파일을 변환, 번들링, 빌드 진행
    2. plugin을 통해서 output 파일을 튜닝해줌
6. Hot Module Replacement를 처음으로 제안하고, 사용중이다.
7. code splitting
    1. 번들 로드 최적화 작업
    2. 파일들을 여러 파일로 분리하여, **스크립트를 병렬로 로드**
        1. → 페이지 로딩 속도 개선
    3. 초기에 구동될 필요가 없는 코드를 분리
        1. → lazy로딩으로 페이지 로딩 속도 개선

# Rollup

1. module bundler
2. Rollup의 정체성은 **“확장”**
    1. compiles small pieces of code into something larger and more complex, such as a library or application
3. 어플리케이션은 webpack 라이브러리는 rollup이라는 여론 형성
4. webpack과 거의 유사하게 사용
    1. input, entry를 설정, 추가 기능을 plugin으로 사용
5. vs webpack
    1. webpack은 commonJS를 사용, rollup은 typescript 사용
    2. rollup은 ES6형태로 빌드 가능(webpack도 이제 된다)
    3. rollup이 조금 더 빠르다
    4. rollup이 조금 더 가볍다.
        1. tree shaking은 ES6에서 제대로 동작한다.
        2. 사용되는 모듈만 [AST 트리](https://yceffort.kr/2021/05/ast-for-javascript)에 포함시키는 방식으로 작동
    5. rollup은 CommonJS 코드를 ES6코드에서 사용가능

# ESBuild

1. 지금까지 번들러는 내부적으로 JS를 기반으로 번들링 한다.
2. ESBuild는 Go로 작성되었다.

# Vite

1. ESBuild 단점을 보완했다.
2. ESBuild로 파일을 통합하고, rollup을 통해 번들링

# SWC

1. 기존 Babel의 역할을 대체한다.
2. [카카오 엔터 설명](https://fe-developers.kakaoent.com/2022/220217-learn-babel-terser-swc/)
    1. Terser: parser + (mangler + compressor)(=minifier)
    2. babel과 terser의 역할을 대체한다.(webpack을 대체하지 않는다)
        1. nextjs는 swc-loader를 통해 webpack을 사용중이다.
3. 
4. dapada에 적용
    1. Next.js' swc compiler is used for minification by default since v13. This is 7x faster than Terser.
    2. 13버전부터는 자동으로 적용되어있었다. (그니까 이미 쓰고 있었음)
    3. 85M, 33.4155s
    4. svgr/@webpack이 문제다. 10초정도 느리게 만든다.