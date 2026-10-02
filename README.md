# Text Analytics Portfolio
Portfolio of Text Analytics class (ISM6564) assignments

Assignment 1-5 were individual assignments where I utilized Claude Code to do and explain code, while I wrote the task memos at the end of each task. Assignments 6-10 were group work where this repo was used to collaborate together. Assignments 1-5 are more pure code based while assignments 6-10 focus more on agents and models. Here is a breakdown of the assignments: 

Assignment 1: Tokenization Report
Compared whitespace, regex, and NLTK tokenizers on a 20 Newsgroups corpus, then fit Heaps' law to measure vocabulary growth against two fixed vocabulary subword tokenizers (BERT, GPT-2). Quantified out of vocabulary rates, Unicode normalization pitfalls (NFC vs. NFD), and the cost of running an LLM API across five languages, concluding with a tokenization strategy memo for search, summarization, and international rollout use cases.

Assignment 2: Vector-Space Search Engine
Built a retrieval system over a 128 article help center corpus using four vectorization schemes (count, TF-IDF, min-df, bigrams) and four similarity metrics (cosine, dot, Euclidean, Jaccard), evaluated with precision@5, recall@10, MRR, and nDCG@10. Diagnosed document length bias and near duplicate articles, then tuned and recommended a final configuration (TF-IDF bigrams + stop-word removal + cosine) with a written memo defending the trade offs.

Assignment 3: Support Ticket Router
Trained and benchmarked six classifier families (logistic regression, linear SVM, kNN, random forest, XGBoost, MLP) on TF-IDF features to route 3,000 support tickets into five queues, enforcing a leakage safe train/validation split. Shipped a logistic regression pipeline with a confidence based auto-route/human-triage threshold and used it to route a 400 ticket batch.

Assignment 4: Sequence Models for Sentiment
Compared bag of words models against an LSTM/GRU built from scratch in PyTorch on product reviews specifically engineered to test word order sensitivity (contrast and negation-scope cases). Showed that reading in order resolves cases a bag of words model gets exactly 50/50, and recommended the LSTM with measured accuracy, size, and training time tradeoffs.

Assignment 5: Local LLMs for Ticket Triage
Benchmarked three open weight instruction tuned language models (360M–1.5B parameters) on memory footprint, token generation speed, and classification accuracy over 60 hand labeled support tickets, comparing local inference against a frontier API fixture on cost and accuracy. Delivered a recommendation memo weighing data privacy, cost, and performance trade offs between on device and third party inference.
