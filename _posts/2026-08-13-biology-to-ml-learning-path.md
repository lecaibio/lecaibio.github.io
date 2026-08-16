---
title: "Bio to ML: A Learning Path"
description: "Finishing an ML course does not connect to anything you would actually do at work. This is the sequence that goes around it, written for people coming from biology."
date: 2026-08-13
layout: post
tags: [bio + ai, career, reproducibility]
---

**Who this is for:** you want a career doing ML in biology/healthcare, eventually. You have probably opened an ML course and not finished it, or finished one and still not known what to do next. A course by itself does not connect to anything you would actually do at work. This is what goes around it.

**Who this is not for:** the "can an agent book my flights and organize my calendar" version of AI. Those tools exist and you do not need this document to find them. Same answer if you already work in industry and desperately want something for a task you do every day: buy it. Pay for Claude or ChatGPT, or ask your manager for a Microsoft Copilot license. Building your own takes six months and it will almost certainly be worse than what you can buy today. But if you want to be the person who eventually builds that kind of tool, this one is for you.

**Block 1 is optional.** Not everyone learns from lectures, and some people already have the coursework. <mark>If you already do science work and use Python, start at Block 3, pick a dataset, and come back to the earlier blocks when you get stuck.</mark>

This is the sequence that worked for me, not a curriculum. I am still working through parts of it myself.

---

## Before you start

- Python basics, then **pandas and numpy**. Most of your time will be spent here, not in model code.
- **Command line**: enough to move around a filesystem, look at the contents of a file, and check what a running job is actually using.
- **git**: init, commit, push. About forty minutes. Blocks 3 and 5 both assume it.
- [**Google Colab**](https://colab.research.google.com/): free, no setup, free GPU. Open a notebook, run cells, switch the runtime to GPU, get a CSV in. Know that the runtime resets and takes your files with it.

If you are missing all of this, do not spend a month on it before starting. A beginner ML course will teach you the Python you need as you go, and if you skip the course, a real dataset will do the same thing more painfully. Either way you learn it faster attached to something you want to build.

---

## The map

| Block | Topic                                  | Time                  | What you can do at the end                                |
| ----- | -------------------------------------- | --------------------- | --------------------------------------------------------- |
| 1     | Finish a beginner ML course (optional) | ~10 weeks at 5 hrs/wk | Build a baseline model and defend the choice              |
| 2     | Evaluation and leakage                 | 2 weeks               | Explain how a model can look excellent and still be wrong |
| 3     | A real project on public data          | 6 weeks               | Finish one honest end-to-end analysis                     |
| 4     | Deep learning and pretrained models    | optional              | Tell when a problem needs deep learning                   |
| 5     | LLMs and agentic workflows             | 4 weeks               | Ship a tool-using workflow you can debug                  |
| 6     | Controlled and offline environments    | 2 weeks               | Make any of the above run with no internet                |
| 7     | Engineering practice                   | start in week one     | Hand someone your analysis and have it run                |

Six months at a working scientist's pace if you do all of it, and these are the numbers for when it goes smoothly, which it will not. Block 7 is not a stage. It runs underneath everything else and it is at the bottom only because it has no natural place in the sequence.

Blocks 2 and 6 are the shortest and the easiest to skip. They are also the two that come up at work.

---

## Block 1: Finish a beginner ML course

Optional, as above. If you are taking one, any intro to ML course from the last three years will do. The specific course matters much less than finishing one. Sampling three courses and completing none is how this block fails.

**If you have no basis for choosing, take this one:** [Machine Learning Specialization (Stanford Online / DeepLearning.AI, Andrew Ng)](https://www.coursera.org/specializations/machine-learning-introduction). No prerequisites beyond high school math, and it teaches the Python you need as it goes.

### Do this alongside, not after

Every week, take the concept you just learned and run it on a small tabular dataset of your own, in Colab, with scikit-learn. Block 3 covers where to get the data.

You are after one reflex: load data, split it, fit, score, without looking anything up. Course exercises come with the data already loaded and the problem already framed, which is not what real work looks like.

Keep these in one repository as you go. When you want to try a different setting, copy the cell and change the copy rather than editing in place. By the end you have a dozen small working examples.

→ **You'll be able to:** build a baseline model on a new biological dataset and explain why you chose it.

---

## Block 2: Knowing whether a model is actually good

No course covers this, and it is where scientific training actually helps.

**Concepts:**

- Data leakage: target leakage, train/test contamination, preprocessing fit on the full dataset, duplicate records across splits.
- Split discipline in a scientific setting. The unit of independence is almost never the row. It is the patient, the subject, the batch, the cell line, the site, or the time point. Random `train_test_split` on the row is the single most common way biological ML goes wrong.
- Metric choice: AUROC versus AUPRC under class imbalance, sensitivity and specificity at a chosen operating point, calibration. Accuracy is nearly always the wrong headline number.
- Cross-validation done properly, including nested cross-validation when you are also selecting features or tuning hyperparameters. If model selection happens inside the same loop that reports performance, the reported number is optimistic and you cannot say by how much.
- Baselines. Always report what a trivial model gets: majority class, one strong clinical covariate, or a logistic regression on age and sex. Age and sex predict a lot of clinical outcomes on their own, so a paper reporting AUROC 0.78 from a complex model has told you nothing if age and sex alone would have given 0.75.
- Variability across folds and seeds. A single number with no spread is not a result.
- Batch effects and confounding as a modeling problem, not just a preprocessing problem. If the cases were run in one batch and the controls in another, your classifier is a batch detector.
- Keep a table of your runs from here on: date, what you changed, the number you got. See Block 7. A week later you will not remember which version produced the result in your slide.

**Reading:**

- Kapoor & Narayanan, _Leakage and the Reproducibility Crisis in ML-based Science_.
- Whalen et al., _Navigating the pitfalls of applying machine learning in genomics_ (Nat Rev Genet).
- TRIPOD+AI, if you work anywhere near clinical prediction. It is the reporting standard, and knowing what it asks for changes how you write up a result.

### Do this alongside, not after

**Your first model is almost certainly wrong**, in some way. A model that works is not the same as a model that is right, and this is where ML differs from most code. Broken code fails visibly. So before you improve a result, deliberately build a leaking model. Take a dataset you know well, engineer a leak into it on purpose, get a suspiciously high AUROC, then remove it and watch the number collapse.

Do it once on purpose and you will recognize it later. Otherwise you meet it by accident, six months in, after presenting the result.

→ **You'll be able to:** explain how a model can look excellent and still be wrong.

---

## Block 3: A real project on public data

Courses teach you to run a model on a clean dataset. Nothing teaches judgment except a messy dataset and a question you actually care about. Take **one** project end to end. A small completed analysis with a real evaluation section beats an ambitious unfinished one.

This is also where people who skipped Block 1 should start. You will be looking things up constantly for the first two weeks. That is the cost of this route and it is survivable.

### Is it actually an ML question?

Near the end of a first ML course a reasonable thought arrives. You have twenty measurements per sample and have only ever looked at them one at a time. A model uses all twenty at once. Surely that is where the biomarker is.

Sometimes. But ML predicts Y for a new sample, while statistics estimates whether X affects Y and by how much. Most biology experiments ask the second, and a linear model with proper multiple testing answers it better, because it gives you an effect size and an interval instead of an accuracy number.

Then count independent units, not rows. Six mice per group is six, however many transcripts you measured on each. At that size a model will find something. It will also find something after you shuffle the labels.

Learn on public data, where sample size is someone else's problem. Knowing the boring method wins here is the same judgment as knowing when it doesn't.

### Start here: NHANES

**https://www.cdc.gov/nchs/nhanes/**

CDC population health survey: demographics, questionnaires, labs, exam measurements. Free, no application, downloadable today. The best first choice because it is tabular, human, interpretable, and messy in realistic ways.

- **Teaches:** joining multiple files on a subject ID, missing data that is not missing at random, variable dictionaries and codebooks, cross-sectional confounding.
- **Project shape:** predict a lab-confirmed outcome from questionnaire and exam variables, then ask honestly how much of your signal is age.

### Finding your own dataset

Before you pick from the list below, describe your question to ChatGPT, Claude, or Gemini and ask what public datasets would fit it. They are good at this, and the answer is often better matched than anything here. Ask about access requirements too, then confirm those on the source site.

### Other datasets

**PhysioNet / MIMIC-IV** · https://physionet.org
De-identified ICU and hospital records. Requires CITI training and a signed data use agreement, so apply a few weeks early. Read the agreement while you wait; it is nine clauses and it is the cheapest preview of Block 6 available. The trap is treatment leakage: mortality is easy to predict if vasopressors and comfort-care orders are in your feature window.

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
6. Put it in a repository with a README that a stranger could run, an environment file, and tests on the data-loading functions. Block 7 covers what those look like.

→ **You'll be able to:** carry one dataset from question to defensible conclusion without a tutorial telling you what to do next.

---

## Block 4: Deep learning and pretrained models

Optional, and selective. A beginner course already gives you the working version of neural networks. Come here only if you need more.

For most tabular clinical and omics problems at typical biological data scale, gradient-boosted trees (XGBoost, LightGBM) are still the strong baseline and deep learning does not beat them. Deep learning earns its place with images, sequences, structures, text, and very large datasets. Outside those, check that it beats XGBoost before you commit to it.

**Fundamentals:**

- Deep Learning Specialization (DeepLearning.AI), for the structured version.
- Karpathy, _Neural Networks: Zero to Hero_. Build the thing from scratch once. After that the rest stops looking like magic.
- Karpathy, _A Recipe for Training Neural Networks_. Short, and the best practical debugging guide that exists.
- PyTorch over TensorFlow for anything research-adjacent.

**The mode you will actually work in** is adapting and fine-tuning existing architectures, not training from scratch. Load a pretrained model, freeze parts of it, replace the head, fine-tune. On a new modality, start by using a pretrained model's embeddings as features and decide afterward whether fine-tuning is worth it.

**Domain models that exist:** ESM and AlphaFold for proteins; scGPT and Geneformer for single cell, benchmarked against a simple baseline before you believe any of it; TabPFN, which works surprisingly well on small tabular data and changes what is worth attempting on a few-hundred-sample clinical dataset.

**If you do come here, do it on Block 3's dataset.** Take the analysis you already finished, swap in a pretrained model as a feature extractor, and see whether it beats what you had. Most of the time it will not, and finding that out on your own data is the whole lesson.

→ **You'll be able to:** tell when a problem needs deep learning and when it doesn't, and use a pretrained model as a feature extractor without training anything.

---

## Block 5: LLMs and agentic workflows

The bottleneck in applying these systems to biology is domain judgment, not model engineering, which is why a biologist can become useful here quickly.

**First, get out of the chat window.** Think about the last time you used a chatbot. When it searched the web, ran code, read a file you uploaded, or picked up something you said twenty minutes earlier, none of that was the model. Those were pieces built around it. What you were using was already an agentic workflow, which is the thing this block is about building, and you cannot build one while it still looks like a single object.

A bare LLM is text in, text out. Nothing else. It does not open your files. It does not make you a slide deck. It does not remember yesterday. Everything beyond the string it returns was assembled by someone.

If you have never made an LLM API call, or don't know what I mean, I have something for you here: [cabs-workshop-llm-agents](https://github.com/lecaibio/cabs-workshop-llm-agents). It has a Colab notebook ready to run and instructions for getting a free Gemini API key, no credit card. As of August 2026 the free tier covers it. Send a string, get one back, and you have seen the entire backbone. Everything below is something wrapped around that call.

### 5.1 The knobs that matter

You do not need to know how transformers work internally. You do need:

- Tokens, context windows, and what happens when you exceed them.
- Temperature, and why you set it to zero for extraction and not for drafting.
- Structured output: asking for JSON against a fixed schema. Real pipeline work depends on getting reliable structured output, not prose.
- Batching. One call per row is slow and expensive; too many records in one call and quality drops. Find the batch size that holds up for your task and keep it fixed.
- **Save every raw response.** The same input gives a different answer next time, so a result you did not store is one you cannot reproduce, re-check, or diff against a later run. Write responses to disk before you parse them.
- When fine-tuning is worth it, which is less often than people assume. Prompting, retrieval, and better data usually beat it for knowledge tasks. Fine-tune for format, style, and narrow classification.
- Prompts as versioned artifacts. Keep them in files, in git, with a small evaluation set.
- Let the model draft the prompt. Describe the task, the input, and the output format you want, and ask it to write the prompt; then edit. Faster than writing from scratch, and it surfaces requirements you had not thought to state. Same for schemas, test cases, and the first version of an evaluation set.
- Which provider you call is a config line, not an architecture. Write it so the model call can be swapped for a different API or a local model without touching the rest. Block 6 explains why this matters more than it looks.

### 5.2 Giving it what it does not know

Retrieval, if you need it. Split documents into chunks, embed them, search, pass what comes back to the model.

Retrieval sets the ceiling. If the right passage is never retrieved, the model cannot recover, so check that step on its own before judging the answer. For scientific work the system also has to point at the passage it used, not just the document.

### 5.3 Tools first, then autonomy

These are separate questions and it helps to keep them apart.

**What does it need to be able to do?** That is tools: a function that queries an API, runs a search, reads a file, executes a query against your database. This is where most of the work is and where most of the value sits. A function that pulls from ClinicalTrials.gov and returns clean structured records is useful whether or not a model ever calls it. Build the tool and test it on its own, before any model is involved.

**Who decides when to use it?** This is what people mean by "agentic," and it is a dial rather than a sequence of stages:

1. **You decide, in code.** One call, or a fixed order of them. If a tool is needed, your code calls it. Most tasks stop here.
2. **One branch.** A classification step picks among fixed paths, then fixed code again.
3. **Tool-using agent.** You send the tool descriptions along with the prompt. The model replies with a request to call one, arguments filled in. Your code runs it, returns the result, and the model either calls another or answers. You wrote the tools; you did not write the order.
4. **Multi-agent.** Several specialized agents coordinating, each with its own tools.

Turning the dial up buys flexibility and pays for it in nondeterminism, cost, latency, and debugging difficulty. **Use the least autonomy that works.** The common mistake is reaching for 3 or 4 on a task whose order you already knew.

The two axes are independent. A fixed chain calling four tools is a real system. So is a retrieval pipeline calling none. Tool count is not autonomy.

**Build a tool-using agent yourself once, before you touch a framework.** Define a tool schema, send it with the prompt, receive a tool call, execute it, return the result, repeat until the model stops. It is about eighty lines, and it is the chatbot from the intro, rebuilt. After that, frameworks are conveniences rather than magic.

One more reason to write the loop yourself: SDKs that execute tools internally will hand you the model's summary of what came back instead of what actually came back. Once the raw return values are yours, you can render output deterministically from them and check the model's prose against them. That is the difference between a demo and something auditable.

### 5.4 Frameworks

Once you have written the loop, LangGraph is worth learning, because it makes the workflow an explicit graph rather than an open-ended loop and you can see what ran and in what order. What matters: the state object you thread through it, branching on that state, checkpointing so a long run can pause and resume, and an interrupt step when something needs human approval. Run a tracing tool alongside it.

Alternatives keep appearing. What transfers between them is the ladder above and the state design.

### 5.5 Evaluating what you build

Block 2's discipline on a harder target, and the difference between a demo and something you can leave running.

Keep a fixed set of inputs with known correct outputs, however small. Track cost and latency per run, since agent loops can quietly cost far more than a single call. Set hard stops on steps, cost, and time. And record which kind of failure happened, because the pattern is what tells you where to fix it.

### 5.6 What is worth building in bio

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

This block has two halves. The first you can learn anywhere, tonight. The second you mostly cannot learn outside a job, and it is worth knowing that in advance rather than trying to simulate it.

### 6.1 Ask where the data is going

Calling a model API means sending your data to someone else's servers. That is a transfer, not a computation, and it is usually the exact thing a data use agreement prohibits. Pasting a clinical note into a chat window is the same transfer as doing it in code, and it does not become acceptable because it was convenient.

So before anything leaves the environment, know which agreement governs the data and what it says about third-party processing. If you cannot name the agreement, do not send it. Patient privacy rules, residency requirements, IRB conditions, partner contracts, and unpublished IP all sit behind this. These are the terms the data came with, and breaking them can end access permanently. PhysioNet's [guidance on using MIMIC data with LLMs and online services](https://physionet.org/news/post/llm-responsible-use/) may give you a sense of what data constraints look like in industry.

The restriction is narrower than it looks. It applies to the model call, not the whole system. A pipeline that queries public databases can go on querying them. It is the prompt that has to stay inside. Not everything has to be mirrored, though that would be the safest option.

Speaking of running an LLM model where the data already is, Ollama is where you can get started. AWS Bedrock is a more industry facing solution for that, since it keeps the call inside an account whose terms legal has already signed. Local models handle extraction and classification fine and fall apart on open-ended reasoning. The alternative is to keep the frontier model and structure the work so only de-identified or aggregate output crosses the boundary.

Whichever you end up with is usually not your decision, so write the model call to be swappable (5.1).

### 6.2 Working with no network

This is the half you can practice tonight. Dependencies break first. You cannot install packages at runtime, so either build a container image outside and bring it in, or download the packages on a connected machine and install from that local copy. Pin versions, or the environment drifts into an outage later. This is Block 7's conda and Docker material, met under the condition that makes it non-optional.

Model weights break second. Anything that loads a model by name reaches Hugging Face when it runs. Download the weights ahead of time, note the version and date, and load from a local path. Hugging Face has offline settings for exactly this case; turn them on so a missing file fails immediately instead of hanging.

That last part generalizes. Blocked network calls tend to hang rather than error, so a job that should fail in a second sits for minutes looking merely slow. Besides model loading, the usual sources are library telemetry, packages that quietly download reference files the first time you use them, and notebook widgets pulling code from a CDN, which shows up as a blank cell rather than an error message.

### 6.3 Do not leak data on the way out

Notebook outputs are the most common accident: a cell that prints patient records, committed to a repository, is a data incident. Strip outputs before committing. Keep code and data in separate places with separate access. Scrub logs, since error messages often contain the record that caused the error. Never put identifiers in filenames or ticket titles.

### 6.4 Test it

Run the pipeline once with networking disabled before you deploy it. It takes seconds and tells you what you missed, which is cheaper than finding out during deployment.

→ **You'll be able to:** take any workflow from Blocks 1 through 5 and make it run with no network access, and explain to a compliance reviewer where the data goes.

---

## Block 7: Engineering practice

Not a stage. Start it in week one and let it accumulate. Each item below shows up in an earlier block; this is where they live together.

- conda: creating environments and moving them between machines. conda-pack bundles one into a transferable archive, which is the option that works when the environment has non-Python binaries in it. Pin versions. (Block 6.2 is where this becomes mandatory rather than tidy.)
- Enough Docker to build an image, run a container, mount a volume.
- Model freezing, weight import and export, artifact versioning.
- Track your runs in a table. Date, what you changed (parameters), the metric number you got. MLflow automates this. (Block 2: a result you cannot trace back to a configuration is not a result.)
- Tests for data-loading and preprocessing functions. That is where the silent errors live. (Block 3, step 6.)
- One cloud platform, whichever your organization uses. Object storage, managed notebooks, training jobs, model endpoints, secrets, and access control are common to all of them. Learn those six and the vendor becomes a documentation problem.

For a structured version: MLOps (Machine Learning Operations Specialization, Duke University, Coursera).

→ **You'll be able to:** hand someone your analysis and have it run on their machine.

<br><br>

---

<small>
**A note on this piece**
<br>
Thanks to Yifeng Liang, Lengxi Huang, and Karen Wong, who I met through the [CABS summer intern program](https://ds4cabs.github.io). A lot of this document came out of talking with them. Their questions made me organize thinking I had been carrying loosely, and several of their ideas changed how I see the sequence. Teaching turned out to be its own form of learning. None of them started with the Coursera course, because all three already had intro ML and Python coursework behind them. Each took a real dataset and worked a project through over the summer, picking up the pieces as they needed them. That is Block 1 skipped for the right reason, and it works out great.
<br>
Thanks also to Shucheng Cao, from the same program, who came to this with the ML already behind him and went down the list to find Block 6 was his only blank. That question is why the block now says which half of it you can learn without the job.
</small>
