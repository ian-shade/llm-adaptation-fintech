**fig00_training_length_budget** — Distribution of training sequence lengths against the truncation budget. Examples to the right of the dashed line lost their EOS token under the previous tokenisation.

**fig01_corpus_profile** — Retrieval corpus profile: token length per embedding unit against the encoder limit, and embedding units per bank. Unequal coverage across the three reports is a confound for the per-bank results and is reported here so it can be quoted alongside them.

**fig02_accuracy_by_type** — Objective accuracy by question type for each condition. Open-ended questions are excluded because they have no binary correctness; n per type is shown on the axis to make small-sample types explicit.

**fig03_open_ended_quality** — Open-ended answer quality on four complementary measures. BERTScore is bounded well above zero by construction, so differences rather than absolute levels are the meaningful comparison.

**fig04_hallucination_profile** — Hallucination measured three ways. The automatic proxy counts confident wrong answers, the claim-level rate verifies decomposed claims against the corpus, and the adversarial rate counts non-abstentions on unanswerable questions.

**fig05_accuracy_vs_cost** — Accuracy against inference latency, with marker area proportional to total GPU cost including the one-off training cost carried by the tuned conditions. Upper-left is better.

**fig06_paraphrase_robustness** — Accuracy on the same questions before and after paraphrasing. A steep downward slope indicates a system keyed to surface wording rather than meaning.

**fig07_adversarial_behaviour** — Response to questions with no answer in the corpus. Abstention is the correct behaviour; the red segment is fabricated content.

**fig08_answer_length** — Answer length by condition against the decoder cap. A distribution pressed against the cap indicates the model is not emitting its stop token rather than genuinely producing longer answers.

**fig09_response_profile** — Decomposition of every objective answer into correct, wrong-but-abstaining, and wrong-but-confident. The red band is the share of answers that are both wrong and assertive, which is the operationally dangerous category.

**fig10_retrieval_quality** — Distribution of best-match similarity per question, and objective accuracy by similarity quartile. A flat right-hand panel would indicate that retrieval confidence carries no signal about answer correctness.

**fig11_training_loss** — QLoRA training loss. Colab renders the live curve as a widget that does not survive saving the notebook, so it is reconstructed here from the persisted log history.

**fig12_results_heatmap** — All headline metrics in one panel. Cell values are the raw metric; colour is min-max rescaled within each row so that green always indicates the better result, including for metrics where lower is better.

**fig13_accuracy_by_bank** — Objective accuracy split by bank. Bars that move together across the three reports support the pooled comparison; a condition that wins on one bank and loses on another is reporting coverage or retrieval behaviour rather than a property of the strategy.

**fig14_retrieval_bank_precision** — Retrieval bank-precision and the cross-bank confusion matrix. The three reports use near-identical statement wording, so a passage retrieved with a high similarity score can still belong to the wrong institution; mass off the diagonal is context that was well-matched and wrong.

**fig15_adversarial_by_tier** — Abstention rate on the three adversarial tiers. Fabricated premises are unanswerable for any corpus; out-of-scope-year questions name a real bank and a real metric and are the tier where a fine-tuned model is most likely to answer from parametric memory; bank-absent questions concern institutions outside the index.

**fig16_extraction_gap** — Four-way decomposition of short-answer outcomes for the retrieval conditions. The dotted rule on each bar is the answer-in-context ceiling — the share of questions whose gold figure reached the model at all. Orange is the extraction gap: evidence retrieved successfully and still not converted into the right answer, which responds to prompting and table formatting rather than to a wider retrieval window. Blue is answered correctly without the figure in context, i.e. parametric memory rather than retrieval.

