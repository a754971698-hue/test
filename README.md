# SimpleSlideViewer组件单元测试文档

## 概述
本文档描述了针对`SimpleSlideViewer`组件（一个切片图像查看器）的两个核心交互功能的单元测试：
1. **test_image_drag.py** - 鼠标拖拽平移功能测试
2. **test_wheel_zoom.py** - 滚轮缩放功能测试
3. **test_scale_bar.py** - 比例尺功能测试

## 测试环境要求
- Python 3.7+
- PySide6 (Qt6 Python绑定)
- pytest 框架
- pytest-qt 插件
- unittest 框架

### 环境配置
测试脚本已自动配置以下环境：
- **离屏渲染模式**：`QT_QPA_PLATFORM=offscreen`，无需图形界面即可运行测试
- **OpenSlide模拟**：无需本地安装OpenSlide原生库，测试中使用模拟对象替代
- **Qt事件模拟**：完全模拟鼠标事件，不依赖实际用户输入

## 测试用例详细说明

### 1. test_image_drag.py - 拖拽平移功能测试
此测试验证`SimpleSlideViewer`组件在响应鼠标拖拽操作时的核心逻辑。

#### 测试用例清单
| 测试函数 | 测试目的 | 关键验证点 |
|---------|---------|-----------|
| `test_basic_pan_updates_position_by_mouse_delta_at_scale_1` | 验证在缩放级别为1.0时，基本的平移操作是否正确更新了位置 | 1. 从(100,100)拖到(200,250)<br>2. 位置从(5000,5000)更新为(5100,5150)<br>3. 鼠标偏移量(100,150)正确转换<br>4. 触发图片加载功能 |
| `test_drag_disabled_prevents_dragging_and_position_change` | 验证当主窗口的拖拽功能被禁用(drag_enabled=False)时，拖拽操作不会触发任何变化 | 1. 禁用拖拽功能<br>2. 执行拖拽操作<br>3. 位置保持不变<br>4. 图片加载函数不被调用 |

#### 测试设计特点
- **使用unittest框架**：传统单元测试框架
- **完全模拟的鼠标事件**：通过`QMouseEvent`模拟完整拖拽流程（press → move → release）
- **隔离测试**：专注数学计算，屏蔽图片加载、网络IO等外部依赖
- **边界条件测试**：验证功能开关的有效性

### 2. test_wheel_zoom.py - 滚轮缩放功能测试
此测试验证`SimpleSlideViewer`组件在响应鼠标滚轮事件时的缩放功能。

#### 测试用例清单
| 测试函数 | 测试目的 | 关键验证点 |
|---------|---------|-----------|
| `test_wheel_zoom_in_increases_digital_zoom_factor` | 验证滚轮向上滚动能正确增加数字缩放因子 | 1. 向上滚动120度<br>2. 缩放因子从1.0增加到1.08(增加8%)<br>3. 触发图片加载 |
| `test_wheel_zoom_out_decreases_digital_zoom_factor` | 验证滚轮向下滚动能正确减少数字缩放因子 | 1. 向下滚动-120度<br>2. 缩放因子从1.0减少到0.92(减少8%) |
| `test_current_level_stays_inside_openslide_level_range` | 验证缩放级别不超出OpenSlide支持范围 | 1. 在最大缩放级别时继续放大<br>2. 在最小缩放级别时继续缩小<br>3. 确保级别始终在有效范围内 |
| `test_wheel_zoom_keeps_mouse_anchor_stable` | 验证缩放时鼠标锚点保持稳定（用户体验关键） | 1. 在不同位置(中心/边缘)进行缩放<br>2. 验证鼠标所指的图像坐标基本不变(误差≤1像素) |
| `test_status_updated_emits_zoom_factor_text` | 验证缩放操作触发UI状态更新信号 | 1. 缩放后发出status_updated信号<br>2. 信号包含正确的缩放倍率文本(如"缩放倍率: 21.60x") |

**注意**：`test_wheel_zoom_keeps_mouse_anchor_stable`是参数化测试，实际运行2个测试实例（中心位置和边缘位置），所以总共运行6个测试。

### 3. test_scale_bar.py - 比例尺功能测试
此测试验证`SimpleSlideViewer`组件比例尺功能。

#### 测试用例清单
| 测试函数 | 测试目的 | 关键验证点 |
|---------|---------|-----------|
| `test_scale_bar_calculation` | 验证基本比例尺计算公式的正确性 | 1. 使用参数化测试覆盖3种缩放因子(0.5, 1.0, 2.0)<br>2. 验证比例尺文本的正确格式("2.0mm", "1.0mm", "500μm")<br>3. 测试单位自动切换逻辑(mm ↔ μm) |
| `test_update_scale_bar_uses_mocked_openslide_mpp_value` | 验证update_scale_bar函数正确使用MPP值 |1. 验证调用了get_zoom_factor和get_pixel_size方法<br/>2. 在0.25um/px分辨率下，100px比例尺显示"25μm"<br/>3. 确保计算过程正确使用模拟的OpenSlide数据|
| `test_get_pixel_size_adapts_to_mpp_properties` | 验证像素尺寸获取适应不同MPP属性 |1. 测试有效的mpp-x和mpp-y属性<br/>2. 验证两者都有效时取平均值计算<br/>3. 参数化测试3种MPP组合(0.25, 0.50, 以及非对称0.24/0.26)|
| `test_get_pixel_size_invalid_mpp_falls_back_without_crashing` | 验证无效MPP值时的优雅回退机制 |1. 处理空字典、空值、非法字符串等异常情况<br/>2. 验证非正数值(-1, 0)的容错处理<br/>3. 确保全部回退到默认值0.25um/px而不崩溃|
| `test_scale_bar_large_zoom_remains_readable` | 验证高放大倍数下的比例尺可读性 | 1. 在50倍高放大下测试<br>2. 确保短距离仍用微米单位显示("20μm")<br>3. 验证公式在高倍率下的正确性 |
| `test_scale_bar_tiny_zoom_uses_larger_physical_distance` | 验证低放大倍数下的单位切换 | 1. 在0.1倍低放大下测试<br>2. 验证自动切换到毫米单位<br>3. 确保长距离用毫米显示并保留合适精度("10.0mm") |
| `test_update_scale_bar_dynamically_adapts_to_svs_resolution` | 验证比例尺对不同扫描分辨率的动态适应 | 1. 测试不同MPP值(0.125, 0.25, 0.50 um/px)<br>2. 验证比例尺标签随pixel_size线性变化<br>3. 确保显示结果符合预期("12μm", "25μm", "50μm") |

### 4. test_annotation_rendering.py - 病理标注功能测试

此测试验证`SimpleSlideViewer`组件处理病理标注功能。

#### 测试用例清单

| **测试函数**                                                 | **测试目的**                               | **关键验证点**                                               |
| ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| `test_coordinate_transform_mirror_symmetry_between_draw_and_hit_detection` | 验证渲染与点击判定逻辑的镜像对称性     | 1. 0级坐标点经`map_to_screen`转换 <br/>2. 结果传入`find_feature_at_coords`还原 <br/>3. 还原后偏差小于 $10^{-6}$ 像素 |
| `test_polygon_with_five_points_builds_closed_qpolygonf`      | 验证标准 Polygon（首尾重合）的解析逻辑 | 1. 输入GeoJSON标准的5点序列 <br/>2. 验证`QPolygonF`实例化过程 <br/>3. 确保正确识别闭合图形 |
| `test_tiny_and_huge_polygons_do_not_overflow_or_raise`       | 验证在极端坐标尺度下的数值稳定性       | 1. 输入坐标值为 $10^{-5}$ 和 $10^{10}$ 的几何体 <br/>2. 确保不触发溢出或系统崩溃 <br/>3. 验证绘制函数成功调用 |
| `test_rectangle_geometry_requires_explicit_branch_support`   | 验证对 Rectangle 专项几何类型的支持    | 1. 构造带有`type: "Rectangle"`的Mock数据 <br/>2. 检查解析逻辑分支 <br/>3. 确保矩形标注被正常处理 |
| `test_rectangle_hit_detection_uses_bbox_and_inverse_mapping`   | 验证渲染与点击判定逻辑的镜像对称性    | 1. 0级坐标点经map_to_screen转换 <br/>结果传入find_feature_at_coords还原 <br/>3. 还原后偏差小于 $10^{-6}$ 像素 |
| `test_redundant_pen_and_brush_creation_is_detected_in_large_annotation_batches` | 验证高频绘制下的资源复用与内存管理     | 1. 模拟绘制1000个相同样式的标注<br/> 2. `QPen`实例化次数需 $\le 10$<br/> 3. 确保样式对象已提取至循环外复用 |
| `test_only_visible_annotations_should_be_transformed_and_drawn` | 验证视口裁剪 (Culling) 功能的有效性    | 1. 加载10,000个标注且仅1个可见<br/> 2. 实际执行坐标转换的数量应 $\le 10$ <br/>3. 确保跳过屏幕外标注以提升FPS |
| `test_draw_geojson_labels_early_returns_when_label_button_is_unchecked` | 验证“显示标注”开关的 Early Return 逻辑 | 1. 设置`Btn_show_label`为关闭状态 <br/>2. 触发重绘事件 <br/>3. 确保立即执行Early Return，不消耗计算资源 |

