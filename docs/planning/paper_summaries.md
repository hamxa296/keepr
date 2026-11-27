# Keepr — Paper Summaries

<!--
  Versioned planning doc. The Markdown file is the source of truth for diffs;
  the PDF next to it is a snapshot of the same version. See docs/planning/README.md.
-->
> **Version 1.2** · 28 Nov 2026 · Live doc: https://claude.ai/artifact/EaUXwgDxxbLEUcUwzv931M · PDF snapshot: [`Keepr_Paper_Summaries_v1.2.pdf`](Keepr_Paper_Summaries_v1.2.pdf)
> Changes since 1.0: [CHANGELOG](CHANGELOG.md).


24 papers, grouped by what they give Keepr. Each summary has six parts: the main idea, how it works, what we get, the key concept explained in plain terms, its limits, and a concrete next step for the team. **Read the six marked ★ first:** Chang 2016, FGAesthetics, Co-Instruct, LEAF, Calibrated Selective Classification and VLM position bias.

## Series and burst photo selection

These papers study our exact task: given several near-identical shots, pick the one people prefer.

### 1. ★ Automatic Triage for a Photo Series (Chang et al., SIGGRAPH / ACM TOG 2016)

[Paper](https://gfx.cs.princeton.edu/pubs/Chang_2016_ATF/chang2016-triage-preprint.pdf) · [Reproduction code](https://github.com/zhijian-liu/auto-triage)

**Main idea.** Instead of scoring photos on an absolute good-to-bad scale, rank photos *within* a series of the same scene. The authors built the Phototriage dataset from real personal albums: 15,545 unedited photos in 5,953 series, with 98,780 crowd judgments of which photo in a pair is better. Categories include portraits, groups, selfies, food, animals, interiors and landscapes.

**How it works.** A Siamese network built on VGG scores each photo with the same weights, and the two scores are compared to predict which one people prefer. It reaches 73% accuracy on test pairs where raters clearly agreed (over 70% agreement); the best generic quality model, MemNet, gets 57%. Only about 65% of photo owners agreed with the crowd's pick, which shows how personal taste is.

**What we get.** Our main training and test data, the evaluation protocol (pair accuracy on high-agreement pairs, split by series), the baseline to beat, and a realistic ceiling to quote in the report.

**Concept explained.** A Siamese network is one CNN used twice with the same weights: photo A goes in and gives a score, photo B goes in and gives a score, and the gap between the two scores goes through a sigmoid to give P(A is preferred). Because the weights are shared, the network learns a single "goodness" score that works for any photo, so you can rank a whole series just by sorting the scores. To turn noisy crowd votes into a clean ranking per series, the authors fit a Bradley–Terry model, which estimates a hidden strength for each photo from who beat whom. They report accuracy only on pairs where raters clearly agreed, because 50/50 pairs have no right answer.

**Limits to keep in mind.** The photos are from around 2016, so cameras, phones and editing styles differ from today's. Each pair got only about 3–10 votes (about 20 divided by the series length), so vote fractions on long series are noisy. There are only 7,459 labelled training pairs, which is small for deep learning. The paper's split sizes (4,560 / 195 / 967 series) don't add up to the stated 5,953, so state the exact split you use.

**Try this.** First milestone: reproduce Chang's setup with a modern small backbone (EfficientNet-B0) and the same "over 70% agreement" test pairs. Keep each pair's vote fraction; it's the target for the calibration experiments.

### 2. Real-time Burst Photo Selection (Wang et al., Microsoft; arXiv 2018, IEEE TIP 2019)

[arXiv](https://arxiv.org/pdf/1803.07212) · [TIP version](https://pubmed.ncbi.nlm.nih.gov/31804935/)

**Main idea.** Pick the best moment from a camera burst the instant the shutter is pressed, on the phone itself. The camera keeps a few frames from before and after the press, and a tiny network gives each frame a "goodness" score. The authors captured their own dataset of about 15k bursts.

**How it works.** A light-head network trained with an adversarial (GAN-style) objective. The final model is only 0.47 MB and scores a frame in 13 ms on an iPhone 7. Its pick matched the user's top choice in 64.1% of bursts (about 11 frames each) and was in the user's top 3 in 86.2%.

**What we get.** Proof that burst selection can run on a phone with a very small model, plus user-study numbers to compare our own pilot against.

**Concept explained.** Keeping frames from just before and after the shutter press means the camera has already captured the moment you meant to take. Choosing among them is a *relative* problem: frame A only needs to beat frame B, it doesn't need a perfect absolute score. That's why such a tiny model can work. Top-1 means the model's pick was the user's favourite; top-3 means the user's favourite was among the model's three best.

**Limits to keep in mind.** The model was trained on the authors' own bursts, not a public benchmark, so you can't compare numbers directly with Phototriage results. The authors also note that Chang's triage setup doesn't really cover true bursts: burst frames differ far more subtly than the shots in a Phototriage series.

**Try this.** Use top-1 and top-3 agreement as the metrics for our user pilot, so we can quote 64.1% / 86.2% as a reference point. Keep our student well under 25M parameters; this paper shows that far smaller models are possible.

### 3. Series Photo Selection via Multi-view Graph Learning (Huang et al., 2022)

[arXiv](https://arxiv.org/html/2203.09736)

**Main idea.** A single feature extractor misses useful cues, so look at each photo through several "views" (deep VGG-16 features plus handcrafted ones such as saturation) and model how those views relate with a graph.

**How it works.** A graph convolutional network links the views, an adaptive self-attention module weights them, and a Siamese network compares two photos and predicts which one wins. Trained and tested on Phototriage, where it beats earlier single-view models.

**What we get.** Evidence that adding simple low-level cues (colour, saturation, sharpness) on top of deep features helps. That supports our separate image-quality head.

**Concept explained.** A graph convolutional network (GCN) treats each feature view as a node and lets nodes pass information to each other, so the saturation view can be influenced by the deep-semantics view and vice versa. Self-attention then learns how much to trust each view for a given photo, instead of using fixed weights.

**Limits to keep in mind.** The backbone is VGG-16, which is large and slow by today's standards, and the handcrafted views add extra computation. The authors re-shuffled the training and validation splits, so their numbers aren't strictly comparable with Chang's original split.

**Try this.** A cheap version for an ablation: compute a few simple statistics per photo (sharpness as the variance of the Laplacian, a brightness histogram, mean saturation), concatenate them to the CNN features, and see whether accuracy improves.

### 4. Modeling Multiple Aesthetic Views for Series Photo Selection (SPSNet, IEEE 2023)

[IEEE Xplore](https://ieeexplore.ieee.org/document/10168263/)

**Main idea.** The same multi-view idea, pushed further: combine deep, generic and handcrafted features of each image.

**How it works.** A Siamese network with parallel deep and shallow branches fuses local and global features, reaching 75.36% accuracy on Phototriage.

**What we get.** A stronger published number for our baseline table, and more support for fusing low-level and high-level features.

**Concept explained.** Deep layers of a CNN capture meaning (who and what is in the photo); shallow layers capture edges, texture and blur; handcrafted features measure things explicitly (contrast, colourfulness). Local features look at small regions such as a face, and global features look at the whole frame. Fusing all of these lets the model notice both "someone blinked" and "the whole shot is dark".

**Limits to keep in mind.** We've only read the abstract, so check the exact data split and test protocol before putting 75.36% in a table next to our own numbers.

**Try this.** Put it in the baseline table as the best published result before 2026, labelled "reported". Read the method section only if you need ideas for the fusion layer.

### 5. Focusing on Subtle Differences: Feature Disentanglement for Series Photo Selection (IEEE TMM 2024)

[ACM/IEEE page](https://dl.acm.org/doi/10.1109/TMM.2024.3382469)

**Main idea.** Photos in a series are almost identical, so the useful signal is tiny. This paper separates (disentangles) the features that relate to the preference label from those that don't.

**How it works.** An encoder learns to capture all label-relevant information and push the rest aside, so the model focuses on the small differences between near-duplicates. Reported to beat earlier methods on Phototriage.

**What we get.** A design lesson: our model should focus on *what differs* between two shots (eyes, blur, expression), not on the scene they share. That justifies the face-state heads.

**Concept explained.** Disentanglement means splitting what a network has learned into separate parts, here roughly "what is in the scene" (shared by every photo in the series, so useless for choosing) and "how well this particular shot was taken" (what actually decides the winner). The model is pushed to rely only on the second part.

**Limits to keep in mind.** We've only seen the abstract, with no exact accuracy, so cite it for the idea rather than for a number.

**Try this.** A simple way to borrow the idea: also give the ranking head the *difference* of the two photos' features (feature A minus feature B), which cancels out the shared scene. Run it as one ablation.

### 6. ★ FGAesthetics / FGAesQ: Learning Discriminative Scores from Relative Ranks (arXiv, March 2026)

[arXiv](https://arxiv.org/html/2603.03907v1)

**Main idea.** The newest and closest work. It builds FGAesthetics, a fine-grained dataset of 32,217 images in 10,028 series with 44,863 pairs, each judged by 10 annotators. It mixes natural photo series (built from Chang's data), AI-generated image series and cropping series. Pairs that people couldn't tell apart were removed.

**How it works.** FGAesQ, a CLIP ViT-B/16 model (86.3M parameters), adds three parts: difference-preserving tokens, comparative text alignment and rank-aware regression. It reaches 0.779 pair accuracy on natural series and beats large VLM scorers such as Q-Align (0.711), UNIAA (0.709) and RealQA (0.658).

**What we get.** The current best number to compare against, a possible second dataset with 10 votes per pair (ideal for checking confidence), and the idea of training on *differences* between photos.

**Concept explained.** The three parts each tackle "the photos look almost the same". DiffToken keeps the small differences between images when they are turned into tokens instead of averaging them away. Comparative text alignment teaches the model with sentences that describe how two images differ (generated with Gemini 2.5 Pro and GPT-4o). Rank-aware regression makes the scores respect the human ranking inside each series. At the series level (ranking all photos in a series) it reaches 0.729 rank correlation on natural photos.

**Limits to keep in mind.** The model is 86.3M parameters and was trained on a datacenter A800 GPU, with no speed or on-device numbers. It reports no confidence calibration and never says "not sure". Indistinguishable pairs were deleted from the data rather than kept as "toss-ups". The natural series include 150 series taken from video frames (LSVQ), and the split is 8:1:1.

**Try this.** On day 1, check the paper for a data or code release. If the data is out, evaluate on the natural subset so our numbers sit directly next to 0.779, and use the 10-vote pairs to check whether our confidence matches the human split.

### 7. Burst Image Quality Assessment (BuIQA, AAAI 2026)

[arXiv](https://arxiv.org/abs/2511.07958)

**Main idea.** A new task: score the quality of each frame within a burst, for whatever the burst is used for next (denoising, super-resolution, or human viewing). Ordinary single-image quality models struggle because burst frames differ in almost invisible ways.

**How it works.** A dataset of 7,346 bursts (45,827 images, 191,572 quality scores) and a framework with a prompt-generation network that tells a quality network which downstream task to judge for. Picking frames this way improved downstream denoising and super-resolution by 0.33 dB PSNR.

**What we get.** A possible extra dataset if released, and confirmation that within-burst differences need specialised models rather than generic quality scores.

**Concept explained.** "Task-aware" means the best frame depends on what you'll do with it. For denoising you want the frame with the least motion; for a person viewing photos you may want the best expression. Prompt tuning learns a small set of extra input vectors that steer a shared network toward one task, instead of training a separate network per task.

**Limits to keep in mind.** Much of the benchmark targets machine tasks (denoising, super-resolution) rather than which photo a person would keep, and we haven't confirmed whether the data is publicly released.

**Try this.** Use it in the report to justify defining our task as "the frame a person would keep". If the subjective part of the dataset is released, it makes a good extra robustness test.

### 8. Attractiveness of Human Action Shots (arXiv 2017)

[arXiv](https://arxiv.org/pdf/1711.00677v1.pdf)

**Main idea.** For action photos (jumps, sports), the best frame in a burst is mostly decided by the person's pose at the peak of the action. This paper is the first data-driven study of what makes an action shot attractive, with a dataset of about 8,000 shots.

**How it works.** A deep Siamese network trained with a hybrid loss that uses both absolute ratings and pairwise comparisons. It finds that attractiveness is subjective but still predictable.

**What we get.** Support for combining two kinds of labels in one loss (our human votes plus teacher labels), and a reminder that non-portrait bursts (sports, jumps) depend on pose, which is a good failure case to discuss.

**Concept explained.** "Peak action" is the instant at the top of a jump or swing when the subject briefly hangs motionless, usually the shot people want. A hybrid loss trains one network on two label types at once (here, absolute ratings and pairwise comparisons) so neither kind of label is wasted.

**Limits to keep in mind.** It's from 2017, specific to action photography, and only the first arXiv version is linked here.

**Try this.** While doing error analysis, tag sports and jump series in Phototriage. If our model does worse there, that's a ready-made failure case: pose timing matters, and our face and blur heads don't capture it.

## Faces and group photos

People photos are where closed eyes and looking away ruin a shot; these papers show which face signals matter.

### 9. All Smiles: Automatic Photo Enhancement by Facial Expression Analysis (Google, CVMP 2012, best paper)

[Google Research](https://research.google/pubs/pub41101/)

**Main idea.** In a set of group shots, someone always has their eyes closed, is looking away or isn't smiling. Score every face in every frame, then build the best possible photo.

**How it works.** Each face gets a score for eyes, gaze and smile. The system picks the frame with the best overall score, then replaces any low-scoring face with the same person's best face from another frame, using alignment and seamless blending. This is the idea behind Google's later "Best Take" feature.

**What we get.** The face signals to score (eyes open, looking at camera, smiling) and a clear v2 product feature: when every frame has someone blinking, offer a face swap instead of just picking the least-bad shot.

**Concept explained.** The pipeline has two steps. *Selection* scores each person's face in each frame and picks the frame with the highest total. *Composition* then finds the same person in another frame, aligns their face to the base photo, and blends it in so the seam isn't visible. The result is a photo that never actually happened, but in which everyone looks their best.

**Limits to keep in mind.** It predates deep learning, so its face scoring relies on older hand-built methods that struggle with tilted, small or partly hidden faces. Swapping faces also changes reality, which raises honesty and consent questions; Keepr should always say when a photo has been edited.

**Try this.** Not for AI341. Keep it as a v2 feature: once our face heads score every face, "swap in the best face" becomes an add-on rather than a new model. Mention it in the report's future work.

### 10. Aesthetic Quality Assessment of Group Photographs (arXiv 2020)

[arXiv](https://arxiv.org/pdf/2002.01096v1.pdf)

**Main idea.** General photo-quality models ignore what matters most in group photos: the state of each person. The authors design features from photography principles and build a group-photo dataset.

**How it works.** Hand-designed features for open eyes, gaze direction, smile and occlusion (faces blocked by other people or objects), fed into a machine-learning model. They also point to a guidance system that could help people while taking group photos.

**What we get.** A checklist for our face heads. It adds **occlusion**, which our plan didn't have at first, and it's a reference for why face state matters more than general aesthetics in people photos.

**Concept explained.** Hand-designed features are rules a person writes (for example, "eye openness = distance between eyelids divided by eye width"), while learned features are whatever a CNN discovers during training. Hand-designed features are easy to explain but brittle; learned features are flexible but harder to interpret. Occlusion means a face is partly blocked, by another person's head, a hand, a drink or hair.

**Limits to keep in mind.** It scores single group photos rather than comparing near-duplicates, and its hand-designed features predate modern face models. We haven't checked the size or availability of its dataset.

**Try this.** Use its feature list as the label checklist for our face heads: eyes open, gaze at camera, smile, **occluded**. Show the hand-designed approach (an eye-openness rule from MediaPipe landmarks) next to our learned heads in the ablation table; it's an easy, explainable baseline.

### 11. Group Photo Ranking with rGroup (arXiv 2020)

[arXiv](https://arxiv.org/pdf/2006.03898.pdf)

**Main idea.** Rank several group photos of the same scene by combining how happy the group looks, how aesthetic the photo is, and its technical quality.

**How it works.** Separate scores for group emotion, aesthetics and image quality are combined into one ranking. They release the small rGroup database: 70 sets of 3 group photos from indoor and outdoor events.

**What we get.** Support for fusing face/emotion scores with image-quality scores into one rank, exactly what our fusion layer does. The dataset is too small to train on but could serve as an extra sanity check.

**Concept explained.** This is late fusion: separate models each produce a score (emotion, aesthetics, quality), and the scores are combined at the end. The alternative, early fusion, mixes features before any score is made. Late fusion is easier to debug because you can see which part pushed a photo up or down.

**Limits to keep in mind.** The dataset is tiny (70 sets of 3), and a happy-looking group isn't always the photo people keep, since a candid laugh can beat a stiff smile.

**Try this.** Build Keepr's fusion as late fusion first (face score + image-quality score, then a small layer), because that also gives the app its explanation text for free ("ranked lower: 1 person blinking"). Use rGroup only as a sanity check.

### 12. An Image Quality Assessment Dataset for Portraits (PIQ23, arXiv 2023)

[arXiv](https://arxiv.org/pdf/2304.05772)

**Main idea.** Existing quality datasets use artificial distortions (compression, noise) that don't match how modern smartphones process portraits. This paper builds a portrait-specific quality dataset from smartphone photos.

**How it works.** Portraits are judged by over 30 image-quality experts. The paper also reviews earlier smartphone datasets, such as SPAQ (11,125 photos from 66 phones), and notes their weaknesses, like resized images.

**What we get.** A source of phone-portrait quality labels if our image-quality head needs them, and a warning: train and test at realistic phone resolutions, not on tiny resized images.

**Concept explained.** Modern phones don't just record light; they apply heavy processing (HDR merging, sharpening, skin smoothing, night modes), and that processing creates its own kinds of flaws. Datasets built by adding artificial noise or JPEG compression to clean photos don't capture those flaws, so models trained on them can misjudge real phone portraits. Expert labels are more consistent than crowd labels but more expensive to collect.

**Limits to keep in mind.** It measures portrait *quality*, which isn't the same as which photo a person would keep. Check its access terms before using it.

**Try this.** Optional pre-training for the image-quality head if SPAQ alone isn't enough. Its main lesson for us: build the shifted test set from real, unedited phone photos at their original resolution.

## VLMs that judge photo quality

These are the candidate "teachers": large vision-language models that can look at photos and say which is better, and explain why.

### 13. Q-Align: Teaching LMMs for Visual Scoring via Discrete Text-Defined Levels (ICML 2024)

[arXiv](https://arxiv.org/abs/2312.17090) · [Code](https://github.com/Q-Future/Q-Align)

**Main idea.** Human raters don't think in numbers like 3.457; they think in words like "good" or "poor". So teach a vision-language model to answer with five rating words (bad, poor, fair, good, excellent) instead of raw scores.

**How it works.** Existing score labels are converted into those words for training. At test time, the model's probabilities for each word are combined into a weighted average, which gives a fine-grained score. One model, OneAlign, handles image quality, image aesthetics and video quality, and generalises well across datasets. It is large (about 8.2B parameters) and takes roughly 100 ms per image on a desktop RTX 3090.

**What we get.** The trick for turning a VLM's answer into a number: read the *probabilities of the answer words*, not just the text. We use the same trick to read P("A") vs P("B") from our teacher. It's also a ready-made baseline (FGAesthetics measured it at 0.711 on natural series).

**Concept explained.** A language model never outputs just one word; it outputs a probability for every possible next word. If it says "good" with 0.6 and "excellent" with 0.3, the expected score is a weighted average (good = 4, excellent = 5, and so on), which is far finer than the single word it prints. This is how a chat-style model becomes a precise scoring machine.

**Limits to keep in mind.** It scores one image at a time, and two near-identical photos often get almost the same score, which is why it trails purpose-built series models (0.711 vs 0.779 on FGAesthetics). At about 8.2B parameters it needs a desktop GPU: roughly 100 ms per image on an RTX 3090, so a 300-photo trip takes about 30 seconds there and is impossible on a phone.

**Try this.** Run OneAlign on the Phototriage validation pairs as a ready-made baseline: score both photos and pick the higher. The Hugging Face model card shows it takes only a few lines of code. It also tests whether our lab A2000 (12 GB) setup works for VLMs.

### 14. ★ Co-Instruct: Towards Open-ended Visual Quality Comparison (ECCV 2024, oral)

[arXiv](https://arxiv.org/abs/2402.16641) · [Model](https://huggingface.co/q-future/co-instruct)

**Main idea.** People judge quality most reliably by comparing images, not by scoring one alone. Co-Instruct is an open-source VLM trained specifically to compare two to four images and explain its answer.

**How it works.** The training set, Co-Instruct-562K, comes from two imperfect "teachers": single-image quality descriptions merged into comparisons, and GPT-4V answers on unlabelled images. The paper also introduces MICBench, 2,000 multiple-choice questions comparing several images. Co-Instruct scores 80.18% on paired comparison questions and beats its own GPT-4V teacher. The same model trained *without* comparison data scores only about 53%.

**What we get.** Our best candidate teacher, since it was built for exactly "which image is better". It also shows that a student can beat its teacher when trained on many teacher labels, and that comparison training matters a lot.

**Concept explained.** Instruction tuning trains a model on many question–answer examples so it follows requests like "Which image is sharper, and why?". "Weak supervisors" are imperfect label sources (here, merged single-image descriptions and GPT-4V); each is noisy alone, but together they give enough signal. A student beating its teacher sounds odd, but it happens because the student learns from hundreds of thousands of teacher answers and averages out the teacher's random mistakes.

**Limits to keep in mind.** It was trained to compare technical *quality* (blur, noise, exposure), not which photo people would *keep*, so it may ignore faces, expressions and timing. It's a large model, so check memory needs before planning around it: on our 12 GB A2000 or a 16 GB Kaggle T4 it will likely need 4-bit loading.

**Try this.** In the teacher probe, run Co-Instruct and Qwen2.5-VL on the same 300 Phototriage pairs, in both orders. Compare accuracy and flip rate, and pick the better one as the teacher.

### 15. DepictQA: Depicting Beyond Scores (ECCV 2024)

[Paper](https://arxiv.org/html/2312.08962.pdf)

**Main idea.** Replace a bare quality score with a language explanation: what is wrong with the image, and why image A is better than image B.

**How it works.** A multi-modal language model is trained on a hierarchy of tasks, from spotting distortions to full comparative reasoning. Multi-source training data and special image tags help it handle several images at once.

**What we get.** A model for the explanation feature ("photo 3 is blurry, photo 5 has closed eyes") in a later server-side version of Keepr, and a citation for why explanations improve trust.

**Concept explained.** A bare score ("3.2") doesn't tell the user what to fix or why a photo lost. An explanation ("the left photo is blurry on the faces; the right one is slightly underexposed") is easier to trust and lets the user disagree sensibly. The hierarchy goes from simple tasks (name the distortion) to hard ones (reason about which of two images is better).

**Limits to keep in mind.** It's large and slow, so it's not usable on-device, and it was trained mostly on distortion-style quality data rather than personal photo preferences.

**Try this.** Not for AI341. Note it as a premium "explain my picks" feature that could run on a server, and cite it in the report's future work. For now, Keepr's explanations come from the face and quality heads ("1 person blinking").

### 16. The Photographer's Eye: PhotoEye and PhotoBench (arXiv 2025)

[arXiv](https://arxiv.org/html/2509.18582v1)

**Main idea.** General VLMs understand what's *in* a photo but are weak at judging it like a photographer (exposure, composition, equipment). PhotoEye teaches a multimodal LLM to critique photos the way photographers do.

**How it works.** Vision modules tuned to low-level and aesthetic features are added. In a series of increasingly overexposed photos, PhotoEye flagged overexposure from the second photo, while other models only noticed in the third or fourth. The authors also release PhotoBench to test aesthetic understanding.

**What we get.** Evidence that general VLMs miss subtle technical flaws. So a raw VLM teacher may be weak on blur and exposure, and our separate image-quality head (and good teacher selection) is justified.

**Concept explained.** Most VLMs use a vision encoder trained to recognise *content* (dog, beach, person), which learns to ignore exposure and blur, since a dark dog is still a dog. PhotoEye adds vision modules that pay attention to exactly those low-level details, which is why it catches overexposure earlier.

**Limits to keep in mind.** It's about critique and aesthetic understanding, not choosing among near-duplicates, and it's a large model.

**Try this.** A 10-minute sanity test for whichever teacher we pick: take one good photo, make 5 copies with increasing brightness and blur, and ask the teacher to compare neighbouring pairs. If it can't rank them correctly, don't trust its labels on blur or exposure; rely on our image-quality head for those.

## Distillation and pairwise learning

These show how to move a big model's judgement into a small one, and how to learn from "A is better than B" labels.

### 17. ★ LEAF: Label-Efficient Image Quality Assessment (arXiv, January 2026)

[arXiv](https://arxiv.org/html/2601.20689)

**Main idea.** A large VLM already "sees" image quality well; what it lacks is the exact score scale humans use. So use the VLM as a teacher to train a much smaller model, then fine-tune that student on only a few human labels.

**How it works.** A frozen InternVL-3.5-8B teacher gives two kinds of soft labels: a score from the probabilities of the five rating words (as in Q-Align), and pairwise "A or B" preferences. Each pair is weighted by how confident the teacher was (lower entropy means higher weight), and very uncertain pairs are dropped. The student is a ConvNeXt-Base (88.6M parameters). With **no** human scores it already reaches 0.777 rank correlation on KonIQ and 0.861 on SPAQ; with 10% of the labels, 0.867 and 0.896.

**What we get.** Our distillation recipe almost as-is: confidence-weighted pairwise loss, drop low-confidence pairs, then fine-tune on human votes. Our version uses a much smaller student and works within a photo series.

**Concept explained.** Entropy measures how unsure a probability is: 50/50 has maximum entropy, 99/1 almost none. LEAF's pair weight is 1 minus the entropy divided by log 2, so a confident teacher answer gets a weight near 1 and a coin flip gets near 0. Training has two stages: (1) learn from the teacher with no human labels, using a point-wise loss (match the teacher's score) plus a pair-wise loss (match the teacher's A-vs-B preference); (2) a short fine-tune on a few human scores to fix the scale. Their ablation shows each piece helps: on AGIQA-3K with 10% of labels, fine-tuning alone gives 0.759, adding point-wise distillation 0.834, and the full method 0.841 rank correlation.

**Limits to keep in mind.** You still have to run an 8B teacher over every unlabelled image once, which takes GPU hours. The student (88.6M) is much bigger than ours will be. It reports no confidence calibration and no speed numbers, and code was only promised on publication. Bigger teachers (32B–38B) helped only slightly, and a bigger student (ConvNeXt-Large) didn't help, so a medium teacher plus a small student is a sensible choice.

**Try this.** Copy the pair-wise loss and the entropy weighting directly. Tune the drop threshold on validation. Cache the teacher's outputs to a file once on the lab A2000 so every student run reuses them.

### 18. Visual-Language Model Knowledge Distillation for Image Quality Assessment (arXiv 2025)

[Summary page](https://pith.science/paper/2507.15680)

**Main idea.** Transfer CLIP's knowledge of image quality into a small image encoder that is cheap enough to deploy.

**How it works.** CLIP becomes a quality scorer using five quality-graded text prompts, and only its image encoder is fine-tuned on human labels. A lightweight student then learns from a mix of matching the teacher's features (soft) and the human labels (hard), with the balance changing over training on a cosine schedule.

**What we get.** A second distillation recipe that copies the teacher's *features*, not just its answers. Worth trying as an ablation if answer-level distillation is weak; CLIP is also a cheaper teacher than a full VLM.

**Concept explained.** There are two ways to learn from a teacher. *Answer distillation* copies the teacher's outputs (scores or probabilities). *Feature distillation* makes the student's internal feature vector look like the teacher's, which passes on richer information. Cosine annealing gradually shifts the balance during training, here between copying the teacher and fitting human labels.

**Limits to keep in mind.** We've read a summary page, not the full paper, so treat the details as approximate. It works on single images, not pairs.

**Try this.** If answer distillation from the VLM is weak or too slow, try feature distillation from CLIP's image encoder into the student as one ablation. CLIP is much cheaper to run than a full VLM.

### 19. PSDistill: Preferences Order, Ratings Anchor (arXiv, May 2026)

[arXiv](https://arxiv.org/html/2605.19776v1)

**Main idea.** Pairwise preferences are good at saying which image is better; ratings are good at anchoring the absolute scale. Combine both. It also builds expert aesthetic ground truth by fusing the judgements of five experts.

**How it works.** A VLM compares each image against a pool of reference images. Those results are turned into calibrated pseudo-scores with an Elo-style system (like chess ratings), a Bayesian update and a sigmoid calibration. The same VLM is then trained with a confidence-weighted ranking loss, so it scores an image in a single pass instead of many comparisons.

**What we get.** A clean way to convert many pairwise judgements into one score per photo (useful for ranking a whole burst), and more support for confidence-weighted ranking losses.

**Concept explained.** Elo is the chess rating system: every win against a strong opponent raises your rating a lot, and a win against a weak one raises it a little. By comparing each image against a fixed pool of reference images, every image gets a rating that can be compared with any other, even if the two were never directly compared. A Bayesian update keeps track of how certain each rating is.

**Limits to keep in mind.** The final scorer is still a full VLM, so it's not deployable on a phone. The work targets single-image aesthetics with expert ratings, not near-duplicate series.

**Try this.** Use the same idea inside a burst: let the student compare every pair, then turn the wins into one score per photo (Bradley–Terry or Elo) and rank by it. That's more stable than "most wins" when two photos are close.

### 20. PICNIQ: Pairwise Comparisons Are All You Need (DXOMARK / ENS, arXiv 2024)

[arXiv](https://arxiv.org/pdf/2403.09746v2)

**Main idea.** Human pairwise comparisons are more consistent than absolute ratings, so build a quality model that predicts the *probability that one image is better than another*.

**How it works.** The model estimates the preference likelihood for image pairs. Psychometric scaling turns many pairwise comparisons into scores on a just-objectionable-difference (JOD) scale, with a sampling strategy designed for sparse comparisons. It uses and publishes the comparison matrices of the PIQ23 portrait dataset.

**What we get.** Theory behind our design: predict P(A beats B) directly (the Bradley–Terry loss), and treat a 50/50 human split as real information, not noise.

**Concept explained.** Psychometric scaling is the maths psychologists use to turn "A was chosen over B 70% of the time" into distances on a scale. A just-objectionable-difference (JOD) is the gap at which a clear majority of viewers notice that one image is worse. Asking "which is better?" is quicker and more consistent for people than asking them to rate on a 1–10 scale.

**Limits to keep in mind.** It targets portrait image quality measured by experts, not personal "which would I keep" preferences, and its models are built for comparisons between different scenes, not near-duplicates.

**Try this.** Cite it to justify our pairwise design. Then apply the same scaling to Phototriage votes to get a per-photo score for each series, so we can report series-level metrics (did we pick the top-ranked photo?) alongside pair accuracy.

## Trust and bias

These cover when a model should say "not sure", and why a VLM teacher can't be trusted blindly.

### 21. ★ Calibrated Selective Classification (Fisch, Jaakkola & Barzilay, MIT, 2022)

[arXiv](https://arxiv.org/abs/2208.12084v2)

**Main idea.** Letting a model abstain ("I don't know") on uncertain inputs raises accuracy on what it does answer. But the confidence it gives on the answered cases should *also* be honest: 70% confidence should mean right about 70% of the time. They call this selective calibration.

**How it works.** A separate small *selector* network decides which predictions to accept, trained to minimise calibration error on the accepted set at a chosen coverage. To stay reliable when data shifts, training adds simulated perturbations to the inputs. Tested on image classification and lung-cancer risk prediction.

**What we get.** The vocabulary and metrics for our "not sure" pile: coverage, selective risk, calibration on accepted pairs. It also suggests a stronger version of our threshold rule (a learned selector) and the idea of testing calibration under corruptions, which matches our shifted phone-photo set.

**Concept explained.** *Coverage* is the share of cases the model answers (80% coverage means it says "not sure" on 20%). *Selective risk* is the error rate on the cases it does answer. A good selector lowers risk sharply while giving up little coverage. "Robust" here means the confidence stays honest when inputs change, simulated during training by deliberately perturbing images, a little like our blur and low-light stress tests.

**Limits to keep in mind.** It's written for classification (one input, one label), not pairwise ranking, so we have to adapt it: our "prediction" is "A beats B" and our confidence is that probability. A trained selector also adds another model to train and explain in the viva.

**Try this.** Version 1: abstain when the temperature-scaled confidence is below a threshold chosen on validation. Stretch goal: a tiny selector (a small MLP on the student's features) trained on validation, compared on the risk–coverage curve.

### 22. Reliable VQA: Abstain Rather Than Answer Incorrectly (FAIR & UC Berkeley, 2022)

[arXiv](https://arxiv.org/pdf/2204.13631)

**Main idea.** For visual question answering, a wrong answer can be worse than no answer, so frame it as selective prediction: answer or abstain.

**How it works.** With the model's own raw probabilities as confidence, a state-of-the-art VQA model could only answer about 5% of questions while keeping errors very low. A separately learned selection function, using both image and question, estimates confidence much better and lets the model answer far more questions at the same low error.

**What we get.** A strong warning: raw softmax confidence is a poor abstention signal. Budget time for calibration (temperature scaling at minimum) and compare a learned confidence head against raw probabilities in the trust experiments.

**Concept explained.** Softmax probabilities are often overconfident: a network can say 99% and still be wrong, especially on unusual inputs. A *learned selection function* is a second small model trained specifically to predict "will the main model be right here?", and it usually separates right from wrong answers much better than raw probabilities. The paper also argues that a wrong answer should cost more than an abstention when scoring a model.

**Limits to keep in mind.** It's about visual question answering, so the exact numbers don't transfer to photo selection, only the lesson does.

**Try this.** In the trust experiment, compare three confidence signals on the same risk–coverage plot: raw sigmoid probability, temperature-scaled probability, and (stretch) a learned confidence head. If the raw one is clearly worse, that's a strong finding for the report.

### 23. ★ Identifying and Mitigating Position Bias of Multi-image VLMs (Tian et al., ANU & GE Research, 2025)

[arXiv](https://arxiv.org/html/2503.13792v1)

**Main idea.** When a VLM is given several images, its answer can change just because the images are in a different order, even for GPT-4o. The model handles some positions better than others, a bias inherited from its language part.

**How it works.** They pick tasks whose correct answer doesn't depend on image order, shuffle the order, and measure how accuracy changes per position. They then propose a method to reduce the bias.

**What we get.** The reason our teacher must see every pair in **both** orders (A,B) and (B,A), with the results averaged, and why we drop pairs where the two orders disagree. Report the flip rate.

**Concept explained.** A VLM is built on a language model, and language models are known to favour content at certain positions in their input (often the start or end). With images, that means "photo shown first" can get an unfair advantage. Averaging the two orders cancels a constant bias. Pairs where the two orders disagree are ones the model can't really judge, so they're safer to drop.

**Limits to keep in mind.** The study covers general multi-image tasks, not photo-preference pairs, so we must measure the bias on our own data rather than assume its size.

**Try this.** Make the flip test part of the day 4 teacher probe: run 500 pairs as (A,B) and as (B,A) and report the percentage whose verdict follows position. Keep only order-consistent pairs for distillation.

### 24. TASTE: Designer-Annotated Preferences for AI-Generated Graphic Design (arXiv 2026)

[arXiv](https://arxiv.org/pdf/2605.20731)

**Main idea.** A dataset of professional designers' pairwise preferences on AI-generated graphic designs across several criteria, used to test how well VLMs work as judges.

**How it works.** Every pair is judged twice, in normal and flipped order, and under eight paraphrases of the question. Qwen3-VL-4B reached only 0.53 accuracy, and its verdict stayed the same after flipping on 77.5% of pairs, meaning it was answering by position, not content. On the order-consistent pairs alone, accuracy rose to 0.63.

**What we get.** A ready-made protocol for vetting our teacher (flip test, accuracy on order-consistent pairs, sensitivity to prompt wording), and hard evidence that small VLMs can be near-random judges unless filtered. This is why the teacher is a stretch goal, not a dependency.

**Concept explained.** The paper checks a VLM judge in three ways: (1) **position bias**, whether the answer changes when the images are swapped; (2) **conditional accuracy**, how accurate it is on the pairs where it's order-consistent; (3) **paraphrase stability**, how much accuracy moves when the question is reworded (Qwen3-VL-4B varied by about 0.055 across eight wordings). Together they tell you whether a judge is actually looking at the images.

**Limits to keep in mind.** It's about graphic design, not photos, and it tested a small 4B model; bigger or comparison-tuned models may do better.

**Try this.** Copy the protocol for our teacher check: both orders, accuracy on order-consistent pairs, and 3 rewordings of the prompt. If the teacher fails these, drop it and train on human votes only. The plan doesn't depend on it.
