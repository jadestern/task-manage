# 걸킨 구문 개발 통합 가이드

PRD의 Given-When-Then 시나리오를 실제 코드와 테스트로 직접 변환하는 구체적인 방법론을 제시합니다.

## 1. 걸킨 구문 개발 통합 원칙

### 1.1 직접 매핑 원칙
```
PRD 걸킨 시나리오 ↔ 코드 구조 ↔ 테스트 케이스
```

- **Given**: 초기 상태 설정 (setup, mock data, 사전 조건)
- **When**: 사용자 액션 처리 (event handler, API call, 비즈니스 로직)  
- **Then**: 결과 검증 (assertion, UI update, 상태 변경)

### 1.2 추적 가능성 확보
- 모든 코드는 특정 걸킨 시나리오와 연결
- 코드 변경 시 해당하는 시나리오 및 테스트 자동 업데이트
- 시나리오별 구현 상태 추적 대시보드

## 2. 걸킨 시나리오를 코드로 변환

### 2.1 단계별 변환 프로세스

**1단계: 시나리오 분석**
```gherkin
Scenario: 할 일 완료 처리
Given 사용자가 '오늘 할 일' 목록에 "PRD 작성"이라는 할 일을 보고 있을 때
When 사용자가 "PRD 작성" 할 일 옆의 '완료' 체크박스를 클릭하면
Then "PRD 작성" 할 일은 '오늘 할 일' 목록에서 사라지고 '완료된 할 일' 목록으로 이동되어야 한다
```

**2단계: 코드 구조 설계**
```javascript
// Given: 초기 상태 설정
class TodoListScreen {
  setupInitialState() {
    this.todayTodos = [
      { id: 1, content: "PRD 작성", status: "pending" }
    ];
    this.completedTodos = [];
  }
}

// When: 사용자 액션 처리  
class TodoActions {
  toggleTodoComplete(todoId) {
    const todo = this.findTodoById(todoId);
    todo.status = todo.status === "pending" ? "completed" : "pending";
    this.moveTodoToAppropriateList(todo);
  }
}

// Then: 결과 검증
class TodoResultValidator {
  verifyTodoMovement(todoContent) {
    const todayList = this.getTodayTodos();
    const completedList = this.getCompletedTodos();
    
    expect(todayList.find(t => t.content === todoContent)).toBeUndefined();
    expect(completedList.find(t => t.content === todoContent)).toBeDefined();
  }
}
```

### 2.2 인수 기준을 코드 검증으로 변환

**PRD 인수 기준:**
```
인수 기준: 할 일이 즉시 이동되며, 사용자는 '완료된 할 일' 목록에서 해당 항목을 확인할 수 있어야 한다.
```

**검증 코드:**
```javascript
const verifyTodoMovement = async (todoContent) => {
  // 즉시 이동 확인 (성능 기준)
  const startTime = performance.now();
  
  await toggleTodoComplete(todoContent);
  
  const endTime = performance.now();
  expect(endTime - startTime).toBeLessThan(100); // 즉시 이동 (100ms 이내)
  
  // 완료된 할 일 목록에서 확인 가능
  const completedTodos = getCompletedTodos();
  const movedTodo = completedTodos.find(todo => todo.content === todoContent);
  
  expect(movedTodo).toBeDefined();
  expect(movedTodo.status).toBe("completed");
  expect(movedTodo.completedAt).toBeDefined();
};
```

## 3. 자동화된 테스트 생성

### 3.1 걸킨 시나리오 → 테스트 케이스 변환

**템플릿 기반 자동 생성:**
```javascript
// 걸킨 시나리오를 받아서 테스트 코드 자동 생성
const generateTestFromGherkin = (scenario) => {
  return `
describe('${scenario.feature}', () => {
  test('${scenario.name}', async () => {
    // Given: ${scenario.given}
    ${generateGivenSetup(scenario.given)}
    
    // When: ${scenario.when}  
    ${generateWhenAction(scenario.when)}
    
    // Then: ${scenario.then}
    ${generateThenAssertion(scenario.then)}
    
    // 인수 기준 검증
    ${generateAcceptanceCriteriaTests(scenario.acceptanceCriteria)}
  });
});
`;
};
```

### 3.2 실제 테스트 코드 예시

```javascript
describe('할 일 완료 처리', () => {
  test('할 일 완료 처리 및 목록 이동', async () => {
    // Given: 사용자가 '오늘 할 일' 목록에 "PRD 작성" 할 일을 보고 있을 때
    const todoApp = renderTodoApp();
    const todoItem = await todoApp.findByText("PRD 작성");
    
    expect(todoApp.getTodayTodosList()).toContain(todoItem);
    expect(todoApp.getCompletedTodosList()).not.toContain(todoItem);
    
    // When: 사용자가 "PRD 작성" 할 일 옆의 '완료' 체크박스를 클릭하면
    const checkbox = todoApp.getCheckboxFor("PRD 작성");
    
    const startTime = performance.now();
    fireEvent.click(checkbox);
    await waitForStateUpdate();
    const endTime = performance.now();
    
    // Then: "PRD 작성" 할 일은 목록에서 이동되어야 한다
    expect(todoApp.getTodayTodosList()).not.toContain("PRD 작성");
    expect(todoApp.getCompletedTodosList()).toContain("PRD 작성");
    
    // 인수 기준: 즉시 이동 (성능)
    expect(endTime - startTime).toBeLessThan(100);
    
    // 인수 기준: 완료된 할 일 목록에서 확인 가능
    const completedTodo = todoApp.getCompletedTodo("PRD 작성");
    expect(completedTodo.status).toBe("completed");
    expect(completedTodo.completedAt).toBeDefined();
  });
});
```

## 4. AI 프롬프트 최적화

### 4.1 걸킨 기반 구현 요청

**효과적인 프롬프트 구조:**
```
🧑: 다음 걸킨 시나리오를 구현해줘:

[걸킨 시나리오 전체 붙여넣기]

구현 요구사항:
1. Given-When-Then 구조를 코드에 명확히 반영
2. 각 인수 기준을 검증할 수 있는 테스트 코드 포함
3. SMART 기준의 측정 가능한 부분을 수치로 검증
4. 우리의 아키텍처 가이드라인 준수

참고: 이 시나리오는 [기능명]의 [n번째] 시나리오입니다.
```

### 4.2 검증 요청

```
🧑: 방금 구현한 기능이 다음 걸킨 시나리오의 모든 인수 기준을 만족하는지 검토해줘:

[시나리오 다시 붙여넣기]

특히 다음 사항들을 확인해줘:
- Given 조건이 정확히 설정되었는지
- When 액션이 예상대로 동작하는지  
- Then 결과가 모든 인수 기준을 만족하는지
- 성능 기준 (예: 1초 이내)이 달성되는지
```

## 5. 개발 과정에서의 실시간 추적

### 5.1 시나리오별 구현 상태 관리

```markdown
## 할 일 관리 기능 구현 상태

### ✅ 완료된 시나리오
- [x] Scenario 1: 새로운 할 일 성공적으로 추가
  - 파일: `src/features/todo-add/TodoAddForm.tsx`
  - 테스트: `tests/todo-add.test.js`
  - 커밋: `feat(todo): implement todo addition scenario`

### 🚧 진행 중인 시나리오  
- [ ] Scenario 2: 할 일 완료 처리 및 목록 이동
  - 진행률: 60% (Given, When 완료, Then 진행 중)
  - 예상 완료: 오늘 오후

### 📋 대기 중인 시나리오
- [ ] Scenario 3: 할 일 내용 미입력 시 추가 실패
- [ ] Scenario 4: 할 일 수정 기능
```

### 5.2 자동화된 진척도 추적

```javascript
// 시나리오 구현 상태 자동 추적
const trackScenarioProgress = () => {
  const scenarios = parseGherkinFromPRD();
  const implementedFeatures = scanCodebaseForFeatures();
  const testCoverage = analyzeTestCoverage();
  
  return scenarios.map(scenario => ({
    id: scenario.id,
    name: scenario.name,
    status: getImplementationStatus(scenario, implementedFeatures),
    testCoverage: getScenarioTestCoverage(scenario, testCoverage),
    lastUpdated: getLastCommitDate(scenario)
  }));
};
```

## 6. 품질 보증 및 검증

### 6.1 자동화된 품질 검사

```javascript
// 걸킨 시나리오 준수 검증
const validateGherkinCompliance = (codeFile, scenario) => {
  const checks = [
    // Given 조건 검증
    hasProperSetup(codeFile, scenario.given),
    
    // When 액션 검증  
    hasUserActionHandling(codeFile, scenario.when),
    
    // Then 결과 검증
    hasResultVerification(codeFile, scenario.then),
    
    // 인수 기준 검증
    hasAcceptanceCriteriaTests(codeFile, scenario.acceptanceCriteria)
  ];
  
  return {
    compliant: checks.every(check => check.passed),
    issues: checks.filter(check => !check.passed)
  };
};
```

### 6.2 지속적인 검증 프로세스

```bash
# Git pre-commit hook
#!/bin/bash
echo "걸킨 시나리오 준수 검증 중..."

# 변경된 파일의 걸킨 준수 검사
npm run validate-gherkin-compliance

# 관련 테스트 실행
npm run test:gherkin-scenarios

# 성능 기준 검증  
npm run test:performance-criteria

echo "모든 걸킨 기준을 만족합니다!"
```

## 7. 실무 적용 팁

### 7.1 효율적인 시나리오 관리

- **시나리오 ID 부여**: 각 시나리오에 고유 ID로 추적성 확보
- **의존성 매핑**: 시나리오 간 의존관계 명확히 정의
- **우선순위 설정**: 비즈니스 가치와 기술적 복잡도 고려

### 7.2 팀 협업 최적화

- **공통 용어집**: 걸킨 용어의 팀 내 표준 정의
- **리뷰 체크리스트**: 걸킨 준수 여부 코드 리뷰 항목 포함
- **지식 공유**: 시나리오별 구현 패턴 팀 내 공유

이러한 걸킨 통합 방법론을 통해 PRD의 요구사항을 정확하고 추적 가능한 코드로 변환할 수 있습니다. 