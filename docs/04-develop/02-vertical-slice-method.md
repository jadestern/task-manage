# 수직 분할 방식 (Vertical Slice Method)

PRD에서 정의된 걸킨 시나리오를 개발 가능한 단위로 분할하여 점진적으로 구현하는 방법론입니다.

## 1. 수직 분할 방식의 정의

수직 분할 방식은 기능을 기술적 계층(프론트엔드, 백엔드, 데이터베이스)별로 나누지 않고, **비즈니스 가치를 제공하는 완전한 기능 단위**로 나누어 개발하는 방법입니다.

### 전통적인 수평 분할 vs 수직 분할
```
❌ 수평 분할 (기술 계층별)
Phase 1: 모든 데이터베이스 모델 설계
Phase 2: 모든 API 엔드포인트 개발  
Phase 3: 모든 UI 컴포넌트 개발

✅ 수직 분할 (기능별 완전 구현)
Phase 1: 로그인 기능 (DB → API → UI 완전 구현)
Phase 2: 할 일 추가 기능 (DB → API → UI 완전 구현)
Phase 3: 할 일 완료 기능 (DB → API → UI 완전 구현)
```

## 2. 걸킨 시나리오를 수직 분할로 변환하기

PRD의 각 걸킨 시나리오를 개발 가능한 수직 분할 단위로 변환합니다.

### 예시: 할 일 추가 기능

**PRD의 걸킨 시나리오:**
```gherkin
Scenario 1: 새로운 할 일 성공적으로 추가
Given 사용자가 TODO 앱에 로그인하여 '오늘 할 일' 화면을 보고 있을 때,
When 사용자가 할 일 내용을 입력하고 '할 일 추가' 버튼을 클릭하면,
Then 새로운 할 일이 '오늘 할 일' 목록의 가장 상단에 1초 이내에 표시되어야 한다.
```

**수직 분할 개발 단계:**
```
Phase 1-1: 기본 할 일 추가 기능 (최소 실행 가능 구현)
- DB: Todo 테이블 생성 (id, content, created_at만)
- API: POST /todos 엔드포인트 구현
- UI: 입력 필드 + 추가 버튼 + 목록 표시

Phase 1-2: 성능 및 UX 개선
- 1초 이내 표시 성능 최적화
- 상단 정렬 로직 구현
- 로딩 상태 UI 추가
```

## 3. 개발 워크플로우

### 3.1 단계별 구현 절차

각 수직 분할 단위에 대해 다음 순서로 진행합니다:

1. **Given 조건 구현** (선행 조건 확인)
2. **When 액션 구현** (사용자 행동 처리)
3. **Then 결과 구현** (기대 결과 출력)
4. **인수 기준 검증** (SMART 기준 확인)

### 3.2 구현 체크리스트

각 시나리오 구현 시 다음을 확인합니다:

```markdown
## Given 조건 구현 체크리스트
- [ ] 사용자 인증 상태 확인
- [ ] 필요한 데이터 사전 로딩
- [ ] UI 초기 상태 설정
- [ ] 권한 검증

## When 액션 구현 체크리스트  
- [ ] 사용자 입력 유효성 검사
- [ ] 이벤트 핸들러 연결
- [ ] API 호출 처리
- [ ] 에러 핸들링

## Then 결과 구현 체크리스트
- [ ] 데이터 저장/업데이트
- [ ] UI 상태 변경
- [ ] 성능 기준 만족 (예: 1초 이내)
- [ ] 사용자 피드백 제공
```

## 4. 실제 개발 예시

### 4.1 할 일 추가 기능 - Phase 1-1 구현

**1단계: Given 조건 구현**
```javascript
// 사용자 로그인 상태 확인
const checkUserAuthentication = () => {
  if (!user.isAuthenticated) {
    throw new Error('사용자가 로그인되어 있지 않습니다');
  }
};

// 할 일 목록 화면 초기화
const initializeTodoListScreen = () => {
  // 기존 할 일 목록 로드
  // UI 컴포넌트 렌더링
};
```

**2단계: When 액션 구현**
```javascript
// 할 일 내용 입력 처리
const handleTodoInput = (content) => {
  if (!content.trim()) {
    throw new Error('할 일 내용을 입력해주세요');
  }
  return content.trim();
};

// 할 일 추가 버튼 클릭 처리
const handleAddTodoClick = async (content) => {
  const todo = await createTodo({
    content: content,
    created_at: new Date().toISOString()
  });
  return todo;
};
```

**3단계: Then 결과 구현**
```javascript
// 새 할 일을 목록 상단에 추가
const addTodoToList = (todo) => {
  const todoList = document.getElementById('todo-list');
  const newTodoElement = createTodoElement(todo);
  todoList.insertBefore(newTodoElement, todoList.firstChild);
};

// 1초 이내 표시 보장
const updateUIWithPerformanceCheck = (todo) => {
  const startTime = performance.now();
  
  addTodoToList(todo);
  
  const endTime = performance.now();
  if (endTime - startTime > 1000) {
    console.warn('UI 업데이트가 1초를 초과했습니다:', endTime - startTime);
  }
};
```

### 4.2 검증 및 테스트

**인수 기준 검증:**
```javascript
describe('할 일 추가 기능', () => {
  test('Given-When-Then 시나리오 1: 새로운 할 일 성공적으로 추가', async () => {
    // Given: 사용자가 로그인하여 할 일 화면에 있을 때
    await authenticateUser();
    const todoScreen = renderTodoListScreen();
    
    // When: 할 일 내용을 입력하고 추가 버튼을 클릭하면
    const todoContent = "프로젝트 PRD 작성";
    const inputField = todoScreen.getByTestId('todo-input');
    const addButton = todoScreen.getByTestId('add-button');
    
    fireEvent.change(inputField, { target: { value: todoContent } });
    const startTime = performance.now();
    fireEvent.click(addButton);
    
    // Then: 새로운 할 일이 상단에 1초 이내에 표시되어야 한다
    await waitFor(() => {
      const todoList = todoScreen.getByTestId('todo-list');
      const firstTodo = todoList.firstChild;
      expect(firstTodo).toHaveTextContent(todoContent);
    });
    
    const endTime = performance.now();
    expect(endTime - startTime).toBeLessThan(1000); // SMART - Measurable
  });
});
```

## 5. 다음 수직 분할로 진행하기

첫 번째 수직 분할이 완료되면:

1. **현재 구현 검토**: 모든 인수 기준이 만족되는지 확인
2. **다음 시나리오 식별**: PRD의 다음 걸킨 시나리오 선택
3. **의존성 확인**: 이전 구현에 의존하는 부분 파악
4. **점진적 개선**: 기존 코드를 확장하여 새 기능 추가

### 예시: 할 일 완료 기능으로 확장

```
Phase 2: 할 일 완료 기능
- 기존 Todo 모델에 status 필드 추가
- 체크박스 UI 컴포넌트 추가  
- 완료 처리 API 엔드포인트
- 완료된 할 일 목록 분리 표시
```

## 6. 수직 분할 방식의 장점

1. **빠른 피드백**: 각 단계마다 동작하는 기능을 확인 가능
2. **위험 감소**: 작은 단위로 개발하여 문제 조기 발견
3. **명확한 진척도**: 완료된 기능 수로 진행 상황 측정
4. **사용자 가치 우선**: 기술적 완성도보다 비즈니스 가치 중심
5. **걸킨 시나리오와 직접 매핑**: PRD와 구현이 1:1 대응

## 7. 주의사항

- **과도한 세분화 금지**: 너무 작은 단위로 나누면 오히려 비효율적
- **기술 부채 관리**: 빠른 구현 후 반드시 리팩토링 계획 수립  
- **통합 테스트**: 각 수직 분할이 전체 시스템과 잘 통합되는지 확인
- **성능 모니터링**: SMART 기준의 측정 가능한 지표 지속 추적

이러한 수직 분할 방식을 통해 PRD의 걸킨 시나리오를 체계적이고 점진적으로 구현할 수 있습니다. 