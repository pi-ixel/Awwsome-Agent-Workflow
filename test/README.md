# AAW Skills 自动化测试与测评体系

本目录存放面向 `module-asis-analysis` / `module-tobe-design` / `module-test-design` / `module-design-gate` 四个 skill 的测试用例。

## 目录组织原则

`test/` 下的子目录**与被测 skill 一一对应**(镜像 `skills/` 的命名)。每个被测 skill 有自己专属的测试空间,在该子目录内放 fixtures、prompt、断言。

一个例外是**横向契约检查**(lint):它天然是跨多个 skill 的(术语在 4 个 skill 间是否一致),无法对应单一 skill,因此保留 `test/lint/` 作为横向测试区,与各 skill 目录并列。

```
test/
├── lint/                              ← 横向:跨 skill 契约一致性检查(不对应单一 skill)
│   ├── README.md
│   └── TestPrompt_skill-contract-lint.md
└── module-design-gate/                ← 对应 skills/module-design-gate/
    ├── README.md
    ├── TestPrompt_gate-defect-detection.md
    └── fixtures/
        └── case-01-退款审计模块/
            ├── .sdd/software_architecture.md
            ├── *.context.md
            ├── *详细设计说明书.md
            ├── *测试用例设计.md
            └── expected-defects.md
```

## 设计哲学

这四个 skill 的 95% 内容是 **LLM 行为指令**(SKILL.md),不是确定性代码。因此本目录的"测试"本质是 **LLM eval(评测)**,不是传统单元测试。判据不是"输出等于期望字符串",而是:
- **lint**:产出/契约是否符合 skill 间的模板契约(确定性可断言)
- **gate**:在预置错误成果物上,LLM 能否抓出已植入的缺陷(行为正确性)

所有用例都以 prompt 文件形式给出,由人/CI 把 prompt 喂给运行中的 agent,再核对结果。

## 如何运行

### lint 用例
1. 打开 `lint/README.md`,执行 `lint/run.md` 中的 prompt。
2. LLM 产出 lint 报告,按 `lint/README.md` 的"通过标准"逐项核对(PASS/FAIL)。

### gate 用例
1. 把 `module-design-gate/fixtures/case-01-退款审计模块/` 复制到干净工作目录。
2. 按 `module-design-gate/README.md` 执行 `run.md` 中的 prompt(替换 `{WORKDIR}`)。
3. LLM 输出门禁结论(C17~C22),按 `expected-defects.md` 核对 5 个植入缺陷是否被抓出。

## 非确定性说明
LLM 行为有波动,建议:
- 同一 fixture 跑 3~5 次,**通过率 ≥ 4/5** 视为通过
- 记录每次 run 的输出到 `runs/`,便于回归对比
- 锁定 `model + temperature`,否则结果不可比

## 扩展指引
新增用例时遵循对应关系:
- 测 `module-asis-analysis` → 建 `test/module-asis-analysis/`,fixtures 放假 context.md
- 测 `module-tobe-design` → 建 `test/module-tobe-design/`,fixtures 放假说明书 + context
- 测 `module-test-design` → 建 `test/module-test-design/`,fixtures 放假测试设计文件
- 横向契约检查(跨多个 skill 的术语/路径/编号一致性)→ 放 `test/lint/`

## 局限性
- 本套测试**不验证** sr-design 的 MCP server(那部分用 `skills/sr-design/test_*.py` 的 pytest 覆盖)
- 本套测试**不覆盖**端到端流水线(需完整 repo + SR + AR fixture)
- gate 用例的 expected-defects 是人工标注的"应被抓出的问题",本身也是评测的一部分,随 skill 演进需更新
