# Why write a Design Doc?

- Discussing possible implementations to answer a problem without actually building prototypes
- Building consensus around a design in the organization
- Knowledge sharing
- Act as an artifact that explains what engineers were thinking before solving the problem (and subsequently, if that changes).

## What is a Design Doc?

You're free to write them in any form you see fit, but here are some cookie cutter ideas to think about:

### Context and Scope

Try to answer these questions:

- What are we trying to build?
- Why are we building this?
- What is the problem we are solving with this design?

### Goals and Non-goals

Try to answer these questions:

- What are some must-haves of the system?
  - How much performance is expected?
  - How many users do we expect to use our system?
  - What features do our users really need?
- What are some things that we don't care about?
  - Does it matter that we allow API access?

## Design

In the design section, you'll want to write down the trade-offs you propose making.
Given the facts, goals and non-goals, try to discuss some possible solutions and show why one solution (the one you choose) is the best for the problem at hand.

### Diagrams

It is important to explain the context of the system with diagrams. Including a `system-context-diagram` which explains how the proposed solution fits in with other parts of the system is helpful.

### APIs

If the system has a callable API, it is a good idea to sketch it out. Try to avoid putting down an actual code interface, instead, try to provide ways how a user might use the API.

### Data Storage

If the system uses or processes data, it's a good idea to discuss how you'll plan to store the data, or otherwise process it, focusing on the tradeoffs of potential technologies/methodologies.

### Code

If this requires a novel approach to code up, a small prototype of an algorithm or explanation of how the system will work should suffice here.

### Degree of Constraint

Is this a greenfield software project, where you have free reign to do whatever you want? Is this project a rearchitecting of a previous system? If so, you'll want to explain which of the two it is, and how many resources you have for either, so you can focus on the necessary conditions to deem the project complete.

### Alternatives

You'll want to discuss other alternative solutions you and your team thought of and discuss why you selected the solution that is the topic of the Design Doc.

### Cross-cutting concerns

Discuss cross cutting concerns here, like security, privacy, ops load, and infrastructure burden. If you require a lot of buy-in from other teams, explain why you will and how you'll do that. If you're lowering the support burden for another team with this project (e.g. this new project lowers support tickets) you'll want to explain why this will do so and how much you expect the support burden to decrease.

### Length

Design docs can vary from 1-3 pages (a mini) to 20+ pages (a large feature set). Pick the length that fits most with your topic. Try to keep it as terse and succinct as possible, since that'll help your readers understand the problem and solutions.

## Lifecycle of a Design Doc

1. Rapid iteration
2. Review
3. Implementation and iteration
4. Maintenance and learning

### Rapid iteration

You write the doc with some co-authors, and work to flesh it out. When you've led the doc to a stable point, you'll want to get some reviewers on it.

### Review

You'll want to select a few people to review your design doc, and at their own leisure, giving them tools to provide feedback (e.g. on Google docs), or through version control. You can establish a mailing list or other pipeline at your company where senior members can review documents, and other members can propose them at any time, allowing for asynchronous communication.

### Implementation and iteration

When review has mainly wrapped up, you're ready to implement the system. Be prepared to make lots of changes to the design as you do this, since lots of things will pop up while implementing a system. If the facts change, change your mind. And the docs. You may want to add a note saying why this change was made and why a previous assumption was wrong in the doc as well, so people reading the doc afterwards can understand your frame of mind.

### Maintenance and learning

It is important to keep the Design doc in sync with the system -- it is an entry point for new engineers joining a team, and one of the first resources they will reach for (along with more structured documentation). Always look at your design docs a while later, so you can figure out what you got right, wrong, how to improve, and how to edit the design doc so it's helpful for readers in the future.

[Example](./example.md)
