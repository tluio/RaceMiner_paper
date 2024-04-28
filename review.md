USENIX Security '24 Winter Paper #40 Reviews and Comments
===========================================================================
Paper #40 LR-Miner: Static Race Detection in OS Kernels by Mining Locking
Rules


Review #40A
===========================================================================

Paper summary
-------------
LR-Miner is a tool designed to identify potential data races within OS kernels, fundamentally utilizing the well-established lock set analysis technique. To adapt this analysis to the complexities of operating system kernels, LR-Miner employs a heuristic approach to deduce the fields within structures that corresponding locks should protect. Specifically, it constructs field graphs as abstract representations to ascertain the structure type and specific field a memory read or write access targets. By integrating these graphs with lock sets, LR-Miner can deduce locking rules and pinpoint unprotected field accesses. Notably, LR-Miner has identified approximately 200 potential data races, with 10 of these instances receiving CVE IDs at the time of submission. Some kernel developers have also expressed interest in incorporating LR-Miner into their CI pipelines.

Detailed comments for authors
-----------------------------
Thanks for submitting to Usenix Security 2024.

Overall, I believe the LR-Miner should be a useful tool for many kernel developers and showcases some novel ideas about performing lock set analysis on modern operating system kernels.

I think the paper can be published nearly as it is after minor proofreading and editing, but it would be great if the revised one could also address the following comments and questions.

First, I think LR-Miner is about applying the traditional idea of lock set analysis (e.g., as suggested in ERaser [65]) for complex modern operating systems. It would be interesting if the paper discusses the difference between such existing static lock set analysis and LR-Miner somewhere. The discussion could also highlight why new ideas, such as field-aware locking rule mining, are needed.

Second, some minor details about LR-Miner are unclear. For example, in Section 3.1, the paper says:

> Field graph is updated by handling each arrow operator (->).

This suggests that LR-Miner does not consider the other forms of field accesses, such as the dot (`.`). I think LR-Miner considers all possible ways of field access because, in the later section, it is said that the tool works at the LLVM IR level.

Third, the following short note in Section 5.1 about pruning false positives does not include a detailed technique.

> 11.1K false races are dropped because their code paths are infeasible or their caller functions have no concurrency

The problem of determining if a caller function has no concurrency or no feasible code paths does not look trivial to me, though I don't think it's impossible or significantly challenging. Please provide more details about his procedure in the revised version.

Fourth, LR-Miner introduces and uses the notion of function summary, which contains the intermediate analysis results needed to build field graphs for each context. Though the included example (in Figure 8) helps me understand what a function summary may look like, the lack of its formal definition makes the description of LR-Miner about function summary vague. 



Some may also question why it is necessary to use the field graph, saying that it would be enough to rely on the types. Discussing possible alternatives and their limitations would also make the paper helpful.

Ethics consideration
--------------------
1. No

Required changes
----------------
- Provide details about how LR-Miner drops the infeasible code paths (about Section 5.1).
- Please proofread the paper to improve the readability of some sentences.

Reasons to accept the paper
---------------------------
- LR-Miner showcases that lock set analysis can be used to find data races statically from modern operating system kernels.
- LR-Miner found many potentially harmful data races from two modern operating system kernels, and 10 received CVE IDs.
- The proposed technique using the field graph is succinct yet effective in identifying the access of fields with each lock set.

Reasons to not accept the paper
-------------------------------
- Some crucial details are missing (e.g., how LR-Miner drops infeasible code paths).
- Experiments do not clearly demonstrate the effectiveness or the need for field graphs.

Recommended decision
--------------------
2. Accept on Shepherd Approval

Questions for authors' response
-------------------------------
- Provide details about how LR-Miner drops the infeasible code paths (about Section 5.1).

Writing quality
---------------
3. Adequate

Confidence in recommended decision
----------------------------------
3. Highly confident (would try to convince others)



Review #40B
===========================================================================

Paper summary
-------------
This paper introduces a new kernel race bug detector, LR-Miner. Using static program analysis techniques, LR-Miner extracts locking rules from the kernel code, then identifies data races using alias-aware checking method, and finally distinguishes harm bugs via pattern-based bug detection. In the evaluation, LR-Minder has detected over 200 harmful race bugs, leading to assignment of 10 CVEs.

Detailed comments for authors
-----------------------------
This paper provides appropriate example code and figures, which facilitate the readers' understanding of the key research problems and their approach. I also appreciate the author's contribution to bug discovery, comparison analysis as well as the detailed description of the evaluation results, including false positive/negative, and case studies. The following has more comments with some concerns about this paper.

Pattern-based bug detection is limited to identifying four fixed types of race bugs. However, I'm not sure the authors adequately justify such a choice of bug types. UAF (use-after-free) bugs are arguably one of most common forms of data race bugs. While partly covered by EHB cases, I expected to explicitly include this type of race bugs for monitoring. So it would be nice to provide justification for the chosen bug types. 

The proposed technique employs lockset analysis, which relies on lock information to detect data races. I was wondering if this technique could also identify harmful race bugs (e.g., UAF bugs) that do not involve lock mechanisms, i.e., there is no relevant lock information available for a shared variable, which can occur in real device drivers within OS kernels, such as Linux kernel.

In the evaluation, while the authors provide the results of runtime performance in tables (e.g., Table 2), there is no detailed explanation about these results. Although the authors made efforts to enhance performance, performing static analysis with high accuracy, through field/flow/context sensitive and inter-procedural analysis (along with allyesconfig), comes with a significant performance overhead. It would be great if the authors could provide explanation/further breakdown of the performance results.

Ethics consideration
--------------------
2. Yes: submission appropriately mitigates potential risks or harms

Reasons to accept the paper
---------------------------
- good motivating example
- evaluation results i.e., numbers of new bugs and CVEs 
- improvement over previous work

Reasons to not accept the paper
-------------------------------
- limited applicability 
- limited bug types for detection
- lack of discussion about runtime performance

Recommended decision
--------------------
3. Accept Conditional on Major Revision

Writing quality
---------------
3. Adequate

Confidence in recommended decision
----------------------------------
2. Fairly confident



Review #40C
===========================================================================

Paper summary
-------------
Data races are one of the most common concurrency issues in kernels, and can cause severe problems. Therefore, a critical step of race detection is identifying locking rules for which variable should be protected by which lock.
Unlike user-space programs, which actively execute code, kernel code is often passively executed via system calls, complicating the race detection.

Previously proposed approaches incur three significant limitations:

1. failing to accurately identify locking rules (that is, which variable should be protected by which lock)
2. failing to consider alias relations
3. they do not consider the security impact of a race

This paper presents a new static analysis approach, LR-Miner, to detect kernel data races by mining locking rules from source code. LR-Miner improves on previous work by employing:

1. Field-aware mining, that is, since many parts of kernel code are not sufficiently commented or documented, LR-Miner mines locking rules from source code by constructing and analyzing the structure field relation between locks and accessed variables. A key insight is the fact that  a shared variable and its protection lock are often represented as different fields but in the same data structure
2. Flow/context/field-sensitive and inter-procedural alias analysis to identify relationships involving both locks and accessed variables. Then, an SMT-based validation code-path feasibility reduces false positives
3. Usage patterns to classify the found races into null-pointer dereferences, double fetch issues, etc.; this helps automatically estimate the security impact of each reported race and identify the harmful ones (developers can introduce benign races to improve efficiency)

The authors evaluated LR-Miner on Linux and FreeBSD. LR-Miner mines 2.7K locking rules from kernel code and finds 306 actual races with a false positive rate of 19.9%. Among those, 200 are estimated to be harmful as they can cause security problems, and kernel developers have confirmed 61.
Moreover, ten harmful races have been assigned with CVE IDs

Detailed comments for authors
-----------------------------
First of all, thanks for your submission, which I enjoyed.

I have only few minor comments:

- the text size of subscripts in Figure 4 and 6 is way too small when printed on paper
- on pg.7, you mention a threshold T without commenting on its value; I see you discuss that in Section 5, and  I think you should add a reference to section 5 here

Ethics consideration
--------------------
1. No

Reasons to accept the paper
---------------------------
- this paper presents a solid approach for data-race detection in kernels, with a low false-positive rate
- Linux/FreeBSD developers have confirmed 61 races automatically found by LR-Miner, and 10 races have been assigned with CVE IDs
- the authors plan to open-source their work upon publication

Recommended decision
--------------------
1. Accept

Writing quality
---------------
2. Well-written

Confidence in recommended decision
----------------------------------
1. Little confidence (would be convinced by differing opinions)



Review #40D
===========================================================================

Paper summary
-------------
This paper introduces LR-Miner, a static analysis-based kernel race condition detection tool. 
LR-Miner leverages the convention that a shared variable is typically protected by a lock from a different field within the same data structure to mine the locking rules (namely, which variable should be protected by which lock).
LR-Miner enhances the classic lockset analysis with alias analysis on both locks and accessed variables to detect data races. 
Besides, it employs a pattern-based strategy to estimate the potential impact of the identified data races. 
In total, LR-Miner identified 200 harmful data races in Linux and FreeBSD, 10 of which have already been assigned CVE IDs.

Detailed comments for authors
-----------------------------
This paper introduces LR-Miner, a static kernel race detection framework, leveraging field-aware locking rule mining and alias-aware race checking. The results demonstrate LR-Miner’s capability to detect more data races with a lower false positive rate.
However, several aspects need further consideration and it will help if these problems could be addressed.

## It is unclear how to recognize the same data field in different functions when calculating the proportion of variables access protected by lock to the total variable access.

When LR-Miner establishes locking relations, it may not recognize that certain field graphs in different functions represent the same data structure. But when it processes locking rule mining, it needs to calculate the proportion of variable access protected by lock to the total variable access. It's necessary to recognize accesses to the same data field in different functions.

## Only considering locks that have the same ancestor as the field will result in missing some lock relations.

The lock relation construction just considers situations in which lock and data have a common ancestor. This will miss some lock relations that data and lock don't have a common ancestor. This may lead to missing some possible data races.

## The pattern-based harmful estimation doesn't describe how to find interleavings that cause data race and how to match patterns.

The harmful estimation first identifies racy-variable accesses. Then it checks usage patterns from racy-variable accesses. However, it is necessary to determine the interleaving of these racy-variable accesses before checking usage patterns. And it would be better to provide the algorithms to check usage patterns.

## Much of the compared works are outdated

Most of the static kernel race detectors used in this paper as comparison baselines are outdated. They are all proposed 5 years ago except CPALockator. This results in the inability to prove that LR-miner is superior to the current state of the art.

## Minor issue about writing
- There should be some explanation about why the evaluation set the threshold in locking rule mining to 0.7.
- Section 2.2 duplicated 'that' in 'we that that identifying locking ...'

Ethics consideration
--------------------
1. No

Required changes
----------------
- Compare with more recent static kernel race detectors in the evaluation. 

- Conduct experiments to verify the best threshold for locking rule mining

Reasons to accept the paper
---------------------------
- This paper introduces a field graph-based locking rule mining technique, which can effectively and precisely find locking rules statically.

- This paper makes a rudimentary step toward estimating the security impact of the uncovered race conditions. 

- The evaluation provides a detailed comparison between LR-Miner and other approaches, demonstrating that LR-Miner discovers more harmful races while generating fewer false positives.

- The writing is very good, providing ample explanation for the work in this paper, including the motivation, design, and evaluation.

- Finds a good number of race condition bugs.

Reasons to not accept the paper
-------------------------------
- LR-Miner cannot uncover lock relations that do not exist within the same data structure. This would miss many data races within the kernel.

- Pattern-based harmful estimation doesn't describe how to find interleavings that cause data race with some racy-variable accesses.

- The majority of the compared works in the evaluation are not from recent years.

Recommended decision
--------------------
2. Accept on Shepherd Approval

Writing quality
---------------
2. Well-written

Confidence in recommended decision
----------------------------------
1. Little confidence (would be convinced by differing opinions)



AuthorFeedback Response by Author [Tuo Li <islituo@163.com>] (821 words)
---------------------------------------------------------------------------
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