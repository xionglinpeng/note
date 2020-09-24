- `-XX:-UseSplitVerifier`：关闭类加载-验证-方法体校验阶段`StackMapTable`优化。
- `-XX:+FailOverToOldVerifier`：要求在类型校验失败的时候退回到旧的类型推导方式进行校验（JD7+（主版本号>50）此参数已无效）。
- `-Xverify:none`：关闭类加载阶段Class二进制数据校验。

- 

