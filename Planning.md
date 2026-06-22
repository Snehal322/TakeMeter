
# Community Description :

Community: LetsTalkMusic

    LetsTalkMusic is a discussion-focused music community where users debate artists, genres, albums, production techniques, and the cultural impact of music. The quality of discourse varies widely, from emotional reactions and personal opinions to detailed analytical arguments supported by musical examples and historical context. Distinguishing between these discussion styles matters because community members value thoughtful analysis and substantive discussion over unsupported claims.

    I chose the Reddit community LetsTalkMusic because it contains a wide range of music discussions, from detailed analytical essays to casual opinions and emotional reactions. My classifier will categorize posts into Analysis, Opinion, and Reaction. These distinctions matter because community members value evidence-based discussion, and separating thoughtful analysis from unsupported opinions or emotional responses helps measure discourse quality within the community.

----------------------------------------------------------------------------------

# Labels:

*Label 1: Analysis*

Definition: A post that presents a structured argument supported by evidence, examples, comparisons, historical context, or detailed reasoning.

Example Post 1:
    "There's nothing like D'Angelo's Voodoo" – The author discusses production quality, instrumentation, vocal layering, individual tracks, and comparisons with other albums to explain why the album is exceptional.

Example Post 2:
    "Do all popular artists chase fame?" – The author examines artist motivations, fame, commercial success, and indie music culture by exploring multiple possibilities and asking broader questions.


*Label 2: Opinion*

Definition: A post that expresses a strong personal judgment or controversial opinion without substantial supporting evidence.

Example Post 1:
    "Underground concerts are MUCH better than mainstream ones." The author argues that underground concerts provide a better experience based primarily on personal preference.

Example Post 2:
    "Insane Clown Posse is actually kind of underrated." The central claim is a controversial opinion about the band's reputation and quality.


*Label 3: Reaction*

Definition: A post that primarily expresses personal feelings, excitement, nostalgia, frustration, or emotional responses rather than making an argument.

Example Post 1:
    "I see no difference between grunge and early post-hardcore, so I love both." The post mainly describes the author's musical journey, excitement about discovering bands, and personal enjoyment of the genre.

Example Post 2:
    A post describing being emotionally overwhelmed by hearing an album for the first time and explaining how it made the listener feel.


*Label 4: Help-Seeking*

Definition: A post whose primary purpose is to ask a question, request information, seek recommendations, or ask for explanations.

Example Post 1:
    "I wonder how popular and well-known Jermaine Jackson was during the 1980s and 1990s." The author is seeking information about a historical music figure.

Example Post 2:
    "Do all popular artists chase fame?" When viewed as a question-driven discussion starter, the post asks the community for explanations and perspectives.

---------------------------------------

# Hard Edge Cases

Some posts will naturally fit more than one label. The most common ambiguity will occur between Analysis and Opinion, and between Analysis and Reaction.

*Edge Case 1: Analysis vs Opinion*

Example:
    "Insane Clown Posse is actually kind of underrated."

This post could be labeled as Opinion because it expresses a strong judgment. However, if the author provides specific song examples, discusses lyrical themes, and explains the reasoning behind the claim, it will be labeled as Analysis.

Decision Rule: If the post contains evidence and reasoning that support the claim, label it Analysis. If it primarily states a belief without substantial evidence, label it Opinion.


*Edge Case 2: Analysis vs Reaction*

Example:
    "I see no difference between grunge and early post-hardcore, so I love both."

The post contains discussion about musical characteristics, but the primary focus is the author's personal experience and excitement.

Decision Rule: If the primary goal is sharing feelings or experiences, label it Reaction. If the primary goal is explaining musical concepts or defending a claim with evidence, label it Analysis.

-----------------------------

# Data Collection Plan

I will collect posts from the r/LetsTalkMusic subreddit. I will gather at least 200 examples from discussion threads and posts.

*Target distribution:*

    Analysis: 50 examples
    Opinion / Hot Take: 50 examples
    Reaction: 50 examples
    Help-Seeking / Recommendation: 50 examples

The dataset will be *split* into:

    Training Set: 140 examples (70%)
    Validation Set: 30 examples (15%)
    Test Set: 30 examples (15%)

If a label is underrepresented after collecting 200 examples, I will intentionally search for posts matching that category and add additional examples until the distribution is reasonably balanced. Maintaining balanced classes is important because highly imbalanced data could cause the model to overpredict the most common label.

---------------------

# Evaluation Metrics

    I will use multiple evaluation metrics because accuracy alone can be misleading.

1. Accuracy

    Accuracy measures the percentage of examples classified correctly overall. It provides a simple summary of model performance but does not show whether some labels perform significantly worse than others.

2. Precision

Precision measures how often a predicted label is actually correct. This is important because a classifier that frequently labels Opinion posts as Analysis would produce misleading results.

3. Recall

Recall measures how many examples of a label the model successfully finds. This is important because I want the classifier to identify most Analysis posts rather than missing them.

4. F1 Score

F1 Score balances precision and recall. Since all four labels are important, F1 Score provides a better measure of overall performance than accuracy alone.

5. Confusion Matrix

A confusion matrix will help identify which labels the model confuses most often. I expect confusion primarily between Analysis and Opinion, and between Analysis and Reaction.

6. Baseline Comparison

I will compare the fine-tuned DistilBERT model against a zero-shot classification baseline using Groq's llama-3.3-70b-versatile model. This comparison will show whether fine-tuning provides a meaningful improvement.

----------------------------------------------------

# Definition of Success

    A useful classifier should perform significantly better than the zero-shot baseline while maintaining reasonable performance across all labels.

*Success criteria:*

    Overall accuracy above 75%
    Macro F1 score above 0.70
    No label with F1 score below 0.60
    Fine-tuned model outperforms the zero-shot baseline by at least 10 percentage points in accuracy


For deployment in a real community tool, I would consider the classifier "good enough" if it consistently distinguishes Analysis, Opinion, Reaction, and Help-Seeking posts with an overall accuracy of 80% or higher and shows balanced performance across all classes. Even if some borderline posts remain difficult, the classifier would still provide useful insights into discourse patterns within the community.

-----------------

# AI Tool Plan


1. Label Stress-Testing

    Before annotating the full dataset, I will use an AI tool (Claude) to test whether my label definitions are clear and mutually exclusive.

_Process_:
    a. Provide the AI with:
        Community description (r/LetsTalkMusic)
        Label definitions

    b. Edge case rules
    Ask the AI to generate 5–10 posts that intentionally sit at the boundary between two labels.

    c. Attempt to classify each generated post myself.

_Goal_:

    If I cannot confidently assign a single label to a generated example, I will revise the label definitions before beginning large-scale annotation.

*Example Stress-Test Prompt*

    Generate 10 LetsTalkMusic-style discussion posts that are difficult to classify between Analysis and Appreciation, or between Hot Take and Analysis. Make them realistic and similar to actual Reddit posts.

_Expected Outcome_:

    This process helps identify overlapping labels before annotating 200+ examples and improves consistency during labeling.


2. Annotation Assistance:

    I will use Calude as an annotation assistant for an initial review pass but will manually verify every label before adding examples to the final dataset.

_Process_:
    
    a. Collect Reddit posts from r/LetsTalkMusic
    b. Ask the AI for a suggested label
    c. Review the suggestion myself
    d. Either accept or correct the label
    e. Record the final human-reviewed label in the dataset

_Tracking AI Assistance_:

    Examples that receive AI-generated preliminary labels will be marked in my annotation notes so I can disclose AI assistance in the README.

_Why Use AI_
    - Speeds up the initial annotation process
    - Helps identify borderline cases
    - Does not replace human judgment

_Human Responsibility_

    All final labels used for training will be reviewed and approved manually.


3. Failure Analysis:

    After evaluating both the fine-tuned model and the zero-shot baseline, I will use Claude to help identify patterns among incorrect predictions.

_Process_:
    
    a. Export misclassified test examples.
    b. Provide the examples, true labels, and predicted labels to the AI.
    c. Ask the AI to identify possible trends.

_Example Prompt_:

    Here are 20 misclassified LetsTalkMusic posts. Identify common patterns among the mistakes and suggest possible reasons why the classifier struggled.

_What I Will Look For_:

    - Confusion between Analysis and Appreciation.
    - Hot Take posts that contain minimal evidence.
    - Short posts lacking sufficient context.
    - Posts containing sarcasm or humor.
    - Posts mixing multiple discourse styles.

_Verification_:

    I will manually review every pattern suggested by the AI before including it in the evaluation report.

----------------------------------------------

# Definition of Success :

_To make success criteria objective:_

1. _Minimum Success Threshold_:
    - Fine-tuned model accuracy ≥ 70%
    - Macro F1-score ≥ 0.65
    - Every label achieves at least 0.60 F1-score

2. _Strong Success Threshold_:
    - Fine-tuned model accuracy ≥ 80%
    - Macro F1-score ≥ 0.75
    - Outperforms the zero-shot baseline by at least 10 percentage points in overall accuracy

3. _Deployment Threshold_:

    I would consider the classifier useful for a real community moderation or discussion-analysis tool if:
        - Accuracy is at least 80%
        - No class has F1 below 0.70
        - Error analysis shows no major systematic failure mode (for example, consistently confusing Analysis with Appreciation)

These thresholds allow me to objectively determine whether the project succeeded rather than simply saying the model "works well."