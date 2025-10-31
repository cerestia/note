通用操作
1. 设置vl
size_t vl = __riscv_vsetvl_e32m1(ARRAY_NUM); //ARRAY_NUM为数组长度
2. 加载数组到矢量
vint32m1_t v1 = __riscv_vle32_v_i32m1(a, vl);
3. 存储矢量到数组
vint32m1_t v2 = __riscv_vle32_v_i32m1(b, vl);