# Phong光照模型
**姓名**：王婧怡　｜　**学号**：202311081051　｜　**专业**：计算机科学与技术

## <font style="color:rgb(15, 17, 21);">实验简介</font>
<font style="color:rgb(15, 17, 21);">本项目使用 Taichi 高性能计算框架，实现了一个基于 Phong光照模型 的交互式3D渲染器。通过光线投射技术，在屏幕上实时渲染一个红色球体和一个紫色圆锥，并提供了材质参数的实时调节功能，帮助理解和掌握局部光照的基本原理。</font>


## 核心功能实现
### 场景构建
#### <font style="color:rgb(15, 17, 21);">几何体定义</font>
+ 红色球体: 圆心 (-1.2, -0.2, 0)，半径 1.2，颜色 (0.8, 0.1, 0.1)
+ 紫色圆锥: 顶点(1.2, 1.2, 0)，底面中心(1.2, -1.4, 0)，底面半径1.2，颜色(0.6, 0.2, 0.8)

#### <font style="color:rgb(15, 17, 21);">摄像机与光源</font>
+ 摄像机位置:(0, 0, 5)(固定视角)
+ 点光源位置:(2, 3, 4)
+ 光源颜色: 纯白光(1.0, 1.0, 1.0)
+ 背景颜色: 深青色 (0.05, 0.15, 0.15)


### <font style="color:rgb(15, 17, 21);"> 光线求交与深度测试 </font>
#### 球体求交算法
使用二次方程求交法：

```plain
oc = ro - center
b = 2.0 * oc.dot(rd)
c = oc.dot(oc) - radius * radius
delta = b * b - 4.0 * c
# 判断delta >= 0且t>0时有效
```

#### 圆锥求交算法
将光线方程代入圆锥隐式方程x² + z² = k(y-h)²，其中 k = (r/H)²，求解二次方程：

```plain
A = rd.x² + rd.z² - k * rd.y²
B = 2.0 * (ro_local.x * rd.x + ro_local.z * rd.z - k * ro_local.y * rd.y)
C = ro_local.x² + ro_local.z² - k * ro_local.y²
```

<font style="color:rgb(15, 17, 21);">关键优化:</font>

+ <font style="color:rgb(15, 17, 21);">验证交点在圆锥高度范围内[-H, 0]
+ <font style="color:rgb(15, 17, 21);">选择较近的有效交点 t_first 或 t_second

#### <font style="color:rgb(15, 17, 21);">Z-Buffer深度竞争</font>
```plain
if 0 < t_sph < min_t:  # 球体更近
    min_t = t_sph
    hit_normal = n_sph
    hit_color = ti.Vector([0.8, 0.1, 0.1])
    
if 0 < t_cone < min_t:  # 圆锥更近
    min_t = t_cone
    hit_normal = n_cone
    hit_color = ti.Vector([0.6, 0.2, 0.8])
```

<font style="color:rgb(15, 17, 21);">效果: 当两个物体在屏幕上重叠时，显示距离摄像机更近的物体，实现正确的遮挡关系。</font>

### Phong着色器实现
#### <font style="color:rgb(15, 17, 21);">光照计算流程</font>
1. **<font style="color:rgb(15, 17, 21);">计算光照向量</font>**<font style="color:rgb(15, 17, 21);">:</font>
    - L = normalize(light_pos - p) (光线方向)
    - V = normalize(ro - p) (视线方向)
    - R = normalize(reflect(-L, N)) (反射方向)
2. **<font style="color:rgb(15, 17, 21);">三个分量计算</font>**<font style="color:rgb(15, 17, 21);">:</font>

```plain
# 环境光 (Ambient)
ambient = Ka * light_color * hit_color

# 漫反射 (Diffuse) - Lambert定律
diff = max(0.0, N.dot(L))
diffuse = Kd * diff * light_color * hit_color

# 镜面高光 (Specular) - Phong反射模型
spec = max(0.0, R.dot(V)) ** shininess
specular = Ks * spec * light_color

# 最终颜色
color = ambient + diffuse + specular
```

3. <font style="color:rgb(15, 17, 21);">颜色裁剪: 使用 ti.math.clamp(color, 0.0, 1.0) 确保颜色值在合法范围内

### UI交互面板
<font style="color:rgb(15, 17, 21);">使用 ti.ui.Window.get_gui() 创建交互面板，提供4个滑动条：

| <font style="color:rgb(15, 17, 21);">参数</font> | <font style="color:rgb(15, 17, 21);">范围</font> | <font style="color:rgb(15, 17, 21);">默认值</font> | <font style="color:rgb(15, 17, 21);">作用</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">Ka (环境光系数)</font> | <font style="color:rgb(15, 17, 21);">0.0 ~ 1.0</font> | <font style="color:rgb(15, 17, 21);">0.2</font> | <font style="color:rgb(15, 17, 21);">控制环境光强度</font> |
| <font style="color:rgb(15, 17, 21);">Kd (漫反射系数)</font> | <font style="color:rgb(15, 17, 21);">0.0 ~ 1.0</font> | <font style="color:rgb(15, 17, 21);">0.7</font> | <font style="color:rgb(15, 17, 21);">控制漫反射强度</font> |
| <font style="color:rgb(15, 17, 21);">Ks (镜面高光系数)</font> | <font style="color:rgb(15, 17, 21);">0.0 ~ 1.0</font> | <font style="color:rgb(15, 17, 21);">0.5</font> | <font style="color:rgb(15, 17, 21);">控制高光强度</font> |
| <font style="color:rgb(15, 17, 21);">N (高光指数)</font> | <font style="color:rgb(15, 17, 21);">1.0 ~ 128.0</font> | <font style="color:rgb(15, 17, 21);">32.0</font> | <font style="color:rgb(15, 17, 21);">控制高光锐利程度</font> |


```plain
with gui.sub_window("Material Parameters", 0.7, 0.05, 0.28, 0.22):
    Ka[None] = gui.slider_float('Ka (Ambient)', Ka[None], 0.0, 1.0)
    Kd[None] = gui.slider_float('Kd (Diffuse)', Kd[None], 0.0, 1.0)
    Ks[None] = gui.slider_float('Ks (Specular)', Ks[None], 0.0, 1.0)
    shininess[None] = gui.slider_float('N (Shininess)', shininess[None], 1.0, 128.0)
```

## 实验结果
### <font style="color:rgb(15, 17, 21);">默认参数效果</font>
+ **<font style="color:rgb(15, 17, 21);">Ka=0.2</font>**<font style="color:rgb(15, 17, 21);">: 物体暗部有微弱亮度，不至于完全黑暗</font>
+ **<font style="color:rgb(15, 17, 21);">Kd=0.7</font>**<font style="color:rgb(15, 17, 21);">: 漫反射明显，体现Lambert定律的余弦衰减</font>
+ **<font style="color:rgb(15, 17, 21);">Ks=0.5</font>**<font style="color:rgb(15, 17, 21);">: 高光区域明亮，体现材质光泽</font>
+ **<font style="color:rgb(15, 17, 21);">Shininess=32</font>**<font style="color:rgb(15, 17, 21);">: 高光区域中等大小，边缘柔和</font>

### <font style="color:rgb(15, 17, 21);">参数调节效果</font>
1. **<font style="color:rgb(15, 17, 21);">增大Ka</font>**<font style="color:rgb(15, 17, 21);">: 整体变亮，暗部细节可见</font>
2. **<font style="color:rgb(15, 17, 21);">增大Kd</font>**<font style="color:rgb(15, 17, 21);">: 面向光源的面更亮，立体感增强</font>
3. **<font style="color:rgb(15, 17, 21);">增大Ks</font>**<font style="color:rgb(15, 17, 21);">: 高光更亮，材质更"闪亮"</font>
4. **<font style="color:rgb(15, 17, 21);">增大Shininess</font>**<font style="color:rgb(15, 17, 21);">: 高光区域缩小，变得更锐利（类似金属质感）</font>

下列是分别调节Ka,Kd,Ks以及Shininess后物体的效果
<img width="804" height="647" alt="ka" src="https://github.com/user-attachments/assets/1213223e-8e43-4ace-b55d-965f79ac1982" />
<img width="804" height="647" alt="kd" src="https://github.com/user-attachments/assets/d18ca80b-825c-4fab-90cd-3d7899f04ebf" />
<img width="804" height="647" alt="ks" src="https://github.com/user-attachments/assets/7cf6c062-871a-41e7-b8bc-94d8f52abbdb" />
<img width="804" height="647" alt="n" src="https://github.com/user-attachments/assets/b5103da2-d90e-4d05-a049-c3af03254947" />


