# Learning Roadmap for *Introduction to Software Testing*

**Book:** *Introduction to Software Testing*, Second Edition  
**Authors:** Paul Ammann and Jeff Offutt  
**Scope:** All 14 chapters, including theory, exercises, practical application, and a final project  
**Suggested pace:** 18 weeks, 2-3 study sessions per week, 60-90 minutes per session  
**Estimated total effort:** 55-75 hours

## 1. The Goal

The goal is not merely to summarize the book. By the end of the roadmap, the learner should be able to:

1. Explain the important testing vocabulary precisely.
2. Design tests systematically instead of choosing inputs randomly.
3. Derive test requirements from input domains, graphs, logic expressions, and syntax.
4. Automate useful tests and build meaningful test oracles.
5. Apply testing throughout the software lifecycle.
6. Defend why each test exists and what risk it addresses.

The book's central idea is that strong test design can be organized around four structures:

| Structure | Main chapter | Main testing purpose |
| --- | ---: | --- |
| Input domains | 6 | Select representative input values |
| Graphs | 7 | Reach important program or model locations |
| Logic expressions | 8 | Exercise conditions so faults affect internal state |
| Syntax and grammars | 9 | Generate structured tests and use mutation to check propagation |

These structures connect to the book's RIPR model: **Reachability, Infection, Propagation, and Revealability**.

## 2. How Every Chapter Will Be Explained

Every chapter lesson will follow the same professional teaching format:

1. **Purpose:** Why the chapter exists and how it connects to earlier chapters.
2. **Learning outcomes:** What the learner should be able to do afterward.
3. **Key vocabulary:** Precise definitions rewritten in understandable language.
4. **Section-by-section explanation:** Every important subsection, in the book's order.
5. **Visual model:** A diagram, table, truth table, graph, or process map where it improves understanding.
6. **Worked example:** A complete example solved step by step.
7. **Practical application:** A Java or Spring-style example when the concept supports one.
8. **Common mistakes:** Confusing terms, invalid shortcuts, and likely exam errors.
9. **Knowledge check:** Short questions that test understanding, not memorization alone.
10. **Practice task:** An exercise completed independently before seeing the solution.
11. **Chapter summary:** The essential ideas in a compact revision sheet.
12. **Mastery check:** Evidence required before moving forward.

When material is added for clarity or modern practice, it will be labeled as an **additional explanation** so it is never confused with the authors' claims. The book's terminology and logic will remain the authority for the lessons.

## 3. Prerequisite Bridge

No advanced mathematics course is required, but the following ideas should be comfortable before the heavy chapters:

- Java methods, classes, parameters, return values, and exceptions
- Basic unit-test structure and assertions
- Sets, subsets, and simple set notation
- Boolean operators and truth tables
- Directed graphs, nodes, edges, and paths
- Requirements, use cases, and basic software lifecycle stages

Any missing topic should be taught as a short bridge immediately before it is needed. In particular:

- Review sets before Chapter 5.
- Review combinatorics before Chapter 6.
- Review directed graphs before Chapter 7.
- Review Boolean algebra and truth tables before Chapter 8.
- Review BNF grammar notation before Chapter 9.

## 4. The 18-Week Roadmap

| Week | Book coverage | Main objective | Required output |
| ---: | --- | --- | --- |
| 1 | Orientation + Chapter 1 | Understand why testing exists and distinguish fault, error, and failure | Testing vocabulary sheet and risk analysis |
| 2 | Chapter 2 | Understand MDTD, testing activities, levels, coverage, and RIPR | MDTD process map for one feature |
| 3 | Chapter 3 | Turn a test design into repeatable automated tests | Small automated test suite |
| 4 | Chapter 4 | Place testing early in agile development and CI | Testing-first workflow and CI checklist |
| 5 | Chapter 5 | Define test requirements and reason about coverage, infeasibility, and subsumption | Coverage-criterion comparison sheet |
| 6-7 | Chapter 6 | Build input domain models and combine blocks systematically | IDM tables plus derived test cases |
| 8-10 | Chapter 7 | Derive structural and data-flow tests from graph models | CFG, test paths, and coverage evidence |
| 11-12 | Chapter 8 | Derive tests from predicates and clauses | Truth tables and active-clause tests |
| 13-14 | Chapter 9 | Use grammars and mutation to design and evaluate tests | Grammar-based tests and mutation report |
| 15 | Chapters 10-11 | Integrate testing into the lifecycle and write a usable test plan | Lifecycle matrix and level test plan |
| 16 | Chapter 12 | Test incomplete or isolated systems using integration order and test doubles | Isolated component test with a double |
| 17 | Chapters 13-14 | Build regression strategy and trustworthy test oracles | Regression policy and oracle design |
| 18 | Final integration | Apply all four test-design structures to one feature | Capstone test portfolio and oral review |

## 5. Chapter-by-Chapter Plan

### Chapter 1 - Why Do We Test Software? (book pages 1-21)

**Focus:** Motivation and foundational language.

**The learner will be able to:**

- Distinguish a fault, an error state, and a failure.
- Explain verification versus validation.
- Describe the testing maturity levels presented in the chapter.
- Explain why testing reduces risk rather than proves perfection.
- Explain why late fault discovery is expensive.

**Applied task:** Analyze one software failure and trace its possible fault-error-failure chain. Then write a precise test objective for a feature.

**Mastery evidence:** The learner can classify new examples without mixing up faults, errors, and failures.

### Chapter 2 - Model-Driven Test Design (book pages 22-42)

**Focus:** The conceptual process used throughout the rest of the book.

**The learner will be able to:**

- Explain testing activities and testing levels.
- Explain the RIPR model.
- Separate test design, automation, execution, and evaluation.
- Move from a software artifact to a model, test requirements, test values, and executable tests.
- Explain why abstraction makes test design more systematic.

**Applied task:** Take one requirement from a booking feature and map it through the complete MDTD pipeline.

**Mastery evidence:** The learner can explain what changes and what remains independent when the same test requirement is implemented in a different framework.

### Chapter 3 - Test Automation (book pages 43-67)

**Focus:** Executable, repeatable tests.

**The learner will be able to:**

- Define software testability.
- Identify the components of a test case.
- Explain setup, input, expected result, execution, comparison, and cleanup responsibilities.
- Build ordinary, data-driven, and parameterized unit tests.
- Distinguish automatable work from the human reasoning required for test design.

**Applied task:** Automate tests for a small Java service. The book's test concepts should be preserved even if the syntax is adapted to the learner's current Java test framework.

**Mastery evidence:** The tests run repeatedly, have meaningful assertions, remain independent, and clearly report failures.

### Chapter 4 - Putting Testing First (book pages 68-80)

**Focus:** Tests as a continuous development safeguard.

**The learner will be able to:**

- Explain the cost-of-change argument and its limits.
- Explain the test harness as a guardian of existing behavior.
- Connect unit, system, and acceptance tests to agile work.
- Describe continuous integration's relationship to fast automated feedback.
- Add tests safely around legacy behavior.
- Discuss weaknesses and risks instead of treating agile testing as automatically sufficient.

**Applied task:** Design a pull-request testing workflow that separates fast checks from slower system tests.

**Mastery evidence:** The learner can justify when each test level runs and what feedback it provides.

### Chapter 5 - Criteria-Based Test Design (book pages 81-92)

**Focus:** The formal foundation for Chapters 6-9.

**The learner will be able to:**

- Define a test requirement and a coverage criterion.
- Explain why exhaustive testing is usually impossible.
- Distinguish a test requirement from a test case.
- Identify infeasible test requirements.
- Reason about subsumption between criteria.
- Explain how criteria provide design goals and stopping guidance.

**Applied task:** Given two criteria and several tests, identify the requirements each test covers and determine whether the stronger criterion subsumes the weaker one.

**Mastery evidence:** The learner never equates "100% coverage" with "fault-free software" and can state exactly which criterion achieved 100%.

### Chapter 6 - Input Space Partitioning (book pages 93-131)

**Focus:** Systematically selecting test inputs without needing source code.

**The learner will be able to:**

- Build interface-based and functionality-based input domain models.
- Define characteristics, partitions, blocks, and representative values.
- Check completeness, disjointness, and relevance of a model.
- Apply combination strategies, including each-choice, pair-wise, base-choice, and stronger combinations.
- Handle constraints among characteristics without creating impossible tests.
- Derive tests from documentation such as JavaDoc.

**Applied task:** Create an IDM for a hotel-booking request using dates, guest count, room capacity, room status, and discount information. Derive tests at more than one combination strength.

**Mastery evidence:** The learner can explain why every characteristic and block exists and can detect invalid or missing partitions.

### Chapter 7 - Graph Coverage (book pages 132-218)

**Focus:** The largest chapter and the main treatment of reachability.

**The learner will be able to:**

- Use nodes, edges, paths, subpaths, cycles, and simple paths precisely.
- Apply structural criteria such as node, edge, edge-pair, and prime-path coverage.
- Distinguish direct tours, sidetrips, and detours.
- Work with definitions, uses, def-clear paths, and du-paths.
- Apply all-defs, all-uses, and all-du-paths criteria.
- Derive graphs from source code, design relationships, state behavior, specifications, and use cases.
- Recognize infeasible paths and compare criteria through subsumption.

**Applied task sequence:**

1. Draw a control-flow graph for a booking-validation method.
2. List structural test requirements.
3. Create test paths that satisfy a chosen criterion.
4. Map the paths back to executable input values.
5. Extend the analysis to one data-flow criterion.

**Mastery evidence:** The learner can derive a test set from a new graph and show coverage explicitly, rather than guessing paths.

### Chapter 8 - Logic Coverage (book pages 219-291)

**Focus:** Designing tests for predicates and clauses to support infection.

**The learner will be able to:**

- Distinguish predicates from clauses.
- Apply predicate coverage, clause coverage, and combinatorial coverage.
- Explain when a major clause determines a predicate.
- Apply active-clause and inactive-clause coverage variants.
- Find satisfying truth assignments and recognize infeasibility.
- Understand how expression rewriting and side effects affect testing.
- Apply logic coverage to programs, specifications, and finite-state machines.
- Treat DNF, implicant coverage, MUMCUT, and Karnaugh maps as an advanced extension.

**Applied task:** Analyze a predicate such as room availability, date validity, capacity, and payment authorization. Build its truth table and derive active-clause tests.

**Mastery evidence:** The learner can prove why a clause determines the predicate in each selected test pair.

### Chapter 9 - Syntax-Based Testing (book pages 292-354)

**Focus:** Grammar coverage and mutation testing, supporting propagation.

**The learner will be able to:**

- Derive tests from regular expressions and BNF grammars.
- Generate valid and invalid strings systematically.
- Explain mutants, mutation operators, killing a mutant, and equivalent mutants.
- Use mutation as a way to evaluate test strength.
- Apply syntax-based ideas to programs, integration, object-oriented software, specifications, XML-like inputs, and other input grammars.

**Applied task sequence:**

1. Define a small grammar for a structured booking request or command.
2. Generate valid and invalid inputs using grammar coverage.
3. Introduce controlled mutations into a validation rule.
4. Run the test suite and analyze surviving mutants.

**Mastery evidence:** The learner can distinguish an equivalent mutant from a merely surviving mutant and knows why mutation score is evidence, not proof of correctness.

### Chapter 10 - Managing the Test Process (book pages 355-363)

**Focus:** Testing activities across the full development lifecycle.

**The learner will be able to:**

- Plan testing during requirements, architecture, design, implementation, integration, deployment, operation, and maintenance.
- Distinguish fault prevention from fault detection activities.
- Assign testing responsibilities and outputs early enough to influence quality.
- Translate the book's MDTD approach into a practical team process.

**Applied task:** Create a lifecycle matrix showing testing objectives, activities, owners, inputs, and outputs for a small software project.

**Mastery evidence:** Testing work begins with requirements and design rather than appearing only after implementation.

### Chapter 11 - Writing Test Plans (book pages 364-368)

**Focus:** Useful test documentation rather than paperwork for its own sake.

**The learner will be able to:**

- Explain the purpose of master and level test plans as presented in the book.
- Define scope, approach, resources, schedule, responsibilities, test items, risks, and completion criteria.
- Connect every plan section to an actual testing decision.
- Avoid vague statements that cannot guide execution or evaluation.

**Applied task:** Write a concise level test plan for a booking-service API using the template discussed in the chapter.

**Mastery evidence:** Another tester could use the plan without needing the author to explain what it means.

### Chapter 12 - Test Implementation (book pages 369-379)

**Focus:** Making tests executable when the complete system is unavailable or difficult to control.

**The learner will be able to:**

- Choose and justify an integration order.
- Explain the purpose and cost of test doubles.
- Distinguish stubs and mocks in the chapter's treatment.
- Replace dependencies in a controlled way while preserving the test objective.
- Recognize that test doubles can also contain faults or make tests unrealistic.

**Applied task:** Isolate a service from a repository or external payment component and test its behavior with an appropriate double.

**Mastery evidence:** The learner can explain why the double is necessary, what behavior it simulates, and what the test cannot establish.

### Chapter 13 - Regression Testing for Evolving Software (book pages 380-383)

**Focus:** Maintaining confidence as software changes.

**The learner will be able to:**

- Explain why regression testing must be automated to remain practical.
- Decide which tests belong in a regression suite.
- Select and prioritize tests when the full suite is too expensive.
- Interpret failures caused by intended behavior changes, unintended regressions, or outdated tests.
- Connect regression suites with continuous integration and version control.

**Applied task:** Classify a project's tests into fast commit checks, broader integration checks, and scheduled full regression tests.

**Mastery evidence:** The learner can update tests after a requirement change without blindly deleting a failing test or preserving an obsolete expectation.

### Chapter 14 - Writing Effective Test Oracles (book pages 384-393)

**Focus:** Deciding what correct behavior looks like and revealing failures.

**The learner will be able to:**

- Define a test oracle and connect it to revealability.
- Decide what outputs and state should be checked.
- Balance checking too little against checking irrelevant details.
- Determine expected values using direct specification, redundant computation, consistency checks, and metamorphic testing.
- Recognize oracle limitations and observability problems.

**Applied task:** Design oracles for price calculation and booking confirmation. Add at least one consistency relation and one metamorphic relation.

**Mastery evidence:** Each assertion has a reason, and the learner can explain what fault it could reveal.

## 6. Practice Project Used Across the Book

A single small **hotel-booking service** can connect the chapters and reduce context switching. It should contain:

- A booking request with check-in and check-out dates
- Room type and capacity
- Availability checking
- Price calculation and an optional discount
- Booking status transitions
- A repository dependency
- One external dependency, such as payment or notification

The same feature evolves throughout the roadmap:

| Chapters | Project application |
| --- | --- |
| 1-2 | Define risks, test levels, artifacts, and the MDTD flow |
| 3-4 | Build automated tests and place them in a repeatable workflow |
| 5-6 | Define requirements, partitions, blocks, and combination criteria |
| 7 | Build control-flow and data-flow models |
| 8 | Test compound booking predicates |
| 9 | Evaluate suite strength with grammars and mutation |
| 10-11 | Build the process matrix and test plan |
| 12 | Isolate repository and external dependencies |
| 13 | Organize a regression strategy |
| 14 | Improve assertions and expected-value logic |

## 7. Assessment System

### After every chapter

- A 5-question terminology check
- Three application questions
- One worked problem completed without looking at the solution
- One short "teach it back" explanation
- One practical artifact or automated test

### Part I gate - after Chapter 5

The learner must be able to explain the complete route:

**software artifact -> test model -> coverage criterion -> test requirements -> test values -> executable tests -> results -> evaluation**

### Part II gate - after Chapter 9

Given a new feature, the learner must be able to decide whether an input-domain, graph, logic, or syntax model is most useful and justify the choice.

### Final gate - after Chapter 14

The final portfolio must contain:

1. A test plan.
2. An input-domain model.
3. A graph and its test paths.
4. Logic-based test requirements.
5. A mutation or grammar-based analysis.
6. Automated tests using a test double where appropriate.
7. A regression strategy.
8. Clearly justified test oracles.

## 8. Mastery Standard

A chapter is complete only when the learner can:

- Explain its central idea without copying the text.
- Use its terminology correctly.
- Solve a new example, not only repeat the book's example.
- Connect the chapter to the MDTD and RIPR models when relevant.
- Identify the limitations of the technique.
- Produce the chapter's practical artifact.

A useful target is at least **80% on knowledge checks** and a correct independent solution to the core practice problem. If a chapter is weak, review the specific prerequisite or subsection instead of rereading the entire chapter.

## 9. Recommended Starting Point

Begin with Chapter 1 as a complete lesson. Do not begin Chapter 2 until the distinctions among **fault, error, failure, verification, validation, testing, and debugging** are secure. These terms become the language used by every later chapter.
