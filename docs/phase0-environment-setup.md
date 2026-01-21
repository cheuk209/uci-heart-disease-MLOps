# Phase 0: Environment & Project Foundation

> **Theme:** "The Right Foundation Prevents Technical Debt"

## Learning Objectives

- Understand why reproducible environments matter for ML (not just "works on my machine")
- Learn project structure patterns that scale from prototype to production
- Establish the mental model of "infrastructure as code" for ML projects
- Internalize the "reproducibility tax" - investing upfront saves debugging time later

---

## Key Concepts

### Why Reproducible Environments Matter for ML

ML has dependency hell worse than traditional software development. Consider this: sklearn 1.0 changed the default value of a parameter. Your model trained on 1.0 behaves differently when loaded on 0.24. No error. No warning. Just different predictions.

This isn't hypothetical. NumPy, pandas, scikit-learn, and every other library you'll use have subtle version-to-version differences. Some break loudly (API changes). Many break silently (numerical differences, default parameter changes).

**Why this matters:** In traditional software, a bug usually manifests as an error. In ML, a "bug" often manifests as slightly worse predictions - which you might not notice until production metrics degrade weeks later.

The solution is environment pinning: every dependency locked to an exact version. Not "pandas>=1.0" but "pandas==1.5.3". Every time. No exceptions.

### Project Structure as Documentation

A well-organized repository tells you where things live without reading documentation. This is especially critical for ML projects because they have more artifact types than traditional software:

- **Source code** (your Python modules)
- **Data** (raw, processed, interim)
- **Notebooks** (exploration, experimentation)
- **Models** (trained artifacts)
- **Configs** (hyperparameters, paths)
- **Documentation** (decisions, learnings)

Each needs a home. If you find yourself asking "where did I put that script?" or "which notebook has the latest version?", your structure has failed.

**The principle:** Anyone (including future you) should be able to navigate the repo without explanation. The structure IS the explanation.

### The "Reproducibility Tax"

Every hour spent on proper environment setup saves ten hours of "but it worked yesterday" debugging.

This feels painful upfront. Setting up a proper virtual environment, pinning dependencies, organizing directories - it's slower than just writing code in a notebook. But consider:

- How many times have you lost work because you couldn't recreate an environment?
- How much time have you spent debugging issues that turned out to be version mismatches?
- How often have you been unable to reproduce a result from a month ago?

The tax is real. Pay it early, pay it once. Or pay it repeatedly with interest every time something breaks mysteriously.

### Infrastructure as Code for ML

In DevOps, "infrastructure as code" means your servers, networks, and configurations are defined in version-controlled files. The same principle applies to ML:

- Your environment is code (requirements.txt, pyproject.toml, Dockerfile)
- Your data pipeline is code (DVC pipelines, not manual steps)
- Your training process is code (reproducible scripts, not notebooks with cell execution order dependencies)

If it can't be recreated by running a command, it's not real infrastructure.

---

## Pattern Descriptions

### The Standard ML Project Layout Pattern

Separate concerns between:
- `data/` - Raw, interim, and processed datasets (often gitignored, tracked by DVC)
- `src/` or `<project_name>/` - Production-quality Python modules
- `notebooks/` - Exploratory analysis and experimentation (numbered for order)
- `configs/` - Configuration files (YAML, JSON)
- `models/` or `artifacts/` - Trained model files (often gitignored, tracked by DVC)
- `tests/` - Unit and integration tests
- `docs/` - Documentation and decision records

**Key consideration:** Data and models should NOT be in git. They're too large, and binary diffs are useless. Use DVC or similar for versioning these artifacts.

### Dependency Management Patterns

Three main approaches, each with tradeoffs:

**requirements.txt:**
- Pros: Simple, universal, everyone understands it
- Cons: Doesn't distinguish dev vs. production dependencies, no lock file by default
- Best for: Small projects, prototypes, when working with teams who know nothing else

**pyproject.toml (with pip-tools or Poetry):**
- Pros: Modern standard, handles dev/prod separation, lock files for exact reproducibility
- Cons: Learning curve, tooling fragmentation
- Best for: Projects that will grow, teams that want best practices

**conda environment.yml:**
- Pros: Handles non-Python dependencies (CUDA, system libraries)
- Cons: Slower, larger environments, cross-platform reproducibility issues
- Best for: When you need non-Python packages, data science teams already using Anaconda

**The key insight:** Whichever you choose, PIN VERSIONS. `pandas==1.5.3`, not `pandas>=1.5`.

### The "Makefile as Documentation" Pattern

Common commands should be one word. A new developer should be able to:

- `make install` - Set up the environment
- `make test` - Run tests
- `make train` - Train the model
- `make serve` - Start the prediction service

The Makefile becomes executable documentation. It answers "how do I...?" before the question is asked.

**Key consideration:** Make targets should be idempotent where possible (running twice gives the same result as running once).

---

## Common Pitfalls

### Pitfall 1: "I'll organize it later"
You won't. Technical debt accrues faster in ML projects because experiments proliferate. Establish structure before writing a single line of model code. Moving files later breaks paths in notebooks, configs, and your memory.

### Pitfall 2: Unpinned dependencies
"Latest version should be fine" until it isn't. Pin everything. Use a lock file. If you can't recreate your environment from your config files, your environment isn't real.

### Pitfall 3: Data in git
Git is for code, not data. Binary files don't diff. Large files slow clones. Use DVC, Git LFS, or similar. The rule is simple: if it's generated or downloaded, it shouldn't be in git.

### Pitfall 4: Notebooks as primary artifacts
Notebooks are for exploration, not production. They have hidden state (cell execution order). They don't diff cleanly. They're hard to test. Extract reusable code to modules. Keep notebooks for what they're good at: exploration and visualization.

### Pitfall 5: "It works on my machine"
If you can't give someone a single command to reproduce your environment, you don't have a reproducible environment. Test this: delete your virtual environment and recreate it from scratch. Does it work?

---

## Questions to Answer

Before proceeding to Phase 1, you should be able to answer:

1. **Can a colleague clone your repo and run your code in under 5 minutes?**
   - What commands would they need to run?
   - What would they need to install first?

2. **If you delete your virtual environment, can you recreate it identically?**
   - What files define your environment?
   - Are versions pinned or floating?

3. **Where will data live? Where will artifacts live? Why?**
   - How will these be versioned?
   - What's gitignored and what isn't?

4. **What commands should be "one word" in your project?**
   - What does a developer need to do daily?
   - How can the Makefile document these?

---

## Success Criteria

You have completed this phase when:

- [ ] Project structure exists with clear separation between code, data, notebooks, and artifacts
- [ ] Environment can be recreated from configuration files (test this by deleting and recreating)
- [ ] All dependencies are pinned to exact versions
- [ ] A Makefile (or similar) provides common commands
- [ ] A new developer could set up and run the project in under 5 minutes
- [ ] You've documented any non-obvious choices in a decisions file

---

## Exercises

### Exercise 1: Destruction Testing
Delete your virtual environment. Time how long it takes to recreate. If it's more than 2 minutes, your process needs improvement.

### Exercise 2: Fresh Clone Test
Clone your repo to a different location. Follow only your documented setup instructions. Note every missing step or assumption.

### Exercise 3: Dependency Archaeology
Pick a pinned dependency. Look up what changed between your version and the previous major version. Understand what could break.

---

## References

- **Chip Huyen "Designing ML Systems"** - Chapter 2 (Introduction to ML Systems Design)
  - Particularly the section on reproducibility and environment management

- **Python Packaging User Guide** (pypa.io)
  - The authoritative source on Python packaging standards

- **Cookiecutter Data Science**
  - A popular project template with rationale for structure decisions
  - https://drivendata.github.io/cookiecutter-data-science/

- **"Twelve-Factor App"**
  - Particularly Factor I (Codebase), Factor II (Dependencies), and Factor X (Dev/prod parity)
  - https://12factor.net/

---

## What's Next

With a solid foundation in place, Phase 1 will focus on acquiring and exploring the UCI Heart Disease dataset. You'll learn to treat data exploration as a discipline - systematic, documented, and prerequisite to any cleaning or transformation.
