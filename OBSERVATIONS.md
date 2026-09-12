# Observations

## Project

This project fine-tunes `Qwen/Qwen3-1.7B-Base` as an IT support-ticket router using LoRA and LLaMA-Factory.

The source dataset contains 585 tickets across seven categories:

| Category | Examples |
| --- | ---: |
| Support general | 150 |
| Fileservice | 138 |
| O365 | 92 |
| EOL | 58 |
| Software | 58 |
| Active Directory | 48 |
| Computer-Services | 41 |

The training split contains 467 examples. The held-out `val_split.csv` contains 118 examples.

## Environment observations

- The training environment was macOS with Apple MPS available.
- CUDA was unavailable, so CUDA-specific acceleration and 4-bit bitsandbytes quantization were not used.
- LoRA training completed successfully with the Qwen3 1.7B base model.
- The training configuration used no quantization, batch size 2, gradient accumulation 8, and 3 epochs.
- The completed run reported 90 optimization steps and 8,716,288 trainable parameters.

## Training and inference observations

1. The initial `default` chat template caused generated text to include formatting such as `Human:` and `Assistant:`.
2. Switching to the Qwen-specific `qwen3_nothink` template produced cleaner Qwen-formatted prompts.
3. The model sometimes produced the correct category as its first label. For example, Outlook-related tickets produced `O365`, and a label-printer ticket produced `Computer-Services`.
4. The Chat tab also generated unwanted continuation tokens, including Chinese characters and text such as `user`.
5. Some prompts produced an incorrect first label, so the current Chat output should not be treated as a measured accuracy result.
6. The training run did not report validation metrics because validation size was set to zero during training.

## Conclusions

The end-to-end workflow works: the dataset can be prepared, the LoRA adapter can be trained, the model can be loaded, and predictions can be generated. However, the current model needs formal evaluation before it can be considered a reliable classifier.

The Chat tab performs open-ended text generation; it does not constrain the output to the seven category labels. For a production router, the output should either be constrained during inference or post-processed to keep only a valid category.

## Recommended next steps

1. Evaluate all 118 records in `val_split.csv` and report accuracy, per-category precision/recall, and a confusion matrix.
2. Repeat training with a non-zero validation split so evaluation loss is recorded.
3. Compare the Qwen-specific template and training settings using the same validation set.
4. Add constrained decoding or a small output parser that returns only one of the seven allowed labels.
5. Keep local model checkpoints, caches, virtual environments, and the cloned LLaMA-Factory source outside the GitHub repository.
