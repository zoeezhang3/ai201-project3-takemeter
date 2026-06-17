# Planning: Fine-Tuned Text Classifier for r/NYKnicks Discourse Quality

## 1. Community

I chose **r/NYKnicks**, a Reddit community focused on discussion of the New York Knicks. I chose this community because it has a strong mix of basketball analysis, emotional fan reactions, jokes, memes, complaints, and sometimes hostile arguments between fans. This makes it a good fit for a discourse-quality classification task because the comments are not all the same type: some users write thoughtful analysis about players, coaching decisions, trades, rotations, and playoff matchups, while others post short hype reactions, jokes, or frustration after games.

This community is especially interesting because sports discourse can change quickly depending on team performance. After a win, the subreddit may contain more hype and celebration. After a loss, it may contain more blame, sarcasm, criticism of players or coaches, and arguments between users. Because of this variation, a classifier has to do more than detect whether a comment is positive or negative. It needs to distinguish between useful basketball discussion, harmless fan reaction, and comments that reduce the quality of the conversation.

## 2. Labels

I will use three labels for this classification task.

### Label 1: `constructive_analysis`

A comment should be labeled `constructive_analysis` when it adds meaningful basketball-related discussion, such as reasoning about player performance, coaching decisions, roster construction, strategy, statistics, matchups, or team trends.

Example 1:

> Brunson is still creating good looks, but the spacing gets much worse when two non-shooters are on the floor. Teams can help off the corners and make every drive harder.

Example 2:

> I do not think the problem is just Randle taking shots. The bigger issue is that the offense becomes predictable when the ball stops moving late in the fourth quarter.

### Label 2: `casual_reaction`

A comment should be labeled `casual_reaction` when it is mostly a fan reaction, joke, meme, hype comment, complaint, emotional response, or short unsupported opinion that does not add much detailed analysis but is not directly hostile or abusive.

Example 1:

> BING BONG, Knicks are back.

Example 2:

> I cannot watch another fourth quarter like that lol.

### Label 3: `toxic_bad_faith`

A comment should be labeled `toxic_bad_faith` when it lowers the quality of the discussion through personal attacks, direct insults, trolling, baiting, harassment, or hostile comments toward another user, player, coach, fanbase, or group.

Example 1:

> You are an idiot if you think that trade would help this team.

Example 2:

> This fanbase is full of clowns who do not know basketball.

## 3. Hard Edge Cases

Some comments will be genuinely ambiguous between two labels. The hardest cases will probably be comments that mix basketball criticism with emotional language or insults.

One difficult edge case is a comment that criticizes a player or coach very strongly but still gives real basketball reasoning. For example, a comment might say that a coach made a terrible rotation decision and then explain how the lineup hurt spacing and defense. I will label this as `constructive_analysis` if the main purpose of the comment is to explain a basketball point rather than attack someone personally.

Another difficult edge case is sarcasm. Knicks fans often use sarcasm after losses, and some sarcastic comments may sound negative without being truly toxic. If the comment is sarcastic or emotional but does not directly insult another person or group, I will label it as `casual_reaction`.

A third difficult edge case is a comment that includes some basketball content but also directly insults another user. For example, if a comment says, “Only an idiot would think this lineup works because the spacing is terrible,” it includes basketball reasoning, but the personal insult damages the discourse. I will label comments like this as `toxic_bad_faith` because the hostile framing is the most important feature for discourse quality.

When I encounter ambiguous examples during annotation, I will follow these rules:

1. If the comment contains meaningful basketball reasoning and no direct insult, label it `constructive_analysis`.
2. If the comment is emotional, funny, sarcastic, or low-effort but not abusive, label it `casual_reaction`.
3. If the comment directly attacks, insults, baits, or harasses someone, label it `toxic_bad_faith`, even if it also contains some basketball reasoning.
4. If I am still unsure, I will add a note explaining why the example was difficult and make a consistent final decision based on the comment's overall effect on discussion quality.

## 4. Data Collection Plan

I will collect examples from public posts and comments in **r/NYKnicks**. I plan to focus mainly on comments because comments provide more examples of direct discourse between community members. I may include original posts if they contain enough text to classify, but the main unit of analysis will be one Reddit comment.

I will collect comments from several types of threads to capture a wide range of discourse:

- Game threads
- Post-game threads
- Trade or free-agency discussion threads
- Injury news threads
- Player performance discussion threads
- General discussion or daily discussion threads

I will collect at least **200 labeled examples** total. My target distribution is:

| Label | Target Count |
|---|---:|
| `constructive_analysis` | 70 |
| `casual_reaction` | 70 |
| `toxic_bad_faith` | 60 |
| **Total** | **200** |

This distribution is not perfectly equal because toxic or bad-faith comments may be less common than casual reactions and basketball analysis. However, I still want enough toxic examples for the model to learn the difference between harmless negativity and genuinely harmful discourse.

If one label is underrepresented after collecting 200 examples, I will take the following steps:

1. Search more targeted thread types where that label is more likely to appear. For example, toxic or bad-faith comments may be more common in post-game loss threads, controversial trade discussions, or referee complaint threads.
2. Continue collecting additional examples beyond 200 until the underrepresented label has enough examples to be useful.
3. If the label is still rare, I will document the imbalance honestly and use evaluation metrics such as macro F1 score and per-label recall instead of relying only on accuracy.
4. I will avoid artificially inventing examples because the goal is to model real discourse from the community.

After annotation, I will split the dataset into train, validation, and test sets using a stratified split so that each label appears in all three sets. For 200 examples, my planned split is:

| Split | Count |
|---|---:|
| Train | 140 |
| Validation | 30 |
| Test | 30 |
| **Total** | **200** |

## 5. Evaluation Metrics

Accuracy alone is not enough for this task because the labels may not be perfectly balanced. For example, if casual reactions are the most common label, a model could get a decent accuracy score by overpredicting `casual_reaction` while failing to identify toxic comments or constructive analysis. That would not be useful for a real discourse-quality tool.

I will use the following evaluation metrics:

### Accuracy

Accuracy will show the overall percentage of examples classified correctly. It is useful as a general summary, but I will not rely on it by itself.

### Precision per label

Precision will show how often the model is correct when it predicts a particular label. This is especially important for `toxic_bad_faith` because falsely labeling normal fan discussion as toxic could be unfair and frustrating for users.

### Recall per label

Recall will show how many true examples of each label the model successfully catches. This is especially important for `toxic_bad_faith` because a moderation-support tool should not miss too many harmful comments.

### Macro F1 score

Macro F1 is important because it gives equal weight to each label, even if one label is less common. This is a better metric than accuracy for my task because I care about performance across all three discourse-quality categories, not just the largest class.

### Confusion matrix

I will also inspect a confusion matrix to understand where the model fails. For example, I want to know whether the model often confuses `constructive_analysis` with `casual_reaction`, or whether it confuses sarcastic comments with toxic comments. This will help me understand the model's weaknesses and explain them honestly.

## 6. Definition of Success

A successful classifier should be able to separate thoughtful basketball discussion from casual fan reactions and toxic or bad-faith discourse. It does not need to be perfect, because sports discussions are full of sarcasm, slang, jokes, and emotional reactions that can be hard even for humans to label consistently.

For this project, I would consider the model useful if it achieves:

- Overall accuracy of at least **75%**
- Macro F1 score of at least **0.70**
- `toxic_bad_faith` recall of at least **0.75**
- `toxic_bad_faith` precision of at least **0.70**

For a real community tool, I would not want the classifier to automatically delete comments by itself. A “good enough” deployment would use the classifier as a moderation assistant or sorting tool. For example, it could flag likely toxic comments for human review, identify high-quality discussion for highlighting, or help moderators understand the overall tone of a thread.

For real deployment, I would want stronger performance than in the class project. I would consider it good enough for a real community tool only if it achieves around:

- Macro F1 score of at least **0.80**
- `toxic_bad_faith` precision of at least **0.85**
- `toxic_bad_faith` recall of at least **0.80**

The higher precision requirement matters because false accusations of toxicity can harm trust in the tool. The model should be conservative when labeling something toxic. It is better for borderline cases to be sent to human review than for the model to automatically punish users.

Overall, success means the classifier can support better community moderation and analysis without replacing human judgment.

## 7. AI Tool Plan

This project is not mainly an implementation project, so I do not plan to use AI tools primarily for generating code. Instead, I will use AI tools in three parts of the workflow where they can help improve the quality of the dataset, annotation process, and evaluation write-up.

### 7.1 Label Stress-Testing

Before annotating the full dataset, I will use an AI tool to stress-test my label definitions. I will give the tool my three labels, definitions, and hard edge case rules, then ask it to generate 5–10 example r/NYKnicks-style comments that sit near the boundary between two labels.

The goal is not to add these generated examples to the dataset. The goal is to test whether my labels are clear enough for difficult cases. If the AI produces examples that I cannot classify cleanly, that means my label definitions are still too vague. I will revise the label definitions before annotating the real 200-example dataset.

The most important boundary cases I will test are:

- `constructive_analysis` vs. `casual_reaction`: comments that contain some basketball reasoning but are mostly emotional, sarcastic, or low-effort.
- `casual_reaction` vs. `toxic_bad_faith`: comments that are negative, sarcastic, or aggressive but may not contain a direct insult.
- `constructive_analysis` vs. `toxic_bad_faith`: comments that include real basketball analysis but also insult another user, player, coach, or fanbase.

Based on this stress test, I will use the following clarified rules during annotation:

1. A comment should only be labeled `constructive_analysis` if it contains a meaningful basketball reason, explanation, comparison, or evidence-based claim.
2. A comment should be labeled `casual_reaction` if it is mostly hype, frustration, sarcasm, joking, or unsupported opinion, as long as it does not directly attack a person or group.
3. A comment should be labeled `toxic_bad_faith` if it includes direct insults, hostile baiting, harassment, or personal attacks, even if it also contains some basketball analysis.
4. General criticism of a player, coach, team, or referee is not automatically toxic. It becomes `toxic_bad_faith` only when the wording becomes personally insulting, abusive, or clearly baiting.
5. If a comment remains ambiguous after applying these rules, I will record it in the annotation notes and make the most consistent decision based on the comment's overall effect on discussion quality.

### 7.2 Annotation Assistance

I may use an LLM as an annotation assistant, but I will not let it be the final annotator. If I use an AI tool, I will use it to pre-label a batch of examples and then manually review every label myself.

The purpose of pre-labeling is to speed up the first pass and help identify difficult examples. However, the final dataset labels will be my responsibility. I will correct any AI labels that do not match my definitions.

If I use AI pre-labeling, I will track it in the dataset with an additional column such as:

| Column | Purpose |
|---|---|
| `ai_prelabel` | The label suggested by the AI tool. |
| `final_label` | The final human-reviewed label used for training and evaluation. |
| `label_changed` | Whether I changed the AI-suggested label. |
| `annotation_notes` | Notes for difficult or ambiguous examples. |

This will allow me to disclose AI assistance honestly in the final AI usage section. I will not include AI-generated comments as training, validation, or test data. The dataset will contain real examples collected from r/NYKnicks.

### 7.3 Failure Analysis

After fine-tuning and evaluating the classifier, I will collect the model's wrong predictions from the validation and test sets. I will give these misclassified examples to an AI tool and ask it to identify possible error patterns.

I will look for patterns such as:

- The model confusing sarcastic fan reactions with toxic comments.
- The model labeling strong but reasonable criticism as `toxic_bad_faith`.
- The model missing toxic comments when the insult is indirect or uses sports slang.
- The model confusing short basketball opinions with `constructive_analysis`.
- The model overpredicting the most common label if the dataset is imbalanced.
- The model struggling with comments that require Knicks-specific context, such as nicknames, memes, or references to recent games.

I will use the AI's pattern suggestions as a starting point, not as final evidence. I will verify the patterns myself by reviewing the actual misclassified examples and checking whether the same mistake happens repeatedly. In the final evaluation write-up, I will only report failure patterns that I can confirm from the model's outputs.

This use of AI will help organize the error analysis, but the final interpretation will be based on my own review of the evidence.

