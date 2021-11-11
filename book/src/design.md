# 设计

## 语言方面的注解

我们用和其他语言稍微不同的语言描述PLONK的相关概念。总体上：

1. 我们把PLONK相关的证明看成表，每一列对应一根“线”。表中元素我们称为“单元格”。
2. 我们喜欢说“选择子多项式”和“固定列”。当一个在固定列的单元格用来控制一个行中的约束是否开启的时候，我们称“选择子约束“。
3. 其他多项式，我们称为”advice列“。这些列由证明者提供。
4. 我们用”规则“字眼表述一个”门“，比如
   $$A(X) \cdot q_A(X) + B(X) \cdot q_B(X) + A(X) \cdot B(X) \cdot q_M(X) + C(X) \cdot q_C(X) = 0.$$
   - TODO: Check how consistent we are with this, and update the code and docs to match.
