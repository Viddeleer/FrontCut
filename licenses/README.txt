Third-party licenses and notices for withoutbg.exe

Built with DINOv3.

Component                        License                File(s)
-------------------------------  ---------------------  ------------------------------------------
withoutBG Open Weights model     Apache-2.0 + DINOv3    withoutBG-LICENSE.txt, withoutBG-NOTICE.txt,
  (the three .onnx files)        License                withoutBG-THIRD_PARTY_NOTICES.md, Apache-2.0.txt
DINOv3 (Meta Platforms)          DINOv3 License         DINOv3-LICENSE.md
Depth Anything V2 small          Apache-2.0             Apache-2.0.txt
BiRefNet / BiRefNet_HR           MIT                    BiRefNet-LICENSE.txt
  (ZhengPeng7; birefnet-general.onnx in the withoutBG bundle; optional
   BiRefNet_HR-general-epoch_130.onnx from github.com/ZhengPeng7/BiRefNet releases v1)
Resized models (in memory)       as their source        (see withoutBG rows above)
  (withoutbg.exe rebuilds the withoutBG backbone and matting graphs for a 1344 input and extracts
   the Depth Anything V2 subgraph for a 1288 input at run time; no modified model file is
   distributed)
ONNX Runtime (Microsoft)         MIT                    onnxruntime-LICENSE.txt,
                                                        onnxruntime-ThirdPartyNotices.txt

Sources:
  https://withoutbg.com/open-model/license
  https://huggingface.co/withoutbg/withoutbg-openweights-onnx (LICENSE, THIRD_PARTY_NOTICES.md)
  https://github.com/facebookresearch/dinov3/blob/main/LICENSE.md
  https://github.com/ZhengPeng7/BiRefNet/blob/main/LICENSE
  https://www.apache.org/licenses/LICENSE-2.0.txt
  onnxruntime-win-x64-1.23.2.zip (LICENSE, ThirdPartyNotices.txt)
