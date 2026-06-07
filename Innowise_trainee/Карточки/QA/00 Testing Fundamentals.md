# Flashcards — Testing Fundamentals

Source: [[00 Testing Fundamentals]]

#flashcards/QA/Fundamentals

---

What is the relationship between QA, QC, and Testing?
?
QA ⊇ QC ⊇ Testing. Testing is a subset of QC, and QC is a subset of QA.

What is Quality Assurance (QA) — focus, goal, approach?
?
Focus: the **process**. Goal: **prevent** defects. Approach: proactive — build quality into the whole development lifecycle.

What is Quality Control (QC) — focus, goal, approach?
?
Focus: the **product**. Goal: **find** defects in what was built. Approach: reactive — check quality after something is created.

What is the difference between Verification and Validation?
?
Verification — a **static** check of documents, design, and code against requirements (*are we building the product right?*). Validation — a **dynamic** test of the actual running product against user needs (*are we building the right product?*).

What is the chain Error → Defect → Failure?
?
A human makes an **Error** (mistake) → which introduces a **Defect** (flaw in code/docs) → which causes a **Failure** (observable wrong behavior) when the defective code is executed.

What is an Error?::A mistake made by a human (developer, analyst, designer).

What is a Defect?::A flaw in the code or documentation caused by an error — this is what testers find.

What is a Failure?::Observable incorrect behavior that occurs when defective code is executed in a real environment.

What is the main goal of software testing?
?
To **reduce the risk of failure** in production by finding defects as early as possible. The primary practical activity that achieves this is finding bugs.

Why should testing start early in development?
?
Defects found early are far cheaper and easier to fix, and early testing stops defects from compounding through later phases.

What is the Pesticide Paradox?
?
Running the same tests repeatedly eventually stops finding new bugs. Test cases must be regularly reviewed, updated, and expanded.

What does "testing shows the presence of defects, not their absence" mean?
?
Passing all tests does not prove the product is bug-free — a test suite only covers the scenarios it was written for.

What is the "absence of errors fallacy"?
?
The false belief that fixing all bugs guarantees success. A technically clean product can still fail if it doesn't meet real user needs.

What does "testing is context-dependent" mean?
?
The approach, tools, and depth of testing depend on the product type — a banking app, a game, and medical software each need different testing.

Why does the defect clustering principle matter?
?
A small number of modules usually contain the majority of defects, so testing effort should be focused there based on risk.
<!--SR:!2026-06-08,1,230-->
