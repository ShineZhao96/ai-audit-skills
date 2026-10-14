# Example Input (Synthetic)

> **Illustrative Example / Synthetic Data.**
> The example below is a synthetic corporate policy fragment. It does
> not represent any real organization, document, or regulatory text.

---

## 《示例企业采购管理制度》

**编号：** Q/EX-PROC-2024-001

**版本：** v1.0

**发布日期：** 2024 年 5 月 1 日

**生效日期：** 2024 年 7 月 1 日

---

### 第一章  总则

**第一条**  为规范本企业采购活动，依据国家相关法律法规和上级集团制度要求，结合本企业实际情况，制定本制度。

**第二条**  本制度适用于本企业及其下属各单位的采购活动。

**第三条**  采购活动应当遵循公开、公平、公正、诚实信用原则。

### 第二章  采购方式

**第四条**  采购分为公开招标、邀请招标、竞争性谈判、单一来源采购、询价采购等方式。

**第五条**  采购活动应遵守《示例采购管理办法》（示例文号），按照规定的采购程序组织实施。

**第六条**  采购人员应按照相关法律法规开展采购活动，并参照本企业内部采购工作指引执行。

### 第三章  供应商管理

**第七条**  本企业建立供应商库，对供应商进行动态评价管理。

**第八条**  供应商的准入、评价和退出应按照《供应商管理实施细则》执行。

### 附录：规范性引用文件

1. 《中华人民共和国招标投标法》
2. 《中华人民共和国政府采购法》
3. GB/T 19001-2008《质量管理体系 要求》
4. 《示例采购管理办法》（示例文号）
5. 《示例供应商管理细则》

---

## How to run this example

1. Load the Skill:

   ```
   Load skills/regulatory-reference-check/SKILL.md
   ```

2. Paste the policy text above as the input.

3. The Agent will:

   - extract the formal references (1)–(5) above,
   - detect implicit references in Articles 1, 5, 6, 8,
   - classify each reference into A / B / C / D / E / S,
   - run verification per class,
   - record candidate issues,
   - **pause** for human confirmation,
   - produce the final report after confirmation.

The expected output is in `example_output.md`.
