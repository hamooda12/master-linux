# Improved Learning Roadmap for *Introduction to Software Testing*

**Book:** *Introduction to Software Testing*, Second Edition
**Authors:** Paul Ammann and Jeff Offutt
**Coverage:** All 14 chapters
**Recommended duration:** 22 weeks
**Weekly schedule:** 2–3 sessions
**Session duration:** 60–90 minutes
**Estimated total effort:** 65–90 hours

---

# 1. Roadmap Purpose

This roadmap is designed to help the learner understand and apply the complete testing process presented in the book.

The objective is not merely to read or summarize each chapter. By the end of the roadmap, the learner should be able to:

1. Use software-testing terminology precisely.
2. Explain why a test exists and which risk it addresses.
3. Design tests systematically instead of selecting inputs randomly.
4. Derive test requirements from software artifacts and testing models.
5. Apply coverage criteria correctly.
6. Convert abstract test requirements into executable tests.
7. Build meaningful test oracles.
8. use automation without confusing automation with test design.
9. Apply testing throughout the software lifecycle.
10. Evaluate the strengths and limitations of a test suite.
11. Build a professional software-testing portfolio.
12. Explain and defend testing decisions independently.

---

# 2. Central Learning Model

The roadmap follows the Model-Driven Test Design process used throughout the book:

```text
Software artifact
        ↓
Testing model
        ↓
Coverage criterion
        ↓
Test requirements
        ↓
Test values
        ↓
Executable tests
        ↓
Test execution
        ↓
Result evaluation
```

A software artifact may be:

* A requirement
* Source code
* A design model
* A use case
* A state model
* An input grammar
* A logical expression

The model simplifies the artifact into a structure that can be analyzed systematically.

---

# 3. The RIPR Model

The roadmap connects the testing techniques to the RIPR model:

1. **Reachability:** The test must reach the faulty location.
2. **Infection:** The fault must create an incorrect internal state.
3. **Propagation:** The incorrect state must propagate toward an observable result.
4. **Revealability:** The tester must observe and identify the incorrect result.

The four major test-design structures in the book support different parts of this process.

| Testing structure   | Main chapter | Main purpose                                                         |
| ------------------- | -----------: | -------------------------------------------------------------------- |
| Input domains       |            6 | Select representative input values                                   |
| Graphs              |            7 | Reach important locations and paths                                  |
| Logic expressions   |            8 | Exercise conditions so faults infect program state                   |
| Syntax and grammars |            9 | Generate structured inputs and evaluate propagation through mutation |
| Test oracles        |           14 | Reveal and identify incorrect behavior                               |

These connections are learning aids. They should not be treated as strict boundaries because one testing technique may support more than one RIPR condition.

---

# 4. Learning Principles

## 4.1 Active learning

Every important concept must be used in a problem, model, test, or explanation.

Reading alone does not demonstrate mastery.

## 4.2 Progressive difficulty

Each chapter should move through four levels:

1. Understand the terminology.
2. Follow a solved example.
3. Complete a similar guided problem.
4. Solve a new problem independently.

## 4.3 One continuing project

A single hotel-booking service will be used throughout the roadmap.

This reduces context switching and shows how different testing techniques can be applied to the same software feature.

## 4.4 Book terminology remains authoritative

The book’s definitions, terminology, organization, and logical framework remain the foundation of the lessons.

Any modern tool, Java example, Spring example, or clarification not presented directly by the authors must be labeled:

> **Additional explanation**

This prevents supporting material from being confused with the book’s original claims.

## 4.5 Mastery before progression

Completing a chapter means being able to apply its concepts to a new problem.

It does not mean merely finishing the assigned pages.

---

# 5. Standard Format for Every Chapter

Every chapter lesson should follow this format.

## 5.1 Chapter purpose

Explain:

* Why the chapter exists
* What problem it addresses
* How it connects to earlier chapters
* How it contributes to MDTD or RIPR

## 5.2 Learning outcomes

State what the learner should be able to explain, calculate, design, or implement after completing the chapter.

## 5.3 Prerequisite check

Identify the knowledge needed before beginning the chapter.

Any missing prerequisite should be taught through a short bridge lesson.

## 5.4 Key vocabulary

Provide:

* The book’s terminology
* A precise definition
* A plain-language explanation
* A simple example
* A comparison with commonly confused terms

## 5.5 Section-by-section explanation

Explain important sections in the book’s order.

The explanation should preserve the authors’ conceptual structure instead of replacing it with a different testing framework.

## 5.6 Visual model

Use an appropriate visual representation, such as:

* A process map
* Input domain table
* Directed graph
* Control-flow graph
* Truth table
* Subsumption diagram
* Grammar tree
* Lifecycle matrix

## 5.7 Worked example

Solve one complete example step by step.

The example must show the reasoning process, not only the final answer.

## 5.8 Guided practice

Provide a similar problem with limited hints.

## 5.9 Independent practice

Provide a new problem that the learner completes without seeing the solution first.

## 5.10 Practical application

Apply the concept to Java or a Spring-style hotel-booking feature when appropriate.

The practical example should implement the chapter’s concept rather than replace it with framework-specific instructions.

## 5.11 Common mistakes

Identify:

* Incorrect definitions
* Invalid shortcuts
* Common calculation errors
* Misleading interpretations of coverage
* Likely examination mistakes
* Common automation mistakes

## 5.12 Knowledge check

Include:

* Five terminology questions
* Three application questions
* One analysis question
* One “teach it back” explanation

## 5.13 Chapter artifact

Produce a concrete result, such as:

* A vocabulary sheet
* An input-domain model
* A graph
* A truth table
* An automated test suite
* A mutation report
* A regression strategy

## 5.14 Revision sheet

Create a one-page summary containing:

* Essential terminology
* Central formulas or rules
* A small example
* Common mistakes
* Connections to other chapters

## 5.15 Mastery check

Define the evidence required before progressing.

---

# 6. Weekly Session Structure

Each study week should normally contain three sessions.

## Session 1 — Concepts

**Duration:** 60–90 minutes

Activities:

1. Review the previous lesson.
2. Read the assigned section.
3. Explain the terminology.
4. Create the chapter’s visual model.
5. Answer short understanding questions.

## Session 2 — Worked problems

**Duration:** 60–90 minutes

Activities:

1. Review the main definitions.
2. Follow one complete worked example.
3. Complete one guided example.
4. Compare the result with the solution.
5. Record mistakes in an error log.

## Session 3 — Independent application

**Duration:** 60–90 minutes

Activities:

1. Solve an independent problem.
2. Apply the idea to the hotel-booking project.
3. Produce the required chapter artifact.
4. Complete the knowledge check.
5. Update the portfolio repository.

When only two sessions are available, combine the worked problem and independent application into the second session.

---

# 7. Prerequisite Diagnostic

Before starting the main chapters, complete a short diagnostic assessment.

The diagnostic is not intended to test advanced software-testing knowledge. It identifies which bridge lessons are necessary.

## 7.1 Java foundations

The learner should be able to explain:

* Methods
* Parameters
* Return values
* Classes and objects
* Conditional statements
* Loops
* Exceptions
* Basic collections

## 7.2 Unit-testing foundations

The learner should recognize:

* Test setup
* Test input
* Expected result
* Actual result
* Assertion
* Test cleanup
* Test independence

## 7.3 Sets

The learner should understand:

* Sets
* Elements
* Subsets
* Union
* Intersection
* Empty sets
* Set membership

## 7.4 Basic combinatorics

The learner should be able to count simple combinations of choices.

This will support Chapter 6.

## 7.5 Directed graphs

The learner should understand:

* Nodes
* Edges
* Paths
* Direction
* Cycles

This will support Chapter 7.

## 7.6 Boolean logic

The learner should understand:

* AND
* OR
* NOT
* True and false
* Truth tables
* Compound conditions

This will support Chapter 8.

## 7.7 Grammar notation

The learner should recognize the basic purpose of:

* Regular expressions
* Production rules
* Terminals
* Nonterminals
* BNF-style notation

This will support Chapter 9.

## Diagnostic result

Classify each prerequisite as:

* **Secure:** No bridge lesson required.
* **Developing:** Complete a short review.
* **Weak:** Complete a full bridge lesson and practice exercise.

Only review the prerequisite areas that are actually weak.

---

# 8. The Improved 22-Week Schedule

|  Week | Coverage                                   | Main objective                                               | Required output                                    |
| ----: | ------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- |
|     1 | Diagnostic, orientation, Chapter 1         | Establish prerequisites and foundational testing language    | Diagnostic report, vocabulary sheet, risk analysis |
|     2 | Chapter 2                                  | Understand MDTD, testing activities, levels, and RIPR        | MDTD process map                                   |
|     3 | Chapter 3                                  | Convert test designs into repeatable automated tests         | Small automated test suite                         |
|     4 | Chapter 4                                  | Place testing early in development and CI                    | Testing-first workflow                             |
|     5 | Review Chapters 1–4 and Chapter 5          | Consolidate foundations and understand criteria-based design | Part I concept map and coverage comparison         |
|   6–7 | Chapter 6                                  | Build and evaluate input domain models                       | IDM tables and derived tests                       |
|     8 | Chapter 6 assessment and cumulative review | Apply partitioning independently and reconnect Chapters 1–6  | Completed input-space case study                   |
|  9–12 | Chapter 7                                  | Master structural and data-flow graph coverage               | CFGs, test paths, data-flow analysis               |
|    13 | Chapter 7 assessment and cumulative review | Demonstrate independent graph-based test design              | Graph-testing portfolio section                    |
| 14–16 | Chapter 8                                  | Derive tests from predicates and clauses                     | Truth tables and active-clause tests               |
| 17–18 | Chapter 9                                  | Apply grammar-based testing and mutation testing             | Grammar tests and mutation report                  |
|    19 | Chapters 10–11                             | Integrate testing into the lifecycle and create a test plan  | Lifecycle matrix and level test plan               |
|    20 | Chapter 12                                 | Test incomplete or isolated components                       | Test-double implementation                         |
|    21 | Chapters 13–14                             | Design regression strategy and effective test oracles        | Regression policy and oracle analysis              |
|    22 | Final integration                          | Apply the complete roadmap to one feature                    | Capstone portfolio and oral review                 |

---

# 9. Chapter-by-Chapter Learning Plan

## Chapter 1 — Why Do We Test Software?

**Book pages:** 1–21
**Recommended time:** One week

### Focus

Motivation, risk, and foundational terminology.

### Learning outcomes

The learner will be able to:

* Distinguish faults, error states, and failures.
* Explain verification and validation.
* Explain the purpose of testing.
* Describe the testing maturity levels introduced in the chapter.
* Explain why testing reduces risk rather than proving perfection.
* Explain why discovering faults late can be expensive.
* Distinguish testing from debugging.

### Worked example

Analyze a failure in a booking system:

1. Identify the observable failure.
2. Identify the incorrect internal state.
3. Suggest a possible fault that created the state.
4. Explain how a test might reveal the failure.

### Applied task

Choose one hotel-booking feature and write:

* The feature’s main risks
* A precise test objective
* Possible fault-error-failure chains
* The consequences of failure

### Required artifact

`chapter-01-foundations.md`

It should contain:

* Vocabulary definitions
* Fault-error-failure examples
* Verification versus validation
* Testing versus debugging
* Feature risk analysis

### Mastery evidence

The learner can classify new examples correctly without confusing:

* Fault
* Error
* Failure
* Testing
* Debugging
* Verification
* Validation

---

## Chapter 2 — Model-Driven Test Design

**Book pages:** 22–42
**Recommended time:** One week

### Focus

The conceptual testing process used throughout the rest of the book.

### Learning outcomes

The learner will be able to:

* Explain the main testing activities.
* Explain testing levels.
* Explain the RIPR model.
* Separate test design, automation, execution, and evaluation.
* Move from a software artifact to executable tests.
* Explain the purpose of test models.
* Explain why abstraction makes test design systematic.
* Distinguish test requirements from test values.

### Worked example

Use a booking cancellation requirement and identify:

1. The software artifact
2. The testing model
3. The coverage criterion
4. The test requirements
5. The test values
6. The executable tests
7. The evaluation process

### Applied task

Create the complete MDTD pipeline for one booking feature.

### Required artifact

`chapter-02-mdtd-map.md`

### Mastery evidence

The learner can explain which parts of a test design remain conceptually stable when the implementation framework changes.

---

## Chapter 3 — Test Automation

**Book pages:** 43–67
**Recommended time:** One week

### Focus

Creating repeatable and executable tests.

### Learning outcomes

The learner will be able to:

* Define software testability.
* Identify the parts of a test case.
* Explain setup, execution, comparison, and cleanup.
* Build ordinary unit tests.
* Build parameterized or data-driven tests.
* Write meaningful assertions.
* Keep tests independent.
* Explain which testing tasks require human reasoning.
* Explain why automation does not replace test design.

### Practical application

Automate tests for a small Java booking-pricing service.

Example responsibilities:

* Validate dates
* Calculate the number of nights
* Calculate the base price
* Apply an optional discount
* Reject invalid input

### Required artifact

A small executable test suite containing:

* Normal tests
* Boundary tests
* Exception tests
* Parameterized tests
* Clear assertions

### Mastery evidence

The tests:

* Run repeatedly
* Produce consistent results
* Remain independent
* Report failures clearly
* Avoid unnecessary implementation dependence

---

## Chapter 4 — Putting Testing First

**Book pages:** 68–80
**Recommended time:** One week

### Focus

Using tests continuously during development.

### Learning outcomes

The learner will be able to:

* Explain the cost-of-change argument and its limitations.
* Explain tests as guardians of existing behavior.
* Connect testing to agile development.
* Explain the role of unit, system, and acceptance tests.
* Explain continuous integration as rapid feedback.
* Add tests around legacy behavior.
* Identify weaknesses in automated test suites.
* Avoid treating testing-first practices as automatically sufficient.

### Applied task

Design a pull-request workflow containing:

1. Static checks
2. Fast unit tests
3. Integration tests
4. System tests
5. Acceptance checks
6. Failure-handling rules

### Required artifact

`chapter-04-testing-first-workflow.md`

### Mastery evidence

The learner can justify:

* Which tests run on every commit
* Which tests run before merging
* Which tests run less frequently
* What feedback each test level provides

---

## Chapter 5 — Criteria-Based Test Design

**Book pages:** 81–92
**Recommended time:** One week, including cumulative review

### Focus

The formal foundation of Chapters 6–9.

### Learning outcomes

The learner will be able to:

* Define a test requirement.
* Define a coverage criterion.
* Distinguish test requirements from test cases.
* Explain why exhaustive testing is usually impossible.
* Identify infeasible test requirements.
* Explain criterion satisfaction.
* Reason about subsumption.
* Explain how criteria support design and stopping decisions.
* State precisely which criterion has achieved a given coverage level.

### Worked problem

Given:

* A small model
* Two coverage criteria
* Several test requirements
* Several test cases

Determine:

1. Which requirements each test satisfies
2. Whether any requirement is infeasible
3. Whether one criterion subsumes another
4. Whether the provided test set satisfies each criterion

### Required artifact

`chapter-05-coverage-criteria.md`

### Mastery evidence

The learner never treats “100% coverage” as meaning “fault-free software.”

The learner must always identify the exact criterion being measured.

---

## Chapter 6 — Input Space Partitioning

**Book pages:** 93–131
**Recommended time:** Three weeks including assessment

### Focus

Systematic input selection without requiring source-code access.

### Prerequisite bridge

Review:

* Sets
* Partitions
* Simple combinatorics
* Constraints among values

### Learning outcomes

The learner will be able to:

* Build interface-based input domain models.
* Build functionality-based input domain models.
* Define characteristics.
* Define partitions and blocks.
* Select representative values.
* Evaluate completeness.
* Evaluate disjointness.
* Evaluate relevance.
* Apply each-choice coverage.
* Apply pair-wise coverage.
* Apply base-choice coverage.
* Apply stronger combinations where appropriate.
* Handle constraints among characteristics.
* Identify impossible combinations.
* Derive tests from documentation such as JavaDoc.

### Continuing project example

Create an input domain model for a booking request containing:

* Check-in date
* Check-out date
* Guest count
* Room capacity
* Room status
* Room type
* Discount status
* Customer category

### Practice progression

#### Stage 1

Identify characteristics.

#### Stage 2

Create valid and invalid blocks.

#### Stage 3

Check each partition for:

* Completeness
* Disjointness
* Relevance

#### Stage 4

Select representative values.

#### Stage 5

Apply each-choice coverage.

#### Stage 6

Apply pair-wise or base-choice coverage.

#### Stage 7

Resolve impossible combinations.

#### Stage 8

Convert abstract test values into executable requests.

### Required artifact

`chapter-06-input-space-partitioning/`

It should contain:

```text
characteristics.md
partition-table.md
constraints.md
each-choice-tests.md
pairwise-or-base-choice-tests.md
executable-tests/
reflection.md
```

### Mastery evidence

The learner can:

* Explain why every characteristic exists.
* Explain why every block exists.
* Identify overlapping blocks.
* Identify missing blocks.
* Identify irrelevant blocks.
* Avoid impossible test combinations.
* Convert the model into concrete tests.

---

## Chapter 7 — Graph Coverage

**Book pages:** 132–218
**Recommended time:** Five weeks including assessment

### Focus

Structural and data-flow testing, with emphasis on reachability.

### Prerequisite bridge

Review:

* Directed graphs
* Nodes
* Edges
* Paths
* Subpaths
* Cycles
* Basic source-code control flow

### Learning outcomes

The learner will be able to:

* Define nodes and edges precisely.
* Distinguish paths and subpaths.
* Identify cycles and simple paths.
* Apply node coverage.
* Apply edge coverage.
* Apply edge-pair coverage.
* Apply prime-path coverage.
* Explain tours.
* Distinguish direct tours, sidetrips, and detours.
* Identify definitions and uses.
* Identify def-clear paths.
* Identify du-paths.
* Apply all-defs coverage.
* Apply all-uses coverage.
* Apply all-du-paths coverage.
* Derive graphs from source code.
* Derive graphs from designs.
* Derive graphs from specifications.
* Derive graphs from use cases.
* Derive graphs from state behavior.
* Identify infeasible paths.
* Compare graph criteria through subsumption.
* Map abstract graph paths to executable inputs.

### Week 9 — Graph foundations

Topics:

* Graph terminology
* Paths
* Cycles
* Simple paths
* Test paths
* Touring

Required output:

* Graph vocabulary sheet
* Basic path exercises

### Week 10 — Structural graph coverage

Topics:

* Node coverage
* Edge coverage
* Edge-pair coverage
* Prime-path coverage
* Subsumption relationships

Required output:

* Structural test requirements
* Test paths satisfying selected criteria

### Week 11 — Control-flow graphs

Topics:

* Deriving a graph from source code
* Branches
* Loops
* Entry and exit nodes
* Mapping paths back to inputs
* Infeasible paths

Required output:

* CFG for a booking-validation method
* Concrete tests for selected paths

### Week 12 — Data-flow coverage

Topics:

* Definitions
* Uses
* Def-clear paths
* du-paths
* All-defs
* All-uses
* All-du-paths

Required output:

* Data-flow table
* Data-flow test requirements
* Test paths

### Week 13 — Assessment and integration

Independent task:

1. Receive a new method or model.
2. Draw the graph.
3. Select a criterion.
4. Derive test requirements.
5. Create test paths.
6. Identify infeasible paths.
7. Map paths to test inputs.
8. Demonstrate coverage.

### Required artifact

`chapter-07-graph-coverage/`

```text
graph-terminology.md
structural-criteria.md
control-flow-graph.md
prime-path-analysis.md
data-flow-analysis.md
infeasible-paths.md
executable-tests/
coverage-evidence.md
```

### Mastery evidence

The learner can derive a justified test set from a new graph without guessing paths.

---

## Chapter 8 — Logic Coverage

**Book pages:** 219–291
**Recommended time:** Three weeks

### Focus

Testing predicates and clauses to support infection.

### Prerequisite bridge

Review:

* Boolean operators
* Truth tables
* Compound expressions
* Equivalence of expressions

### Learning outcomes

The learner will be able to:

* Distinguish predicates from clauses.
* Identify major and minor clauses.
* Apply predicate coverage.
* Apply clause coverage.
* Apply combinatorial coverage.
* Explain when a major clause determines a predicate.
* Apply active-clause coverage.
* Apply inactive-clause coverage.
* Find satisfying truth assignments.
* Identify infeasible logical requirements.
* Explain the effect of short-circuit evaluation.
* Recognize side effects in logical expressions.
* Apply logic testing to programs.
* Apply logic testing to specifications.
* Apply logic testing to finite-state machines.

Advanced topics may include:

* DNF
* Implicants
* Implicant coverage
* MUMCUT
* Karnaugh maps

These advanced topics should be studied after the main logic criteria are secure.

### Continuing project predicate

Consider a booking condition based on:

* Room availability
* Date validity
* Guest capacity
* Payment authorization

Example structure:

```text
available AND validDates AND sufficientCapacity AND paymentAuthorized
```

### Week 14 — Logic foundations

Activities:

* Identify predicates and clauses.
* Construct truth tables.
* Apply predicate and clause coverage.
* Compare coverage results.

### Week 15 — Active-clause coverage

Activities:

* Select a major clause.
* Fix minor clauses.
* Prove determination.
* Generate pairs of test requirements.
* Identify infeasible assignments.

### Week 16 — Application and assessment

Activities:

* Analyze program predicates.
* Analyze specification predicates.
* Convert logical requirements into executable tests.
* Explain expression rewriting and side effects.

### Required artifact

`chapter-08-logic-coverage/`

```text
predicate-analysis.md
truth-tables.md
predicate-and-clause-coverage.md
active-clause-coverage.md
infeasible-requirements.md
executable-tests/
```

### Mastery evidence

For every active-clause test pair, the learner can prove why changing the major clause changes the predicate.

---

## Chapter 9 — Syntax-Based Testing

**Book pages:** 292–354
**Recommended time:** Two weeks

### Focus

Grammar-based testing and mutation testing, with connections to propagation.

### Prerequisite bridge

Review:

* Regular expressions
* Grammar rules
* Terminals
* Nonterminals
* Derivations
* BNF-style notation

### Learning outcomes

The learner will be able to:

* Derive tests from regular expressions.
* Derive tests from BNF grammars.
* Generate valid strings systematically.
* Generate invalid strings systematically.
* Explain grammar coverage.
* Define mutants.
* Define mutation operators.
* Explain what it means to kill a mutant.
* Identify surviving mutants.
* Explain equivalent mutants.
* Use mutation to evaluate test-suite strength.
* Explain why mutation score is evidence rather than proof.
* Apply syntax testing to programs, specifications, integration, object-oriented software, and structured inputs.

### Week 17 — Grammar-based testing

Applied task:

1. Define a grammar for a booking command or structured request.
2. Generate valid examples.
3. Generate invalid examples.
4. Connect each example to a grammar requirement.
5. Convert examples to executable tests.

### Week 18 — Mutation testing

Applied task:

1. Introduce controlled mutations.
2. Run the test suite.
3. Identify killed mutants.
4. Analyze surviving mutants.
5. Investigate possible equivalent mutants.
6. Improve the test suite where justified.
7. Recalculate the mutation result.

### Required artifact

`chapter-09-syntax-and-mutation/`

```text
booking-grammar.md
valid-inputs.md
invalid-inputs.md
mutation-operators.md
mutation-results.md
surviving-mutant-analysis.md
test-suite-improvements.md
```

### Mastery evidence

The learner can distinguish:

* A killed mutant
* A surviving mutant
* An equivalent mutant
* A weak oracle
* Missing test input
* Unreachable mutated code

---

## Chapter 10 — Managing the Test Process

**Book pages:** 355–363
**Recommended time:** Combined with Chapter 11

### Focus

Testing throughout the software lifecycle.

### Learning outcomes

The learner will be able to:

* Plan testing during requirements.
* Plan testing during architecture.
* Plan testing during design.
* Plan testing during implementation.
* Plan testing during integration.
* Plan testing during deployment.
* Plan testing during operation.
* Plan testing during maintenance.
* Distinguish fault prevention from fault detection.
* Assign testing responsibilities.
* Define testing inputs and outputs.
* Apply MDTD as a team process.

### Applied task

Create a lifecycle testing matrix.

| Lifecycle stage | Testing objective | Input artifact | Activity | Owner | Output |
| --------------- | ----------------- | -------------- | -------- | ----- | ------ |
| Requirements    |                   |                |          |       |        |
| Architecture    |                   |                |          |       |        |
| Design          |                   |                |          |       |        |
| Implementation  |                   |                |          |       |        |
| Integration     |                   |                |          |       |        |
| Deployment      |                   |                |          |       |        |
| Operation       |                   |                |          |       |        |
| Maintenance     |                   |                |          |       |        |

### Mastery evidence

Testing activities begin with requirements and design rather than appearing only after implementation.

---

## Chapter 11 — Writing Test Plans

**Book pages:** 364–368
**Recommended time:** Combined with Chapter 10

### Focus

Creating useful testing documentation.

### Learning outcomes

The learner will be able to:

* Explain the purpose of a master test plan.
* Explain the purpose of a level test plan.
* Define testing scope.
* Define the testing approach.
* Identify required resources.
* Define responsibilities.
* Define the schedule.
* Identify test items.
* Identify risks.
* Define entry and completion criteria.
* Avoid vague statements that cannot guide testing.

### Applied task

Create a concise level test plan for the booking-service API.

The plan should include:

1. Test-plan identifier
2. Purpose
3. Scope
4. Features to test
5. Features not to test
6. Testing approach
7. Test environment
8. Test data
9. Responsibilities
10. Risks
11. Schedule
12. Entry criteria
13. Completion criteria
14. Deliverables

### Required artifact

`chapter-10-11-test-management/test-plan.md`

### Mastery evidence

Another tester can use the plan without requiring a verbal explanation from its author.

---

## Chapter 12 — Test Implementation

**Book pages:** 369–379
**Recommended time:** One week

### Focus

Making tests executable when the full system is unavailable or difficult to control.

### Learning outcomes

The learner will be able to:

* Choose an integration order.
* Justify an integration order.
* Explain the role of test doubles.
* Explain stubs in the chapter’s treatment.
* Explain mocks in the chapter’s treatment.
* Replace dependencies in a controlled way.
* Preserve the test objective while using doubles.
* Recognize that test doubles may contain faults.
* Recognize when a double makes a test unrealistic.
* State what a test using a double cannot establish.

### Applied task

Test a booking service that depends on:

* A booking repository
* A payment service
* A notification service

Choose an appropriate double for each test objective.

### Required artifact

`chapter-12-test-implementation/`

```text
integration-order.md
dependency-analysis.md
test-double-design.md
automated-tests/
limitations.md
```

### Mastery evidence

The learner can explain:

* Why the double is required
* What behavior it simulates
* What interaction is checked
* Which risks remain untested
* Why the test does not prove that the real dependency works

---

## Chapter 13 — Regression Testing for Evolving Software

**Book pages:** 380–383
**Recommended time:** Combined with Chapter 14

### Focus

Maintaining confidence as software changes.

### Learning outcomes

The learner will be able to:

* Explain why regression testing should be automated.
* Decide which tests belong in a regression suite.
* Select tests when the complete suite is too expensive.
* Prioritize tests.
* Classify regression failures.
* Distinguish intended behavior changes from regressions.
* Identify outdated tests.
* Update tests after requirement changes.
* Connect regression testing to version control and CI.

### Applied task

Organize project tests into:

1. Fast commit checks
2. Pull-request checks
3. Integration regression tests
4. System regression tests
5. Scheduled full-suite tests

For each group, explain:

* Purpose
* Trigger
* Expected duration
* Failure response
* Selection criteria

### Required artifact

`chapter-13-regression-strategy.md`

### Mastery evidence

The learner does not automatically delete every failing test or preserve every outdated expectation after a requirement changes.

---

## Chapter 14 — Writing Effective Test Oracles

**Book pages:** 384–393
**Recommended time:** Combined with Chapter 13

### Focus

Determining what correct behavior looks like and revealing failures.

### Learning outcomes

The learner will be able to:

* Define a test oracle.
* Connect oracles to revealability.
* Decide which output should be checked.
* Decide which state should be checked.
* Avoid checking too little.
* Avoid checking irrelevant implementation details.
* Determine expected values from specifications.
* Use redundant computation.
* Use consistency checks.
* Use metamorphic testing.
* Identify observability problems.
* Explain oracle limitations.

### Applied task

Create oracles for:

* Booking price
* Booking status
* Availability changes
* Payment result
* Confirmation behavior

Add:

* At least one consistency relation
* At least one metamorphic relation
* At least one negative assertion
* A justification for each assertion

### Required artifact

`chapter-14-test-oracles.md`

### Mastery evidence

For every assertion, the learner can explain:

* Why it exists
* Which behavior it observes
* Which possible fault it may reveal
* Whether it depends unnecessarily on implementation details

---

# 10. Continuing Hotel-Booking Project

The project should remain small enough to understand completely.

## Core features

### Booking request

* Check-in date
* Check-out date
* Guest count
* Room type
* Optional discount information

### Room behavior

* Capacity
* Availability
* Base price
* Room status

### Booking behavior

* Request validation
* Price calculation
* Booking creation
* Status transitions
* Cancellation rules

### Dependencies

* Booking repository
* Payment service
* Notification service

## Chapter integration

| Chapters | Project application                                                 |
| -------- | ------------------------------------------------------------------- |
| 1–2      | Define risks, artifacts, test levels, RIPR, and MDTD                |
| 3–4      | Automate tests and integrate them into development                  |
| 5–6      | Define criteria, characteristics, partitions, and test combinations |
| 7        | Build control-flow and data-flow models                             |
| 8        | Test compound booking predicates                                    |
| 9        | Generate structured inputs and evaluate tests using mutation        |
| 10–11    | Create the lifecycle process and test plan                          |
| 12       | Isolate dependencies with test doubles                              |
| 13       | Organize the regression suite                                       |
| 14       | Improve expected results and assertions                             |
| 22       | Combine all techniques in the final capstone                        |

---

# 11. Cumulative Review System

A short cumulative review should occur regularly.

## Review after Chapter 4

The learner should explain:

* Fault, error, and failure
* Verification and validation
* Testing and debugging
* MDTD
* RIPR
* Test design versus automation
* Testing-first workflows

## Review after Chapter 6

The learner should connect:

```text
Artifact
→ Model
→ Criterion
→ Test requirement
→ Test value
→ Executable test
```

The learner should then produce an input domain model independently.

## Review after Chapter 7

The learner should compare:

* Input-domain testing
* Structural graph testing
* Data-flow testing

The learner should explain when each model is useful.

## Review after Chapter 9

The learner should compare all four major test-design structures:

| Structure           | Best suited for                                        | Main artifact                |
| ------------------- | ------------------------------------------------------ | ---------------------------- |
| Input domains       | Selecting representative values                        | Partition table              |
| Graphs              | Exercising locations, transitions, and data flow       | Test paths                   |
| Logic expressions   | Exercising compound conditions                         | Truth assignments            |
| Syntax and grammars | Testing structured inputs and evaluating test strength | Generated strings or mutants |

## Final review

The learner should explain how the chapters collectively support:

* Test design
* Test implementation
* Test execution
* Test evaluation
* Regression
* Revealability

---

# 12. Assessment System

## After every chapter

Complete:

* Five terminology questions
* Three application questions
* One worked problem without viewing the solution
* One “teach it back” explanation
* One practical artifact
* One short reflection on mistakes

## Error log

Maintain an error log with these columns:

| Date | Chapter | Problem | My incorrect reasoning | Correct reasoning | Prevention rule |
| ---- | ------: | ------- | ---------------------- | ----------------- | --------------- |

The purpose is not merely to record wrong answers. It is to identify recurring reasoning problems.

## Knowledge target

A useful target is:

* At least 80% on terminology and understanding checks
* A correct independent solution to the chapter’s central problem
* A completed practical artifact
* A correct verbal or written explanation of the technique’s limitations

The percentage alone is not sufficient. A learner should not progress after memorizing answers without understanding the application.

---

# 13. Mastery Gates

## Gate 1 — Foundations

**After Chapter 5**

The learner must be able to explain:

```text
Software artifact
→ Testing model
→ Coverage criterion
→ Test requirements
→ Test values
→ Executable tests
→ Results
→ Evaluation
```

The learner must also distinguish:

* Test requirement
* Test case
* Test value
* Coverage criterion
* Coverage measurement

## Gate 2 — Input-space design

**After Chapter 6**

Given a new feature, the learner must be able to:

1. Identify characteristics.
2. Create partitions.
3. Check completeness.
4. Check disjointness.
5. Check relevance.
6. Select a combination strategy.
7. Generate valid test values.
8. Handle constraints.

## Gate 3 — Graph design

**After Chapter 7**

Given a new graph or method, the learner must be able to:

1. Identify nodes and edges.
2. Derive test requirements.
3. Select test paths.
4. Explain tours.
5. Identify infeasible paths.
6. Map paths to test values.
7. Demonstrate coverage.

## Gate 4 — Four testing structures

**After Chapter 9**

Given a new testing problem, the learner must decide whether the most useful model is:

* Input-domain based
* Graph based
* Logic based
* Syntax based

The learner must justify the choice.

## Gate 5 — Final integration

**After Chapter 14**

The learner must produce and defend:

1. A test plan
2. An input-domain model
3. A graph model
4. Graph-based test paths
5. Logic-based test requirements
6. Grammar-based or mutation analysis
7. Automated tests
8. Test-double usage
9. A regression strategy
10. Effective test oracles

---

# 14. Final Capstone Project

## Objective

Apply the complete testing process to one hotel-booking feature.

A suitable feature is:

> Create a booking after validating dates, room availability, capacity, price, discount eligibility, and payment authorization.

## Capstone stages

### Stage 1 — Risk analysis

Identify:

* Important failures
* Business consequences
* User consequences
* Technical risks
* High-priority behavior

### Stage 2 — MDTD mapping

Identify:

* Artifacts
* Models
* Criteria
* Requirements
* Test values
* Executable tests
* Evaluation method

### Stage 3 — Input space partitioning

Create:

* Characteristics
* Blocks
* Constraints
* Representative values
* Combination-based tests

### Stage 4 — Graph coverage

Create:

* Control-flow graph
* Selected structural criterion
* Test requirements
* Test paths
* Coverage evidence

Add a data-flow analysis where appropriate.

### Stage 5 — Logic coverage

Select an important compound predicate and produce:

* Clauses
* Truth table
* Coverage requirements
* Active-clause test pairs
* Executable tests

### Stage 6 — Syntax or mutation testing

Complete at least one:

* Grammar-based input design
* Mutation analysis

Completing both is preferred.

### Stage 7 — Test implementation

Use test doubles where dependencies are unavailable or must be controlled.

### Stage 8 — Regression design

Classify tests by:

* Speed
* Scope
* Importance
* Execution trigger
* Risk coverage

### Stage 9 — Oracle design

Justify every important assertion.

Include:

* Direct expected-value checks
* State checks
* Consistency relations
* Metamorphic relations where appropriate

### Stage 10 — Final defense

The learner should answer:

1. Why was each model selected?
2. Which criterion was applied?
3. Which requirements were infeasible?
4. What does the achieved coverage mean?
5. What does it not mean?
6. Which faults could the tests reveal?
7. Which risks remain?
8. Which tests belong in regression?
9. Which assertions are essential?
10. What are the limitations of the final suite?

---

# 15. Recommended Portfolio Repository

```text
software-testing-book-portfolio/
├── README.md
├── roadmap-progress.md
├── prerequisite-diagnostic/
│   ├── diagnostic-results.md
│   └── bridge-lessons.md
├── chapter-01-foundations/
├── chapter-02-mdtd/
├── chapter-03-automation/
├── chapter-04-testing-first/
├── chapter-05-criteria/
├── chapter-06-input-space-partitioning/
├── chapter-07-graph-coverage/
├── chapter-08-logic-coverage/
├── chapter-09-syntax-and-mutation/
├── chapter-10-11-test-management/
├── chapter-12-test-implementation/
├── chapter-13-regression/
├── chapter-14-oracles/
├── hotel-booking-project/
│   ├── src/
│   ├── tests/
│   └── documentation/
├── capstone/
│   ├── risk-analysis.md
│   ├── mdtd-map.md
│   ├── input-domain-model.md
│   ├── graph-model.md
│   ├── logic-analysis.md
│   ├── mutation-analysis.md
│   ├── regression-strategy.md
│   ├── oracle-design.md
│   └── final-reflection.md
└── revision-sheets/
```

---

# 16. Optional Supporting Tools

The book’s concepts must be learned independently of any particular framework.

The following tools may be used as supporting implementations.

> **Additional explanation: modern practical tools**

| Purpose                   | Possible tool                                          |
| ------------------------- | ------------------------------------------------------ |
| Unit-test automation      | JUnit 5                                                |
| Test doubles              | Mockito                                                |
| Code-coverage measurement | JaCoCo                                                 |
| Mutation testing          | PIT                                                    |
| Build automation          | Maven or Gradle                                        |
| Continuous integration    | GitHub Actions                                         |
| API testing               | JUnit, REST Assured, or another authorized test client |

The learner should always explain the testing concept before using the tool.

For example:

* JaCoCo may measure structural coverage, but it does not design the tests.
* PIT may generate mutants, but the learner must analyze surviving and equivalent mutants.
* Mockito may create a test double, but the learner must justify why the double is appropriate.

---

# 17. Progress Tracking

Use this table throughout the roadmap.

| Chapter | Terminology | Worked problem | Independent problem | Practical artifact | Mastery achieved |
| ------: | ----------- | -------------- | ------------------- | ------------------ | ---------------- |
|       1 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       2 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       3 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       4 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       5 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       6 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       7 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       8 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|       9 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|      10 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|      11 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|      12 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|      13 | ☐           | ☐              | ☐                   | ☐                  | ☐                |
|      14 | ☐           | ☐              | ☐                   | ☐                  | ☐                |

A chapter should only be marked mastered when all five columns are complete.

---

# 18. Final Mastery Standard

A chapter is complete only when the learner can:

1. Explain its central idea without copying the book.
2. Use its terminology correctly.
3. Solve a new example.
4. Produce the required testing artifact.
5. Connect the technique to MDTD.
6. Connect it to RIPR where relevant.
7. Explain its strengths.
8. Explain its limitations.
9. Convert abstract requirements into concrete tests.
10. Defend the testing decisions.

A learner who cannot complete the independent problem should review the specific weak prerequisite or subsection rather than rereading the entire chapter without a clear purpose.

---

# 19. Recommended Starting Procedure

Begin with the prerequisite diagnostic and Chapter 1.

Do not begin Chapter 2 until the learner can confidently distinguish:

* Fault
* Error
* Failure
* Testing
* Debugging
* Verification
* Validation

These concepts form the language used throughout the remaining chapters.

The first practical output should be a risk and fault-error-failure analysis for one hotel-booking feature.

The next lesson should then use that same feature to introduce the complete Model-Driven Test Design process.
