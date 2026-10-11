---
name: faithful-old-photo-restoration
description: Inspect a single old photo to generate an image-specific restoration prompt, then combine it with a fixed general prompt to perform faithful face restoration, remove stains and scratches, export at high pixel dimensions, and run inspection and rework cycles. Suitable for restoring original scans, making local improvements to already restored photos, and restoration assisted by reliable references; not for beautification, face swapping, or redesigning people.
---

# High-Fidelity Old Photo Restoration Skill

## Usage and Task Scope

The user can begin by providing the path to this file and uploading a photo or providing its path. For example:

> Please read and execute this SKILL-ENG.md to restore the old photo I uploaded.

The user may also provide the original scan, an approved restoration version, reference photos, locations requiring rework, and an output directory. Process one photo by default; do not expand the task to an entire folder on your own.

**Only this Skill and the photo provided by the user for the current task are needed.** It does not depend on the author's private photos, fixed sample images, specific filenames, or previous conversations. Person references, quality references, and annotated images are all optional inputs; if none are provided, proceed directly using the original image and the acceptance criteria in this file, without requiring the user to obtain anyone else's photos. The Skill delivery package contains only the general workflow and necessary general-purpose tools, without private originals, sample images, test results, or their absolute paths.

This file is the complete entry point and does not depend on previous chat history. After reading it, actually invoke the available tools to complete restoration, saving, and inspection; merely writing prompts, proposing a plan, changing filenames, or modifying DPI tags does not count as completing photo restoration. If the tools truly cannot execute the task, clearly explain the limitations and what remains unfinished; do not falsely claim actions were performed.

Respect the scope, version, format, and save location explicitly specified by the user for the current task. Ask for necessary information when the original image is missing, the correspondence between people is unclear, or a critical ambiguity cannot be resolved by inference; do not reconfirm requirements that are already clear. Continue any original-image analysis and unambiguous repairs that can be performed independently.

**Determine which project the task belongs to and where outputs will go before creating files.** Production restoration and Skill development/trials are two separate operating modes. Keep private photos, prompts, candidates, comparisons, and records from development testing in an independent validation workspace **outside the Skill delivery directory**; use source photos as read-only inputs. Test samples help validate the workflow; they must not become dependencies for others using the Skill or be packaged for distribution. Do not reuse output directories from past restoration projects, and do not infer the test output location from the directory containing the original or reference images. See Section 8 for the saving rules for ordinary restoration tasks. Before executing any script that writes files, check its output paths to prevent an old script from writing files back into another project.

The execution sequence is: **read the original image and user requirements → analyze this image → generate an additional image-specific prompt → combine it with this Skill's fixed general prompt → perform the actual restoration → inspect against the original → generate the next round's image-specific prompt from failed items → perform local rework → export and recheck after passing**. The general prompt cannot replace inspecting the image and generating an image-specific prompt; generating prompts cannot replace actual restoration. Section 4 provides the two prompt layers and the methods for generating and combining them; Section 7 defines iteration and stopping conditions.

## Core Acceptance Principles

Restore the same photo and the same people, aiming for a result that looks like the original photo after cleaning, repair, and rescanning.

- **Prioritize identity and facial features**: Preserve each person's appearance, apparent age, expression, head angle, pose, and natural asymmetry. A clear result that does not resemble the person is a failure.
- **Protect structure and remove contamination**: Fidelity protects the positions, proportions, contours, and authentic texture of facial features; it does not mean freezing the pixels of an entire face or patch of skin. Forehead black spots, discoloration beside the ears, contamination on arms, and similar issues identified by the user or already confirmed must be removed while preserving structure; their presence in the original scan does not make them identity features.
- **Keep geometry and content unchanged**: By default, preserve the full frame, number of people, aspect ratio, people's positions, and body proportions. Do not crop, squash, stretch, rotate to reconstruct the scene, or extend the background; preview crops cannot replace the complete final image.
- **Damage must be treated**: Do not use faithfulness as a reason to retain obvious stains, scratches, thin colored lines, or surface damage that can reliably be removed.
- **Clarity must be consistent**: Do not substitute a blurred face, a featureless patch of color, or excessive skin smoothing for restoration, and do not make facial clarity conspicuously inconsistent with the hair, body, and surroundings.
- **Protect areas that have already passed**: When the user requests local rework only, such as a mouth corner, ear, or clothing scratch, treat only that defect and the necessary transition area.
- **Preserve source files and approved versions**: Write new copies by default. Even files the user explicitly asks to overwrite should first have a recoverable version saved.

These conditions must all be met. High resolution, large file size, lossless PNG encoding, or unchanged pixels outside a mask cannot by themselves prove faithful restoration of the people.

If the user explicitly requests cropping, removing people, changing the background, or reasonable redrawing, record this as a scope change for the current task; it does not also authorize changes to other people or to other facial features. If the user only says “the other people do not need restoration,” do not directly infer that those people should be deleted.

## 1. Establish Original-Image Evidence and Protection Baselines

1. Read the actual image file and inspect the image. When the original scan is available, do not substitute a chat thumbnail, screenshot, or low-resolution preview for the original file.
2. Record the file path, file format, width and height in the correct display orientation, horizontal/vertical DPI, color mode, bit depth, and readable color profile; retain an input hash or equivalent version identifier.
3. Record two baselines in parallel:
   - **Authenticity baseline**: The original scan and reliable copies of the same photo, which determine the people's structure and the actual content.
   - **Editing baseline**: The current version specified by the user, which determines which already repaired areas must be preserved.
4. Label the role of every input: original scan, current editing target, another physical copy of the same photo, a reference photo of a person confirmed to be the same individual, a restoration quality reference approved by the user, or an annotated screenshot used only to identify problems. Red boxes, arrows, and text on screenshots are not actual photo content.
5. Reference reliability, from highest to lowest: a clear copy from the same negative/of the same photo; a photo of the same person whose correspondence the user has confirmed; a restoration version approved by the user. Generated facial features cannot serve as independent evidence of the person's actual features.
6. “Possibly the same person” does not count as confirmation. When filling in missing facial features, first clarify exactly which person in the reference photo is intended and whether they are definitely the same individual. References showing different ages, angles, or expressions can only help with understanding features; an entire face must not be transferred directly.
7. Assign each person an identifier and mark their position, including babies, distant people, and obscured people. Check the number of people first to prevent omissions, duplicates, or mistakenly restoring a front-facing person as someone seen from behind.

When the user provides an image for the current task and says “achieve this effect,” it may be used as an **optional quality reference**. Actually inspect it and record the specific gaps: whether stains have been removed, whether skin still has discoloration, whether the areas beside the ears/on the arms/at object edges are clean, and whether clarity and grain are consistent. By default, a quality reference participates only in acceptance review; it is not included in generation calls, editing inputs, or compositing material. Even if it is a restored version of the same photo, do not copy it or perturb it and pass it off as a result produced by this workflow. If the user separately and explicitly designates an image as the editing baseline or a person reference, use it in its authorized role and record that role. A quality reference cannot stand in for a result preview. When no quality reference is available, apply the complete criteria in Section 6 for acceptance review; do not reduce stain-removal requirements or suspend restoration because of its absence.

Save working files in a separate process directory to keep failed candidates out of the finished-output directory. For each round, retain traceable inputs, prompts/operation instructions, candidates, and inspection conclusions; do not rewrite the original scan and pass it off as a restoration result.

## 2. Analyze Damage and Confirmable Facial Information

Inspect the full frame, then inspect every face and damaged area using unstretched crops at native pixel resolution. Crops are only for analysis and local processing; the final result must return to the original canvas.

Create a **defect log** that can be updated throughout the task. Assign a fixed identifier to each issue and record its location/crop, damage type, evidence, proposed method, protected structure, before/after results, and status individually. When a single screenshot boxes several different areas, record them separately; do not summarize the forehead, areas beside the ears, chin, and arms as “facial processing complete.”

Use the statuses “Pending work / Pending review / Resolved / Awaiting evidence / Unresolved.” Mark an item “Resolved” only after actually inspecting the same-location crop after repair and verifying that the defect is gone and the structure looks natural. Immediately reopen the corresponding item when the user points out a new omission; the earlier passing conclusion no longer applies to that item. The next round's image-specific prompt must carry forward all unresolved items; they must not be lost during iteration.

Record **actionable defects** separately from **the precision of historical detail that cannot be recovered**. Remaining stains, large anomalies whose removal has not yet been decided, and insufficient evidence for key facial features are all open issues. If a missing area has already been reasonably filled in as authorized and has passed checks against original-image constraints and naturalness, but it cannot be proven that every fiber exactly matches its state at the moment of photography, this is a precision limitation that must be disclosed. It does not require endless retries, and the actual details must not be claimed as recovered.

| Information status | Treatment principle |
| --- | --- |
| Clearly visible and undamaged | Protect the original structure; do not regenerate facial features |
| Contaminated or faded, but contours remain continuous and discernible | Remove contamination, correct color, repair actual scratches, and restore the existing structure |
| Small missing area with sufficient neighboring structure | Fill in locally, continuing the authentic material and contours |
| Large missing area with unconfirmable facial features | Seek reliable references; mark it unresolved if evidence is insufficient, rather than inventing a replacement face |

Focus on locating the following damage instead of mechanically applying the same treatment to every image:

- Gray haze, yellowing, pink/purple or cyan/green color casts, local colored contamination, and fading.
- Mold spots, white flaking, black spots, cracks, creases, long thin diagonal scratches, vertical colored lines, and edge damage.
- Continuous scratches crossing skin, fingers, clothing, a platform, shoes, or floor tiles: record their complete starting and ending points and paths.
- Fine, shallow abrasion around major damage and on skin: low-contrast short white lines, networks of fine cracks, and intermittent faint marks also require individual localization. They often become visible only after discoloration is removed or contrast restored; do not classify them as normal grain merely because they do not look like major scratches.
- Artifacts introduced by earlier restoration: crooked mouths, unnatural eyes, duplicate facial features, changes to face shape, patches of color, black outlines, sharpening halos, mosaic artifacts, and face–neck seams.

First distinguish stains from actual content. Do not delete a hat crown, clothing, fingers, shadows, wrinkles, moles, or decorations present in the original merely because they look odd. However, “this spot is in the original” is also insufficient to prove it is actual content: check whether contamination crosses skin and background, whether edges show flaking/halos, and whether color is inconsistent with neighboring materials; cross-check against a reliable copy of the same photo when available. A quality reference measures only cleanliness and the naturalness of repairs; it is not evidence for identifying objects in the original or deleting actual content. Do not first label an entire flaked area or patch of discoloration as an object such as a disk or balloon, then preserve it in the name of fidelity.

For stains that the user has already explicitly identified or asked to remove, record them as restoration targets for this round; do not continue using an earlier assumption that they “might be moles, so should be retained.” Mark other uncertain suspected defects that affect acceptance review as “Awaiting evidence,” and continue verifying or ask when necessary; do not simultaneously list them as uncertain residual defects and claim the entire image has passed stain-removal acceptance review. This does not mean indiscriminately removing normal grain or genuine skin features supported by evidence.

## 3. Choose the Minimum Necessary Restoration Method for This Round

Choose methods according to the evidence and scope, and follow the tool rules of the current environment; do not invent tool parameters, silently switch to an API/paid route requiring separate authorization, or promise resolutions the tools do not support.

- **Minor damage or local rework**: Prioritize conventional retouching operations that comply with the tool rules, such as local color correction, healing brushes, cloning, frequency separation, neighborhood-based defect filling, and moderate detail recovery. Clear, intact facial features do not participate in generation.
- **Generative filling is needed**: Generate candidates only for damaged areas or areas that genuinely cannot be treated using conventional methods, with the original scan always serving as a constraint. Have the image-editing tool generate repair material; do not default to replacing the original scan with a low-resolution redraw of the entire image.
- **The user permits reasonable redrawing**: Treat the necessary facial areas continuously under the constraints of the original image and reliable references; preserve recognizable key structures. Permission for reasonable redrawing does not authorize beautification, changing the person's appearance, or adding distinctive features that cannot be substantiated.
- **Severe loss in the original image**: References with clearly established person correspondence are needed. If there is insufficient evidence, complete the areas that can be handled reliably, preserve the baseline that has passed, and explain the remaining issues; do not treat a faceless patch of color, a grotesquely assembled face, or an obvious face swap as a finished image.

If the tool cannot directly guarantee the scope of local edits, place repair content that has passed inspection back onto the high-resolution original canvas when reliable compositing capabilities are available. If these capabilities are unavailable, explain the limitation; do not treat a prompt saying “leave everything else unchanged” as proof that scope protection has been implemented.

When compositing original-scan content back into the image to preserve identity, **first clean confirmed contamination from that content, then protect the continuous original structure**. Identity-protection areas and stain areas may overlap; do not let a protection mask reintroduce green contamination beside the ears, white spots on the chin, flaking on the arms, or similar defects. Simply lowering opacity, reducing color differences, or blurring a stain does not remove the defect; if its outline remains visible, continue repairing that specific defect. Do not use this as a reason to expand the work to redrawing the entire face.

**Inspect the material and its entire boundary before compositing it back, and check for regressions afterward.** First explain the specific structural differences that need to be restored; do not automatically paste back the entire original face or hand merely because the generated candidate is clearer. Material proposed for compositing back must have its contamination removed item by item; low-frequency color adjustment does not count as completed stain removal. Determine the scope along the actual structures requiring protection; do not use convenient rectangles, polygons, or ellipses that also cover background outside the head, neighboring objects, or skin that has not been cleaned. Check alignment at every overlap to avoid ghosting in fingers, beads, ear contours, and lip lines. List contamination within structural protection areas as separate repair items; do not preserve it along with the structure by lowering the weight assigned to stain removal.

If structure, texture, or clarity does not match, change the repair source, narrow the scope again, or abandon that compositing-back approach. Do not conceal seams by softening the entire image, increasing feathering, or blending in more of the original. For every compositing operation, examine the original scan, the candidate before compositing, and the result after compositing together; comparing only the original and final images will miss areas that had already been repaired at an intermediate stage and were then damaged again.

For fine, shallow scratches, establish small repair areas along the actual marks, preserving adjacent eyelids, mouth corners, ear contours, and jaw structure. First repair using clean neighboring texture of the same material or continuous tonal transitions, then check for any remaining linear bright edges; correcting only low-frequency skin color may leave high-frequency cracks, while retaining only the original's high-frequency information may also bring the cracks back. Do not remove line marks by smoothing an entire patch of skin or escalate local fine marks into redrawing complete facial features. Inspection must target the final composited image; clean generated material does not guarantee that the result remains clean after being composited back.

### Explicit Limits on Facial Treatment

- Do not beautify, slim the face, widen the face, or change facial-feature proportions, apparent age, or head orientation.
- Do not force the left and right eyes into symmetry, or fill in complete eyes that were originally obscured by bangs, glasses, or other objects.
- Do not copy the other eye to complete the facial features or assemble one face from different people's facial parts.
- Do not locally stretch, liquify, or distort the original face using dense deformation to accommodate newly generated facial features.
- Do not cut through and splice the same iris, lip line, or side of the nose, or overlay blurred structures from the original with sharp details from another mismatched face to create double contours.
- When a continuous structure needs repair, use consistent anatomical direction and lighting, placing transitions in surrounding areas that can blend naturally; feathering cannot conceal inconsistent internal structure.
- Identify source-image noise, scratches, and flaking edges first; do not bring them back into the restoration result as “authentic texture” or gradients.

## 4. General Prompt + Generate an Image-Specific Prompt by Inspecting the Image + Actual Execution

This section must be carried out before each actual restoration round. Use two layers of prompts: **G is the fixed general prompt below, which can be reused directly; Sₙ is the additional image-specific prompt for round n, written after inspecting this image; Pₙ is the full execution prompt used for actual execution, Pₙ = G + Sₙ.** The first round is denoted S₁, and revision rounds after review are denoted S₂, S₃, and so on. The round number does not specify how many times the image must be redrawn; stop once it passes review.

### 4.1 Fixed General Restoration Prompt G

The following code block is the complete general prompt that can be used directly for restoration. Retain these shared constraints in every round; put the facts specific to this image and this round's tasks in the additional Sₙ, rather than rewriting G as a few generic requirements.

```text
Please restore the same old photograph provided so that the result looks like the original photograph after cleaning, local repairs, and rescanning.
Execute this together with the image-specific prompt for this round provided afterward. The original scan and reliable copies of the same photograph are the basis for authentic content;
the current editing baseline is used to protect areas that have already been repaired. Annotated images only locate problems; the annotations themselves must not appear in the finished image.

Prioritize fidelity to each person's identity, facial features, and expression, while removing stains and scratches that can be reliably repaired.
Preserve each person's original eyes, eyebrows, nose, ears, cheeks, jawline, and chin outline, as well as their apparent age,
head angle, natural asymmetry, and the positional relationships between facial features. Do not redesign clear, intact facial features.
Do not beautify, slim or widen faces, or arbitrarily add eye bags, wrinkles, pupils, or other facial features.
Protecting these structures does not mean retaining contaminated pixels. Black spots on foreheads, discoloration beside ears,
and stains on skin and clothing that the user has identified or that have been confirmed must be removed locally; do not paste stains back in under the pretext of protecting the original face.

Use each person's own mouth in the original image as the basis for preserving mouth width, the relative thickness of the upper and lower lips, the Cupid's bow, the positions
and curvature of the mouth corners, the course of the line between the lips, how open or closed the mouth is, and the mouth's positional relationships with the base of the nose, philtrum, cheeks, and chin.
Do not substitute a generic mouth shape, artificially correct existing asymmetry, or create a crooked mouth or a new smile.
If the mouth is closed in the original, keep it closed; if teeth are visible in the original, preserve their visible extent, evidence-based arrangement, and natural brightness.
Do not arbitrarily add, remove, or whiten teeth, or make them sparse; do not artificially darken the lip line or arbitrarily change
natural lip color supported by evidence. Color differences caused by contamination or previous restoration may be corrected within the scope specified for this round.

Repair only actual damage, contamination, and missing content supported by evidence. First use adjacent visible structures and reliable references;
do not invent facial features that are clear but different. Do not splice in mismatched faces, eyes, noses, or mouths, or use stretching,
liquifying, forced symmetry, or misaligned overlays to accommodate new structures. Do not introduce duplicate facial features, distorted faces, or harsh seams.
Treat uncertain details conservatively, but do not conceal problems with blurred faces, pixelation, or smeared blocks of color.

Within the scope given in this round's image-specific prompt, completely remove stains, colored contamination, mildew spots,
scratches, cracks, fine colored lines, and flaking that actually exist. Treat continuous scratches along their entire paths; where they cross skin, hair,
clothing, background, and ground, restore each corresponding material while preserving real clothing seams, folds, and scene content.
After repair, leave no black lines, white lines, broken lines, repeated cloning patterns, smeared bands, bright edges, or local blocks of color.
Pay particular attention to low-contrast fine white lines, short cracks, and web-like abrasions on skin and object edges; these differ from normal
grain and must be removed locally along their actual paths. Do not remove only discoloration while retaining fine lines, use broad skin smoothing
to conceal fine marks, or reintroduce these contaminated textures while protecting existing structures.
Address every defect ID and user annotation for this round individually; do not merely fade contamination and then claim it has been removed.
When a quality reference is available, compare the degree of cleanliness, naturalness of edges, and consistency of sharpness, without borrowing its facial features.

Restore natural colors, soft tonal transitions, and moderate detail only within the scope required for this round, retaining slight photographic grain.
Skin tones should blend naturally with their surroundings; sharpness should be consistent between faces, hair, bodies, and background, without producing
plastic skin, excessive sharpening, strong HDR, or oversaturation. Do not retain stains as if they were real texture.

Except for changes explicitly authorized by the user, preserve the complete original frame, original aspect ratio, number of people, their positions, poses,
clothing, fingers, relationships between legs, shoe shapes, and background. Do not crop, extend the scene, squash, stretch, or invent text or decorations.
Modify only the defects specified for this round and the necessary transition areas, protecting areas that already pass review and areas not authorized for modification.

Use high-pixel-count processing methods that the tool genuinely supports, and follow the pixel dimensions, aspect ratio, and export plan specified for this round.
Preserve the high-resolution master canvas and lossless master file; do not replace a high-pixel-count original with a whole-image redraw at the default approximately 1K resolution,
or pass off an increased DPI tag or larger file size as recovered detail. Do not call a JPEG compatibility export lossless encoding.
Local tool outputs may serve as repair material; subsequent compositing and export ensure the complete frame and lossless master file.
Do not regenerate content outside the task scope simply to output the whole photograph.
This output is a candidate and must be checked against the original scan for the people, damage, modification scope, and actual export quality before delivery.
Pasting back or compositing must not reintroduce contamination, patch edges, double images, or blurred bands into areas that have already been cleaned.
The quality reference is for acceptance review only; do not copy its pixels, use it as the editing target, or use it as the result preview.
```

### 4.2 Inspect This Image and Generate the Additional Image-Specific Prompt Sₙ for This Round

Complete the original-image inspection, damage analysis, and method selection in Sections 1–3, read the dimensions and export rules in Section 5, and then carry out the following **prompt-generation instructions**. They guide the restoration agent in writing Sₙ; they are not to be sent verbatim to the image tool, and there is no need to find another model. Without actually seeing the image, do not invent damage or facial-feature analyses.

```text
Based on the original scan, current editing baseline, reliable references, quality reference, and the user's feedback for this round that you have just actually inspected,
write an additional image-specific restoration prompt Sₙ for this photograph. It will be appended to the fixed general prompt G
for the actual restoration immediately afterward. Do not repeat generic requirements for fidelity or high definition; specify exactly how to repair this image.

First distinguish between initial restoration and revision, and clarify the role of each input file. Identify people with numbers based on their positions in the image,
and separately record the facial features relevant to this round's restoration that can be confirmed from the original image, particularly mouth shape and facial contours.
Do not infer the original appearance from an existing generated version. Explicitly mark uncertain details as unconfirmable, without filling in imagined features.

List by area the problems that actually need to be addressed in this round. For each, specify its location, visible evidence, the start and end or extent of the damage,
the materials it crosses, the method to be used, and the expected improvement; for local revision, list only the problems still unresolved this time.
Reuse the IDs from the defect log; every stain, discoloration, and residual edge annotated by the user must have a corresponding task.
List major damage, local discoloration, and fine shallow scratches separately; for fine marks, clearly describe their paths and the surrounding real structures to protect.
In the next round, inspect again for damage revealed only after cleaning or color correction, rather than carrying forward only the problems found in the first round.
Describe the specific aspects of the result that still fall short of the quality reference; do not substitute "the original image pixels have been preserved" for the cleaning objective.
Specify the intact facial features and already acceptable areas to protect, as well as the permitted modification scope and necessary transitions.
When stains overlap protected facial-feature areas, explain how to preserve contours and positions while removing contamination; do not classify an entire
patch of stained skin as unmodifiable. Reinspect areas where the original image was previously pasted back for fidelity to determine whether damage was reintroduced.
If proposing a local redraw, explain why traditional repair is insufficient and what evidence supports the redraw; do not default to redrawing the entire face.

For revision, turn the failures found in the previous round's review into specific tasks: which part, where fidelity is lacking or
defects remain, what the original-image evidence is, and what method or scope will change this time. Remove resolved items from the repair list,
and add areas that have passed review to the protection list. If identity fidelity has deteriorated, use a reliable editing baseline after rollback.

State the original width and height actually read from the file, the original aspect ratio, the selected processing and export dimensions, DPI, format, and local
protection method; these must come from file information and tool capabilities. Do not invent dimension parameters or treat 4K as a guarantee.
Finally, list the specific checks this round's candidate must pass, corresponding item by item to this round's restoration tasks.
Output a complete image-specific prompt that can be used directly for the current photograph, without leaving fields to be filled in.
```

The generated Sₙ must contain at least the following; this specifies the output structure, and **section names or an empty template cannot be treated as an already generated image-specific prompt**:

1. **Round and inputs**: The actual filenames and roles of the original image, editing baseline, supporting images, and quality reference; this round's objectives and person IDs; the established project output root directory. If there are no reference images, write "None" rather than assuming they exist.
2. **Image evidence and identity constraints**: Confirmable features and unconfirmable details related to the damaged areas; describe each person separately for group photos, rather than merely writing "preserve the original face." When the mouth requires repair, state the evidence visible in the original image for its openness, mouth corners, line between the lips, and so on; do not invent precise anatomical descriptions from blurred pixels.
3. **Specific restoration tasks**: For each item, write "Defect ID and location → Evidence → Method → Expected change," including all user annotations, complete scratch paths, and treatment across materials. Coordinates or masks must come from actual inspection, rather than being invented; if only textual directions are provided, do not claim that a precise mask has been established.
4. **Protection scope and execution parameters**: Areas that may be modified and areas that must be protected in this round, actual pixel dimensions, and the export plan; the division of work between traditional operations and generative completion. Do not add whole-image beautification to a local revision that does not require whole-image color correction.
5. **This round's acceptance review and changes**: The removal criteria for each defect, the method for comparing the same locations, and the gap relative to the quality reference; for revision rounds, also state the reasons for the previous round's failures, this round's adjustments, and the rollback version. Merely saying "more faithful, higher definition, try again" does not constitute a new-round plan.

### 4.3 Combine Pₙ, Verify It, and Immediately Execute the Restoration

Combine the complete body of G and the Sₙ already written for this round, in that order, into Pₙ. Save a readable record in the process directory, for example `round-01.md`, clearly separating "General Prompt G," "Image-Specific Prompt S₁," and "Execution and Review Record." The first two sections together form the actual prompt for this round. If the text actually received by the tool is adjusted, also record the final text sent; do not pass off a formula as a record of the actual prompt.

Before execution, check whether Sₙ corresponds to this image and the current version, contains unfilled content or unsupported guesses, omits problems specified by the user, conflicts with G's fidelity constraints, or exceeds the local scope, and whether the planned dimensions are achievable. Explicit new user requirements take priority; record any related scope changes. Sₙ cannot independently cancel identity protection using reasons such as "enhance detail." If problems are found, correct Sₙ before executing.

- **Image editing tools**: Actually submit the complete Pₙ, attaching the correct original image, editing baseline, and necessary reference images; identify each input's corresponding role in the prompt. Do not send only G, send only Sₙ, or merely display the prompt in chat without calling the tool. If the work is divided into multiple local calls, every call must include G and the image-specific instructions applicable to that area, use the same original-image evidence, and record its corresponding inputs and results.
- **Traditional photo editing tools**: Still generate G + Sₙ first, then translate Sₙ into a concrete list of operations for the current file, such as what kind of cleaning, local color adjustment, repair, and protection to apply in which area. Actually execute the operations and save the image; do not force traditional photo editing to become generative redrawing.

When the user has already requested restoration and there is sufficient evidence, continue with execution after writing the prompt, without adding a confirmation checkpoint asking "whether to use this prompt." If a tool does not support a parameter, use a verifiable approach and record the limitation; do not pretend in the prompt that the tool has that capability.

### 4.4 Obtain a Candidate, Then Proceed to Review and the Next Round

After execution, read the actual candidate image, confirm that the file opens, and record its real width and height, inputs, tool, and method. If there is no image output or the call fails, the round is incomplete and cannot be described as having restored the image. A candidate automatically displayed by the tool must be labeled as awaiting review, rather than initially called a finished product.

Next, carry out the review in Section 6 and record the itemized results as Cₙ. When there are failed items that can be located, roll back or retain acceptable areas according to Section 7, return to 4.2, and regenerate Sₙ₊₁ based on **the original image + a reliable editing baseline + Cₙ + new user feedback**. Then combine a new Pₙ₊₁ and actually perform the revision. Do not skip generating a new targeted prompt, or simply give the tool the previous round's failed version with "continue optimizing."

## 5. High-Pixel-Count Processing and Canvas Constraints

1. **Work on the complete high-resolution master canvas**. Use the original width and height `W×H`, with the correct display orientation, as the baseline, and record the aspect ratio. Previews may be reduced, but the final image must not become approximately 1K because of tool defaults.
2. First check the dimension parameters and limitations actually supported by the tool, and actively select a higher native resolution when available; after the call, read the output file's actual width and height to confirm, rather than relying on the words "4K" in the prompt.
3. **Not falling below the original image's pixel dimensions is a hard minimum; 4K or higher is the preferred goal when the tool supports it**. When capabilities permit, the target long edge must be at least `max(original image long edge, 3840)`; retain larger dimensions when the original is larger, and increase them when doing so has practical value. Do not force all photographs to 3840×2160, crop edges, or fabricate image content to reach a particular size.
4. Use the same scaling factor in both directions, allowing at most a 1-pixel discrepancy caused by rounding to integer pixel values. For example, a 3:2 photograph may be exported at 3840×2560 and a 2:3 portrait-orientation photograph at 2560×3840; an original at 6000×4000 must not be reduced to those dimensions.
5. When the generative tool's output is insufficient to support the entire original photograph, preferentially use it as repair material for the corresponding local area, preserving the remaining original pixels on the master canvas. Do not simply enlarge a redrawn 1K whole image and then claim that the high-definition original has been restored.
6. When the original image is itself smaller than the target, use proportional upscaling suitable for preserving fidelity, prioritizing the avoidance of newly generated facial structures; inspect facial features and textures again after upscaling. Record the native processing resolution and final export resolution, making clear that upscaling does not mean recovering details that did not originally exist.
7. When the tool genuinely does not support the high-pixel-count target, retain the highest-quality file at no less than the original dimensions and explicitly state the limitation; do not fake quality by increasing DPI, file size, or adding noise. If the user has specified 4K as mandatory, mark "dimension requirement incomplete" when it is not met; if the user only requested 4K preferentially when supported by the tool, delivery is allowed when all other acceptance checks pass and the original pixels are fully preserved, but the actual resolution must be disclosed. When the tool clearly supports a higher output resolution, do not use the default approximately 1K output instead of actively choosing a high pixel count.

### DPI, Color, and Encoding

- Treat pixel dimensions, real visible detail, and DPI separately. DPI is primarily metadata for print scale; increasing the tag alone does not add detail.
- Determine DPI from valid file tags; do not treat a default 72/96 returned by an image library as actual stored scan precision. For example, when the JFIF unit is 0, density indicates only a ratio. When tags are missing or cannot be confirmed, record that truthfully, then export using the missing-value rule.
- Record valid horizontal and vertical DPI separately; by default, set each to the greater of its original value and 300, using 300 when missing or invalid. Allow minor rounding errors caused by the format's storage precision, and do not reduce valid high DPI without reason. When the user specifies a higher print DPI, values such as 600 or 800 may be set, with an explanation that this is an output setting, rather than passing it off as the original scan's sampling precision. Do not reduce pixel dimensions when adjusting DPI.
- Preserve the original color profile and bit depth when they can be handled correctly; when color-space conversion is needed, convert correctly rather than merely changing the tag and causing a color shift. Do not lower the original bit depth without reason.
- In this workflow, "no compression" means no downsampling, no lossy compression as the sole means of saving, and no repeated encoding. Reversible compression in PNG or lossless TIFF does not lose pixels; if the user explicitly specifies an uncompressed file, use a tool-supported format such as uncompressed TIFF.
- Always retain a lossless master file. JPEG is not a lossless format and cannot be called lossless even at quality 100; encode only once at final export, using the tool's supported high-quality settings and 4:4:4 sampling, and avoid repeatedly saving back to JPEG.
- Saving an original JPEG as PNG only prevents subsequent encoding losses and cannot reverse existing JPEG losses; a larger file does not automatically mean more real detail.

## 6. Checks Required in Every Round

Check the candidate first, and consider export only after it passes. The review must always include the original scan, the currently approved baseline, and this round's candidate; do not compare the candidate only with the previous generated version.

### A. Check Each Person's Identity and Facial Features

Create comparison crops at the same proportional scale, orientation, and location. Check each person separately; do not overlook others because one person's face passes.

| Check | What Must Be Verified |
| --- | --- |
| Mouth and philtrum | Width, upper and lower lip thickness, Cupid's bow, height/curvature of the mouth corners, direction of the line between the lips, degree of opening, and asymmetry; any crooked mouth, abnormally dark lines, blurred patches of color, or mosaic artifacts |
| Teeth and expression | The visible extent and natural brightness of the original teeth; whether teeth, a smile, or changes to the mouth have appeared without evidence |
| Eyes and eyebrows | Eye positions, length and width of the eye openings, degree of opening, visible evidence for gaze direction, obstruction by bangs, and eyebrow shape; any abnormal difference in eye size, duplicated pupils, under-eye bags, or excessive symmetrization |
| Nose and ears | Nasal bridge, nasal wings, nostrils, ear positions, and visible contours; whether the nose has been replaced with a high, narrow one, scratches remain on the nose, or ear structures absent from the original have been added |
| Cheeks and chin | Face width, contours, cheekbone area, jawline, chin shape, head angle, and apparent age |
| Connections between facial features | The positional relationships among nose, mouth, and chin; any ghosting, misalignment, twisted facial structures, or a sense of parts spliced together from multiple sources |
| Outer head and face | Continuity and tones across hair–forehead, cheek–ear, and chin–neck–collar transitions; do not check only the interiors of the eyes, nose, and mouth |

Similarity scores or landmark detection can only assist review. They cannot replace comparison with the original image or decide on the user's behalf whether a person resembles their relative. Explicitly retain uncertainty for details that lack sufficient evidence in the original image.

Before claiming that “facial features or object shapes have changed,” verify this with native-pixel crops of the same location, distinguishing uncertainty in the original's blurred boundaries from actual geometric changes. Inferring that the mouth has grown or the eye shape has changed merely because the candidate's edges are clearer causes repeated redrawing without evidence. When opinions conflict, return to the original for verification. If evidence is insufficient, do not force changes to structures that are already continuous and natural; confirmed contamination must still be removed.

### B. Check Whether Damage Has Actually Been Removed

- **Close each item individually**: For every ID in the defect log, actually view before/after crops of the same location and record the result. Do not skip any area marked by the user; “Not checked,” “Awaiting evidence,” “Only faded,” and “Residue remains” are all failures to pass. Check original-image features and the quality reference separately; facial similarity cannot compensate for inadequate stain removal.
- Follow the entire scratch path using the area checklist, including easily missed curves near fingers, trousers, shoes, and floor tiles; do not repair only the middle segment.
- Check the edges, top, bottom, and dark areas for mold spots, white flaking, and fine colored lines; separately check for residual colored stains.
- For each person, check the forehead/hairline, around both ears, chin/neck, both hands/forearms and sleeves, and the boundary between the person and the background. Pay particular attention to areas pasted back from the original and mask edges, avoiding a situation where “major damage is repaired, but small stains are pasted back in.”
- **Add a fine-mark recheck**: After treating major stains and scratches, inspect each face section by section along both cheeks, in front of the ears, outside the mouth corners, and along the jawline. Check for low-contrast short white lines, networks of fine cracks, pale traces formed by strings of grain, and patch edges. Confirm continuous paths using native-pixel crops of the same location and moderate magnification; if necessary, create a separate diagnostic preview with enhanced local contrast. Do not use the diagnostic preview to overwrite the finished image directly. Interpolation textures created solely by image enlargement are not evidence of damage, and natural skin texture, contour highlights, and expression lines must not be mechanically removed.
- If the final result uses local compositing, pasting back from the original, sharpening, or enlargement, recheck every fine-mark ID in the **actual composited and exported files**. Assess residual marks, naturalness, and facial structure separately; continue rework if any of them fail. “Very slight” and “Nothing outside the marked area changed” cannot substitute for closing a defect specified by the user.
- Distinguish removable spots from normal grain, avoiding both “claiming restoration is complete while noise has not been removed” and smoothing normal texture into plastic.
- Repaired areas must not retain black lines, white lines, broken lines, blurry smeared bands, repeated clone patterns, bright edges, dark rings, or obvious color patches.
- When a user-approved quality reference is available, inspect corresponding types of areas at comparable display magnifications: whether contamination has disappeared, whether skin tones and edges are natural, and whether clarity is consistent. Different original resolutions cannot be expected to produce identical real detail from nothing, but this limitation cannot justify retaining repairable stains.

### C. Check Consistency between the Full Image and Local Areas

First view the complete image fitted to the screen, then inspect faces, repair edges, and major textures at 100% native-pixel scale. If necessary, use approximately 200%–400% magnification to help locate seams, but do not infer that fine structures existed in the original from details produced by interpolated enlargement.

Verify the number of people, their orientation, clothing types and colors, fingers and gestures involving an arm draped over a shoulder, seated postures, overlapping legs, shoe shapes, and background objects. Faces must not be noticeably blurrier than their surroundings or so sharp that they look pasted on; skin tones should transition naturally to the neck, ears, and arms.

### D. Verify the Scope of Changes

When pixel analysis is available, compare the candidate with the editing baseline using the same dimensions, orientation, color interpretation, and encoding baseline. Both the local mask and its feathered band belong to the permitted modification area; pixels outside the mask in the lossless master file should remain unchanged.

If this round truly requires global color correction, proportional enlargement, or color-space conversion, record that global transformation first, then verify the local scope against a common processing baseline. Do not simultaneously claim “global color correction” and “all original pixels outside the marked area are completely unchanged.” Re-encoding the full image as JPEG also precludes a strict promise of identical pixels outside the marked area.

When the user says “keep everything else unchanged,” protect its content, geometry, and approved details, and preserve pixels in the lossless master file wherever possible. If the user explicitly requires every pixel outside the marked area in the final JPG to remain identical, but the current encoder cannot achieve this, state that format limitation directly and retain a verifiable lossless master file. Do not misrepresent ordinary JPEG encoding as satisfying that condition.

Do not claim “pixels outside the marked area are unchanged” without pixel verification. Even if verification finds zero changes, still check the interior of the face for fidelity and naturalness.

### E. Check Whether This Round Damaged Previously Accepted Areas

Create a review coverage table: use **all actual operations and masks from this round** to list their interior areas, complete boundaries, and adjacent transition areas, then add all unresolved and previously resolved defects. Do not crop only the small area of the most recent rework; tight crops around the eyes, nose, and mouth also cannot replace checks outside the top of the head, of both hands, and of surrounding objects.

At the same orientation and magnification, compare the original scan, the candidate before compositing, the result after compositing, and the actual exported file. For each area, separately record identity/structure, cleanliness, clarity, tone, and seams. Check the affected area after every compositing operation, paste-back from the original, denoising, sharpening, and enlargement. New patches, boundaries, ghosting, or abnormal blur in a previously repaired area constitute a regression; improvements elsewhere, a lossless file, or zero changes outside the mask cannot offset it.

If a regression is found, reject the operation that caused it, return to the reliable baseline preceding that operation, and change the method to address the cause. Record the specific area and before/after evidence; keep unviewed items marked “Pending review.” Passing spot checks, another agent's pass conclusion, or “it looks clean overall” cannot replace item-by-item verification. When the user identifies a new defect, recheck the actual image and revoke any conflicting previous pass conclusion.

## 7. Rework Loop and Rollback after Failure

Follow this process instead of delivering directly after a single generation:

```text
Original scan analysis + editing baseline + defect list
    → Read the fixed general prompt G
    → Additionally generate image-specific prompt S₁ from the evidence in this image
    → Combine and record P₁ = G + S₁; verify inputs, scope, and parameters
    → Actually perform the restoration or corresponding traditional operations; save the candidate
    → Compare with the original image (and any quality reference supplied for this request): identity + individual closure of defects + full-image/scope checks; record C₁
        ├─ Identity and stain removal both pass, with no open items in the log: export, then check the actual exported file
        ├─ Problems that can be located: preserve accepted areas and regenerate S₂ based on C₁
        └─ Identity regression or structural abnormalities: reject the candidate, return to a reliable baseline, adjust the method, then generate S₂
    → Verify and execute P₂ = G + S₂ → Review to obtain C₂
    → Continue generating the next round's Sₙ as needed until the result passes or a stopping condition is triggered
```

- G provides the constraints shared by every round; Sₙ defines the actual task for that round. Every round must complete “view the image and analyze feedback—generate Sₙ—combine Pₙ—execute—review Cₙ.” Do not repeatedly generate images with the first round's prompt unchanged. After the checks pass, do not generate unnecessary rework prompts.
- Advance only confirmed issues in each round; do not redo areas that have already passed. A failed candidate must not automatically become the facial ground truth for the next round.
- When the user identifies remaining damage again, change the corresponding defect from “Resolved” back to “Unresolved,” revoke the affected overall acceptance conclusion, and generate the next round's Sₙ from the new feedback. Verification must inspect the actual image; do not simply carry forward a conclusion because “Pass” was written in the previous round.
- For severely distorted earlier versions, first locate the earliest reliable version and the original scan. Do not keep layering changes onto a crooked mouth, a spliced face, or distorted structures.
- The “most recent accepted version” may be determined by the relevant area. If no historical full image ever passed in its entirety, protect its verified areas and continue using the original scan to constrain areas needing repair; do not relabel a problematic full image as an accepted final result.
- For every round, record the actual G/Sₙ prompts, input versions, specific problems and evidence locations, method used, candidate path, review Cₙ, and the decision to retain or reject. The next round's Sₙ₊₁ must address every failed issue in Cₙ and change the actual method or scope; do not merely change a sentence in the prompt and repeatedly generate images.
- If there is no visible improvement on the same issue for two consecutive rounds, stop repeating that approach and switch to another reasonable method or request the missing reliable reference. Immediately reject obvious face replacement, grotesque splicing, duplicated facial features, or abnormal distortion; do not wait until all rounds have been used.
- By default, each request permits at most three rounds of generative candidate redrawing; necessary traditional local stain removal, color correction, and review may continue. If the result still fails after reaching the generation limit, retain the most recent accepted version and clearly identify unresolved items and the evidence needed; do not force delivery of the last result. Targeted experiments may continue when the user explicitly authorizes more attempts.
- Stop when the requirements are met; do not keep altering facial features merely to fill a round quota. Use “Completed/Restored” as the result status only when identity preservation, every specified defect, the applicable outcome requirements for this request, and export checks all pass together. If any acceptance items remain Pending work, Pending review, Awaiting evidence, or Unresolved, save the result as a candidate awaiting acceptance review; do not deliver prematurely on the basis that “it is mostly fine overall.”
- If insufficient evidence or tool limitations prevent all requirements from being met together, mark the result “Incomplete/Reference needed.” Partial results with real value may be delivered, but their scope must be described accurately. Do not present disturbing trial versions as finished work, or say “Completed” while supplying only an explanation or a copy of the original image.

## 8. Naming, Saving, and Delivery

Prioritize the user's specified name, format, and location, and choose the output root directory according to the current project's purpose:

- **Skill development, trials, or regression verification**: Create an independent verification workspace outside the Skill delivery directory. Within it, `fixtures/` holds necessary copies of test inputs, `runs/<case-or-version>/` holds candidates and process files, and `outputs/` holds only test results that have passed acceptance review. Do not distribute these private test materials with the Skill. Treat external original scans and optional references as read-only, and do not write into their directories; saving conventions from previous production restoration projects do not automatically carry over to Skill testing.
- **Ordinary photo restoration**: If the user has not specified a directory, save in a separate output directory for the current restoration task and report its path. If a writable task directory cannot be determined, use a newly created dedicated restoration-output subdirectory under the input directory. Do not scatter process files throughout the user's existing photo directory.
- Before running scripts, generating outputs, or exporting, resolve the actual output paths to absolute paths and confirm that every candidate, comparison, record, and finished image is within the output root selected for this request. Check hard-coded paths in old scripts; referencing a photo in a directory does not authorize writing new files there. After changing directories, update the scripts' output paths accordingly; do not move or delete existing user files in bulk.

By default, insert a restoration suffix between the base filename and the extension: use `_已修复` for a Chinese workflow and `_restored` for English delivery names. For an initial restoration, use the main original image's base filename; for rework, use the base filename of the editing target specified by the user for this request. Do not use an auxiliary reference's name as the output base. Handle only the final extension, preserving dots within the base filename. If a rework file already contains a restoration suffix, do not append it again; use a version number such as `_v02` or the copy name specified by the user.

- PNG input: default to `original-name_已修复.png`.
- JPEG input: provide `original-name_已修复.jpg` as the compatible finished image, together with `original-name_已修复_无损.png` or a lossless TIFF master file.
- TIFF input: retain TIFF and choose lossless saving when reliably supported by the tools. If other formats cannot be reliably written back, export an available lossless format and state the actual encoding.
- Do not disguise a format merely by changing the extension. When a name conflicts, create an explicit version; do not overwrite user files by default.
- Name trials that have not passed acceptance review with terms such as “Candidate/Incomplete,” store them in the process directory, and do not label them as final restored versions.

When the user explicitly accepts only one format, deliver that format as requested. The lossless master file used to avoid repeated lossy encoding and support review may remain in the process directory; do not substitute another extension for the user's specified delivery format without authorization.

Reopen the actual exported file and confirm that it decodes normally, its extension matches its actual format, its pixel dimensions and original aspect ratio are correct, its DPI is correct, its colors display normally, and its faces and edges have not been damaged by export compression or color conversion. When necessary, recheck the local modification scope in the lossless file and verify that the original file has not been overwritten.

**The preview and downloadable file must correspond to the same final version.** Export PNG and JPG from the same checked master file, and record the paths, dimensions, and hashes of the master and exported files. Read the actual delivery file directly for the final preview: prioritize showing the lossless master when it is delivered; show the JPG when only JPG is delivered. If the interface requires a thumbnail, generate it only by proportionally reducing the corresponding delivery file, and identify its source. Also extract review and comparison crops from the actual delivery file. A generated candidate automatically displayed earlier by a tool cannot serve as the preview of the subsequently composited finished image; explicitly show the actual file in the final response instead of reusing a candidate or reference image. If further edits are made after export, re-export, recheck, and update all links and previews together.

Finally, provide:

1. The actual saved path/link for the complete finished image and the lossless master file; if the user specified JPG, do not deliver only PNG.
2. Proportionally matched original/before-restoration/after-restoration comparisons of faces or key reworked areas; comparison images cannot replace the finished image.
3. A traceable record: the general prompt G actually used, the additional image-specific prompt Sₙ generated for each round and the combined full execution prompt Pₙ, the corresponding traditional operation list (if any), input and candidate versions, review conclusions, native processing and export pixel dimensions, DPI, and details that still cannot be verified. The full prompt text may be saved in the process record and linked; the delivery description should briefly state the changes and acceptance review results for this request, without repeating lengthy prompt text in the chat.

Begin the response with the changes actually completed and the file locations, then briefly state the review results. Report tool failures, rejected candidates, or incomplete work truthfully; do not use unverifiable claims such as “absolutely faithful,” “100% restored,” or “losslessly reconstructed every missing facial feature.”

## 9. Development Verification and Conditions for Finalizing the Skill (Only When Improving This Skill)

After improving the Skill, actually complete “original-image analysis → general and image-specific prompts → actual restoration → review → targeted rework → export recheck” in an external verification workspace. Text review, format validation, and identifying failures in an earlier version cannot replace this end-to-end execution.

**A complete regression run starts from the original scan.** In a new, independent run directory, reread the original file, build the defect log, generate G + S₁, and actually execute it. Previous finished images, intermediate images, repair material, and masks must not be used as pixel inputs, the editing baseline, or the initial candidate for the new run; they may only help compare before/after results during acceptance review. Generic tool code unrelated to the photo may be reused, but the damage scope and this round's parameters must be determined afresh from the original scan. User feedback may help identify review priorities, but cannot replace checking the entire original image from the start. Candidates newly produced from the original in this run may continue through the normal rework loop within the same run.

External verification records should retain this Skill version or hash, the original scan identifier, the actual input list for every operation, prompts and tool operations, candidates, before/after evidence, and acceptance review conclusions, so that the absence of old restoration material from the inputs can be verified. After changes to rules affecting restoration or acceptance review, rerun the complete process with the new version. Repairing only an old result can validate a local operation, but cannot demonstrate the new Skill's end-to-end effectiveness; merely giving an old result the new version's name does not count as verification either.

Developers may use authorized private samples and approved results to assess quality, but should distill the experience into general rules. Do not include that photo's names, filenames, locations, coordinates, or access requirements in the Skill. Ordinary users must be able to start by providing only their own photo; optional references must not become implicit requirements. During verification, do not directly copy or rename an approved sample or transplant its face to pass it off as a new result produced by the process.

Mark the current Skill “Verification passed for this round” only when the actual result simultaneously meets the requirements for identity fidelity, removal of clear stains/scratches, natural and continuous repairs, consistent clarity, original aspect ratio, and high-quality export, with no user feedback still awaiting resolution. If a failure is found, save specific evidence, revise the corresponding rules and actual method, then rerun the revised version completely from the original scan and compare; do not declare the Skill finalized after changing only its instructions. When the user supplies an outcome standard, the actual result must reach that standard before entering acceptance review. Do not pass the test by feeding the reference image into the tool, transplanting from it, or saving it as a new result. Report the scope of the samples verified; do not claim that all old photos will necessarily succeed because one image passed. Record internal review passes separately from user approval; reopen verification if the user's feedback still reports problems, and do not assume that the user has approved the final version.
