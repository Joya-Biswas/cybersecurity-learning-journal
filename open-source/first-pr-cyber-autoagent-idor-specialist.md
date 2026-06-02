# First Open Source PR Attempt: Cyber-AutoAgent-ng IDOR Specialist

## Project

* **Repository:** `Cyber-AutoAgent-ng`
* **Issue:** `#22`
* **Topic:** Add an `idor_specialist.py` tool and allow it in the web and ctf modules

## Why I picked this

I am an undergraduate student interested in cybersecurity, and I wanted to start contributing to open source. Since this repository is related to cybersecurity and automation, it felt relevant to my interests.

The issue was labeled as a good first issue, so I decided to try it even though I was new to open source workflow. As I worked through it, I realized it was still more challenging than I expected for a first contribution.

## What I understood the issue to mean

My understanding was that the project wanted a specialist helper tool related to IDOR reasoning.

IDOR stands for **Insecure Direct Object Reference**, which usually involves applications exposing object identifiers such as user IDs, order IDs, invoice IDs, or other references that may be predictable. If authorization is weak, changing one identifier to another may expose data that should not be accessible.

The issue seemed to suggest that the existing general model was not very strong at reasoning about these object references by itself. So instead of expecting the main model to figure everything out alone, the project wanted a more specialized helper tool focused on identifying likely object IDs and possible nearby values or ranges worth checking.

I understood this not as a request for a full vulnerability scanner, but more as a request for a **specialized reasoning component** inside the project.

## What confused me at first

Several things were confusing to me in the beginning:

* I did not fully understand the issue wording
* I was not sure what "add a specialist tool" really meant inside this codebase
* I did not know where tools were stored or how they were registered
* I was unsure whether I should fork first or create a branch first
* I was worried that following an existing project pattern would feel like copying instead of contributing
* I also did not know how much testing I was expected to do before opening a pull request

This was one of the biggest lessons for me: understanding the **project structure** is part of the work, not something separate from it.

## How I explored the codebase

Before making changes, I tried to understand where similar functionality already existed.

I found that the repository already had specialist-style tools and that the `web` and `ctf` modules had YAML configuration files listing the tools they were allowed to use. That helped me realize that this issue likely required two kinds of changes:

1. adding a new Python tool file
2. registering that tool in the relevant module configuration

I also looked at an existing file, `validation_specialist.py`, to understand the general structure used for specialist tools in this project.

That file helped me see that the repo pattern involved:

* a Python file for the tool
* a `@tool`-decorated function
* a system prompt or methodology string
* creation of a specialist agent
* execution of a task
* returning structured JSON-like output

Even though I did not understand every line at first, this helped me understand the overall shape of the solution.

## What I changed

I made three main changes.

### 1. Added `idor_specialist` to the `web` module tool list

In:

`src/modules/operation_plugins/web/module.yaml`

I added:

```yaml
- idor_specialist
```

My understanding is that this tells the `web` module that it is allowed to use the new tool during web-related analysis.

### 2. Added `idor_specialist` to the `ctf` module tool list

In:

`src/modules/operation_plugins/ctf/module.yaml`

I also added:

```yaml
- idor_specialist
```

My understanding is that this makes the same tool available in CTF-oriented workflows, which matches the issue description.

### 3. Added an initial `idor_specialist.py` implementation

I created:

`src/modules/operation_plugins/web/tools/idor_specialist.py`

This file was based on the general structure of existing specialist tools already present in the project.

The implementation I added is a **first-pass specialist tool**, not a complete scanner or exploit engine. My understanding is that it is meant to help the system reason about identifiers that may represent internal objects, such as:

* numeric IDs
* UUID-like IDs
* hex-like values
* slug-like values with numeric parts

The purpose of the tool is to help the larger system think more systematically about which values might be worth checking in IDOR-related situations.

## Why these changes matter

This is the part I did not understand clearly at first.

The YAML changes matter because the tool file by itself is probably not enough. A tool can exist in the repository, but if the relevant module does not list it in its configuration, that module may never use it.

So the Python file is the **implementation**, while the YAML entries are part of the **integration**.

That means my changes were not only about writing a new file. They were also about connecting that file to the parts of the system that are expected to use it.

This helped me understand something important about real projects: adding a feature often means changing both:

* code
* configuration

## What the script is intended to do

My current understanding is that the script acts as a **helper tool for IDOR reasoning**, not a complete vulnerability detector.

In plain language, I think its role is something like this:

* take information about identifiers seen in a URL, parameter, or API path
* analyze whether those identifiers look like object references
* classify the format of those identifiers
* suggest nearby candidate values or ranges that might be worth testing
* return structured output that the larger agent system can use

For example, if a system sees an identifier such as `42`, the tool may treat it as a numeric object ID and suggest nearby values like `41`, `43`, or a small surrounding range.

If it sees a UUID, it may classify it differently and indicate that it is less directly enumerable.

So the tool is more about **supporting reasoning and candidate generation** than proving a vulnerability on its own.

## What the script does not do

It is also important to be honest about what this first implementation does **not** prove.

It does not:

* automatically confirm an IDOR vulnerability
* guarantee exploitation
* replace proper authorization testing
* prove that the identified values actually expose unauthorized data

At best, it helps the larger system think in a more focused way about likely object identifier patterns.

That distinction was important for me to understand while documenting the contribution.

## What I verified

I did a few checks before opening the pull request.

* I validated the Python file syntax locally with:

```bash
python -m py_compile src/modules/operation_plugins/web/tools/idor_specialist.py
```

This confirmed that the file was at least valid Python syntax.

* I inspected the Git diff before committing so I could see exactly what changed.
* I attempted to run the test suite with:

```bash
python -m pytest
```

## Problems I ran into

I ran into several environment and workflow problems during the process.

### Python path issue

At first, pytest could not import modules correctly, and I learned that the repository uses a `src` layout. I needed to set:

```bash
PYTHONPATH=src
```

before imports worked properly.

### Missing dependencies

After fixing the path issue, I encountered missing dependencies such as `pyyaml`, which meant I had to install additional packages before tests could continue.

### Platform-specific test issue

Later, pytest failed because of a shell-related import issue involving `termios`. Since I was working on Windows, this appeared to be a platform-specific issue related to Unix/Linux terminal functionality rather than my exact code change.

This was an important lesson: sometimes test failures are not directly caused by the feature being added.

## Tools and help I used

I did not do this completely alone.

During this contribution, I used help from **ChatGPT** and **Claude** to:

* understand what the issue was asking for
* understand how open source workflow works
* think through the likely meaning of the repository structure
* understand the role of the YAML configuration and the tool file
* get guidance on testing, debugging, and writing documentation

I think it is important to be honest about that.

At the same time, I still went through the actual workflow myself, including:

* editing files
* checking diffs
* running syntax checks
* attempting tests
* documenting what I understood and what I still found unclear

So while I used AI tools for guidance and explanation, the contribution process still involved my own effort, decisions, and learning.


## Reflection
Even though I did not feel fully confident at every step, this was still a valuable first contribution attempt because it helped me experience the real workflow of reading, changing, validating, and documenting a codebase.
