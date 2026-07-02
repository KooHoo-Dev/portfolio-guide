# 관통 예제 스펙: 크래프팅 시스템 (CraftingSystem)

> 이 문서는 가이드 Part 4(Mermaid), Part 5(AI 워크플로우), Part 6(프롬프트)의 **공통 소재**로 사용된다.
> 4종 다이어그램(클래스 / 시퀀스 / 상태 / 플로우차트)이 모두 자연스럽게 도출되도록 설계했다.

---

## 1. 도메인 개요

생존/제작 게임의 **아이템 크래프팅 시스템**.
플레이어가 인벤토리의 재료를 소비하여 레시피에 정의된 결과물을 제작한다.

- 예: `나무 x3 + 돌 x2 → 돌도끼 x1`
- **확률 기반 성공**: 레시피마다 성공 확률(`successRate`)이 있음
- **실패 페널티**: 실패 시 재료의 일부(50%)만 소비 (완전 손실 아님)
- **이벤트 기반**: 제작 결과는 이벤트로 외부(UI, 사운드, 통계)에 전파

이 세 요소 덕분에 "단순 if 하나"가 아니라 **검증 → 확률 판정 → 분기 → 상태 전이**가 생겨
클래스/시퀀스/FSM/플로우차트가 모두 의미를 가진다.

---

## 2. 핵심 클래스 구성

| 클래스 / 인터페이스 | 역할 | 비고 |
|---|---|---|
| `CraftingManager` | 제작 총괄. 요청을 받아 검증·판정·결과 전파 | 이벤트 발행자 |
| `ICraftingService` | 제작 서비스 계약(인터페이스) | DIP, 테스트/교체 용이 |
| `Recipe` | 레시피 데이터 (필요 재료, 결과물, 성공 확률) | 데이터 홀더 |
| `Inventory` | 재료 보관·소비·추가 | LINQ로 재료 조회 |
| `Item` | 아이템 정의 (id, name) | 값 객체 성격 |
| `ItemStack` | 아이템 + 수량 묶음 | Recipe/Inventory가 사용 |
| `CraftingUI` | 사용자 입력 → 제작 요청, 결과 표시 | 이벤트 구독자 |
| `CraftingResult` | 제작 결과 (성공 여부, 결과물, 사유) | 이벤트 페이로드 |

### 관계 요약 (클래스 다이어그램용)
- `CraftingManager` **implements** `ICraftingService`
- `CraftingManager` **uses** `Inventory` (재료 검증/소비)
- `CraftingManager` **holds** `Recipe` 목록
- `CraftingUI` **depends on** `ICraftingService` (인터페이스에 의존, 구현체 아님 → DIP)
- `Recipe` **composed of** `ItemStack` (필요 재료 목록 + 결과물)
- `ItemStack` **references** `Item`
- `Inventory` **aggregates** `ItemStack`
- `CraftingManager` **publishes** `OnCraftCompleted(CraftingResult)` 이벤트 → `CraftingUI`가 **subscribes**

---

## 3. 상태 흐름 (FSM용)

`CraftingManager`의 내부 제작 상태:

```
Idle ──(RequestCraft)──▶ Validating
Validating ──(재료 부족)──▶ Failed
Validating ──(재료 충분)──▶ Crafting
Crafting ──(확률 성공)──▶ Succeeded
Crafting ──(확률 실패)──▶ Failed
Succeeded ──(결과 전파 완료)──▶ Idle
Failed ──(결과 전파 완료)──▶ Idle
```

- 진입 상태: `Idle`
- 종료 후 항상 `Idle`로 복귀 (반복 제작 가능)

---

## 4. 처리 시퀀스 (시퀀스 다이어그램용)

성공 케이스 기준 상호작용 순서:

1. `Player` → `CraftingUI` : 제작 버튼 클릭 (recipeId 전달)
2. `CraftingUI` → `CraftingManager` : `RequestCraft(recipeId)`
3. `CraftingManager` → `Inventory` : `HasIngredients(recipe)` 재료 검증
4. `Inventory` → `CraftingManager` : `true` 반환
5. `CraftingManager` → `CraftingManager` : 확률 판정 (`Roll < successRate`)
6. `CraftingManager` → `Inventory` : `Consume(recipe.Ingredients)` 재료 소비
7. `CraftingManager` → `Inventory` : `Add(recipe.Result)` 결과물 추가
8. `CraftingManager` -->> `CraftingUI` : `OnCraftCompleted(result)` 이벤트 발행
9. `CraftingUI` → `Player` : 성공 연출/토스트 표시

(실패 케이스는 5에서 확률 실패 → 재료 50% 소비 → 실패 이벤트)

---

## 5. 분기 로직 (플로우차트용)

`RequestCraft` 내부 의사결정 흐름:

```
[제작 요청]
   │
   ▼
[레시피 존재?] ──No──▶ [실패: 레시피 없음] ──▶ [Idle 복귀]
   │Yes
   ▼
[재료 충분?] ──No──▶ [실패: 재료 부족] ──▶ [Idle 복귀]
   │Yes
   ▼
[확률 판정: Roll < successRate?]
   │
   ├─Yes──▶ [재료 100% 소비] ──▶ [결과물 추가] ──▶ [성공 이벤트] ──▶ [Idle 복귀]
   │
   └─No───▶ [재료 50% 소비] ─────────────────────▶ [실패 이벤트] ──▶ [Idle 복귀]
```

---

## 6. 핵심 코드 스니펫 (Part 2·4·5에서 재사용)

> Jay 선호 반영: 네임스페이스 명시, PascalCase/camelCase, 표현식 본문 멤버,
> 이벤트 기반, LINQ, 인터페이스 의존(DIP).

```csharp
namespace Game.Crafting
{
    public interface ICraftingService
    {
        event Action<CraftingResult> OnCraftCompleted;
        void RequestCraft(string recipeId);
    }

    public sealed class CraftingManager : ICraftingService
    {
        private readonly Inventory inventory;
        private readonly IReadOnlyList<Recipe> recipes;
        private readonly Func<float> roll; // 0.0 ~ 1.0, 테스트 주입 가능

        public event Action<CraftingResult> OnCraftCompleted;

        public CraftingManager(Inventory inventory, IReadOnlyList<Recipe> recipes, Func<float> roll)
        {
            this.inventory = inventory;
            this.recipes = recipes;
            this.roll = roll;
        }

        // 표현식 본문 + LINQ로 레시피 조회
        private Recipe FindRecipe(string recipeId) =>
            recipes.FirstOrDefault(r => r.Id == recipeId);

        public void RequestCraft(string recipeId)
        {
            var recipe = FindRecipe(recipeId);
            if (recipe == null)
            {
                Publish(CraftingResult.Fail(recipeId, "레시피를 찾을 수 없습니다."));
                return;
            }

            // (Why) 소비 전에 먼저 검증 → 방어적 처리로 재료 유실 방지
            if (!inventory.HasIngredients(recipe.Ingredients))
            {
                Publish(CraftingResult.Fail(recipeId, "재료가 부족합니다."));
                return;
            }

            // (Why) 확률 판정을 주입된 roll로 분리 → 테스트 시 결정론적 검증 가능
            var success = roll() < recipe.SuccessRate;

            // (Why) 성공/실패에 따라 소비율을 달리해 '실패 페널티' 규칙을 표현
            var ratio = success ? 1.0f : 0.5f;
            inventory.Consume(recipe.Ingredients, ratio);

            if (success)
                inventory.Add(recipe.Result);

            Publish(success
                ? CraftingResult.Success(recipeId, recipe.Result)
                : CraftingResult.Fail(recipeId, "제작에 실패했습니다."));
        }

        // 이벤트 발행을 한 곳으로 모아 null 체크 중복 제거
        private void Publish(CraftingResult result) =>
            OnCraftCompleted?.Invoke(result);
    }
}
```

---

## 7. 다이어그램별 도출 매핑 (Part 4 실습용)

| Mermaid 유형 | 이 스펙에서 뽑는 대상 |
|---|---|
| classDiagram | 2절 클래스 관계 (인터페이스 구현·의존·구성) |
| sequenceDiagram | 4절 성공 시퀀스 (UI→Manager→Inventory→이벤트) |
| stateDiagram-v2 | 3절 제작 상태 전이 |
| flowchart | 5절 RequestCraft 분기 로직 |

---

## 8. 확정 스펙 요약 (변경 금지 기준선)

- 클래스: CraftingManager, ICraftingService, Recipe, Inventory, Item, ItemStack, CraftingUI, CraftingResult (8개)
- 규칙: 확률 성공 / 실패 시 재료 50% 소비 / 이벤트 기반 결과 전파
- 네임스페이스: `Game.Crafting`
- 예시 레시피: 나무 x3 + 돌 x2 → 돌도끼 x1 (successRate 0.8)
