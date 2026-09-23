# models/

**This directory intentionally contains no model weights.**

Trained weights are large binaries and must never be committed. They are
written to Google Drive instead:

```
/content/drive/MyDrive/BSM_Mineral_Classification/models/
├── VGG16_best.pth
├── ResNet18_best.pth
├── ResNet34_best.pth
└── D_ResNet_best.pth
```

## Checkpoint policy

- **Selection metric:** best **validation** accuracy.
- The test set is never used for selection, early stopping, or any other
  decision. It is evaluated exactly once, after training is complete.
- `D_ResNet_best.pth` holds a torchvision **ResNet50** with a 6-class head —
  the documented practical approximation of the paper's D-ResNet, not the
  original D-ResNet. The checkpoint filename preserves the paper's model name;
  the approximation is disclosed here, in the notebook, and in the README.

## Recovering a model

```python
import torch
from torchvision import models

net = models.resnet50(weights=None)
net.fc = torch.nn.Linear(net.fc.in_features, 6)
net.load_state_dict(torch.load(PATH, map_location="cpu"))
```

A minimal per-model metadata sidecar (`<model>_best_meta.json`) is saved next to
each checkpoint in Drive with the class order, image size, normalisation and
best epoch, so a checkpoint can never be loaded against the wrong label order.
