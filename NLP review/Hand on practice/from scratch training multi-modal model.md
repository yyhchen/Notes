
---

## 优雅下载huggingface模型方法


>[!TIP]
>**先安装依赖**:
>`pip install -U huggingface_hub`
>
>**下载模型**：
>`huggingface-cli download <model name> --local-dir <local path>`
>如： `huggingface-cli download Qwen/Qwen2.5-7B-Instruct --local-dir /home/model/Qwen/Qwen2.5-7B-Instruct`
>
>**下载数据**:
>```bash
>huggingface-cli download --repo-type dataset lavita/medical-qa-shared-task-v1-toy
>```
>
>[参考资料 知乎](https://zhuanlan.zhihu.com/p/663712983)






>[!NOTE]
>**数据集**： [LLaVA-CC3M-Pretrain-595K](https://huggingface.co/datasets/liuhaotian/LLaVA-CC3M-Pretrain-595K), 
>	   [minimind-v_dataset](https://huggingface.co/datasets/jingyaogong/minimind-v_dataset/tree/main) 
>	   [Chinese-LLaVA-Vision-Instructions](https://huggingface.co/datasets/LinkSoul/Chinese-LLaVA-Vision-Instructions)
>
>
>**底座模型**：[Qwen2.5-0.5B-Instruct](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)
****



具体多模态的训练细节，可以参考 blog [多模态模型训练总结](https://yyhchen.github.io/2025/04/14/%E5%A4%9A%E6%A8%A1%E6%80%81%E6%A8%A1%E5%9E%8B%E8%AE%AD%E7%BB%83%E6%80%BB%E7%BB%93/)


具体代码: [Handon-LLM-Practice](https://github.com/yyhchen/yyhchen-HandOn-LLM-Practice/tree/main/from_scratch_train_multi_modal_model)


