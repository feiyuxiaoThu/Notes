# `Planning` and `Monte Carlo Tree Search`

---

**`"Monte Carlo Tree Search with Reinforcement Learning for Motion Planning"`**

- **[** `IEEE ITSC 2020` **]**
**[[:memo:](https://ieeexplore.ieee.org/abstract/document/9294697)]**
**[** :mortar_board: `Philippe Weingertner`**]**

- **[** _`MCTS`, `RL` **]**

<details>
  <summary>Click to expand</summary>
![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/1-mcts-nnet-1.gif) 

Authors: Vermaelen, J., Dinh, H. T., & Holvoet, T.

- Motivations:
  - A review.
  - `1-` Beside the **main objective** of traditional `(PO)MDP`, i.e. maximizing the `expected return`, try to **enforce [probabilistic] `safety guarantees`**.
  - `2-` Give **illustrative examples** for each concept.
    - Here the authors consider the deployment of a **drone** for inspection tasks on industrial sites.

- Note:
  - As I understand, an **explicit model** of the **environment** is required.
  - Hence **no model-free `RL`** seems directly applicable.

- **Robust** `(PO)MDP`.
  - Ideas:
    - The **`transition` model** is not perfect and **may not be known exactly**.
    - The **derived `policy`** and the `expected return` can be **very _sensitive_** to **small changes** in the `transition` probabilities.
    - > "An **uncertainty set `τ a,s`** can be used to denote the **uncertain `transitions`**, rather than using known, fixed probabilities. Furthermore, probabilistic information regarding the **unknown parameters** can be considered, for example, using **`confidence regions`**. The resulting policy has to attain the **highest _worst-case_ performance over that `confidence region`**."
  - Example:
    - The **`transition` model** of the `MDP` says that the drone’s **`Lift` action** has a **known and fixed** probability of `0.8` to succeed and `0.2` to fail.
    - These **probabilities** were estimated from **data gathered over previous flights**.
    - These parameters become **representative** after sufficient flights, but some level of **uncertainty remains**.
    - A `Robust (PO)MDP` takes the **confidence in the transition model**, here the gathered flight data, into account.
  - It **does not achieve explicit safety guarantees**.

- **Constrained**: `C-(PO)MDP`
  - Idea:
    - **Constraints** on **secondary objectives**.
    - > "Apart from the traditional `cumulative reward` that is to be **maximized**, **`n` other `objectives`** are expressed, related to the **`n` respective `costs`**. On these **secondary objectives**, **`constraints` are applicable**."
    - > "Such `constraints` can **limit expected `costs`** or utilization of certain **resources** to a maximum threshold."
  - Example `1`:
    - Each `action` has a specific **power consumption**.
    - Secondary objective: the **total power consumption**.
    - Constraint: limit the **expected power usage** to a threshold smaller than the **capacity of the battery**.
  - Example `2`:
    - Each `action` requires some **time to execute**.
    - Secondary objective: **total duration**.
    - Constraint: limit the **expected duration** of the agent’s total execution.
  - > "It turns the problem into a **`constrained` optimization problem**. The resulting `policy` is still a `policy` that maximizes the overall reward, **yet _obeys_** [_well, no hard constraints!_] **the posed `constraints`.**"
  - Limitations:
    - Simply assigning a high cost to `states` that correspond with a **low battery level**, **provides no guarantees**.
    - The planner simply **tries to avoid these `states`**.
    - Lowering the penalty does not solve the problem either: the planner can switch from **being too conservative** to **too risky**.

- **Chance Constrained**: `CC-(PO)MDP`
  - Why _"chance"_?
    - **`probabilistic` constraints** = **`chance` constraints**.
    - The objective is still to maximize the **expected cumulative reward**.
    - But the **probability (“`chance`”)** of **violating safety constraints** is **bounded**.
  - Example:
    - `p`(`collision`) should stay below `0.01`.
    - > "We can demand that the **expected probability** of the `UAV` showing such **unsafe behavior** remains below some arbitrarily **low threshold**."
  - Limitation:
    - > "It is important to remark that an **accurate model** of the environment of the `UAV` and the effects of its `actions` is required to **obtain precise guarantees**."
    - In particular, the `transition` probabilities might not be straightforward to **estimate** but **have to be known precisely**.
  - > "[It has been observed](https://www.ijcai.org/Proceedings/2019/775) that the **`C-MDP` is more general than the `CC-MDP`**. Any `CC-MDP` problem can be reduced to a `C-MDP` problem."

- **Path Constrained**: `PC-(PO)MDP`
  - Motivation: combine a **probabilistic model checking** aspect with a **planning approach**.
  - Idea:
    - Constraints on **consecutive `states`**.
    - Using Probabilistic Computation Tree Logic (`PCTL`) for instance.
  - Example:
    - In each possible **`path`**, **no landing procedure** should be initiated before all environment checks have been run successfully.
    - This **_temporal_ logic formula** must hold with a **probability** of at least `0.99`.

- **Safe-Reachability Objectives**: `SRO-(PO)MDP`
  - Comparison with `C-(PO)MDPs` and `CC-(PO)MDPs`:
    - Their `policy` maximize the agent’s expected `cumulative reward` while **bounding an expected `cost` or `risk`**.
    - They typically hold large `reward` values for `goal states`, to attract the agent, but there is no **hard guarantees** to reach the `goal state`.
  - Motivation: satisfy a `safe-reachability objective` in **all possible execution paths**, that is, including **worst-case scenarios**.
  - Idea: enforce a **double guarantee** regarding:
    - `1` the **behaviour** of the `UAV`: the probability of **visiting an `unsafe` state** is always kept **below some threshold**.
    - `2` its **mission**: a **`goal state`** is **eventually reached** with a **probability above a certain threshold**.
  - Example:
    - The drone **reaches its destination** with a **minimum probability**, while the **probability of crashing is bounded**.

- **Risk Sensitive**: `RS-(PO)MDP`
  - Comparison: traditional `(PO)MDPs` are **minimizing the `cost` itself**.
  - Idea:
    - `risk` = "_possibility_ of something bad happening".
    - The policy should **maximize the `probability`** of the **cumulative `cost`** being below a certain predefined threshold.
  - Example:
    - Maximize the **probability** of **_"`p`(`collision`) stays below `0.01`"_**.

</details>

---
