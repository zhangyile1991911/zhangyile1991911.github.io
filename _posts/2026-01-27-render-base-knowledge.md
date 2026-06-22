1. 输入装配器 Assembly
2. 顶点着色器阶段 Vertex
3. 分割阶段
    1. 分割参数 控制点
    2. 细化器 Tessellation
    3. 域着色器阶段 新顶点
4. 几何着色器 生成新 点 线 面
5. 光栅化 几个物体 在屏幕上物理像素，插值顶点数据
    1. 裁剪测试
    2. 早起深度测试 early-z
6. 片段着色器
7. 输出合并
    1. Alpha测试 目前用clip()替代
    2. 模板测试
    3. 深度测试
    4. 混合

Gamma Space[sRGB] <=> 线性空间Linear Space
对暗部存储更多的颜色
y=x^1/2.2

物理线性空间0.2 -> gamma空间0.5

对纹理启用sRGB Unity会将纹理0.5（Gamma Space）转为 0.2（Linear Space）

最后画面绘制 帧缓冲区 将颜色0.2(Linear Space) 转换 0.5(Gamma Space)

显示器输出帧缓冲 将缓冲区0.5(Gamma Space)转换为0.2(Linear Space)

mipmaps
采样策略，根据原纹理 生成 小纹理 按2的倍数减少
根据 纹理在屏幕上的占比 选择 对应 尺寸纹理
纹理占比 通过 DDX和DDY对比当前纹理和相邻纹理UV变化率
占比越小，变化率越大 mipmap等级越大
1 过度采样 提高性能
2 采样异常 摩尔纹（不连贯）


filter
1. point uv最近像素
2. bilinear uv最近四个像素
3. trilinear 在两张mipmap之间插值


