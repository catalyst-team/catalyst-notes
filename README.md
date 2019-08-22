<div align="center">

![Catalyst logo](https://raw.githubusercontent.com/catalyst-team/catalyst-pics/master/pics/catalyst_logo.png)

**Catalyst info**
 
[![Pipi version](https://img.shields.io/pypi/v/catalyst.svg)](https://pypi.org/project/catalyst/)
[![Docs](https://img.shields.io/badge/dynamic/json.svg?label=docs&url=https%3A%2F%2Fpypi.org%2Fpypi%2Fcatalyst%2Fjson&query=%24.info.version&colorB=brightgreen&prefix=v)](https://catalyst-team.github.io/catalyst/index.html)
[![PyPI Status](https://pepy.tech/badge/catalyst)](https://pepy.tech/project/catalyst)
[![License](https://img.shields.io/github/license/catalyst-team/catalyst.svg)](LICENSE)

[![Build Status](https://travis-ci.com/catalyst-team/catalyst.svg?branch=master)](https://travis-ci.com/catalyst-team/catalyst)
[![Telegram](./pics/telegram.svg)](https://t.me/catalyst_team)
[![Gitter](https://badges.gitter.im/catalyst-team/community.svg)](https://gitter.im/catalyst-team/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![Slack](./pics/slack.svg)](https://opendatascience.slack.com/messages/CGK4KQBHD)
[![Donate](https://raw.githubusercontent.com/catalyst-team/catalyst-pics/master/third_party_pics/patreon.png)](https://www.patreon.com/catalyst_team)


</div>

**Catalyst-info** is a series of posts about the structure of [Catalyst library](https://github.com/catalyst-team/catalyst) and about its [ecosystem](https://github.com/catalyst-team)!

## #1
catalyst-version: `19.08.6` date: `2019-08-22`

### Hello, everyone!

After the release of Catalyst in February it has a lot of new features, which, unfortunately, not everyone still knows about. Finally, we came up with an idea to post a random fact about catalyst every. So, the first release of catalyst-info!

---

In Catalyst we all have implemented our favorite Unet's: 
`Unet`, `Linknet`, `FPNUnet`, `PSPnet` and their brothers with resnet-encoders 
`ResnetUnet`, `ResnetLinknet`, `ResnetFPNUnet`, `ResnetPSPnet`. 
Any `Resnet` model can be fitted with any pre-trained encoder (resnet18, resnet34, resnet50, resnet101, resnet152)

Usage
```python
from catalyst.contrib.models.segmentation import ResnetUnet # or any other
model = ResnetUnet(arch="resnet34", pretrained=True)
```
It's easy to load up a `state_dict`
```python
model = ResnetUnet(arch="resnet34", pretrained=False, encoder_params=dict(state_dict="/model/path/resnet34-5c106cde.pth")
```
---

[Link to the model's code.](https://github.com/catalyst-team/catalyst/tree/master/catalyst/contrib/models/segmentation)

All models have a common general structure `encoder-bridge-decoder-head`, 
each of this part can be adjusted separately or even replaced by their own modules!
```python
# In the UnetMetaSpec class
def forward(self, x: torch.Tensor) -> torch.Tensor:
    encoder_features: List[torch.Tensor] = self.encoder(x)
    bridge_features: List[torch.Tensor] = self.bridge(encoder_features)
    decoder_features: List[torch.Tensor] = self.decoder(bridge_features)
    output: torch.Tensor = self.head(decoder_features)
    return output
```

To bolt your model as an encoder for segmentation, you need to inherit it from 
`catalyst.contrib.models.segmentation.encoder.core.EncoderSpec` ([Code](https://github.com/catalyst-team/catalyst/blob/master/catalyst/contrib/models/segmentation/encoder/core.py#L11)).

---

When creating your own block (for any `encoder/bridge/decoder/head`) using the [function](https://github.com/catalyst-team/catalyst/blob/master/catalyst/contrib/models/segmentation/blocks/core.py#L10) `_get_block` 
you can specify the `complexity` parameter, which will create a sequence of [complexity times by](https://github.com/catalyst-team/catalyst/blob/master/catalyst/contrib/models/segmentation/blocks/core.py#L34) `Conv2d + BN + activation`

---

The Upsample part can be specified [either by interpolation or by convolution](https://github.com/catalyst-team/catalyst/blob/febcb66ade07b231348fd8e19614bdd37d548125/catalyst/contrib/models/segmentation/head/unet.py#L18).
