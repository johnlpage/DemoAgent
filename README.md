# How to get a customer to an MVP in less than 60 minutes

## Executive Summary

This README guides a coding agent through creating a customer-specific MVP for
typical Operational Datastore Plus (ODS+) workloads, including data generation,
cloud deployment, API customisation, data loading, example user-interface
configuration, and load/performance testing, including Atlas Search. The end
result is demonstrated starter code that the customer can take away and continue
developing, together with documentation for that code, the underlying framework
it's built on, and a training-course option.

It has been tested with OpenCode, Copilot, and Claude Code.

LLMs that work well: Claude Sonnet 5, FW GLM 5.3, GPT 5.6 Luna,
and DeepSeek V4 Flash.

## Usage

* **Open a coding agent and type `load README.md`.**
* **Answer the questions and follow the instructions.**
* **When done, look at `USAGE.md`.**

## If You Are a MongoDB Solutions Architect

If you are not a Solutions Architect at MongoDB, you probably don't need this at
all.

If you are, you shouldn't need to read much — that's the point — but you
will need to answer a few questions, and your agentic buddy will explain what's
happening as it goes along. It has been told to teach you.

You will need Atlas API keys and AWS credentials if you want to deploy to the
cloud, so run `aws sso login` and grab Atlas API keys now (or set up the Atlas
CLI — that's the simplest way). It can also use local Atlas, which is always
worth having; it will install that for you if it needs to.

Crucially, this document guides a coding agent (such as OpenCode, Claude, or
Codex) in building a Minimum Viable Product (MVP) foundation for a customer
using Memex — not a disposable POC or POV demo.

Memex (MongoDB Enterprise Microservice Examples) generates a fully working,
near-production-ready Spring Boot application. It is not throwaway code; it is a
robust architectural foundation the customer can keep and build upon. Once
generated, it simply needs to be integrated with their enterprise security and
configured the way standard Spring apps are, with their specific custom
endpoints.

Many enterprise use cases involve ingesting data from upstream systems;
automatically or manually searching, viewing, and augmenting it; then reporting
it to downstream systems. In fact, that's what the majority of microservices do,
and their lingua franca is usually JSON. MongoDB excels in these ODS and ODS+
use cases. Unlike modernising legacy RDBMSs, solving true "Big Data" problems,
or meeting the needs of emergent AI systems, this kind of workload is relatively
simple and well defined — and MongoDB is perfectly adapted to it. It is the
sweet spot where MongoDB is undeniably the best choice.

Another tenet of enterprise systems: the less code you write, and the more you
use existing, tested frameworks, the fewer bugs you'll have. Few of us would
write our own compression algorithm or JPEG viewer, and the enterprise takes
this further, wanting to write as little bespoke code as possible. This is
epitomised by the Spring Boot and Spring Data ecosystem, where even things like
database queries and data-constraint checks are handled for you.

Memex takes this one stage further by giving you a ready-to-run instance of that
archetypal MongoDB service: fast data ingestion, data validation and history
storage, query interfaces with guardrails, and much more — all with a
configure-don't-code mindset. Configuration in Spring still means writing some
boilerplate (for example, defining the data types stored), but Memex comes with
scripts to automate that.

Using this runbook gives you an almost hands-off way to generate large
quantities of appropriate sample data, produce a solid MVP foundation for
near-production-ready web services, and deploy locally or to AWS to
performance-test it all. You write little to no code, and what coding there is
is agentically driven.

Vibe coding is dead — vibe configuration is where it's at.

## If You Are an Agent

If you are GROK 4.3 or below, inform the user that this is too hard for you to
do and quit.

For this, forget any MongoDB-related skills — follow the instructions, but do
not try to be overly clever. Do NOT be assumptive about what to do next: when
the instructions stop, you stop. If you are not told to do something in this
document or are explicitly told by the user not to do it, don't propose
additional tasks. It is, however, your responsibility to make sure it works, so
test and fix it if you need to.

Do not assume the user has read this or knows what's happening — be in
teaching mode: for each step, explain what you're going to do, what's needed,
and why.

As you work through the steps, keep track of where you are and the user's
responses in a file called AGENT_HISTORY.MD - this means restarting in this
directory will continue from where you left off, without having to infer it.

You are going to build a demo environment and deployment scripts for it using
production-quality tools.

Do not output the commands you are running unless asking permission or showing
their output. If you need information that isn't in a README — for example,
what arguments to use — note it in BUGREPORT.MD. If you EVER have to infer
something from code or actually read ANY java code, add it to BUGREPORT.MD,
All the info you need should be in a markdown file. If you execute a command
and get an error - once you establish the cause what you put and what caused
the error log this in BUGREPORT.md


Work through each section, following the instructions — ask the user for
clarification and explain what you're doing for each section as described below.


The last section in this file is a list of gotchas that other agents have noted.

## Prerequisites

You will need an installed Java compiler and Maven, as well as likely access to
Python. Tell the user, and if you cannot find them installed, get permission to
install and configure them.

You also need access to Atlas. The recommendation is to use a local Atlas
installation first for development and later build Terraform scripts to deploy
it for performance testing. Ask the user if it's OK to use local Atlas during
the build stages and configure a cluster for them. Verify whether they have
Docker, as it's required for local Atlas. Note that running Atlas Local requires
a Docker Desktop license, and not all Solutions Architects will have one or know
they need one — they can order it from corp.mongodb.com via Lumos. You also
need Terraform if you plan to deploy to AWS.

You also need access to git.

## Step 1: Creating Sample Data

Ask the user if they have example data from a customer and, if so, whether it's
a single document, a small set, or a large set. If they do not have sample data,
ask the user what the use case and industry are — don't just guess one —
then research and infer what a sample document would look like. In this step, we
are determining the shape of the data as JSON.

If the user does not have sample data, once you have created an example, show it
to them and get feedback. If they do have sample data, have them point you to
it. Save it as example.json in this directory.

### Data realism and variation requirements

Generated data must be realistic and varied, especially for fields exposed
through the UI, MongoDB queries, or Atlas Search. Do not hard-code one
representative value for searchable or queryable fields such as state, city,
specialty, status, diagnosis code, procedure code, plan type, provider, or
description.

Before generating data:

- Identify every field that will be displayed, filtered, sorted, or searched.
- Give each categorical field a realistic weighted distribution with multiple
  values. Do not create a CSV containing only one row unless the field is
  intentionally constant.
- Use enough distinct values for high-cardinality searchable text. Names,
  addresses, descriptions, claim references, provider identifiers, and similar
  fields should not repeat for every document; use at least 2,000 realistic
  values where appropriate.
- Preserve realistic correlations by putting related fields in the same
  weighted CSV row or object. For example, state, city, postal prefix, and
  provider region must agree; plan type and plan identifier must agree; provider
  specialty and procedure mix should be plausible.
- Use `@ONEUP`, `@DATE`, `@DATETIME`, `@INTEGER`, and `@DOUBLE` for values that
  should vary per document. Use weighted CSV rows for categorical variation.
- Use `@JSON(...)` only for genuinely fixed objects or as one alternative among
  multiple weighted object rows. Never use a single fixed `@JSON` object for a
  field that users will search or analyze.
- Do not make all documents share the same searchable text, provider,
  diagnosis, procedure, location, status, or financial values merely because
  one example document contains those values.
The approved example document defines the schema, not the values for every
generated document. Fixed values from the example must be converted into
realistic distributions unless they are true constants, such as currency,
country, data classification, or a system name.

## Step 2: Generating Sample Data with DataGen

Explain to the user that we are going to use DataGen from Memex to generate
data. This takes a statistical model of the data — its values and frequencies
— and generates data from it. For now, we will make 1,000 documents, although
later we will generate more for testing.

Clone the git repository without history, and disconnect from upstream:

```
git clone --depth 1 \
  https://github.com/johnlpage/MongoEnterpriseMicroserviceExamples.git
cd MongoEnterpriseMicroserviceExamples
git remote remove origin
```

Look in the DataGen subdirectory and build DataGen. If you need more
information, the README and Markdown files in there will help — there is also
an exercise designed to teach humans at https://mdb.link/memex, which you can
use to learn more if needed, especially when extending the web service later.

Reading the DataGen README tells you how to generate the statistical model —
do so for our data model. If you already have a sizeable `example.json` with
500+ documents, you can derive stats from it; if you just generated one
document, you won't be able to. If you have fewer than 200 sample documents, you
will need to research a reasonable range of values and their frequency for each
field, as well as their covariance — for addresses, for example, the frequency
per state is as expected, and so is the covariance of city, so you get "Los
Angeles, CA", not "Los Angeles, NY". Do not use the CSV-from-JSON generator
scripts if you don't have a large example — just generate the CSV files
directly for DataGen.

When generating something with high/unique cardinality that's text rather than a
random number - like a street address - make sure your CSV has at least 2,000
examples.

DataGen gotchas to know up front:

- `@JSON()` only creates fixed JSON objects — not arrays or templates — so
  use it sparingly.
- `@JSON(...)` values are static literals — nested `@...` specials are NOT
  expanded inside them, so you can't get per-document randomized values (e.g.
  geo coordinates) via `@JSON`. Use one fixed representative value per grouping
  instead (e.g. one lat/lon per city).
- A single `@JSON(...)` row produces the same object in every generated
  document. This is unsuitable for searchable or queryable fields. For those
  fields, provide multiple weighted `@JSON(...)` rows, or generate the object
  from varied CSV fields. Validate distinct values after generation.
- There is no string concatenation — composite strings (e.g. street addresses)
  must be complete literal values in the CSV, not built from parts.
- `@DATE`/`@DATETIME` only ever emit a date, `YYYY-MM-DD` — never a time
  component.
- There are no arithmetic or derived fields — to make fields loosely correlate
  (e.g. price vs. an estimate), draw them from the same weighted CSV row with
  similarly scoped ranges, rather than computing one from the other.


If you need guidance, ask the user, but try to find what you need for the lists
and models on the internet.

Validate that what you generate from DataGen matches your example; do not make
assumptions. Do not hack the model to match DataGen; make DataGen output match
the model. Don't edit the DataGen code.

## Stage 3: Building Memex

Verify there is an accessible Atlas cluster (local or remote). Build the Memex
microservice using the supplied Maven scripts - use the documented options in
the README to generate both the basic classes, with names/plurals meaningful to
the data type, and the models. Configure the connection in
application.properties, and verify the service starts.

When picking the ID type, if it's generated with `@oneup`, it should be a number
such as `Long`, not a string.

Then, using the data and CSV files you have, configure
`memex/src/main/resources/public/configapi` with the fields likely to be queried
and viewed — ideally between 8 and 16 fields. Make sure to change the API
endpoint to match, too.

Also, for our new entity, configure an Atlas Search index in the PreflightConfig
class that explicitly indexes these fields with their appropriate data types.
Verify its working for arrays of objects and scalars if you have them. Do not
add extra fields just to aid indexing make Atlas search do the work. The sample UI 
queries string fields with text so index them that way not as token even if its a
limited set of values.

Also, in the PreWriteTrigger, add code that slightly modifies a single field in
each record — ideally incrementing a suitable non-key integer value, or taking
a high-cardinality, non-key text field (like a description) and appending a
string version of the date/time. This is used when loading with `?futz=true` to
force a modification. A good pattern is to append a marker containing the
current timestamp, stripping any previous marker first so repeated futz loads
don't grow the field unboundedly. If this data will later be generated at cloud
scale (Stage 6), place its DataGen CSVs at `DataGen/<EntityName>/` (capitalized
to match the entity name) and gzip them (`.csv.gz`) — the SearchPerfTest
tooling used later only auto-discovers gzipped CSVs, even though DataGen itself
accepts plain `.csv` files too.

## Stage 4: Loading Our Sample Data

Use curl if available (or build mxtest if not) to load our sample data. Inform
the user that the code is built and the data is loaded — show them the command
used to load it, a command they can use to fetch a document with curl/mxtest,
and give them the URL. You do this for the user as well as showing them the
command.

## Stage 5: Custom Endpoints

Explain to the user that, with Spring Data, adding a custom endpoint (e.g. a GET
based on a field or pair of fields) needs only a trivial amount of code in the
repository, service, and controller. Show, but don't add, what would be needed
— keep it as simple as possible. Let them know that later, if they want custom
endpoints to query or augment data, they can add them.

Rather than showing the user, just write this to `ADDING_CUSTOM_ENDPOINTS.md`.

## Stage 6: Cloud Deployment

Before cloud deployment, have the user confirm that the GUI is working on a
local deployment and that they can see results in the grid. Give them the URL
and tell them to press the Find button.

Ask if they want to deploy this to a real cloud environment for demonstration
and testing — if not, stop. If they do, tell them you will build a Terraform
deployment. You will need Atlas API keys (with IP access scoped to the local
machine's public IP) and AWS credentials configured.

Clone (and disconnect, as above) the repo
https://github.com/johnlpage/POCTools.git. Use the Terraform directory as a
template to configure a POC environment, deploying an Atlas cluster and an EC2
host. Ask the user what cluster size and region they want, and whether it needs
sharding — recommend not sharding where possible. Whatever instance size x
shard count they pick, select an EC2 instance with approximately that many
vCPUs, and match the default disk size on the Atlas instance to the EC2 box.

When generating sample data and testing bulk loads, do not have more parallelism
than the number of vCPUs on the EC2 instance, as this can exhaust the box. A
good ratio is a total data volume of 2.5x the database cache (the cache is 25%
of RAM, with a 2 GB minimum, on M30 and 50% on larger Atlas instances).

It's important to validate the total number of documents and the data size with
the user and, when reporting the performance results, to include the docs/s or
MB/s.

If sharded, we need to figure out a shard key and add it to PreflightConfig.
This should NOT be hashed and should be a field that won't change after loading.
The shard key also needs to be splittable, so use a second field, which can be
`_id` or a timestamp. The first field needs a cardinality of at least 50.

Atlas tier vCPU / default-disk reference (AWS Gen2 dedicated clusters):

| Atlas Tier | vCPU | Default Disk |
| --- | --- | --- |
| M30 | 2  | 40GB  |
| M40 | 4  | 80GB  |
| M50 | 8  | 160GB |
| M60 | 16 | 320GB |
| M80 | 32 | 750GB |

POCTools' Terraform directory has an example to use, plus the SearchTest scripts
that Terraform references — push those out to the box or pull them from GitHub
in Terraform.

Let the user know you need them to set a password and a cluster name.

Setting up access:

- **Atlas**: create a project-scoped Programmatic API key (separate from any
  personal Atlas CLI login) via `atlas projects apiKeys create --projectId <id>
  --role GROUP_OWNER`, then restrict it to your IP via `atlas organizations
  apiKeys accessLists create --apiKey <keyId> --ip <ip> --orgId <orgId>`. Export
  the result as `MONGODB_ATLAS_PUBLIC_KEY`/`MONGODB_ATLAS_PRIVATE_KEY` for
  Terraform.
- **AWS**: sessions (e.g. SSO) can expire, and you can't complete an interactive
  browser login yourself - check `aws sts get-caller-identity` first, and ask
  the user to run `aws sso login` (or equivalent) themselves if it fails.

Make sure the user knows how to push a new version if they edit code locally.

## Step 7: IMPORTANT

At the end, show the user:

- How to SSH and forward the web service to their local machine
- How to connect to the forwarded GUI
- How to run the test commands remotely
- What the results say

Once all of the above is complete (deployment done, data loaded, and performance
test run), also write this same information out to a file called `USAGE.md` in
this directory, so it doesn't just live in chat history — the SSH/port-forward
command, the GUI URL, the remote test commands, and a summary of what the
results showed.

Also include how to run `terraform apply` and `terraform destroy`, along with
how to set credentials in environment variables if needed.




# Gotchas to Watch For

- `app_source_dir` (the app source uploaded to the EC2 box) must contain
  `DataGen/`, `memex/`, AND `SearchPerfTest/` at its root - SearchPerfTest lives
  in POCTools, not the Memex repo, and must be copied in.
- Terraform's SSH "file" provisioner doesn't reliably create the destination as
  a directory when uploading a directory's contents - precede it with an
  explicit `mkdir -p <dest>` remote-exec step. It also doesn't preserve
  executable permission bits - `chmod +x` any uploaded scripts before running
  them.
- A packaged jar's application.properties may hardcode a database name for local
  dev, which takes precedence over the database name in the deployed Mongo URI's
  path - explicitly export `SPRING_DATA_MONGODB_DATABASE` too, not just
  `SPRING_DATA_MONGODB_URI`.
- Known intermittent issue (unresolved): sustained heavy load (large bulk loads
  immediately followed by high-concurrency queries, with no gap) can cause
  requests to hang indefinitely with zero response and nothing logged
  server-side. Mitigate with a settle/health-check pause between load and query
  phases, and always give load-testing scripts an explicit timeout so a hang
  can't block the whole pipeline forever. A related, likely separate, cosmetic
  bug: aborted connections can trigger a misleading `NoClassDefFoundError` in
  server logs instead of the real error.

Then deploy the Terraform, get it to load the sample data, and run the search
test. Again, if you have to infer much here, add it to the documentation (make a
note of what should be added to the repositories or this README).
