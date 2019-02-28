## fragment shader和 GPU
- GPU是一大堆小的微处理器，并行计算
- GPU处理数学函数可以硬件加速
- GPU上每个线程是独立的，不能修改输入数据，不能使用其他线程的输出结果
- 微处理器（管道）会尽可能忙碌，通用（根据位置输出结果），无状态，无记忆
## GLSL
- main函数
- 像素颜色是全局变量gl_FragColor
- 有宏

- 可以设置浮点数的精度，例如
'''
precision mediump float;
'''

- 并不保证自动类型转换，语言规范精简方便显卡加速
- 所有线程的输入值必须是uniform，并且只读
- 统一值通常为 float, vec2, vec3, vec4, mat2, mat3, mat4, sampler2D, samplerCube
- 给类型设定精度后，定义统一值

