---
base_model: amazon/chronos-2
library_name: peft
tags:
- base_model:adapter:amazon/chronos-2
- lora
- time-series-forecasting
---

# Chronos-2 LoRA adapter for the Tennessee Eastman Process

LoRA adapter for [`amazon/chronos-2`](https://huggingface.co/amazon/chronos-2), fine-tuned to forecast the 41 measurements and 12 valve commands of a simulated Tennessee Eastman Process during set point changes, with the 11 changed set points as known covariates. This is case 6 of the TSFM-TEP repository, made with `TEP-Chronos2-FineTune.ipynb` and evaluated with `TEP-Chronos2-FineTune-Compare.ipynb`.

## Training

- Data: 3 simulation runs (`tep_sp_finetune_noisy.h5`, `tep_sp_finetune_seed1001_noisy.h5`, `tep_sp_finetune_seed1002_noisy.h5`; 1000 h + 500 h + 500 h at 6 min), each with its own set point schedule, AR(1) disturbances and measurement noise. The test data of the article is not used.
- LoRA: r = 8, alpha = 16, on the attention q/k/v/o projections and the output layer.
- Context 2048, horizon 256, learning rate 2e-5, 4 windows per optimizer step.
- Validation: the last 25% of every run (34 forecasts of 256 samples). Training stopped at step 1400 after 8 validations without improvement; these are the weights of the best step (1000).

## Use

```python
from chronos import Chronos2Pipeline

pipeline = Chronos2Pipeline.from_pretrained("chronos-2-tep-sp-3runs-ctx2048-lora-final", device_map="cuda",
                                            import_allowlist=["chronos.chronos2.model"])
```

The adapter is only meant for data from this simulator and these set points.
