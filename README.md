The Aya-Expanse language model represents a significant advancement in the realm of low-resource languages. By leveraging synthetic data, Aya-Expanse has effectively learned to understand and generate underrepresented languages. its training pipeline is noteworthy for its inclusion of fine-tuning on synthetic data, iterative preference training and model merging.this innovative approach allows Aya-Expanse to harness diverse data sources, but it also raises problems on it's certainty to answer questions.

A simple experiment has been conducted to demonstrate the uncertainty present in both the Aya-Expanse and Gaokerena models. in this experiment, Aya-Expanse, Gaokerena-V, Gaokerena-R, and Med-Gemma are utilized to tackle the Iranian Basic Medical Science Entrance Exam held in September 2023, each model was run five times on the same set of questions given a Chain of Thought propmt, The results revealed that Med-Gemma consistently selected the same option for all questions, while the aya-expanse based models showed utter uncertainty

|                       | Gaokerena-V  | aya-expanse | Medgemma |
|-----------------------|--------------------|----------------------------|------------------------|
| **Accuracy**  | **48.14**          | 14.07                      |   25.18                |
| **Number of Single Option Taken** | **53.0**    | 20.0                       | 35.0                   |
| **Number of All Option Taken** | **43.93**   | 19.08                      | 27.17                  |
| **Entropy**     | **55.47**                        | 27.54                 | 31.70               |
| **Prompt**  | **47.05**                    | 17.27                  | 33.82               |
| **Number of Parameters**      | **47.22**                        | 18.75                  | 31.25                |
