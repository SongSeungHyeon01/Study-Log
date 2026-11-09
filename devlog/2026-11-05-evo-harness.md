# 2026-11-05 논문 읽기: EVO-HARNESS

## 논문 정보
| 항목 | 내용 |
|---|---|
| 제목 | EVO-HARNESS: Context-to-Harness Skill Compilation for Self-Evolving Agents |
| 저자 | Tianxin Wei 외 (UIUC, Amazon, Penn State, Emory, AG2 AI, Northeastern) |
| 게재처 | EMNLP 2026 Main, pp. 22343–22358 |
| 링크 | [ACL Anthology 2026.emnlp-main.969](https://aclanthology.org/2026.emnlp-main.969.pdf) |
| 학회 기간 | 2026-10-24 ~ 10-29 |
| 읽은 날 | 2026-11-05 |

## 한 줄 요약
모델은 고정해 두고, 과제를 하나 풀 때마다 실패 경험을 다시 쓸 수 있는 스킬로 정리해서 하네스를 계속 업데이트한다. 기존 메모리·경험 재사용 방법보다 벤치마크 5개 모두에서 나았다.

## 문제 제기
- 기존 경험 학습 방법(reflection, memory, skill)은 궤적을 잔뜩 모아 둔 뒤 나중에 패턴을 캐는 오프라인 방식이다.
- 실제로는 새 과제가 계속 들어오고, 과제마다 배울 기회가 한 번뿐이다. 실행 기록에는 쓸모 있는 교훈과 그 과제에만 해당하는 잡음이 섞여 있다.
- 그래서 과거 사례를 그대로 꺼내 쓰는 것(retrieve)으로는 부족하고, 경험을 다른 과제에도 쓸 수 있는 지침으로 가공(compile)해야 한다고 본다.

## 방법: 온라인 하네스 학습
과제 묶음이 순서대로 들어오는 흐름에서 다음을 반복한다.

1. **Select / Inject:** 지금 하네스에서 이 과제에 맞는 지침을 최대 b개 골라 프롬프트에 넣는다.
2. **실행:** 고정된 solver가 과제를 푼다. 입력, 행동 궤적, 결과, 피드백(테스트 결과, 에러 등)이 남는다.
3. **Execution reflection:** 실패했거나 부정적 피드백을 받은 경우에만 후보 메모리 `(lesson, trigger, evidence, scope_hint)`를 만든다. 성공한 실행은 과제 고유의 잡음이 섞이기 쉬워서 쓰지 않는다.
4. **Harness evolution:** evolver가 후보 메모리와 지금 하네스를 비교해서 ADD / MERGE / REVISE / SKIP 중 하나로 편집한다. 그대로 쌓지 않고 걸러서 합친다.

하네스는 두 층으로 나뉜다.
- General 스킬: 여러 과제에 통하는 패턴
- Topic 스킬: 특정 과제 유형이나 도메인의 절차

## 실험
- 벤치마크: TerminalBench-2(커맨드라인), SWE-bench Lite(코드 수정), CL-Bench(적응형 추론), τ-bench(도구 사용), WebArena-Infinity(웹)
- 모델: 기본은 Claude Opus 4.6이 solver와 evolver를 모두 맡음. 모델 비교에 Opus 4.7, Opus 4.5, Kimi-K2.5, GPT-OSS
- 비교 대상: AWM, Dynamic Cheatsheet, Evo-Memory, ACE, XSkill

### 전체 성능 (Opus 4.6, 성공률 %)
| | CL-Bench | TerminalBench-2 | SWE-bench Lite | τ-bench | WebArena-Inf |
|---|---|---|---|---|---|
| No Evolve | 29.5 | 62.9 | 63.7 | 72.7 | 72.5 |
| XSkill (가장 강한 비교 대상) | 31.4 | 66.3 | 64.7 | 73.9 | 73.8 |
| EVO-HARNESS | 34.0 | 73.0 | 67.0 | 77.0 | 76.3 |

- TerminalBench-2에서 +10.1로 가장 컸다. 파일 확인, 명령 실행, 에러 해석, 복구처럼 반복되는 절차가 있는 과제일수록 이득이 크다.
- 기존 방법 중 일부는 No Evolve보다 낮았다. 외부 경험을 재사용한다고 자동으로 좋아지지는 않는다.

### 모델과 과제 유형에 따른 차이 (CL-Bench)
- Opus 계열은 +3.7~4.5, Kimi-K2.5는 +1.1, GPT-OSS는 +0.8. 강한 모델일수록 하네스 지침을 잘 활용한다.
- 절차형 과제(PTE)는 최대 +9.8로 크게 올랐다. 탐색형 과제(EDS)는 오히려 떨어진 모델도 있었다. 지나치게 구체적인 지침이 탐색을 방해할 수 있다.

### 하네스에 무엇이 쌓였나
- 벤치마크마다 달랐다. WebArena는 워크플로, TerminalBench는 검증과 복구, τ-bench는 API 사용과 도메인 추론, CL-Bench는 출력 제약 처리가 많았다.
- 어블레이션: CL-Bench는 Topic만 써도 33.7로 거의 같았고, SWE-bench는 General만 써도 66.7로 거의 같았다. 과제 성격에 따라 필요한 층이 다르고, 둘 다 쓸 때가 가장 좋았다.

### 전이와 온라인 업데이트 (SWE-bench Lite)
- No-Evolve 68.8, Train-Split Transfer 73.4, Online-Updated 75.0
  - Train-Split Transfer: Sonnet 4.5가 train split에서 만든 스킬을 Opus 4.7이 test에서 사용. 다른 모델이 다른 과제에서 만든 스킬도 쓸모가 있었다.
  - Online-Updated: test 과제 흐름 안에서 바로 업데이트. 가장 높았다.
- solver와 evolver 조합
  - Opus가 solver일 때 Sonnet이 만든 스킬(76.0)이 Opus 자신이 만든 스킬(75.3)보다 조금 나았다.
  - Sonnet이 solver일 때는 어떤 evolver를 써도 No-Evolve(58.0)보다 낮았다(55.3, 55.7). 지침을 소화할 능력이 없는 모델에게는 오히려 해가 된다.

### 피드백 종류
| 피드백 | CL-Bench | SWE-bench Lite |
|---|---|---|
| No Evolve | 29.5 | 63.7 |
| Self-Generated (모델이 스스로 판단) | 28.0 | 61.7 |
| Minimal (성공/실패만) | 29.9 | 67.3 |
| Standard (에러 메시지, 테스트 출력) | 34.0 | 67.0 |

- 외부 근거 없이 모델이 스스로 판단한 피드백은 성능을 떨어뜨렸다.
- 자세한 피드백이 늘 좋은 것도 아니다. SWE-bench에서는 성공/실패만 준 쪽이 조금 더 높았다. 에러를 자세히 주면 스킬이 그 과제에 과하게 맞춰질 수 있다는 해석.

## 한계 (저자 언급)
- 단일 에이전트, 텍스트 기반 환경만 다뤘다. 물리 환경 에이전트나 멀티에이전트는 다루지 않았다.
- 하네스는 자연어 지침 형태만 다뤘고 실행 가능한 코드 스킬은 다루지 않았다.
- 사람이 직접 고친 피드백은 평가하지 않았다.

## 생각해 볼 점
### [Wang et al. 평가 비판 논문](2026-10-27-harness-evolution.md)의 기준으로 보면
맞는 부분
- 과제마다 한 번만 풀고 그 피드백은 이후 과제에만 쓰는 온라인 구조라서, 같은 과제를 여러 번 다시 푸는 탐색량 효과와는 성격이 다르다.
- Train-Split Transfer 실험(73.4)은 처음 보는 과제로의 전이를 어느 정도 보여 준다.
- 모델의 자기 판단 피드백이 오히려 해가 된다는 결과는 그 논문과 같다.

빠진 부분
- 비교 대상이 모두 경험·메모리 기반 방법이다. 같은 예산의 test-time scaling(병렬 샘플링, 순차 개선)과는 비교하지 않았고, evolver 호출 비용도 따로 맞추지 않았다.
- 반복 실행이나 분산 보고가 없다. TerminalBench-2는 89문제라 +10점은 약 9문제 차이다.
- Online-Updated의 최고 점수는 평가하는 test 흐름 자체로 하네스를 업데이트한 결과다.

### 정리
절차가 반복되는 과제에서, 외부 피드백이 있고, 충분히 강한 모델이라면 실패를 스킬로 정리하는 방식이 효과가 있다는 조건부 근거로 읽는 게 맞아 보인다. 이후에 읽은 [HarnessCompass](2026-11-09-harness-compass.md)도 피드백을 외부 근거로 검증한다는 점이 같다.
