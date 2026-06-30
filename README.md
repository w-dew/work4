# Phong光照模型
| 202311081051 | 王婧怡 | 计算机科学与技术 |
| --- | --- | --- |


## <font style="color:rgb(15, 17, 21);">实验简介</font>
<font style="color:rgb(15, 17, 21);">本项目使用 Taichi 高性能计算框架，在原有Phong光照模型渲染器的基础上，实现了Blinn-Phong光照模型和硬阴影两个扩展功能。通过光线投射技术实时渲染红色球体和紫色圆锥，并对比分析了两种光照模型在视觉表现上的差异，以及硬阴影对场景真实感的提升效果。</font>

## 核心功能实现
### <font style="color:rgb(15, 17, 21);">场景构建</font>
#### <font style="color:rgb(15, 17, 21);">几何体定义</font>
+ **<font style="color:rgb(15, 17, 21);">红色球体</font>**<font style="color:rgb(15, 17, 21);">: 圆心 (-1.2, -0.2, 0)，半径 1.2，颜色 (0.8, 0.1, 0.1)</font>
+ **<font style="color:rgb(15, 17, 21);">紫色圆锥</font>**<font style="color:rgb(15, 17, 21);">: 顶点 (1.2, 1.2, 0)，底面中心 (1.2, -1.4, 0)，底面半径 1.2，颜色 (0.6, 0.2, 0.8)</font>

#### <font style="color:rgb(15, 17, 21);">摄像机与光源</font>
+ **<font style="color:rgb(15, 17, 21);">摄像机位置</font>**<font style="color:rgb(15, 17, 21);">: (0, 0, 5) (固定视角)</font>
+ **<font style="color:rgb(15, 17, 21);">点光源位置</font>**<font style="color:rgb(15, 17, 21);">: (2, 3, 4)</font>
+ **<font style="color:rgb(15, 17, 21);">光源颜色</font>**<font style="color:rgb(15, 17, 21);">: 纯白光 (1.0, 1.0, 1.0)</font>
+ **<font style="color:rgb(15, 17, 21);">背景颜色</font>**<font style="color:rgb(15, 17, 21);">: 深青色 (0.05, 0.15, 0.15)</font>

### <font style="color:rgb(15, 17, 21);">光线求交与深度测试</font>
#### <font style="color:rgb(15, 17, 21);">球体求交算法</font>
<font style="color:rgb(15, 17, 21);">使用二次方程求交法：</font>

```plain
oc = ro - center
b = 2.0 * oc.dot(rd)
c = oc.dot(oc) - radius * radius
delta = b * b - 4.0 * c
# 判断delta >= 0且t>0时有效
```

#### <font style="color:rgb(15, 17, 21);">圆锥求交算法</font>
<font style="color:rgb(15, 17, 21);">将光线方程代入圆锥隐式方程 x² + z² = k(y-h)²，其中 k = (r/H)²：</font>

```plain
A = rd.x² + rd.z² - k * rd.y²
B = 2.0 * (ro_local.x * rd.x + ro_local.z * rd.z - k * ro_local.y * rd.y)
C = ro_local.x² + ro_local.z² - k * ro_local.y²
```

+ <font style="color:rgb(15, 17, 21);">验证交点在圆锥高度范围内 [-H, 0]</font>
+ <font style="color:rgb(15, 17, 21);">选择较近的有效交点 t_first 或 t_second</font>

#### <font style="color:rgb(15, 17, 21);">Z-Buffer深度竞争</font>
<font style="color:rgb(15, 17, 21);">通过比较不同物体的交点距离，实现正确的遮挡关系：</font>

```plain
if 0 < t_sph < min_t:  # 球体更近
    min_t = t_sph
    hit_normal = n_sph
    hit_color = ti.Vector([0.8, 0.1, 0.1])
```

### <font style="color:rgb(15, 17, 21);">选做：Blinn-Phong光照模型</font>
#### <font style="color:rgb(15, 17, 21);">实现原理</font>
<font style="color:rgb(15, 17, 21);">Blinn-Phong模型是对经典Phong模型的改进，通过引入半程向量来简化高光计算：</font>

<font style="color:rgb(15, 17, 21);">半程向量计算：H = normalize(L + V)  # L为光线方向，V为视线方向</font>

<font style="color:rgb(15, 17, 21);">高光计算：</font>

```plain
spec = ti.max(0.0, N.dot(H)) ** shininess[None]
specular = Ks[None] * spec * light_color
```

#### <font style="color:rgb(15, 17, 21);">与Phong模型的对比分析</font>
| <font style="color:rgb(15, 17, 21);">特性</font> | <font style="color:rgb(15, 17, 21);">Phong模型</font> | <font style="color:rgb(15, 17, 21);">Blinn-Phong模型</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">高光计算</font> | <font style="color:rgb(15, 17, 21);">spec = (R·V)ⁿ，需计算反射向量R</font> | <font style="color:rgb(15, 17, 21);">spec = (N·H)ⁿ，无需反射向量</font> |
| <font style="color:rgb(15, 17, 21);">计算效率</font> | <font style="color:rgb(15, 17, 21);">需额外计算反射向量，开销较大</font> | <font style="color:rgb(15, 17, 21);">仅需向量加法，效率更高</font> |
| <font style="color:rgb(15, 17, 21);">大入射角表现</font> | <font style="color:rgb(15, 17, 21);">高光边缘可能出现"断裂"</font> | <font style="color:rgb(15, 17, 21);">高光过渡更平滑自然</font> |
| <font style="color:rgb(15, 17, 21);">物理精确性</font> | <font style="color:rgb(15, 17, 21);">更符合物理镜面反射</font> | <font style="color:rgb(15, 17, 21);">经验模型，但效果接近</font> |


**<font style="color:rgb(15, 17, 21);">视觉差异详解</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">在大入射角情况下：</font>

1. <font style="color:rgb(15, 17, 21);">Phong模型的：</font>
    - <font style="color:rgb(15, 17, 21);">当观察方向与反射方向快速偏离时，高光强度急剧下降</font>
    - <font style="color:rgb(15, 17, 21);">高光区域边缘可能出现不自然的截断或"锯齿"</font>
    - <font style="color:rgb(15, 17, 21);">在物体轮廓附近，高光可能突然消失</font>
2. <font style="color:rgb(15, 17, 21);">Blinn-Phong模型：</font>
    - <font style="color:rgb(15, 17, 21);">半程向量H始终位于L和V之间，变化更平缓</font>
    - <font style="color:rgb(15, 17, 21);">高光衰减更柔和，边缘过渡自然</font>
    - <font style="color:rgb(15, 17, 21);">即使在掠射角下，高光也能保持平滑</font>
    - <font style="color:rgb(15, 17, 21);">视觉效果更接近真实材质的光泽表现</font>

### <font style="color:rgb(15, 17, 21);">选做：硬阴影实现</font>
#### <font style="color:rgb(15, 17, 21);">阴影射线算法</font>
```plain
@ti.func
def is_in_shadow(p, light_pos, objects):
    L = normalize(light_pos - p)
    # 从p点向光源方向发射阴影射线，稍微偏移避免自相交
    shadow_ro = p + L * 1e-4
    shadow_rd = L
    
    # 测试与场景中所有物体的相交
    # 如果阴影射线在到达光源前击中任何物体，返回True
    return shadow_t < light_dist
```

#### <font style="color:rgb(15, 17, 21);">阴影渲染流程</font>
1. **<font style="color:rgb(15, 17, 21);">阴影检测</font>**<font style="color:rgb(15, 17, 21);">：从交点向光源方向发射阴影射线</font>
2. **<font style="color:rgb(15, 17, 21);">相交测试</font>**<font style="color:rgb(15, 17, 21);">：检测阴影射线是否在到达光源前击中其他物体</font>
3. **<font style="color:rgb(15, 17, 21);">光照计算</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - **<font style="color:rgb(15, 17, 21);">在阴影中</font>**<font style="color:rgb(15, 17, 21);">：仅计算环境光</font>
    - **<font style="color:rgb(15, 17, 21);">不在阴影中</font>**<font style="color:rgb(15, 17, 21);">：完整计算环境光+漫反射+高光</font>

```plain
if in_shadow:
    # 阴影中只计算环境光
    color = Ka[None] * light_color * hit_color
else:
    # 完整Blinn-Phong光照模型
    color = ambient + diffuse + specular
```

#### <font style="color:rgb(15, 17, 21);">关键技术细节</font>
+ **<font style="color:rgb(15, 17, 21);">避免自相交</font>**<font style="color:rgb(15, 17, 21);">：阴影射线起点沿光线方向偏移</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1e-4</font>`
+ **<font style="color:rgb(15, 17, 21);">深度比较</font>**<font style="color:rgb(15, 17, 21);">：比较阴影射线交点距离与光源距离</font>
+ **<font style="color:rgb(15, 17, 21);">硬阴影特征</font>**<font style="color:rgb(15, 17, 21);">：阴影边界清晰锐利，无半影过渡</font>

## <font style="color:rgb(15, 17, 21);">实验结果与分析</font>
### <font style="color:rgb(15, 17, 21);">Blinn-Phong模型效果</font>
**<font style="color:rgb(15, 17, 21);">默认参数效果</font>**<font style="color:rgb(15, 17, 21);">（Ka=0.2, Kd=0.7, Ks=0.5, Shininess=32）：</font>

+ <font style="color:rgb(15, 17, 21);">高光区域比Phong模型更柔和自然</font>
+ <font style="color:rgb(15, 17, 21);">在物体边缘和轮廓处，高光过渡平滑无断裂</font>
+ <font style="color:rgb(15, 17, 21);">整体光照效果更接近真实材质</font>

**<font style="color:rgb(15, 17, 21);">与Phong模型对比</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">球体高光：Blinn-Phong产生更圆润的高光形状</font>
+ <font style="color:rgb(15, 17, 21);">圆锥高光：在大倾斜表面，Blinn-Phong高光保持连续</font>
+ <font style="color:rgb(15, 17, 21);">视觉效果：Blinn-Phong整体更柔和，Phong在某些角度更"闪耀"</font>

### <font style="color:rgb(15, 17, 21);">硬阴影效果</font>
**<font style="color:rgb(15, 17, 21);">阴影特征</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">阴影边界清晰锐利</font>
+ <font style="color:rgb(15, 17, 21);">球体在圆锥上投射阴影，圆锥在球体上投射阴影</font>
+ <font style="color:rgb(15, 17, 21);">阴影区域仅保留环境光成分，产生明显的明暗对比</font>

**<font style="color:rgb(15, 17, 21);">场景真实感提升</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">硬阴影增强了物体的空间位置感和立体感</font>
+ <font style="color:rgb(15, 17, 21);">通过阴影可以直观判断物体间的相对位置</font>
+ <font style="color:rgb(15, 17, 21);">环境光在阴影区域提供基础照明，避免完全黑暗</font>

### <font style="color:rgb(15, 17, 21);"> 参数调节效果</font>
1. **<font style="color:rgb(15, 17, 21);">增大Ka</font>**<font style="color:rgb(15, 17, 21);">：阴影区域和暗部整体变亮，细节更可见</font>
2. **<font style="color:rgb(15, 17, 21);">增大Kd</font>**<font style="color:rgb(15, 17, 21);">：光照面漫反射增强，立体感提升</font>
3. **<font style="color:rgb(15, 17, 21);">增大Ks</font>**<font style="color:rgb(15, 17, 21);">：高光更亮，材质光泽度增强</font>
4. **<font style="color:rgb(15, 17, 21);">增大Shininess</font>**<font style="color:rgb(15, 17, 21);">：高光区域缩小锐利化，呈现金属质感</font>

### 实验结果展示
下列图1~4是Phong光照模型的结果展示，视频5是Blinn-Phong模型的效果
Phong光照模型
<img width="804" height="647" alt="ka" src="https://github.com/user-attachments/assets/1213223e-8e43-4ace-b55d-965f79ac1982" />
<img width="804" height="647" alt="kd" src="https://github.com/user-attachments/assets/d18ca80b-825c-4fab-90cd-3d7899f04ebf" />
<img width="804" height="647" alt="ks" src="https://github.com/user-attachments/assets/7cf6c062-871a-41e7-b8bc-94d8f52abbdb" />
<img width="804" height="647" alt="n" src="https://github.com/user-attachments/assets/b5103da2-d90e-4d05-a049-c3af03254947" />

Blinn-Phong光照模型
https://github.com/user-attachments/assets/258541d8-22ba-430f-9098-9ec7e025a1ab



