# Octo: A Lightweight, Browser-Based Toolkit for Estimating and Reducing AI-Style Patterns in Written Text

**Aryan Sengar**  
**M.Tech Student, [Department], [Institution]**  
**GitHub: github.com/isdapperdude404  |  November 2026**  

## Abstract

Text written with large language models now turns up in coursework, job applications and online posts, and many people want a quick way to see how much a piece of writing leans on the habits of such models. Most detectors are web services that need the text to be uploaded, and many do not explain their scores. This paper describes Octo, a free tool that runs entirely in the web browser. Octo gives a passage a score from 0 to 100 based on five surface signals: stock phrases, even sentence lengths, missing contractions, em dashes and formal sentence openers. It also includes a rule-based rewriter that replaces those patterns, and a small set of text and file tools. I set out the design, give every scoring rule, and walk through one worked example in which a deliberately stiff sample passage falls from 67 to 44 after rewriting. I also discuss where the method fails, including the risk of wrongly flagging writers who are not native English speakers. Octo gives an estimate, not proof of authorship, and should not be used as proof. The source code is released under the MIT license.

**Keywords:** AI-generated text, text detection, heuristics, readability, web application, privacy

## 1. Introduction

Since late 2022, chat assistants built on large language models have become part of everyday writing for many students and professionals. Teachers, editors and employers have started to ask a plain question about the text they receive: does this read like a model wrote it? Commercial detectors try to answer that question, but they come with problems. Many need the text to be sent to a remote server, many do not say how a score was produced, and a single number with no explanation is hard to act on.

I wanted something different. The aim was a tool whose whole method fits on a page, that runs on the reader’s own device, and that shows which parts of a text triggered the score. I also wanted it to be useful on a weak connection and on an ordinary phone, since many of the students I have in mind do not have fast internet or powerful laptops.

The paper makes four contributions:

- A transparent, rule-based detector whose every weight is stated in this paper.
- A rewriter that replaces stock AI wording with plainer language and reports the score before and after.
- A single-file application that bundles these with everyday tools (file reading, counting, summarizing, trend projection) and needs no installation.
- A frank discussion of what the approach cannot do, including fairness and misuse.

## 2. Background and related work

Work on detecting machine-generated text can be grouped into three broad families. The first looks at how predictable a text is to a language model. GLTR [1] colours each word by its rank under a model, so a person can see when a passage is suspiciously smooth. DetectGPT [2] compares the probability a model gives a passage with the probability it gives small rewrites of that passage, and treats text that sits at a sharp peak as likely to be machine-made. The second family is watermarking. Kirchenbauer et al. [3] show how a generator can quietly favour a random subset of words so that a detector who knows the secret can count them later.

Both families need something Octo does not have: access to a model, or the cooperation of the system that wrote the text. Octo therefore works only with features that anyone can see on the page. That choice makes it weaker, and I come back to this in Section 6.

The third strand of work asks whether detection can be trusted at all. Sadasivan et al. [4] argue that as models improve, the difference between human and machine text shrinks, and that paraphrasing can defeat many detectors. Liang et al. [5] found that several popular detectors often marked writing by non-native English speakers as machine-generated. This matters for a tool meant for students in India, and it shaped the cautious wording in Octo’s interface. For readability, Octo uses the Flesch Reading Ease formula [6].

## 3. System design

Octo is one HTML file. There is no server, no account and no database. All analysis runs in the browser, and the only data kept between visits are the user’s tasks and notes, stored in the browser’s own local storage. Libraries for reading PDF, Word and Excel files (PDF.js 3.11.174, Mammoth 1.6.0, SheetJS 0.18.5) and for writing Word files (JSZip 3.10.1) are loaded from a public content delivery network, and only when the user opens a file. Table 1 lists the tools and the method behind each.

*Table 1. Tools in Octo version 1.0.0 and the method behind each.*

| Tool | Method |
|---|---|
| AI detector | Weighted sum of five surface signals (Section 4.1) |
| AI remover | Ordered rewrite rules (Section 4.2) |
| Word counter | Counts, reading time, Flesch Reading Ease |
| Text cleaner | Regular-expression clean-up and case change |
| Summarizer | Word-frequency extractive summary |
| Predictor | Ordinary least squares line with R² |
| File converter | PDF.js, Mammoth and SheetJS to read; JSZip to write Word |
| Voice | Browser dictation and speech synthesis |
| Face detection | Browser FaceDetector API; counts faces only |
| Tasks, timer, passwords | Local storage, a countdown timer, Web Crypto random values |

Face detection deserves a note. Octo counts faces and draws boxes around them. It does not try to tell who anyone is. I left identification out on purpose: it raises serious privacy questions, and it is not needed for the purpose of the tool.

## 4. Methods

### 4.1 The detector

Let W be the number of words in a passage and S the number of sentences. The detector refuses passages under 40 words, because patterns in very short text are mostly noise. For longer text it adds up five scores, each capped, to reach a total of at most 100. The signals and weights, summarised in Table 2, were chosen by hand from my own reading of typical model output. They were not fitted to a data set.

*Table 2. Signals used by the detector. h is the number of stock-phrase matches, d the number of em dashes and t the number of sentences that open with a formal connective.*

| Signal | Rule | Maximum |
|---|---|---|
| Stock phrases | min(45, 900 × h / W), from a list of 43 phrases such as “delve”, “tapestry” and “it is important to note” | 45 |
| Even sentence length | 25 × clamp((0.55 − CV) / 0.55, 0, 1), where CV is the standard deviation of sentence length divided by its mean; used only when S ≥ 4 | 25 |
| No contractions | 10 if W ≥ 60 and no contraction such as “don’t” or “it’s” appears | 10 |
| Em dashes | min(10, 4 × d) | 10 |
| Formal openers | min(10, 4 × t), counting openers such as “Moreover”, “Furthermore” and “In conclusion” | 10 |

The total is rounded to a whole number. A score below 30 is labelled likely human, 30 to 59 mixed signals, and 60 or more likely AI-written. The interface also lists which signals fired and highlights each stock phrase in the text, so the reader can judge the result instead of trusting a number.

### 4.2 The rewriter

The rewriter applies seven steps in a fixed order. It first removes closing phrases such as “In conclusion,”. It then replaces sentence-opening connectives (“However,” becomes “But”, “Moreover,” becomes “Plus,”). Next it swaps stock words for plainer ones using a table of 42 entries, trying the longest match first, and joins 20 common word pairs into contractions. Em dashes become commas. Any sentence longer than 26 words is split at its first “, and”, “, but” or “, so” that falls after the 40th character. A last pass repairs spacing and capital letters. The interface then shows the detector score before and after.

### 4.3 Supporting tools

The word counter estimates syllables by counting groups of vowels, then computes Reading Ease as 206.835 − 1.015(W/S) − 84.6(Y/W), where Y is the syllable count [6]. The summarizer ignores a short list of common words, counts the rest, scores each sentence by the sum of its word counts divided by the square root of its length, and returns the top-scoring sentences in their original order. The predictor fits a straight line to the numbers the user enters by ordinary least squares, extends it a chosen number of steps, and reports R² as a “fit strength” so the user can see how well a line describes the data.

## 5. A worked example

Table 3 shows the detector on three passages. The first is the sample text built into Octo. I wrote it to be crowded with stock phrases, so a high score on it says very little about real-world accuracy. The second is the same passage after the rewriter. The third is an informal first-person paragraph written for this test.

*Table 3. Detector output on three test passages.*

| Passage | Words | Score | Signals reported |
|---|---|---|---|
| Built-in sample | 75 | 67 | 20 stock-phrase matches; even sentence lengths; 4 formal openers |
| Sample after rewriting | 62 | 44 | 1 stock phrase; even sentence lengths; no contractions |
| Informal paragraph | 89 | 10 | 1 stock phrase |

The rewriter lowered the score by 23 points, but the result is not good writing. It produced phrases such as “open up smooth productivity” and “a proof of what we can achieve”, which no careful writer would choose. Simple substitution cannot tell which meaning of a word is in play. The rewritten text also kept an even sentence rhythm and still had no contractions, two of the signals the detector looks for. The tool therefore asks users to read the output and add their own details, and I think that is the right instruction.

This example is an illustration, not an evaluation. I have not measured accuracy, precision or recall on labelled data. A proper study should collect matched human and machine texts across several subjects and writing levels, include writing by non-native English speakers, report the false-positive rate for each group, fit the weights on one part of the data and test on another, and compare Octo with at least one published detector.

## 6. Limitations and responsible use

The detector looks at surface habits only. A person can write with many of the same habits, especially in formal academic prose, where phrases such as “it is important to note” are common and contractions are avoided. Such writing may score high even though no model touched it. Given the findings in [5], I expect non-native English writers to be at particular risk of unfair scores, and I would not trust Octo for that group without testing. In the other direction, a model asked to write casually, or a text edited by hand, can score low. For these reasons a score should start a conversation and never end one.

The rewriter is a dual-use tool. It was built to help people replace stiff wording with plain language, but it also lowers a detector score, and someone could use it to hide AI assistance where disclosure is required. The interface says plainly that the rewriter cannot guarantee how any other detector will score a text. Users should follow the rules of their school, journal or employer and should say when they have used AI help.

Smaller limits are practical ones. The rewriter works on English only, scanned PDFs have no selectable text and so come out empty, and some features (browser dictation, face detection) depend on what the browser supports. An Android wrapper that places the same page in a WebView has been prepared, but I have not yet tested it on physical devices.

## 7. Conclusion and future work

Octo shows that a useful, explainable check of AI-style writing can be built as one small file that never sends the user’s text anywhere. It is honest about its method, and it is limited by that method. The most important next step is a labelled evaluation with a fairness analysis. After that, I plan to tune the weights on real data, support more languages, read text from scanned documents, and add a better rewriter that considers context instead of swapping words one by one.

The source code, README and wiki are available at https://github.com/isdapperdude404/octo under the MIT license.

## References

[1] S. Gehrmann, H. Strobelt and A. M. Rush, “GLTR: Statistical detection and visualization of generated text,” in Proc. 57th Annu. Meeting Assoc. Comput. Linguistics: System Demonstrations, 2019, pp. 111–116.

[2] E. Mitchell, Y. Lee, A. Khazatsky, C. D. Manning and C. Finn, “DetectGPT: Zero-shot machine-generated text detection using probability curvature,” in Proc. 40th Int. Conf. Machine Learning (ICML), 2023.

[3] J. Kirchenbauer, J. Geiping, Y. Wen, J. Katz, I. Miers and T. Goldstein, “A watermark for large language models,” in Proc. 40th Int. Conf. Machine Learning (ICML), 2023.

[4] V. S. Sadasivan, A. Kumar, S. Balasubramanian, W. Wang and S. Feizi, “Can AI-generated text be reliably detected?,” arXiv:2303.11156, 2023.

[5] W. Liang, M. Yuksekgonul, Y. Mao, E. Wu and J. Zou, “GPT detectors are biased against non-native English writers,” Patterns, vol. 4, no. 7, p. 100779, 2023.

[6] R. Flesch, “A new readability yardstick,” J. Applied Psychology, vol. 32, no. 3, pp. 221–233, 1948.

[7] A. Sengar, “Octo,” GitHub repository, 2026. [Online]. Available: https://github.com/isdapperdude404/octo
