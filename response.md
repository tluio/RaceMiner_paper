We sincerely thank all the reviewers for insightful comments, and hope our response can help solve main concerns about our paper.

## Review A
### A1: Difference between existing static lockset analysis and LR-Miner
In Section 1, we have analyzed the limitations of existing static lockset analyses (L1--L3 in Page 1), to motivate solution techniques in LR-Miner. Thus, we believe the difference can be found here. To emphasize the difference, we will add a separate paragraph in Section 7.

### A2: Handling field access
In LLVM bytecode, both `.` and `->` are represented as the getelementptr instruction, which is handled by LR-Miner. Thus, we believe that LR-Miner handles all possible ways of field access in source code. We will explain this in Section 4.

### A3: Dropping false positives
We have introduced the related techniques in "False positive dropping" paragraph in Page 9. First, LR-Miner uses Z3 [79] to validate code-path feasibility, by translating LLVM instructions into SMT constraints and computing their satisfiability. Second, for concurrency, LR-Miner drops races in functions whose names have keywords like 'init' and 'remove', because they are expected to have no concurrency during execution. We implemented these techniques by referring to existing approaches like DCUAF [6] and SADA [8]. In the evaluation, 8.9K and 2.2K false races were dropped by these techniques, respectively. We will provide more implementation and evaluation details in Section 4 and Section 5.1.

### A4: Formal definition of function summary
As described in Page 7, "this function summary records the access paths and related instructions for all the lock-acquire/release function calls and variable accesses in the analyzed function". We will add a formal definition of function summary and make it more clear here.

### A5: Limitations of type-based approach
Field-based analysis is a common type-based approach (used by DCUAF [6] and SADA [8]) that differentiates variables using structure type and field name. In Section 5.4, we have experimentally compared LR-Miner to field-based analysis (LR-Miner_FieldBased), and LR-Miner produces better results in Table 4. Indeed, field-based analysis neglects the alias relationships of different structure fields, and thus identifies many false locking rules. We will provide more discussion about possible alternatives and their limitations in Section 6.

## Review B
### B1: Bug types of harmful races
We have described the reasons of selecting the four bug types in Page 8. The main reason is that these bug types are common and harmful in OS kernels, and existing approaches focus on finding these bugs in sequential code not concurrency code. As described in Section 6, we believe that LR-Miner can support the detection of other concurrency issues including concurrency UAF bug. For example, a possible way of identifying concurrency UAF bug is to check whether a racy variable is handled by a memory-free operation. We will provide more details in Section 3.3 and Section 6.

### B2: Detecting races involving no lock mechanism
LR-Miner focuses on detecting lock-related races, and cannot find race bugs involving non-lock concurrency mechanisms like wait queue and completion mechanisms. We will discuss this limitation in Section 6.

### B3: Breakdown of performance results
LR-Miner benefits from function summary usage to achieve good performance. As described in "Efficiency improvement" paragraph in Page 10, many repeated analyses of the same function are avoided in this way, which can largely accelerate code analysis. In the evaluation, we tried LR-Miner without function summary, and its time usage exceeded our timeout (one week, namely 168 hours). We will provide more details in Section 5.4.

## Review C
### C1: Minor comments
Thanks a lot for your appreciation. We will address these issues.

## Review D
### D1: Recognizing of the same structure field
The same structure field in different functions can be recognized by comparing their access paths (expressed by the higher-layer structure type followed by a sequence of field names). We will provide more details in Section 3.1.

### D2: Missing lock relations without common ancestor
It is indeed a limitation of LR-Miner. We will discuss this in Section 6.

### D3: Interleavings in harmful estimation
The value of the racy variable is uncertain due to non-determinism of concurrent execution, and thus the access to it may cause problems. LR-Miner does not fully consider interleavings in harmful estimation, and just validates the concurrency of functions containing the found races, as described in "False positive dropping" paragraph in Page 9. We think missing full consideration of interleavings can indeed introduce some inaccuracy in harmful estimation. We will discuss this limitation in Section 6.

### D4: Outdated works in comparison
We have investigated papers of static race detection in OS kernels, and found that CPALockator is the only public-available tool published in recent five years. As for other recent papers, we found that just a few of them (like Lockpick [Security'23]) involve (but do not focus on) static race detection in OS kernels, but their tools are not publicly available. We will provide methodology comparison to these papers in Section 7.

### D5: Writing issues
We will address these issues and carefully check the whole paper.