# OSS Week 12 — React Hooks Playground (Student Starter)

## 실행 방법
Node.js 20.19+ 또는 22.12+ 권장. 프로젝트 폴더에서 실행:

```bash
npm install
npm run dev
```

브라우저에 터미널의 Local URL을 열고 LAB 탭을 이동하세요. 제출 전 `npm run build`도 실행하세요.

## 수행 방법
1. `src/components/` 아래의 LAB 1~4 파일에 있는 `TODO`를 직접 완성합니다.
2. `src/App.jsx`, `src/App.css`는 공통 화면이므로 가급적 수정하지 않습니다.
3. 각 LAB의 동작 화면과 콘솔 결과를 기록합니다.
4. 한 LAB에서 다른 LAB으로 탭을 전환하면 기존 컴포넌트가 언마운트됩니다. Cleanup 결과를 관찰합니다.


## 이해도 확인 질문
1. setCount(prev => prev + 1)은 왜 유용한가?: 이전 state 값을 받아서 계산하기 때문에 처음 count 값인 0만 계속 참조하는 문제를 막아준다.
2. useEffect(fn, [])과 useEffect(fn)은 어떻게 다른가?: 빈 배열을 넣으면 처음 화면에 나타날 때 한 번만 실행되고, 배열을 생략하면 화면이 다시 그려질 때마다 실행된다.
3. Cleanup은 언제 실행되는가? 개발 환경 StrictMode 영향은?: 의존성 값이 바뀌어 Effect가 다시 실행되기 전과 컴포넌트가 화면에서 사라질 때 실행된다. 개발 환경의 StrictMode에서는 정리가 잘 되는지 확인하려고 처음에 Effect와 Cleanup이 한 번 더 실행될 수 있다.
4. useState와 useRef의 값 변경 시 렌더링 차이는?: useState는 값이 바뀌면 화면을 다시 그리고, useRef는 current 값을 바꿔도 화면을 다시 그리지 않는다.
5. LAB 2와 LAB 4에서 Timer를 관리하는 방식의 차이는?: LAB 2는 running 상태 변화에 따라 useEffect가 타이머를 켜고 끄고, LAB 4는 timerRef에 타이머 ID를 넣어두고 버튼 클릭 함수에서 직접 켜고 끈다.

## Project Structure: App.jsx와 주요 Lab 컴포넌트의 역할
- App.jsx: 전체 화면과 탭 전환을 담당한다. 선택한 LAB 컴포넌트만 보여주고 탭을 바꾸면 이전 LAB은 화면에서 사라진다.
- Lab1_EffectTimer.jsx: 처음 열리면 useEffect로 1초마다 count를 올리고 사라질 때 타이머를 정리한다.
- Lab2_TimerControl.jsx: running 상태와 useEffect를 연결해서 Start, Stop, Reset 기능을 처리한다.
- Lab3_InputFocus.jsx: useRef로 input 요소에 직접 접근해서 Focus와 Clear 버튼을 동작시킨다.
- Lab4_RefTimer.jsx: useRef에 타이머 ID를 저장해두고 Start, Stop, Reset 함수에서 직접 타이머를 제어한다.

## useEffect: Effect, Dependency Array 및 Cleanup의 동작 원리

화면이 그려진 뒤에 타이머 같은 작업을 실행할 때 사용한다. 두 번째 인자인 Dependency Array로 언제 다시 실행할지 정한다. 빈 배열이면 처음 한 번만, 배열이 없으면 매 렌더링마다, 값이 있으면 그 값이 바뀔 때마다 실행된다. return으로 넘긴 Cleanup 함수는 Effect가 다시 실행되기 직전이나 컴포넌트가 사라질 때 이전 타이머를 정리한다.

## useRef: .current 속성과 useState의 차이

화면을 다시 그려도 값이 유지되면서 .current로 값을 읽고 바꿀 수 있는 Hook이다. DOM 요소에 접근하거나 타이머 ID를 저장할 때 쓴다. useState는 값을 바꾸면 화면을 다시 그리지만, useRef는 .current를 바꿔도 화면을 다시 그리지 않는다.

## Code Analysis: useEffect와 useRef 관련 주요 코드 설명

### useEffect (Lab2_TimerControl.jsx)

running 값이 바뀔 때마다 Effect가 실행된다. running이 false면 타이머를 만들지 않고 끝낸다. true면 1초마다 count를 올리고, 상태가 바뀌거나 탭을 옮길 때 Cleanup에서 clearInterval로 타이머를 멈춘다.

```
useEffect(() => {
  if (!running) return;

  const id = setInterval(() => {
    setCount(prev => prev + 1);
  }, 1000);

  return () => clearInterval(id);
}, [running]);
```

### useRef (Lab4_RefTimer.jsx)

타이머 ID는 화면에 보일 필요가 없어서 timerRef.current에 저장한다. start에서는 이미 타이머가 돌고 있으면 바로 끝내서 중복 실행을 막는다. stop에서는 clearInterval로 타이머를 멈추고 current를 다시 null로 바꾼다.

```
const timerRef = useRef(null);

const start = () => {
  if (timerRef.current !== null) return;
  timerRef.current = setInterval(() => {
    setCount(prev => prev + 1);
  }, 1000);
  setRunning(true);
};

const stop = () => {
  clearInterval(timerRef.current);
  timerRef.current = null;
  setRunning(false);
};
```

## Lab 비교 분석

- useEffect의 Dependency Array가 필요한 이유는?: Effect가 매번 불필요하게 실행되거나 무한 반복되는 것을 막고, 원하는 시점에만 실행되도록 조건을 정하기 위해 필요하다.
- Cleanup을 사용하지 않으면 어떤 문제가 발생할 수 있는가?: 컴포넌트가 사라지거나 Effect가 다시 실행되어도 이전 타이머가 계속 남아서 돌아간다. 그래서 타이머가 중복 실행되거나 숫자가 더 빨리 올라가는 문제가 생긴다.
- useState와 useRef의 가장 중요한 차이는?: 값이 바뀔 때 화면을 다시 그리는지 여부다. useState는 화면을 다시 그리고, useRef는 다시 그리지 않는다.
- LAB 2와 LAB 4의 Timer 관리 방식은 어떻게 다른가?: LAB 2는 running 상태를 바꾸면 useEffect가 알아서 타이머를 만들고 정리한다. LAB 4는 timerRef에 타이머 ID를 저장해두고 start, stop 버튼 함수에서 직접 타이머를 제어하며, useEffect는 화면에서 사라질 때 정리하는 용도로만 쓴다.
