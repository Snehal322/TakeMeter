#Project Overview

This project builds a discourse classification model for posts from r/LetsTalkMusic. 

The goal is to classify music discussion posts into one of four categories:
    Analysis – Structured arguments supported by evidence, examples, comparisons, or detailed reasoning.
    Opinion – Personal judgments or controversial viewpoints without substantial supporting evidence.
    Reaction – Emotional responses, personal experiences, excitement, nostalgia, or frustration.
    seeking_help – Questions, requests for information, recommendations, or explanations.

The project compares a zero-shot Large Language Model baseline against a fine-tuned DistilBERT classifier trained on a manually annotated dataset.

---------------------------------
**Dataset**

    Dataset Size:

        Split	Examples
        Train	139
        Validation	30
        Test	30
        Total	199


    Train Label Distribution:

        Label	Count
        Analysis	57
        seeking_help	43
        Reaction	20
        Opinion	19


    Test Label Distribution
    
        Label	Count
        Analysis	13
        seeking_help	9
        Opinion	4
        Reaction	4

-------------------------------------
**Baseline Model**

    Model
    - Groq API
    - Llama 3.3 70B Versatile
    - Zero-shot prompting

    Baseline Accuracy = 33.3% 

    Baseline Per-Class Metrics 

        Label	Precision	Recall	F1
        Analysis	0.00	0.00	0.00
        Opinion	    0.17	0.50	0.25
        Reaction	0.17	0.25	0.20
        seeking_help	0.58	0.78	0.67

    Baseline Reflection

    The zero-shot baseline struggled to distinguish between Analysis, Opinion, and Reaction. 
    Most Analysis posts were incorrectly classified, resulting in an F1-score of 0.00 for that category. 
    The model performed best on seeking_help because questions and requests for information contain recognizable 
    linguistic signals such as question marks and direct requests.

    I hypothesized that the baseline would struggle because the distinctions between Analysis, Opinion, and 
    Reaction depend on subtle discourse intent rather than obvious keywords. Fine-tuning was expected to 
    help the model learn these community-specific distinctions.

--------------------------------------
**Fine-Tuning** 

Model
    - distilbert-base-uncased
    - 4 output labels
    - HuggingFace Transformers
    - Google Colab T4 GPU

Hyperparameter Experiments

    Experiment 1 (Default)
        Epochs: 3
        Learning Rate: 2e-5
        Batch Size: 16

        Validation Accuracy: 46.7%

    Experiment 2
        Epochs: 5
        Learning Rate: 2e-5
        Batch Size: 16

        Validation Accuracy: 46.7%

    Observations:

        Validation accuracy steadily improved.
        Validation loss consistently decreased.
        No obvious signs of overfitting.
        Model continued learning through all epochs.

    Experiment 3
        Epochs: 5
        Learning Rate: 3e-5
        Batch Size: 16

        Validation Accuracy: 43.3%

    Observations:

        Training loss decreased faster.
        Validation loss stopped improving after epoch 2.
        Accuracy decreased during later epochs.
        Mild overfitting was observed.

Final Model Selection

    I selected the model trained with:
        Epochs: 5
        Learning Rate: 2e-5
        Batch Size: 16

This configuration produced the most stable training behavior and showed continued improvement without 
clear signs of overfitting.

------------------------

**Evaluation Results**

#Overall Accuracy

    Model	                        Accuracy
    Groq Zero-Shot Baseline	         33.3%
    Fine-Tuned DistilBERT	         56.7%


#Improvement

    Fine-tuning improved accuracy by 23.3 percentage points.

-------------------------------------------------------------------

**Fine-Tuned Per-Class Metrics**

Label	Precision	Recall	F1
Analysis	0.50	0.77	0.61
Opinion	    0.00	0.00	0.00
Reaction	0.00	0.00	0.00
seeking_help	0.70	0.78	0.74

The strongest-performing categories were Analysis and seeking_help. 
The model learned to identify evidence-based discussions and information-seeking questions relatively well.

However, the model completely failed to distinguish Opinion and Reaction posts from Analysis posts.

---------------------------------

**Fine-Tuned Confusion Matrix**


| True Label   | Predicted Analysis | Predicted Opinion | Predicted Reaction | Predicted seeking_help |
| ------------ | ------------------ | ----------------- | ------------------ | ---------------------- |
| Analysis     | 10                 | 0                 | 0                  | 3                      |
| Opinion      | 4                  | 0                 | 0                  | 0                      |
| Reaction     | 4                  | 0                 | 0                  | 0                      |
| seeking_help | 2                  | 0                 | 0                  | 7                      |

Key observations:

    10 of 13 Analysis posts were classified correctly.
    7 of 9 seeking_help posts were classified correctly.
    All Opinion posts were misclassified as Analysis.
    All Reaction posts were misclassified as Analysis.


#Key Pattern

The dominant error pattern was:

    Opinion → Analysis and Reaction → Analysis

Every Opinion and Reaction example in the test set was classified as Analysis.

This indicates that the model learned a simplified decision boundary where any post discussing music 
in some depth was treated as Analysis.

---------------------------------------------------

**AI-Assisted Failure Analysis**

    To identify broader trends, I provided the list of misclassified examples to ChatGPT and asked it to 
    identify recurring patterns.

    The AI suggested:

    1. The model heavily over-predicts Analysis.
    2. Very short posts are difficult to classify correctly.
    3. Discussion-starter questions are often confused between Analysis and seeking_help.

    After manually reviewing the examples, I confirmed all three observations.

    One AI-generated hypothesis that I rejected was that post length alone explained most errors. 
    While many short posts were misclassified, several longer posts were also incorrect. 
    The real issue appears to be ambiguity between discourse categories rather than length itself.

-----------------------------

**Error Analysis**

Error 1: Opinion → Analysis

Post:

    "Insane Clown Posse is actually kind of underrated"

        True Label: Opinion

        Predicted Label: Analysis

        Confidence: 0.43

This post primarily expresses a personal judgment. However, many Opinion posts in the dataset contain supporting explanations.
The model appears to associate any evaluative music discussion with Analysis, causing this boundary to collapse.

To improve performance, the dataset would need more examples that clearly separate unsupported opinions from evidence-based 
arguments.


Error 2: Reaction → Analysis

Post:

    "I finally listened to Mutiny After Midnight by Johnny Blue Skies and have really mixed feelings about it."

        True Label: Reaction

        Predicted Label: Analysis

        Confidence: 0.44

This post focuses on the author's emotional response to an album. According to the annotation guidelines, 
emotional reactions belong in the Reaction category.

The model appears to focus more on the fact that the post discusses music than on the emotional language 
indicating a personal reaction.

Additional Reaction examples emphasizing feelings and personal experiences would likely help.


Error 3: seeking_help → Analysis

Post:

    "What first caught people's attention about Calgary post-punk"

        True Label: seeking_help

        Predicted Label: Analysis

        Confidence: 0.51

This post is clearly asking for information from the community. However, because it is short and lacks explicit question 
formatting, the model interpreted it as the beginning of a discussion rather than a request for information.

More short question-based examples would help the model recognize these cases.

--------------------

#Sample Classifications

    Example Post	                                                Predicted Label	    Confidence
    What first caught people's attention about Calgary post-punk	    Analysis	        0.51
    Insane Clown Posse is actually kind of underrated	                Analysis	        0.43
    I finally listened to Mutiny After Midnight and have mixed feelings about it	Analysis	0.44
    Alright everybody, please break down Deftones for me	            Analysis	        0.45
    Is there something about Paramore that I'm missing?	                seeking_help	    0.48

The final example demonstrates a reasonable prediction because the post directly asks the community 
for clarification and information.

------------------------------------------
#Reflection: What the Model Learned vs What I Intended

My label definitions focused on the author's intent. Analysis was intended to capture reasoning and evidence, 
Opinion was intended to capture unsupported judgments, Reaction was intended to capture emotional responses, 
and seeking_help was intended to capture requests for information.

The model only partially learned these distinctions. It successfully learned the difference between Analysis 
and seeking_help but largely collapsed Opinion and Reaction into Analysis.

This suggests that the model relied on topical discussion cues rather than fully learning the discourse function 
of the post. Additional labeled examples and a more balanced dataset would likely improve performance.

----------------------------------------------------

**Spec Reflection**

#How the Specification Helped

    The specification provided clear guidance for label design, annotation consistency, edge-case handling, and 
    evaluation methodology. Defining edge cases before annotation helped maintain consistency across the dataset.

#Where My Implementation Diverged

    The specification encouraged balanced label distributions. Despite attempts to balance the dataset, Analysis 
    examples remained more common than Opinion and Reaction examples. This likely contributed to the model's 
    tendency to over-predict Analysis.

If I repeated the project, I would collect more Opinion and Reaction examples to create a more balanced training set.

-------------------------------

**AI Usage**

#AI Use #1: Label Development

I used ChatGPT to generate example posts that sat near the boundary between Analysis, Opinion, and Reaction. 
These examples helped refine the label definitions and edge-case rules before annotation.

#AI Use #2: Annotation Review

After completing manual annotation, I used ChatGPT to review the labeled dataset and identify potentially 
inconsistent labels. I manually reviewed all suggestions and only applied corrections that clearly matched 
the annotation rules.

#AI Use #3: Error Analysis

After evaluating the fine-tuned model, I used ChatGPT to analyze the list of misclassified examples and 
identify recurring failure patterns. I then manually verified those patterns against the confusion matrix 
and prediction outputs before including them in the final report.

All final labels and evaluation conclusions were manually reviewed and approved.

---------

#evaluation_result.json

The fine-tuned DistilBERT model achieved an accuracy of 56.67%, compared to the zero-shot Groq baseline accuracy of 33.33%. 
This represents an improvement of 23.33 percentage points, indicating that fine-tuning on the labeled r/LetsTalkMusic dataset
 helped the model learn the distinctions between the four discussion categories.

The model performed best on Analysis and seeking_help posts, while Opinion and Reaction remained more difficult to distinguish.
Error analysis showed that many Opinion and Reaction posts were predicted as Analysis because music discussions often contain 
personal opinions mixed with explanations or supporting details. Similarly, some question-based discussion starters were confused
 between Analysis and seeking_help.

Although the overall accuracy improved substantially over the baseline, the confusion matrix suggests that additional labeled 
examples—especially for Opinion and Reaction posts—would likely improve performance further.