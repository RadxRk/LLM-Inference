# LLM Inference Fundamentals

A four-part series on how large language models are served in production, written for engineers and architects who work with these systems but have not built one. It starts from the hardware and ends at the autoscaler, and every number in it is one you can derive with a pen.

## Executive summary

An LLM request runs in two phases with different physics. **Prefill** processes the whole prompt in one pass and is limited by arithmetic. **Decode** produces the answer one token at a time, and every step re-reads all of the model's weights, so it is limited by memory bandwidth. That asymmetry explains almost everything an operator sees: why time to first token grows with prompt length while inter-token latency does not, why batching requests is nearly free until a knee and then expensive, why the KV cache rather than the weights decides how many requests fit on a GPU, and why the 99th percentile explodes long before the median moves.

Three formulas carry the series:

- **Decode floor** = weight bytes ÷ memory bandwidth (about 21 ms per token for a 70B model in FP8 on one H100)
- **Prefill time** = input tokens × 2 × parameters ÷ FLOPS (about 0.4 s for a 3,000-token prompt on a 70B)
- **KV cache per token** = 2 × layers × KV heads × head dimension × bytes (about 320 KB for Llama 3 70B in FP16)

With those, a benchmark table becomes something you can rebuild from first principles, every optimization in the field can be classified by which limit it attacks, and an autoscaling setting becomes a statement about cold starts. The series is vendor-neutral; every figure is cited to NVIDIA datasheets, model cards, or the original papers.

## The series

| Part | Title | What it covers | PDF |
|---|---|---|---|
| 1A | What Inference Is, and Why It Became Everyone's Problem | Training once vs inference forever; the inference stack; what open-weight models changed; the latency, throughput and cost triangle | [Inference-Fundamentals-Part-1A.pdf](Inference-Fundamentals-Part-1A.pdf) |
| 1B | The Physics of an LLM Request | Tokens, weights, layers, the two operations per weight; the two phases; where the memory goes; why decode is memory-bound and where the knee and the cliff come from | [Inference-Fundamentals-Part-1B.pdf](Inference-Fundamentals-Part-1B.pdf) |
| 2 | What the Serving Engine Does with the Physics | Continuous batching, the paged KV cache, preemption, streaming; every optimization classified by what it attacks; speculative decoding, prefix caching and disaggregation step by step | [Inference-Fundamentals-Part-2.pdf](Inference-Fundamentals-Part-2.pdf) |
| 3 | Reading Benchmarks and Running the System | Anatomy of a request; the metric conversions; rebuilding the curves; queueing and p99; the seven break points; the autoscaler as a control loop; cold starts; a day on the dashboard; glossary | [Inference-Fundamentals-Part-3.pdf](Inference-Fundamentals-Part-3.pdf) |

Read 1A and 1B first; they stand on their own. Parts 2 and 3 build on them. Each part ends with a short self-test and a Sources section.

## Sources

Every technical claim is cited at the end of the part where it appears: NVIDIA product pages for H100, H200, B200 and A100; the Llama 3 configuration and paper; the DeepSeek-V3 technical report; Kaplan et al. on scaling laws; the Orca, PagedAttention, SGLang, speculative decoding, Medusa, EAGLE and lookahead decoding papers; NVIDIA's Dynamo announcement; and the release announcements from Meta, Mistral, Alibaba, DeepSeek and OpenAI.

## Author

Radx Radhakrishnan works on LLM inference and agentic systems, after a decade of performance work on enterprise data platforms.

## License

Text and diagrams: CC BY 4.0. Cite the source when reusing figures.
