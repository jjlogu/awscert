# AI & machine learning: AWS services explained for beginners
> [Home](../../README.md) · [Beginner track](../README.md) · [All services A–Z](README.md) · Deep dive: [Notes 09](../../notes/09-data-analytics-migration.md)

AWS offers two layers of machine learning (ML). The top layer is **ready-made AI services**: you send an image, document, audio clip or text to an API and get a result back, with no ML knowledge needed. The bottom layer is **Amazon SageMaker AI**, where data scientists build and train their own models. The SAA-C03 exam treats AI as a matching game: read what goes in and what should come out, then pick the service.

### The exam only needs one line per AI service: input → output

| Input | Output | Service |
|---|---|---|
| Image or video | Labels, objects, faces, unsafe content, text in a photo | **[Amazon Rekognition](#amazon-rekognition)** |
| Document (scan, PDF, form, invoice) | Text, **forms (key-value pairs) and tables** | **[Amazon Textract](#amazon-textract)** |
| Text | Sentiment, entities, key phrases, language, **PII** | **[Amazon Comprehend](#amazon-comprehend)** |
| Speech (audio) | Text (a transcript) | **[Amazon Transcribe](#amazon-transcribe)** |
| Text | Speech (audio) | **[Amazon Polly](#amazon-polly)** |
| Text in one language | Text in another language | **[Amazon Translate](#amazon-translate)** |
| A user's typed or spoken question | A conversation (chatbot, voice bot) | **[Amazon Lex](#amazon-lex)** |
| Your own data | Your own custom model, trained and hosted | **[Amazon SageMaker AI](#amazon-sagemaker-ai)** |

Memory trick: **Transcribe** writes down what it hears, **Polly** the parrot speaks, **Textract** extracts from documents, and **Rekognition** recognizes what it sees.

| Service | One-liner | Exam importance |
|---|---|---|
| [Amazon SageMaker AI](#amazon-sagemaker-ai) | Build, train and deploy your own ML models | ★★ |
| [Amazon Rekognition](#amazon-rekognition) | Image and video analysis | ★ |
| [Amazon Textract](#amazon-textract) | Extract text, forms and tables from documents | ★ |
| [Amazon Comprehend](#amazon-comprehend) | Natural-language processing: sentiment, entities, PII | ★ |
| [Amazon Transcribe](#amazon-transcribe) | Speech → text | ★ |
| [Amazon Translate](#amazon-translate) | Text → another language | ★ |
| [Amazon Polly](#amazon-polly) | Text → lifelike speech | ★ |
| [Amazon Lex](#amazon-lex) | Chatbots and voice bots | ★ |
| [Amazon Bedrock](#amazon-bedrock) | API access to generative AI foundation models | ★ distractor / not in scope |
| [Amazon Kendra](#amazon-kendra) | ML-powered enterprise search (closed to new customers) | ★ distractor / not in scope |
| [Amazon Personalize](#amazon-personalize) | Real-time recommendations | ★ distractor / not in scope |
| [Amazon Forecast](#amazon-forecast) | Time-series forecasting (closed to new customers) | ★ distractor / not in scope |

---

## Amazon SageMaker AI

> **In one line:** a managed platform for data scientists to build, train, tune and deploy their **own** machine learning models.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like running Kubeflow on your cluster (notebooks, training Jobs, model-serving Deployments), with AWS running the infrastructure. Everyday: a fully equipped professional kitchen you rent, vs ordering a ready meal (the AI services). | Per instance-hour for notebooks, training and endpoints, plus storage |

**What it is.** The AI services below use models AWS trained. SageMaker AI is for when you need a model trained on **your** data for **your** problem, such as fraud scoring or demand prediction. It gives you notebooks to explore data, managed training jobs that spin up GPU or CPU instances and shut them down when done, and managed **endpoints** that host the model behind an API. It was called "Amazon SageMaker" until Dec 2024, when AWS renamed it SageMaker AI and reused the name "Amazon SageMaker" for a broader data and AI platform.

**Key ideas to know**
- **Studio / notebooks**: the development environment for data scientists.
- **Training job**: runs your training code on instances you choose, billed only while it runs (like a Kubernetes `Job`). Managed Spot training can cut the cost.
- **Endpoint (inference)**: real-time, serverless, asynchronous, or **batch transform** for offline bulk predictions.
- **Built-in algorithms and JumpStart**: pre-made algorithms and pre-trained models to start from.
- **Canvas**: a no-code interface for business analysts.

**Use it when / don't use it when**
- ✅ "Data scientists need to **build and train a custom model**", "host a model as an API".
- ❌ A common task such as OCR, translation or sentiment → the ready-made AI service is simpler and cheaper.

**Exam hot spots**
- "Custom model", "train", "data scientists", "deploy an ML model to an endpoint" → SageMaker AI. A pre-built task ("detect faces", "translate text") → the matching AI service.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon S3](storage.md#amazon-s3)

## Amazon Rekognition

> **In one line:** analyzes **images and video** to find objects, scenes, faces, celebrities, text in photos, and unsafe content.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: a security guard who watches the camera feed and describes what's on it | Per image analyzed, per minute of video |

**What it is.** You pass an image (often from S3) or video, and Rekognition returns labels ("car", "beach"), face matches, content-moderation flags or PPE (personal protective equipment) detection. No ML training is needed, though Custom Labels can learn your own objects.

**Exam hot spots**
- "Detect **inappropriate** images uploaded by users", "**face** comparison/search", "identify objects in photos or video" → Rekognition.
- ⚠️ Trap: text in a *photo* (a street sign) → Rekognition. Text, forms and tables in a *document* → Textract.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Textract](#amazon-textract)

## Amazon Textract

> **In one line:** reads scanned documents and PDFs and extracts not just the text, but **form fields and tables** with their structure.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: a clerk who retypes paper forms into a spreadsheet, field by field | Per page processed |

**What it is.** Textract goes beyond plain OCR (optical character recognition): it understands that "Invoice total: 120.00" is a key-value pair and that a grid is a table. It also has specialized analysis for invoices, receipts and identity documents.

**Exam hot spots**
- "Extract data from **scanned forms, invoices, receipts** or tables" → Textract. A common pipeline: S3 upload → Lambda → Textract → Comprehend (for PII) → DynamoDB.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Comprehend](#amazon-comprehend), [Amazon Rekognition](#amazon-rekognition)

## Amazon Comprehend

> **In one line:** natural-language processing (NLP) for text: sentiment, entities, key phrases, language detection, topics and **PII detection**.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: an assistant who reads every customer email and tags it "angry, about billing, mentions a credit card number" | Per unit of text processed |

**What it is.** You send text, and Comprehend tells you what it's about and how the writer feels. It also finds PII (personally identifiable information) such as names, addresses and card numbers, so you can redact it. Comprehend Medical is a variant for clinical text.

**Exam hot spots**
- "Analyze **sentiment** of customer reviews or social media", "detect or redact **PII** in text" → Comprehend.
- ⚠️ Trap: PII in *text* → Comprehend. Sensitive data in *S3 buckets* at scale → [Amazon Macie](security.md#amazon-macie).

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Macie](security.md#amazon-macie)

## Amazon Transcribe

> **In one line:** converts **speech to text**, from recordings or live audio streams.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: a court stenographer | Per second of audio |

**What it is.** Transcribe turns audio into a time-stamped transcript, with speaker identification, custom vocabularies, and automatic PII redaction. Call Analytics adds call-center insights.

**Exam hot spots**
- "Convert **call recordings** or meetings to text", "generate subtitles" → Transcribe. Often chained: Transcribe → Comprehend for sentiment → Translate for other languages.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Polly](#amazon-polly), [Amazon Comprehend](#amazon-comprehend)

## Amazon Translate

> **In one line:** neural machine translation of text between languages.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: an on-call interpreter | Per character translated |

**What it is.** Translate converts text (or documents) from one language to another, in real time or in batch. Custom terminology keeps brand and product names consistent.

**Exam hot spots**
- "**Localize** product descriptions or user content into multiple languages" → Translate. It's text → text; for audio, combine it with Transcribe and Polly.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Transcribe](#amazon-transcribe)

## Amazon Polly

> **In one line:** turns **text into lifelike speech** in many voices and languages.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: an audiobook narrator on demand | Per character converted |

**What it is.** You send text (optionally with SSML, Speech Synthesis Markup Language, to control pronunciation and pauses), and Polly returns an audio stream or file. Neural voices sound most natural.

**Exam hot spots**
- "Read articles **aloud**", "add voice to an app", "text-to-speech" → Polly. ⚠️ Trap: Polly is the *reverse* of Transcribe.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon Transcribe](#amazon-transcribe), [Amazon Lex](#amazon-lex)

## Amazon Lex

> **In one line:** builds **conversational chatbots and voice bots** that understand what the user wants and collect the details.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Everyday: a receptionist who asks the right follow-up questions | Per text or speech request |

**What it is.** You define **intents** ("book a hotel") and the **slots** they need (city, dates), and Lex handles the conversation, then usually calls a Lambda function to do the work. It uses the same speech-recognition technology as Alexa and powers bots in Amazon Connect contact centers.

**Exam hot spots**
- "Build a **chatbot**", "conversational interface", "IVR (interactive voice response) bot for a call center" → Lex (often with Lambda and Amazon Connect).

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [AWS Lambda](compute.md#aws-lambda), [Amazon Polly](#amazon-polly)

## Amazon Bedrock

> **In one line:** a fully managed API for using and customizing **generative AI foundation models** (from Amazon and other model providers) without running any GPUs yourself.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ distractor / not in scope | Like calling a hosted LLM API instead of serving a model in your cluster | Per input/output token, or provisioned throughput |

**What it is.** Bedrock offers many large language and image models behind one API, plus Knowledge Bases (retrieval-augmented generation over your documents), agents and guardrails. It's central to AWS's current AI story, but it isn't on the SAA-C03 in-scope service list.

**Exam hot spots**
- If it appears, it's usually a distractor. "Generate text or images with a foundation model" is the only scenario that points to it.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon SageMaker AI](#amazon-sagemaker-ai)

## Amazon Kendra

> **In one line:** ML-powered enterprise search that answers natural-language questions across company documents. **Closed to new customers since July 30, 2026.**

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ distractor / not in scope | Everyday: a librarian who has read every internal wiki page and manual | Per index-hour plus connectors (existing customers only) |

**What it is.** Kendra indexes content from S3, SharePoint, Confluence and other sources, and answers questions like "what is our parental leave policy?". AWS put it into maintenance mode on June 30, 2026 and stopped accepting new customers on July 30, 2026. It recommends **Amazon Bedrock Knowledge Bases** instead.

**Exam hot spots**
- Distractor only. "Intelligent search over internal documents" pointed to Kendra in older material.

**Go deeper:** [Kendra availability change (AWS docs)](https://docs.aws.amazon.com/kendra/latest/dg/kendra-availability-change.html) · related services: [Amazon Bedrock](#amazon-bedrock), [Amazon OpenSearch Service](databases.md#amazon-opensearch-service)

## Amazon Personalize

> **In one line:** builds real-time **recommendations** ("customers who viewed this also viewed…") from your users' interaction data.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ distractor / not in scope | Everyday: a shopkeeper who remembers what each regular likes | Per data processed, training hours and recommendation requests |

**What it is.** You stream user events (clicks, purchases) and item data to Personalize, and it trains recommendation models and serves personalized rankings via an API. It uses the same kind of technology as Amazon.com's recommendations and is still generally available, but it isn't on the SAA-C03 in-scope list.

**Exam hot spots**
- Distractor only. "Personalized product recommendations without ML expertise" → Personalize.

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon SageMaker AI](#amazon-sagemaker-ai)

## Amazon Forecast

> **In one line:** time-series forecasting (sales, demand, capacity) from historical data. **Closed to new customers since July 29, 2024.**

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ distractor / not in scope | Everyday: a weather forecaster for your sales figures | Existing customers only |

**What it is.** Forecast trained forecasting models on historical time-series data. Existing customers can keep using it, but AWS no longer accepts new customers and points them to **SageMaker Canvas** (part of SageMaker AI) for time-series forecasting.

**Exam hot spots**
- Distractor only. For a new forecasting workload, the current answer is SageMaker AI (Canvas).

**Go deeper:** [Notes 09: 12. Managed ML/AI services](../../notes/09-data-analytics-migration.md#12-managed-mlai-services-one-liners) · related services: [Amazon SageMaker AI](#amazon-sagemaker-ai)

---

**Next:** [All services A–Z](README.md)
