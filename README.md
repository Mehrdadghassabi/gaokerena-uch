# The Problem
The Aya-Expanse language model represents a significant advancement in the realm of low-resource languages. By leveraging synthetic data, Aya-Expanse has effectively learned to understand and generate underrepresented languages. its training pipeline is noteworthy for its inclusion of fine-tuning on synthetic data, iterative preference training and model merging.this innovative approach allows Aya-Expanse to harness diverse data sources, but it also raises problems on it's certainty to answer questions.

A simple experiment has been conducted to demonstrate the uncertainty present in both the Aya-Expanse and Gaokerena models. in this experiment, Aya-Expanse, Gaokerena-V, Gaokerena-R, and Med-Gemma are utilized to tackle the Iranian Basic Medical Science Entrance Exam held in September 2023, each model was run five times on the same set of questions given a Chain of Thought propmt, The results revealed that Med-Gemma consistently selected the same option for all questions, while the aya-expanse based models showed utter uncertainty.
Notice that

|                       | Gaokerena-V  | aya-expanse | Medgemma |
|-----------------------|--------------------|----------------------------|------------------------|
| **Accuracy**  | 38.69        | 34.52                      |   37.5                |
| **Number of Single Option Taken** | 10    | 44                      | 168                   |
| **Number of All Option Taken** |  10  |         7            |        0          |
| **Entropy**     | 1.62                        | 1.21                 | 0               |
| **Prompt**  | COT                    | COT                  | COT               |
| **Number of Parameters**      | 8b                       | 8b                | 4b                |

pov
a. Gaokerena-V's baseline model was aya-expanse and it has inhereted it's uncertainty
b. used self-consistency for getting final answer in Gaokerena-V and aya-expanse


# The solution 
we tried to build a separate head to predict the amount of uncertainty in Aya, but due to the expanses of the hardware renting in Iran we abandoned the project
