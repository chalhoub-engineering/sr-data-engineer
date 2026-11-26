# Senior Data Engineer — Case Study

## About this case study

This case study has two parts. In the first you build a small but complete batch pipeline, from a public dataset through object storage and PostgreSQL to tested, documented reports. In the second you write a short design note on how you would take that pipeline to production for a retail group.

There is no single correct answer. We are interested in the pipeline working, and just as much in the decisions behind it: what you chose, what you deliberately left out, and what you would do differently with more time.

Please limit your work to around 6 hours in total, roughly 4–5 on the build and 1 on the design note. If you run out of time, stop and write down what is missing. A smaller pipeline that runs is worth more than a larger one that does not.

## Part 1: Build the pipeline

Choose a public dataset with transactional data and a few related entities, ideally retail-related. Good options are the [UCI Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) dataset, the [Olist Brazilian e-commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) dataset or the [NYC TLC trip records](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page). Any freely downloadable dataset with at least 100,000 rows is fine.

Everything must run locally: an S3-compatible object store such as MinIO, PostgreSQL, Python and a transformation framework. No cloud account should be needed to run your solution.

Steps 3 and 4 are written for dbt, which is what we use. You may substitute any other transformation framework you know well, such as SQLMesh or Dataform, provided it gives you the same things: version-controlled models with dependencies between them, incremental builds, automated tests and documentation. If you do, tell us briefly why you chose it.

1. **Ingest to object storage.** Write a Python script that downloads the dataset and uploads the raw files to a bucket in your local object store. Organise the files by source and load date, for example `raw/online_retail/2026-11-06/`. The load date must be a parameter, and running the script twice for the same load date must not create duplicate data.
2. **Load to PostgreSQL.** Write a Python script that reads the raw files from object storage and loads them into a `raw` schema in PostgreSQL. Keep the data as close to the source as possible at this stage, and record the load timestamp and source file on each row. Reloading a file that has already been loaded must not duplicate rows.
3. **Model the data.** Create a dbt project (or equivalent) that builds staging models (cleaned and typed, one per source table) and a small dimensional mart, for example `fct_sales` with `dim_customer`, `dim_product` and `dim_date`. Build the fact table incrementally. Add tests for keys, nulls and relationships, and document the models alongside the code.
4. **Build reports.** Add a reporting layer on top of the mart with at least three report models, for example `rpt_monthly_revenue_by_country`, `rpt_top_products_by_month` and `rpt_customer_retention`. Each report should be a model in the same project with a clear grain, tests and documentation, so that a BI tool could read it directly. Include sample output from each report in your README.
5. **Make it runnable.** Provide a `docker-compose.yml` for the object store and PostgreSQL, and a single entry point, for example `python run_pipeline.py` or `make run`, that executes the steps above in order. We will run it on a clean machine with only Docker and Python installed.
6. **Show that it is incremental.** Run the pipeline for at least two load dates; splitting the dataset into two batches is fine. Include evidence in your README, such as row counts or run logs, that the second run added only the new data and that repeating either run changed nothing.

Real transactional data is messy, and your chosen dataset will contain records that are not straightforward sales, are incomplete, or are duplicated. Decide how to treat them and tell us what you found and what you decided.

We expect the engineering standards you would apply to code a team will maintain: configuration through environment variables with no credentials committed, logging and error handling, pinned dependencies, and automated tests for at least one piece of the Python logic. A CI workflow is welcome but optional.

## Part 2: Design note

Now assume the pipeline is going into production for a retail group, and you are the senior engineer responsible for it.

The group has 150 stores. Each store sends a sales file every hour, about 5 million sales lines a day in total, and stores resend corrected files for up to three days after the trading day. The source team changes file layouts occasionally and does not always announce it. Finance and Commercial read the reports each morning at 07:00. Two mid-level engineers will run the pipeline day to day once it is live.

Write a design note of no more than two pages, with one architecture diagram and no code. Cover all six questions briefly, and choose two to explore in depth.

1. **Orchestration.** How would you schedule the pipeline, handle failures and retries, and backfill a week of history?
2. **Late and corrected data.** How would the raw layer, the fact table and the reports stay correct when a store restates a day that has already been published? How would you explain to Finance why yesterday's figure moved?
3. **Schema changes.** What should happen when a file arrives with a new, missing or renamed column?
4. **Quality and monitoring.** What would you check, where, and which failures should wake someone up at night?
5. **Scale and cost.** Which part of your implementation breaks first at this volume, and what would you change? You may assume any cloud warehouse and tooling you know well.
6. **Ownership.** What would you put in place so that the two engineers can run, change and debug this without you: conventions, CI, reviews, runbooks?

## What we look for

| Area | Expectation |
| --- | --- |
| Correctness | The pipeline runs end to end from one command and the reports reconcile with the source data |
| Idempotency and incrementality | Repeated runs are safe at every layer; new load dates add only new data |
| Data modelling | Clear grains, sensible keys, staging separated from marts and marts from reports |
| Data quality | Problems in the source data are found, handled deliberately and covered by tests |
| Engineering quality | Readable, configurable, tested code with a history of meaningful commits |
| Design judgement | The design note makes specific choices for this scenario, explains the trade-offs and says what it would leave for later |
| Communication | A README that lets us run and understand your work without asking you |

## Deliverables

A private Git repository containing:

- the ingestion and load scripts, the transformation project, `docker-compose.yml` and the entry point;
- a README with run instructions, your design decisions, the data issues you found, sample report output, the evidence for step 6 and known limitations;
- the design note for Part 2, as Markdown or PDF.

Please do not commit the dataset itself.

## Practical details

Please create your own private repository rather than forking this one, and grant access to [daniel-rosiak](https://github.com/daniel-rosiak) and [ivansinitsyn-cmy](https://github.com/ivansinitsyn-cmy) at least 24 hours before the session.

The session lasts 60 minutes: a 10-minute demo of the pipeline running, 20 minutes walking us through the code and the design note, and 30 minutes of Q&A. We may ask you to make a small change to your solution during the session, so please have it ready to run.

The brief is intentionally incomplete. Please state any assumptions you make, and feel free to email the recruiter with questions before the session.

You may use AI tools, as you would in your day-to-day work. However, please be prepared to explain and justify every line and every decision.
