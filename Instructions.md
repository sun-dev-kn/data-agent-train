Instructions

## Overview

This is an introductory training project for agentic coding projects on our platform. The main goals of this project are to:

-   Get yourself familiar with what these agentic coding projects typically involve
    
-   Give you an opportunity to try your hand at a simplified version of a typical agentic coding task
    

Completing this training will enable access to families of current and future agentic coding projects. We expect there will be a lot of available work on this front for the foreseeable future!

👉 **If you have already worked on agentic coding projects**, we still recommend that you do this training task as a refresher, and to help ensure you have access to future projects. We'd also be interested in your feedback on whether there are additional topics you think could be helpful to cover in future versions of this training project, e.g., things that you find confusing or challenging in real agentic coding projects that aren't covered in the training task.

### What is agentic coding?

When a model is acting as a coding agent, it doesn't just provide a simple text response in response to a prompt. It has access to tools that enable it to read files, write code, run tests, and make multi-step decisions over time to try to complete a given task, possibly using context from a provided codebase. The model is essentially operating and making a series of choices on its own. You may already have experience working with agentic coding tools, for example through tools like Claude Code, Cursor, and Codex.

Ideally, a coding agent would be able to **act as an autonomous software engineer**. However, in practice, models are not consistently proficient or trustworthy enough to serve this purpose. This is where you come in!

#### Your role: creating problems that challenge frontier models

Many of the agentic coding projects we run are designed to produce training problems and environments that challenge frontier models. The high-level idea is as follows:

1.  There is a starting environment that the coding agent operates in, usually with an existing codebase.
    
2.  The agent is given a problem to solve that requires it to understand the environment, perform multi-step reasoning, and make modifications to the codebase.
    
3.  The agent's solution attempt is scored through verifiers, which can take various forms. Most commonly, these will be rubrics, test suites, and/or rating axes.
    
4.  The scores over different attempts and different tasks can be used as a reward signal to train the model to improve on its deficiencies.
    

In order to generate a meaningful reward signal, we will typically need problems where the model of interest consistently exhibits meaningful failures. There are several potential pitfalls that need to be avoided when creating problems:

-   **Problem difficulty and feasibility:** Problems need to be solvable, but difficult enough to elicit failures.
    
    -   Problems that are too easy are not useful, since there's no usable reward signal if the model typically solves the problem without issues.
        
    -   Similarly, problems that are impossible are not useful, since they would also provide limited reward signal.
        
-   **Verifier alignment:** If the verifiers are misaligned with the problem, the model may be improperly rewarded or penalized. **This is an area where we commonly see issues in submissions in real projects.** For example:
    
    -   Verifiers can be **overspecified**, meaning they impose requirements on solutions that are not actually stated or implied by the problem statement.
        
        -   This can mean that the verifiers are overly tailored to a particular set of possible solutions, and we end up improperly penalizing good solutions.
            
        -   In an extreme case, if there is a verifier that contradicts the problem statement, we end up penalizing all good solutions and rewarding some bad solutions.
            
        -   Simple example: suppose the verifiers include tests that expect the model to write a new function named `foo`, but the prompt does not specify that the function should be named `foo`. A correct solution that names the function `bar` would improperly fail the tests.
            
    -   Verifiers can be **underspecified**, meaning they are missing requirements from the problem statement.
        
        -   This can mean that the verifiers are overly permissive and could allow flawed solutions to pass.
            
        -   Simple example: suppose the problem explicitly specifies that a particular function that the agent will write should be named `foo`. If the verifiers do not check that the function is named `foo`, then a solution that names the function `bar` could pass all the verifiers, even though it has not followed all the instructions from the problem statement.
            
-   **Environment reproducibility:** The starting environment needs to be fully reproducible, meaning that anyone (or any automated system) who runs the agent against the problem should get the same starting conditions.
    
    -   If the environment can't be reliably reconstructed, then failures you observed when creating the problem may not be replicable, and verifiers may produce inconsistent results across runs.
        
    -   Possible pitfalls here include:
        
        -   **Missing dependencies:** If key dependencies are missing from the environment, the problem may not be usable. It is commonly the case that in training environments, models will not have network access post-build and will therefore not be able to download and install missing packages after the initial environment setup.
            
        -   **Dependency drift:** If your environment relies on packages or tools without pinned versions, a different version may be installed in a later run, changing the behavior of the codebase or the verifiers.
            
        -   **Hardcoded local paths or credentials:** References to paths, usernames, or secrets specific to your machine will break when the environment is run elsewhere.
            
    -   The most common way to guarantee reproducibility is to package your environment as a Docker container. A Dockerfile that installs all dependencies, copies in your codebase, and sets up the working environment ensures that the exact same starting state can be reconstructed anywhere.
        

#### What's covered in this training project

In this training project, we'll walk through the steps of an agentic coding task with some exercises for you to practice on:

1.  Codebase sourcing
    
2.  Problem creation
    
3.  Verifier creation
    
4.  Dockerization
    

The exact workflow will not necessarily reflect that of real project tasks, where you may be provided with less explicit guidance, have different task requirements, be using different models and agentic harnesses, etc., but this should give an idea of what real tasks entail.

#### Time expectation

-   You do not need to do the whole task in one sitting. For this training project, it's OK to submit after the timer expires. We don't have any plans to take this training project down, so you're not at risk of spending time doing part of the task and then losing the project before you can submit. (If we do need to pause the training project, we'll send out a notice in advance.)
    
-   **However, note that after the timer expires, you will lose access to the provided ZIP file that's used in several parts of the task.** We recommend that you download [the ZIP file](https://app.dataannotation.tech/publish/s/3f8ed6a9-e917-4990-b611-67774714b6f4.zip) now and save it, in case the timer has expired by the time you do the later parts of the task.
    
-   We expect the task to take roughly 3-7 hours.
    
    -   If you are not on track to finish the task within this time range, skip ahead.
        
    -   You can enter dummy answers in required questions as needed and note in the optional comment at the end what parts you needed to skip. (Note though that this is generally not allowed on actual projects! It's only because this is a training task that we're allowing it here.)
        
    -   **Do not spend an excessive amount of time on this training task**.
        
-   **Update (2026-05-19):** There's been a change made that will require you to have a non-expired task in progress for your provided API key to work. For this project only, we have set up the project such that you can exit Work Mode and then re-enter Work Mode to reset the timer. (It is not safe to do this on other projects, where you may lose your work by exiting Work Mode.)
    

**If you have a question, first check if it's already covered in the FAQ below. We are getting a lot of repeat questions that are already answered in the FAQ.**


### Part 1: Codebase sourcing

Real tasks can vary in whether you will need to source a codebase for them.

-   Some tasks will have the model generate code from scratch, in which case there is no starting codebase.
    
-   In other tasks, you will be provided with a starting codebase and will not need to source one yourself.
    
-   But often, you will need to provide a starting codebase that the prompt you create will apply to and that the agent will act on.
    

In this training task, you will source a codebase. Specifically, please select an open-source repo from GitHub that has one of the following permissive licenses:

-   MIT
    
-   Apache-2.0
    
-   BSD-3-Clause
    

When selecting a repo, keep the following in mind:

-   **Language:** Pick a language you're comfortable reading and writing, since you'll need to understand the codebase well enough to write a problem and assess modifications to the codebase.
    
-   **Scope:** You will likely find it easier to elicit a failure from the agent later on if the codebase has more complex interdependencies and is generally larger.
    
-   **Testability:** The codebase should have behavior that can be verified programmatically. Repos that already have an existing test suite are a good sign — it means the code has testable interfaces. Avoid repos that are primarily configuration, documentation, or tooling with no meaningful runtime behavior.
    
-   **Buildability:** In order to run tests it may be necessary to build the software. Make sure that you're aware of any build process necessary for testing and how long it takes. Avoid repos with lengthy build processes (more than 10 minutes).
    

#### Before committing to a repo, ask yourself:

-   Can I think of two or three realistic coding tasks an engineer might actually do in this codebase — e.g., adding a feature, fixing a bug, refactoring a pattern, tightening a check?
    
-   Are there testable interfaces (pure functions, a CLI, a web API) that would reveal a bug programmatically, not just visually?
    
-   Can I run the existing test suite right now and have it pass cleanly, with no flaky tests or environment-specific failures?
    
-   Does the codebase have interdependencies across multiple files or modules, not just a single-file utility?
    
-   Can I build and run it on my machine right now, not "after I figure out a weird build step"?
    

If you can't answer yes to most of these, the repo will likely bite you later. Pick a different one.

**Strong examples:**

-   A convention-heavy library like `pandas`, `requests`, `click`, or `scikit-learn` — clear public APIs, lots of testable behavior, sibling-pattern enforcement
    
-   A CLI tool like `black`, `ruff`, or `httpie` — deterministic output, fast tests, well-defined interface
    
-   A small web framework like `Flask`, `Starlette`, or `FastAPI` — strong internal conventions, plugin and registry patterns
    
-   A data-validation or data-processing library like `pydantic`, `marshmallow`, or `jsonschema` — invariants to preserve, round-trip semantics
    

**Weak examples:**

-   A documentation-only repo (no runtime logic)
    
-   A tutorial or example repo with a single-file hello-world
    
-   A static site or infrastructure-as-code repo (Terraform, Ansible)
    
-   A thin wrapper around an external API with no significant own-logic
    
-   A repo that's primarily data, fixtures, or model weights
    

**`Note:`** `These examples are for guidance. You may exercise creativity in choosing the repo.`

_Part 1: Codebase sourcing_

#### Repo URL

Enter the base GitHub repo URL (e.g., [https://github.com/pandas-dev/pandas](https://github.com/pandas-dev/pandas)) for the repo you selected.

_Part 1: Codebase sourcing_

#### Repo License

Which license does this repo have?

MIT

Apache-2.0

BSD-3-Clause

Other similarly permissive or more permissive license

_Part 1: Codebase sourcing_

#### Commit SHA

To ensure reproducibility, we will often need to get the SHA for the particular commit that you are using as the starting codebase. For example, if the project requires the Dockerfile to clone the repo to set up the starting environment, it will need a SHA to ensure it is checking out the intended version of the repo.

To find the full SHA:

1.  Go to the commits page for the repo URL you entered above (by appending `/commits/` to the URL).
    
2.  Decide which commit you will use.
    
    -   You can use the most recent commit, but you don't necessarily need to.
        
    -   For example, you can choose an older commit from before a significant feature addition or bug fix, and then later base your problem on having the agent implement the feature addition or bug fix.
        
3.  The row for the commit will display a 7-character abbreviated SHA. Click the copy button to the right of the abbreviated SHA to copy the **40-character** **full SHA** for the commit.
    

Paste the full **40-character full SHA** below.

_Part 1: Codebase sourcing_

#### Local Repo Clone

The rest of the task will involve working with the repo locally. You can run the following commands (with the placeholders filled in) to get a clean starting version of the repo for the task:

```bash
git clone <repo-url>
cd <repo-name>
git checkout <commit-sha>
rm -rf .git
```

-   Remove .git before running the checker or coding agent. This prevents carrot/celery from reading later commit history where the real fix may have landed.
    
-   If you don't already have Git installed, see [https://git-scm.com/install/](https://git-scm.com/install/)
    
-   If you are on Windows, we recommend that you use WSL.
    

-   Repo URL entered above:
    
-   Commit SHA entered above:
    

_Note: After removing .git, git commands such as git diff and git status will no longer work in that copy of the repo. If you want a pristine starting state again later, reclone the repo from the URL and SHA above._

### Part 2: Problem creation

Next, you will create a problem based on your selected codebase and iterate as needed to get a failure from a model. Review the guidelines below to understand what we're looking for in a problem, and then the workflow to understand the steps you should take to produce a problem to meet the guidelines.

### Guidelines

1.  The problem should be **realistic** and representative of a real-world software engineering problem. It should not include unnatural requirements, like "only use variable names that have an odd length".
    
2.  In order to solve the problem, the agent should have to **explore, understand, and modify the codebase**.
    
3.  Requirements in the problem statement should generally correspond to behaviors that are **programmatically testable**. Avoid requirements that are not testable.
    
4.  Between the problem statement and the codebase, the agent should have **sufficient information** to solve the problem. It should not need to make guesses to fill information gaps.
    
5.  The agent should exhibit a **meaningful failure**. For this task, this means that its solution attempt fails to satisfy at least one testable requirement that is stated or implied by the problem statement.
    

Common types of problems include requests for new features, code modifications, bug fixes, and/or refactoring.

### Workflow Summary

1.  Write a problem statement that satisfies guidelines 1-4 above.
    
    -   We don't necessarily expect that your initial draft will be perfect.
        
    -   You will use an agentic checker to check your problem against guidelines 1-4, which will provide feedback and possibly suggestions for improvement.
        
    -   Iterate as needed until you have addressed anything that was validly flagged by the agentic checker.
        
2.  Have a provided custom model generate a solution to the problem locally using **OpenCode**.
    
3.  Assess the solution to see if it fully satisfies the problem's requirements.
    
4.  If it fully satisfies the problem's requirements, return to step 1 and revise the problem to make it more complex.
    
5.  If there is a meaningful failure, then all the guidelines above have been satisfied, so continue to the next part of the task.
    

_Part 2: Problem creation_

#### Problem Statement

Enter your problem statement below.

-   Keep the **<u><a href="/workers/tasks/be6dc699-c848-4451-a0dc-a52413c94e2c?project_id=7ac4e19b-5e27-4253-995b-a685126762a4&amp;task_response_id=372cfe7e-a0f5-4e42-90a6-fba5e5a3b0fb&amp;jumpTo=bookmark%3Aproblem_creation_guidelines"><span>problem creation guidelines</span></a></u>** in mind while you are formulating the problem.
    
-   You will run an agentic checker to help you verify that guidelines 1-4 are met.
    
-   You won't have enough information to verify that guideline 5 (meaningful failure) is satisfied at this point yet, but you can err on the side of making the problem relatively more difficult/complex to have a better chance of getting a failure from the agent. See the tips below.
    
-   Later, if you need to iterate on the prompt, make sure to update this field with the latest version of your problem statement.
    

### Tips for creating sufficiently challenging problems

-   **Require deeper codebase understanding** — make the problem unsolvable without the agent first reading and comprehending more of the existing code, e.g. by requiring it to extend an existing pattern, respect an existing convention, or integrate with logic it needs to discover
    
-   **Increase the scope of the request** — e.g., instead of asking for a single new function, ask for a new feature that requires changes across multiple functions, files, or components
    
-   **Target edge case behavior** — models may be more likely to fail when the problem requires correct handling of boundary conditions, unusual inputs, or interactions between components that a straightforward implementation would overlook
    
-   **Add constraints (but keep them realistic)** — e.g., performance requirements, restrictions on which parts of the codebase can be modified, requirements that existing behavior and interfaces must be preserved
    

**Note:** We do not want to make the problems more "difficult" by introducing ambiguity or contradictory requirements, since that makes it more difficult or impossible to verify solutions.

_Part 2: Problem creation_

#### OpenCode Setup

You will be running agents in OpenCode for the rest of the task, both to generate solution attempts for your problem and to run checks on your work.

1.  Install OpenCode — see [here](https://opencode.ai/docs/#install). As mentioned there, it's recommended that you use WSL if you are on Windows.
    
    -   Avoid using the desktop app. It lacks some features that are necessary for these projects.
        
    -   (optional but recommended) The agents in this project can run commands on your computer, in some cases without asking first, such as installing packages. While OpenCode has some permissions safeguards against this (for example, if it tries to modify files outside of the project folder), to keep your system safe, we recommend running OpenCode inside an isolated environment, such as a Docker container or a virtual machine.
        
        -   If you use Docker, mount only your project folder into the container (not your home directory), then install OpenCode and the repo's dependencies inside the container. For example, from your project folder:
            
            `docker run -it --rm -v "$PWD":/workspace -w /workspace ubuntu bash`
            
        -   Whenever you run OpenCode sessions or code produced from the sessions, you can run in the container for isolation.
            
2.  Download [this ZIP file](https://app.dataannotation.tech/publish/s/3f8ed6a9-e917-4990-b611-67774714b6f4.zip) and extract its contents in the root of your project (the `<repo-name>` directory). All of the following must be **`placed directly in the root directory`** (not nested inside a subdirectory of the root directory).
    
    -   `opencode.json` is a custom configuration file that will let you use two different models in this project:
        
        -   `celery` — This is the relatively weaker model that you will use to generate a solution to your problem and elicit a failure from. We'll refer to this as the **coding agent/model**.
            
        -   `carrot` — This is the relatively stronger model that you will use as a helper at various points in the task. We'll refer to this as the **checker agent/model**, and then later the **test generation agent/model**.
            
    -   `.opencode` contains the helper agent skills. **Note that some operating systems may treat this as a hidden directory, so it may not be visible by default! If you don't see** `.opencode`**, YOU NEED TO EXTRACT THE ZIP'S CONTENTS, NOT JUST COPY THE VISIBLE FILES. This is one of the most common issues asked about in the project chat.**
        
    -   `problem.txt` is an empty file. You will later paste your problem into it.
        
3.  Run `opencode` from the project root.
    
4.  Run `/connect` from the OpenCode TUI.
    
5.  Search for the `DataAnnotation` provider and select it.
    
6.  Enter the following as your API key (unique to your account):
    
    sk-plwkr01-T9VHPRPKWPXX46V64A97MZFDAMXHC9DP4GGXFQXPJDA6YW2G
    
7.  You should see `celery` and `carrot` as the available models. Choose one and send a test prompt to verify that you get a response.
    

Useful commands to know:

-   `/models` — This will let you switch the model that you're using.
    
-   `/new` — This will start a new conversation, without any previous context.
    

_Part 2: Problem creation_

#### Problem Guideline Checker

Next, we'll use `carrot` as an agentic checker for your problem.

1.  Start a new conversation in OpenCode and set `carrot` as the model.
    
2.  Paste your problem statement into `problem.txt`.
    
    Copy problem statement above
    
3.  Submit "Run the problem-checker skill" as the prompt in OpenCode. This should trigger the checker agent.
    
    -   It will evaluate your problem statement and the provided codebase context to flag potential improvements to get the problem to follow **<u><a href="/workers/tasks/be6dc699-c848-4451-a0dc-a52413c94e2c?project_id=7ac4e19b-5e27-4253-995b-a685126762a4&amp;task_response_id=372cfe7e-a0f5-4e42-90a6-fba5e5a3b0fb&amp;jumpTo=bookmark%3Aproblem_creation_guidelines"><span>guidelines 1-4</span></a></u>**.
        
    -   You will see its findings in OpenCode, and the checker agent will also write its findings to `problem_checker_results.md`.
        
    -   If you don't see something like "Skill: problem-checker" after you submit "Run problem-checker", that most likely means that the checker agent hasn't been triggered. Make sure that you have followed the setup steps above and that `.opencode\skills\problem-checker\SKILL.md` exists.
        
4.  Review the checker agent's findings.
    
    -   The checker may not always produce valid or correct findings. You can use your judgment to decide whether each issue that it flags needs to be addressed.
        
    -   But for any issue that it flags, you should either make a change to the problem statement to address the issue or have a reason for why the issue is spurious or doesn't need to be addressed.
        
5.  Revise the **<u><a href="/workers/tasks/be6dc699-c848-4451-a0dc-a52413c94e2c?project_id=7ac4e19b-5e27-4253-995b-a685126762a4&amp;task_response_id=372cfe7e-a0f5-4e42-90a6-fba5e5a3b0fb&amp;jumpTo=bookmark%3Aproblem_statement"><span>problem statement</span></a></u>** as needed and go back to step 1 of running the checker agent, iterating until the checker agent finds no issues that you think are valid and need to be addressed.
    

Once your problem statement has passed the checker agent's evaluation, upload the final `problem_checker_results.md` here.

Drag a file here, click here to browse, or paste it in here

All file formats accepted

Does the final version of `problem_checker_results.md` you uploaded indicate that your problem statement passes on guidelines 1-4?

Yes, there are no outstanding flagged issues

No, the problem checker results still have issues flagged, but I don't believe they need to be addressed

_Part 2: Problem creation_

#### Running the Coding Agent

**_Note:_** _After you finish iterating with the problem checker, remove problem\_checker\_results.md from your directory before switching to the coding agent. This prevents the coding model from reading the checker’s earlier feedback instead of relying only on problem.txt and the codebase._

Next, to finalize the problem, we need to fulfill guideline 5 — the problem needs to be difficult enough that the coding agent (`celery`) fails to satisfy all of the prompt's requirements. To test this, generate a solution as follows:

1.  Change the model in OpenCode to `celery`.
    
2.  Confirm that `problem.txt` contains the latest version of your problem statement.
    
3.  Submit "Solve the problem in `problem.txt`" as the prompt in OpenCode. This should trigger the coding agent to generate a solution to your problem.
    
4.  Let the coding agent do its work. If it asks for permission to edit or run code, you can grant it (assuming it's not trying to do something unsafe, e.g. deleting files outside the project directory).
    
5.  Once the agent has finished, review its changes against the requirements from the problem statement. You can do this by:
    
    -   Reading the agent's edits directly in the codebase
        
    -   Checking for errors in the agent's reasoning shown in OpenCode
        
    -   Running the modified code to observe its behavior
        
6.  If you find at least one requirement that the agent failed to satisfy, the problem is good, and you can move on to the next part of the task. If the agent fully satisfied all requirements, return to the **<u><a href="/workers/tasks/be6dc699-c848-4451-a0dc-a52413c94e2c?project_id=7ac4e19b-5e27-4253-995b-a685126762a4&amp;task_response_id=372cfe7e-a0f5-4e42-90a6-fba5e5a3b0fb&amp;jumpTo=bookmark%3Aproblem_statement"><span>problem statement</span></a></u>** section, revise the problem to make it more difficult and/or complex, and go through the steps to recheck the revised problem against the problem creation guidelines.
    
    -   Note that for this training task, we're only looking at the completeness and correctness of the coding agent's solution. We are not considering other elements of the agent's performance, such as code style, code efficiency, or reasoning efficiency, though these may be in scope in real agentic coding projects.
        
    -   Failures are not always obvious. For example, the agent may have failed to consider and handle certain edge cases that are not explicitly described in the problem statement but are implied.
        

👉 Creating a sufficiently challenging prompt is often one of the most difficult parts of agentic coding projects, so it is not unexpected if you need to iterate on the problem.

⚠️ If you find that you are unable to elicit a failure after several attempts, you can continue without having met guideline 5. But note that this is often not a requirement you can bypass in real agentic coding projects, so we recommend that you at least make a few attempts to make your problem more challenging if you're not getting a failure.

**For this training task, do not spend an excessive amount of time iterating.** We provide the "No, I was not able to elicit a failure even after several attempts to make the problem harder" option in the question below so that you can move on even if you aren't able to meet guideline 5. As a rough estimate, if you have hit 5 hours on the task without getting a failure, you should move on.

👉 If you revise the problem and rerun the coding agent, start from a fresh copy of the repo at the chosen commit SHA. Do not continue from the previously modified working directory.

Were you able to elicit a failure from the coding agent with the last version of your problem?

Yes, the coding agent failed to satisfy one or more requirements

No, I was not able to elicit a failure even after several attempts to make the problem harder

### Part 3: Verifier creation

Now that we have a codebase and problem, we want to create a set of verifiers that can be used to evaluate any arbitrary coding agent solution to the problem. The most common types of verifiers are:

-   **Test suites**: Programmatic tests that verify the correctness and completeness of the codebase's end state. These could be unit tests, integration tests, end-to-end tests, etc.
    
-   **Rubrics**: Natural language conditions that describe the intended behavior and characteristics of the code, as well as possibly the agent's reasoning and tool use.
    
-   **Rating axes:** Different dimensions that the agent's performance is measured on, with ratings to indicate the presence and severity of issues along each dimension.
    

Different projects can use different types of verifiers; some will use just one type, while others will use more than one. There can also be a lot of variation within these types from project to project.

For this training task, we will only be creating a **test suite**, since we are just focusing on the correctness and completeness of the coding agent's code edits.

In real projects, you will usually be allowed to use a coding agent to help write tests. However, **it is your responsibility to ensure that the tests are fully aligned with the problem**. As a reminder, this means that:

-   The tests are not **overspecified**. I.e., they should not impose requirements on solutions that are not actually stated or implied by the problem statement.
    
-   The tests are not **underspecified**. I.e., they should cover all requirements stated or implied by the problem statement.
    

### Workflow Summary

1.  Create a clean copy of your starting repo, without the changes applied by the coding agent in the previous part of the task.
    
2.  In the clean copy, use a provided test generation agent (same model as the checker agent, `carrot`) to generate a test suite based on the prompt.
    
3.  Review the generated tests to check for misalignment with the problem.
    
4.  Modify the tests as needed to fix any misalignments.
    
5.  Copy the test files into the version of the codebase that has the edits made by the coding agent.
    
6.  Run the tests and confirm that the results are as expected.
    

_Part 3: Verifier creation_

#### Clean Copy of Starting Repo

We want the test generation agent to generate tests without knowledge of the coding agent's solution, to lower the risk of the test generation agent over-indexing on the coding agent's solution and thereby producing overspecified tests. We therefore want to run the test generation agent in a clean copy of the starting environment.

1.  Run the following again in a new directory:
    

```bash
git clone <repo-url>
cd <repo-name>
git checkout <commit-sha>
rm -rf .git
```

-   Repo URL entered above:
    
-   Commit SHA entered above:
    

2.  Download [this ZIP file](https://app.dataannotation.tech/publish/s/3f8ed6a9-e917-4990-b611-67774714b6f4.zip) again (same file as earlier) and extract its contents into in the root of your project (the `<repo-name>` directory).
    
3.  Paste your problem statement into `problem.txt` (or copy `problem.txt` over from the project directory that the coding agent had been operating in).
    
    Copy problem statement above
    

_Part 3: Verifier creation_

#### Running the Test Generation Agent

1.  Run `opencode` from the new clean project directory that doesn't have the coding agent's edits.
    
2.  Set `carrot` as the model.
    
3.  Submit "Run the test-generation skill" as the prompt in OpenCode. This should trigger the test-generation agent.
    
    -   If you don't see something like "Skill: test-generation" after you submit this prompt, that most likely means that the test generation agent hasn't been triggered. Make sure that you have followed the setup steps above and that `.opencode\skills\test-generation\SKILL.md` exists.
        

After the test generation agent has finished running, there should be a `da_training_project_tests\` directory added to the project directory containing the generated test suite.

_Part 3: Verifier creation_

#### Test Misalignment Check

Review the generated tests against your problem statement. Are there any instances of misalignment between the tests and problem statement? E.g.:

-   Tests that are too narrow, imposing requirements and assumptions that are not stated or implied by the problem statement
    
-   Incorrect tests that require solutions to display behavior that contradicts the problem statement
    
-   Gaps in test coverage, meaning that some requirements from the problem statement are not being tested
    

You can copy the tests to the other project directory with the agent's solution and run the tests on that solution, in case it's helpful for identifying misalignment.

Yes, there is misalignment

No, the tests are fully aligned

_Part 3: Verifier creation_

#### Running the Tests on the Coding Agent's Solution

Finally, we want to run the tests on the coding agent's solution.

1.  Copy the `da_training_project_tests\` directory into the project directory that the coding agent worked on (if you haven't already done so).
    
2.  Run the tests.
    
3.  Confirm that the test results are as expected:
    
    -   If you had elicited a failure from the coding agent, you should see at least one test fail, and the failed tests should correspond to the failure(s) you described earlier.
        
    -   If you could not elicit a failure from the coding agent, you should see all tests pass.
        
4.  If the test results are not as expected, this may indicate that there is further misalignment in the tests. Revise the tests to correct any misalignment and rerun the tests.
    
5.  Once the test results align with what you expect from the coding agent's solution, upload a screenshot of the test results here.
    

Drag a file here, click here to browse, or paste it in here

All file formats accepted

### Part 4: Dockerization

In order to create tasks that are reproducible, Docker containers are commonly used. This allows for the same starting state to be reconstructed anywhere. You will typically provide the Docker build context as a ZIP file containing a Dockerfile and any other additional files needed to build the container.

**For the sake of simplicity in this training task, rather than Dockerizing the task you made earlier, you will Dockerize a separate, simple example project.** In a real task, you would be Dockerizing the codebase and verifiers from the earlier parts of the task.

_Part 4: Dockerization_

### 📘 Recommended reference: Docker Survival Guide

Before starting the practical exercise parts below, we recommend working through the [**Docker Survival Guide**](https://projects-uploaded-files.s3.us-east-2.amazonaws.com/production/item_response_files/_2f92ce3c-c44b-4270-a5f4-04785bb3b705_eff896c4-48b4-4b3a-99ff-7f54c577a714.html) — an interactive walkthrough with 12 challenges covering dependency pinning, startup scripts, debugging inside containers, and the common pitfalls we see in real agentic coding projects.

It takes about 20–30 minutes. Save the guide — it is a useful reference for real projects too.

_Part 4: Dockerization_

#### Practical Exercise: Part 1: Containerize an Application

In this exercise, you will build and run a container for a small Node.js todo-list app. The later steps modify the container you create here, so complete this one first. You do not need any prior JavaScript experience.

Note that this Part 4 exercise is independent from the earlier parts. In a real project, you would be responsible for Dockerizing the starting environment for your problem, but in the interest of simplifying for learning purposes, we'll be Dockerizing a toy project.

**Note:** This part originally pointed to the Docker tutorial page [here](https://web.archive.org/web/20260821061247/https://docs.docker.com/get-started/workshop/02_our_app/), which was removed from the Docker website on 8/29/26. You can reference this archived snapshot of the webpage as needed, but the key steps that it contained have been transcribed in steps 1-5 below.

**Before you start:** make sure you have Docker Desktop, a Git client, and a text editor or IDE installed.

##### 1\. Get the app source

Clone the sample repository:

```bash
git clone https://github.com/docker/getting-started-app.git
```

The cloned directory should contain `.dockerignore`, `package.json`, `package-lock.json`, `README.md`, `spec/` and `src/`.

##### 2\. Write a Dockerfile

Inside `getting-started-app/`, in the same folder as `package.json`, create a file named `Dockerfile` (no file extension) containing:

```sql
# syntax=docker/dockerfile:1
FROM node:24-alpine
WORKDIR /app
COPY . .
RUN npm install --omit=dev
CMD ["node", "src/index.js"]
EXPOSE 3000
```

That starts from a small Linux image with Node.js preinstalled, sets `/app` as the working directory, copies the source in, installs the production dependencies, sets the command that starts the app, and documents that it listens on port 3000.

##### 3\. Build the image

From inside the `getting-started-app` directory:

```
docker build -t getting-started .
```

`-t` tags the image with a readable name, and the trailing `.` tells Docker to look for the Dockerfile in the current directory. The first build downloads the base image, so it may take a minute.

##### 4\. Run a container

```css
docker run -d -p 127.0.0.1:3000:3000 getting-started
```

`-d` runs the container in the background and returns you to the prompt. `-p` maps `HOST:CONTAINER`, publishing the container's port 3000 to `localhost:3000` on your machine — without it you could not reach the app from the host.

##### 5\. Check that it works

Open `http://localhost:3000` in your browser. Add a couple of todo items and confirm you can mark them complete and delete them. Running `docker ps` should list a container using the `getting-started` image on port 3000.

In the following steps you will make some changes to this container to demonstrate your understanding.

_Part 4: Dockerization_

#### Practical Exercise: Part 2: Change the Dockerfile to clone the repo

If you completed the above workshop correctly, you should have a directory containing the getting-started-app repo and a Dockerfile. In this step you will change the Dockerfile to clone the repo from git instead of including it in the container itself.

-   Delete all files in the directory except for the Dockerfile.
    
-   Edit the Dockerfile so that it clones the repository from git (i.e., [https://github.com/docker/getting-started-app](https://github.com/docker/getting-started-app.git))
    
    -   Hint: use the RUN command to install git and clone the repo.
        
-   Remember to build and run the Docker container to test that it works correctly!
    

_Part 4: Dockerization_

#### Practical Exercise: Part 3: Add additional files to the container

It will often be necessary to include additional non-repo files to containers (tests, configuration, data etc.). To demonstrate you're able to do this, add the contents of [this ZIP file](https://app.dataannotation.tech/publish/s/3f8ed6a9-e917-4990-b611-67774714b6f4.zip) (same file as earlier) to the container.

1.  Extract the contents of the ZIP file into your working directory. Your directory should now look like this:  
    

`your-working-directory/`

`├── .opencode/ ← extracted from the zip`

`├── Dockerfile ← your Dockerfile from Part 2`

`├── opencode.json ← extracted from the zip`

`└── problem.txt ← extracted from the zip`

2.  Modify the Dockerfile so that all files **except** the Dockerfile itself are copied when the container is built.
    

_Part 4: Dockerization_

#### Practical Exercise: Part 4: Pin your dependencies

Reproducibility requires that everyone who builds your container gets the exact same installed package versions. Without pinning, a rebuild a few months later might pull a different version of a library and break your environment — a common source of "works on my machine" pain on real agentic coding projects.

The Docker Workshop's `getting-started-app` is a Node.js project. The simplest form of pinning for Node is using `npm ci` instead of `npm install`: `npm ci` installs the exact versions recorded in `package-lock.json`, while `npm install` resolves version ranges fresh each build and may drift.

You deleted `package-lock.json` from your local working directory back in Part 2, but that is fine — the `git clone` step in your Dockerfile fetches it again (along with `package.json`) every time the container is built. By the time `npm ci` runs inside the container, the lock file is already there.

1.  Open your Dockerfile.
    
2.  Change your `RUN npm install --omit=dev` line to `RUN npm ci --omit=dev`.
    
3.  Rebuild the container and verify the app still runs correctly.
    

_Part 4: Dockerization_

#### Upload the files that build modified Docker container

Upload a ZIP file containing the files that build your Docker container here. It should include the Dockerfile and any files necessary to build the container from the Dockerfile, following the requirements above. After uploading, you'll be able to check the correctness of your upload using an AI.

Drag a file here, click here to browse, or paste it in here

Accepted formats: ZIP

To have an AI check that your uploaded ZIP file meets the guidelines, first press the 'Extract Dockerfile' button below and then the 'Check Dockerfile' button. Note that this AI can make mistakes, and may take a minute or two to respond.

Extract Dockerfile

Check Dockerfile

#### Document your thoughts:

We'd love to hear your thoughts while you worked on this training project. In particular:                                                                                                                                                                                    

-   What drew you to the codebase you picked?                   
    
-   How did you think about what kind of problem to design, and what failure modes you wanted to target?
    
-   What did you learn about Docker that you'll carry into real agentic coding work?
    

Minimum word count: 100

Words: 0

**Optional Comments**

Let us know if you have any comments about this training project. Was it helpful? Was anything confusing? Would you like to see more training projects like this in the future?