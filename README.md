# CASIO

CASIO（Calibrated Surrogate-Informed Optimization）是面向剪力墙截面与材料选型的代理辅助优化方法。它用两套独立训练的 FuseGNN 分别估计候选方案的可行性与配筋质量，再用当前建筑的有限元分析（FEA）结果更新局部校准模型。校准结果用于可行性筛选、材料成本排序和 FEA 名额分配，最终方案仍由 FEA 和结构验算确定。

本仓库包含 CASIO 的核心 Python 实现和命令行入口。论文专用绘图、批量对比实验、排版文件、数据、模型权重和实验产物均不随仓库提供。此次仅整理源码，尚未安装依赖或验证运行。

| 模块 | 源码入口 |
| --- | --- |
| 结构建模、OpenSees 分析与配筋验算 | [`src/shearwall_modeling/`](src/shearwall_modeling/)、[`scripts/shearwall_analyzer_main.py`](scripts/shearwall_analyzer_main.py) |
| FEA 样本生成与布局特征准备 | [`scripts/parametric_dataset.py`](scripts/parametric_dataset.py)、`scripts/surrogate/` 下的 `prepare_dataset.py`、`build_layout_features.py` 和 `make_group_splits.py` |
| FuseGNN 训练与推理 | [`src/surrogate/gnn/`](src/surrogate/gnn/)、`scripts/surrogate/` 下的 `train_gnn_final_pass.py`、`train_gnn_steel.py` 和 `predict_gnn_final_pass.py`。实现中的 `LayoutParamGNN` 是论文中的 FuseGNN。 |
| 在线校准与优化 | [`src/shearwall_optimization/`](src/shearwall_optimization/)，包含候选筛选、成本预选、在线校准、GA、PSO 和随机搜索；统一入口为 [`scripts/shearwall_optimize_main.py`](scripts/shearwall_optimize_main.py) |

`src/data_engine/`、`src/shearwall_pred/` 和 `src/misc/` 仅保留上述代码引用的共享模块。部分脚本仍使用原项目的工作目录约定，后续调试时应从仓库根目录执行，并按入口参数指定自己的输入文件。

原项目使用 Python 3.12。主要依赖包括 PyTorch、PyTorch Geometric、OpenSeesPy、NumPy、pandas、PyArrow（Parquet）、scikit-learn、NetworkX、Shapely、ezdxf 和 Matplotlib。部分优化和辅助模块还使用 LightGBM、Optuna、joblib 和 tqdm；此处不锁定依赖版本。

接回原有资产时，训练通常需要 `data/dxf/cad_json_data/fem_raw/` 的布局 JSON、`data/dxf/fem_raw/` 的 DXF 和 `data/parametric/surrogate_dataset/splits/` 的分组样本。优化和评估还需要 `data/parametric/ckpt/` 下的对应模型工件。这些路径是原脚本的默认约定，仓库中没有附带相应文件。
