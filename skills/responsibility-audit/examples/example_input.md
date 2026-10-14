# Example Input (Synthetic)

> **Illustrative Example / Synthetic Data.**
> The example below is a synthetic department + position responsibility
> excerpt. It does not represent any real organization.

The audit target is one department: **采购管理部** (Procurement Management
Department). It contains two positions: 采购经理 (Procurement Manager) and
采购专员 (Procurement Specialist).

---

## A. 部门职责 (Department Responsibility)

> Source: `《示例公司部门职责及岗位设置管理规定》（Q/EX-ORG-2024-007）》

### 5.2.9 采购管理部

- 5.2.9.2.1 负责采购相关制度修订。
- 5.2.9.2.2 负责采购计划管理。
- 5.2.9.2.3 负责供应商管理。
- 5.2.9.2.4 负责采购合同管理。
- 5.2.9.2.5 负责采购数据分析。
- 5.2.9.2.6 完成上级交办的其他相关工作。

---

## B. 岗位职责 (Position Responsibility)

> Source: `《示例公司岗位职责清单》（Q/EX-ORG-2024-008）》

### 5.2.9.x 采购管理部 — 采购经理

- 主要职责：
  - ① 负责采购计划管理。
  - ② 负责供应商管理。
- 工作内容：
  - ① 编制年度采购计划。
  - ② 维护供应商库，组织年度评价。

### 5.2.9.x 采购管理部 — 采购专员

- 主要职责：
  - ① 抓好采购合同管理。
  - ② 负责供应商管理。
- 工作内容：
  - ① 起草采购合同文本。
  - ② 整理供应商资质材料。

---

## C. 高管岗位（仅执行 Step 2）

> Source: `《示例公司岗位职责清单》（Q/EX-ORG-2024-008）》

### 高管 — 副总经理（采购分管）

- 主要职责：
  - ① 协助总经理分管采购管理工作。
  - ② 负责审定采购管理制度。
- 工作内容：
  - ① 审核年度采购计划。
  - ② 协调跨部门采购争议。

---

## How to run this example

1. Load the Skill:

   ```
   Load skills/responsibility-audit/SKILL.md
   ```

2. Provide the tables above as input.

3. The Agent will:

   - perform Step 1 on 采购管理部 (department responsibility review),
   - pause for human confirmation,
   - perform Step 2 on 采购管理部's two positions (and optionally
     副总经理),
   - pause for human confirmation,
   - perform Step 3 (bidirectional cross-check),
   - pause for human confirmation,
   - move on to the next department.

The expected output is in `example_output.md`.
