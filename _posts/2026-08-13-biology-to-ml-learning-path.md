---
title: "Bio to ML: A Learning Path"
date: 2026-08-13
layout: post
---

# Bio to ML: A Learning Path

**Who this is for:** someone committed to actually learning machine learning. Not someone looking for a paid product to automate a daily task, the "can I get an agent to book my flights and organize my calendar" version of AI. Those tools exist and you do not need this document to find them.

If you have ever paid for an ML course, spent three hours on it, and never finished, this is for you.

This is the sequence that worked for me, not a required curriculum. I'm still working through parts of it myself.

**How to use:** roughly in order. Skip a block only if you can already do the "you'll be able to" line. Blocks 5 and 6 do not depend on Block 4, so 1 → 2 → 3 → 5 → 6 is a legitimate path if language models are closer to your work than deep learning is.

---

## Before you start

What you need:

- Python basics, then **pandas and numpy**. Most of your time will be spent here, not in model code.
- **Command line**: enough to move around a filesystem, look at the contents of a file, and check what a running job is actually using.
- [**Google Colab**](https://colab.research.google.com/): free, no setup, free GPU. Open a notebook, run cells, switch the runtime to GPU, get a CSV in. Know that the runtime resets and takes your files with it.

If you are missing all of this, do not spend a month on it before starting Block 1. A beginner ML course will teach you the Python you need as you go, and you will learn it faster when it is attached to something you want to build.

---

## The map

| Block | Topic                               | Time                  | What you can do at the end                                |
| ----- | ----------------------------------- | --------------------- | --------------------------------------------------------- |
| 1     | Finish a beginner ML course         | ~10 weeks at 5 hrs/wk | Build a baseline model and defend the choice              |
| 2     | Evaluation and leakage              | 2 weeks               | Explain how a model can look excellent and still be wrong |
| 3     | A real project on public data       | 4 weeks               | Finish one honest end-to-end analysis                     |
| 4     | Deep learning and pretrained models | optional              | Tell when a problem needs deep learning                   |
| 5     | LLMs and agentic workflows          | 4 weeks               | Ship a tool-using workflow you can debug                  |
| 6     | Controlled and offline environments | 2 weeks               | Make any of the above run with no internet                |
| 7     | Engineering practice                | alongside 1 to 6      | Hand someone your analysis and have it run                |

Six months at a working scientist's pace, not a bootcamp. Blocks 2 and 6 are the shortest and the easiest to skip. They are also the two that come up at work.

---

## Block 1: Finish a beginner ML course

Any intro to ML course from the last three years will do. The specific course matters much less than finishing one. Sampling three courses and completing none is how this block fails, which is why the note at the top is there.

**If you have no basis for choosing, take this one:** [Machine Learning Specialization (Stanford Online / DeepLearning.AI, Andrew Ng)](https://www.coursera.org/specializations/machine-learning-introduction). No prerequisites beyond high school math, and it teaches the Python you need as it goes.

### Do this alongside, not after

Every week, take the concept you just learned and run it on a small tabular dataset of your own, in Colab, with scikit-learn. Block 3 covers where to get the data.

The point is not the models. The point is the reflex of loading data, splitting it, fitting, and scoring without looking anything up. Course exercises come with the data already loaded and the problem already framed, which is different from real work.

Keep these in one repository as you go. When you want to try a different setting, copy the cell and change the copy rather than editing in place. By the end you have a dozen small working examples.

→ **You'll be able to:** build a baseline model on a new biological dataset and explain why you chose it.

---

## Block 2: Knowing whether a model is actually good

No course covers this, and it is where scientific training actually helps.

**Concepts:**

- Data leakage in all its forms: target leakage, train/test contamination, preprocessing fit on the full dataset, duplicate records across splits.
- Split discipline in a scientific setting. The unit of independence is almost never the row. It is the patient, the subject, the batch, the cell line, the site, or the time point. Random `train_test_split` on the row is the single most common way biological ML goes wrong.
- Metric choice: AUROC versus AUPRC under class imbalance, sensitivity and specificity at a chosen operating point, calibration. Accuracy is nearly always the wrong headline number.
- Cross-validation done properly, including nested cross-validation when you are also selecting features or tuning hyperparameters. If model selection happens inside the same loop that reports performance, the reported number is optimistic and you cannot say by how much.
- Baselines. Always report what a trivial model gets: majority class, one strong clinical covariate, or a logistic regression on age and sex. Age and sex predict a lot of clinical outcomes on their own, so a paper reporting AUROC 0.78 from a complex model has told you nothing if age and sex alone would have given 0.75. Most papers do not report that comparison, which means you cannot tell.
- Variability across folds and seeds. A single number with no spread is not a result.
- Batch effects and confounding as a modeling problem, not just a preprocessing problem. If the cases were run in one batch and the controls in another, your classifier is a batch detector.

**Reading:**

- Kapoor & Narayanan, _Leakage and the Reproducibility Crisis in ML-based Science_.
- Whalen et al., _Navigating the pitfalls of applying machine learning in genomics_ (Nat Rev Genet).

### Do this alongside, not after

Deliberately build a leaking model. Take a dataset you know well, engineer a leak into it on purpose, get a suspiciously high AUROC, then remove it and watch the number collapse.

Do it once on purpose and you will recognize it later. Otherwise you meet it by accident, six months in, after presenting the result.

→ **You'll be able to:** explain how a model can look excellent and still be wrong.

---

## Block 3: A real project on public data

Courses teach you to run a model on a clean dataset. Nothing teaches judgment except a messy dataset and a question you actually care about. Take **one** project end to end. A small completed analysis with a real evaluation section beats an ambitious unfinished one.

### Start here: NHANES

**https://www.cdc.gov/nchs/nhanes/**

CDC population health survey: demographics, questionnaires, labs, exam measurements. Free, no application, downloadable today. The best first choice because it is tabular, human, interpretable, and messy in realistic ways.

- **Teaches:** joining multiple files on a subject ID, missing data that is not missing at random, variable dictionaries and codebooks, cross-sectional confounding.
- **Project shape:** predict a lab-confirmed outcome from questionnaire and exam variables, then ask honestly how much of your signal is age.

### Finding your own dataset

Before you pick from the list below, describe your question to ChatGPT, Claude, or Gemini and ask what public datasets would fit it. This is one of the things they are genuinely good at, and the answer is often better matched than anything here. Ask about access requirements too, then confirm those on the source site.

### Other datasets worth knowing

**PhysioNet / MIMIC-IV** · https://physionet.org
De-identified ICU and hospital records. Requires CITI training and a signed data use agreement, so apply a few weeks early. The trap is treatment leakage: mortality is easy to predict if vasopressors and comfort-care orders are in your feature window.

**ClinicalTrials.gov and AACT** · https://clinicaltrials.gov · https://aact.ctti-clinicaltrials.org
The global trial registry. AACT is the same content as a relational database and is much easier to query. The trap is that registry entries are self-reported and often incomplete.

**DepMap / CCLE** · https://depmap.org
Cancer cell line genomics plus dependency screens and drug sensitivity. Many more features than samples. The trap is lineage: a model predicting drug response can score well by learning tissue type instead of biology. Hold out whole lineages and watch it fall.

### How to run the project so it counts

1. Write the question in one sentence before you download anything.
2. Define the unit of independence and the split **first**, before any modeling.
3. Establish the trivial baseline.
4. Build the simplest model that could work. Stop there if it works.
5. **(important)** Write an evaluation section covering what you tried that failed and where the model should not be trusted.
6. Put it in a repository with a README that a stranger could run.

→ **You'll be able to:** carry one dataset from question to defensible conclusion without a tutorial telling you what to do next.

---

## Block 4: Deep learning and pretrained models

Optional, and selective. Course 2 in Block 1 gives you the working version of neural networks. Come here only if you need more.

For most tabular clinical and omics problems at typical biological data scale, gradient-boosted trees (XGBoost, LightGBM) are still the strong baseline and deep learning does not beat them. Deep learning earns its place with images, sequences, structures, text, and very large datasets. Outside those, check that it beats XGBoost before you commit to it.

**Fundamentals:**

- Deep Learning Specialization (DeepLearning.AI), for the structured version.
- Karpathy, _Neural Networks: Zero to Hero_. Build the thing from scratch once. It permanently demystifies the rest.
- Karpathy, _A Recipe for Training Neural Networks_. Short, and the best practical debugging guide that exists.
- PyTorch over TensorFlow for anything research-adjacent.

**The mode you will actually work in** is adapting and fine-tuning existing architectures, not training from scratch. Load a pretrained model, freeze parts of it, replace the head, fine-tune. On a new modality, start by using a pretrained model's embeddings as features and decide afterward whether fine-tuning is worth it.

**Domain models worth knowing exist:** ESM and AlphaFold for proteins; scGPT and Geneformer for single cell, benchmarked against a simple baseline before you believe any of it; TabPFN for small tabular data, which changes what is worth attempting on a few-hundred-sample clinical dataset; Cellpose and the SAM family for microscopy.

→ **You'll be able to:** tell when a problem needs deep learning and when it doesn't, and use a pretrained model as a feature extractor without training anything.

---

## Block 5: LLMs and agentic workflows

The bottleneck in applying these systems to biology is domain judgment, not model engineering, which is why a biologist can become useful here quickly.

### 5.1 What you actually need to know

You do not need to know how transformers work internally. You do need:

- Tokens, context windows, and what happens when you exceed them.
- Temperature, and why you set it to zero for extraction and not for drafting.
- Structured output: asking for JSON against a fixed schema. Real pipeline work depends on getting reliable structured output, not prose.
- Batching. One call per row is slow and expensive; too many records in one call and the quality drops. Find the batch size that holds up for your task and keep it fixed.
- **Save every raw response.** The same input gives a different answer next time, so a result you did not store is one you cannot reproduce, re-check, or diff against a later run. Write responses to disk before you parse them.
- Embeddings, and how they differ from a generative model.
- When fine-tuning is worth it, which is less often than people assume. Prompting, retrieval, and better data usually beat it for knowledge tasks. Fine-tune for format, style, and narrow classification.
- Prompts as versioned artifacts. Keep them in files, in git, with a small evaluation set.

### 5.2 Let the model write the prompt

Describe the task, the input, and the output format you want, and ask the model to draft the prompt. Then edit it. Faster than writing from scratch, and it surfaces requirements you had not thought to state.

Same for schemas, test cases, and the first version of an evaluation set.

### 5.3 Retrieval, if you need it

Split documents into chunks, embed them, search, pass what comes back to the model.

Retrieval sets the ceiling. If the right passage is never retrieved, the model cannot recover, so check that step on its own before judging the answer. For scientific work the system also has to show which passage it used.

### 5.4 The ladder, and staying low on it

1. **Single call.** One prompt, one response. Most tasks stop here.
2. **Chain.** Fixed sequence of calls, deterministic order.
3. **Router.** One classification step chooses among fixed branches.
4. **Tool-using agent.** The model decides which tools to call and when, in a loop.
5. **Multi-agent.** Several specialized agents coordinating.

Every rung up buys flexibility and pays for it in nondeterminism, cost, latency, and debugging difficulty. The common mistake is building rung 4 or 5 for a problem that rung 2 solves. **Use the least agentic thing that works.**

**Build the raw loop before you touch a framework.** Define a tool schema, send it, receive a tool call, execute it, return the result, repeat until the model stops. It is about eighty lines. After that, frameworks are conveniences rather than magic, and you can debug them.

### 5.5 LangGraph

Worth learning because it makes the workflow an explicit graph rather than an open-ended agent loop, so you can see what ran and in what order. The parts that matter: the state object you thread through it, branching on that state, checkpointing so a long run can pause and resume, and an interrupt step when something needs human approval before it proceeds. Run a tracing tool alongside it.

Alternatives keep appearing, and any of them will do. What transfers between them is the ladder above and the state design.

### 5.6 Evaluating what you build

Block 2's discipline on a harder target, and the difference between a demo and something you can leave running.

Keep a fixed set of inputs with known correct outputs, however small. Track cost and latency per run, since agent loops can quietly cost far more than a single call. Set hard stops on steps, cost, and time. And record what kind of failure happened, not just that one did, because the pattern is what tells you where to fix it.

### 5.7 What is worth building in bio

- Literature triage: screen abstracts against inclusion criteria, structured output with a confidence field.
- Extraction from free text: eligibility criteria, pathology reports, adverse event narratives, assay metadata. Highest immediate value, because it turns unusable text into tables.
- API orchestration: pull from ClinicalTrials.gov, PubMed, or an internal LIMS, normalize, reconcile.
- QC triage: read run logs and metrics, draft the "what went wrong" summary a human then checks.
- Analysis against a known schema: generate and run queries or plotting code in a sandbox, with results verified.

The pattern: a human currently reads unstructured material and produces structured judgment, in volume, and errors are visible when they happen. Where errors are invisible, do not build this.

→ **You'll be able to:** build a tool-using workflow, explain why it sits at the rung you chose, and show the trace when it fails.

---

## Block 6: Working in controlled and offline environments

Tutorials assume an open internet connection and an unrestricted API key. A lot of interesting biological and clinical data assumes neither. Everything Colab taught you is wrong here.

### 6.1 Ask where the data is going

Calling a model API means sending your data to someone else's servers. That is a transfer, not a computation, and it is usually the exact thing a data use agreement prohibits. Pasting a clinical note into a chat window is the same transfer as doing it in code, and it does not become acceptable because it was convenient.

So before anything leaves the environment, know which agreement governs the data and what it says about third-party processing. If you cannot name the agreement, do not send it. Patient privacy rules, residency requirements, IRB conditions, partner contracts, and unpublished IP all sit behind this. These are the terms the data came with, and breaking them can end access permanently.

Two ways through: run a model inside your own environment, accepting that a local model is weaker (extraction and classification hold up, open-ended reasoning does not), or structure the work so only de-identified or aggregate output ever crosses the boundary.

### 6.2 Working with no network

Two things break immediately.

**Dependencies.** You cannot install packages at runtime. Either build a container image outside and bring it in, or download the packages on a connected machine and install from that local copy. Pin versions, or the environment drifts into an outage later.

**Model weights.** Anything that loads a model by name reaches Hugging Face when it runs. Download the weights ahead of time, note the version and date, and load from a local path. Hugging Face has offline settings for exactly this case; turn them on so a missing file fails immediately instead of hanging.

### 6.3 Things that fail quietly

Hugging Face is the usual culprit, so check it first.

The general shape of the problem: blocked network calls tend to hang rather than error, so a job that should fail in a second sits for minutes looking merely slow. Other common sources are library telemetry, packages that quietly download reference files the first time you use them, and notebook widgets pulling code from a CDN, which shows up as a blank cell rather than an error message.

### 6.4 Do not leak data on the way out

Notebook outputs are the most common accident: a cell that prints patient records, committed to a repository, is a data incident. Strip outputs before committing. Keep code and data in separate places with separate access. Scrub logs, since error messages often contain the record that caused the error. Never put identifiers in filenames or ticket titles.

### 6.5 Test it

Run the pipeline once with networking disabled before you deploy it. It takes seconds and tells you what you missed, which is cheaper than finding out during deployment.

→ **You'll be able to:** take any workflow from Blocks 1 through 5 and make it run with no network access, and explain to a compliance reviewer where the data goes.

---

## Block 7: Engineering practice

Run alongside everything else.

- conda: creating environments and moving them between machines. conda-pack bundles one into a transferable archive, which is the option that works when the environment has non-Python binaries in it. Pin versions.
- Model freezing, weight import and export, artifact versioning.
- Enough Docker to build an image, run a container, mount a volume.
- Track your runs in a table. Date, what you changed (parameters), the metric number you got. A week later you will not remember which version produced the result in your slide. MLflow automate this.
- Tests for data-loading and preprocessing functions. That is where the silent errors live.
- One cloud platform, whichever your organization uses. Object storage, managed notebooks, training jobs, model endpoints, secrets, and access control are common to all of them. Learn those six and the vendor becomes a documentation problem.

For a structured version: MLOps (Machine Learning Operations Specialization, Duke University, Coursera).

→ **You'll be able to:** hand someone your analysis and have it run on their machine.

---

Thanks to Yifeng Liang, Lengxi Huang, and Karen Wong, who I met through the [CABS summer intern program](https://ds4cabs.github.io). A lot of this document came out of talking with them. Their questions made me organize thinking I had been carrying loosely, and several of their ideas changed how I see the sequence. Teaching turned out to be its own form of learning. None of them started with the Coursera course. Each took a real dataset and worked a project through over the summer, picking up the pieces as they needed them. That is a different order than the one written here, but it works out great.
