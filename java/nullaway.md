## nullAway
NullAway是一个NullPointerException在Java代码中帮助消除（NPE）的工具。要使用Nullway 首先要在代码中添加@Nullable 注释。在成员变量、方法参数、返回值有可能为空的地方。

## 安装
NullAway 要求使用 [Error Prone](http://errorprone.info/)构建代码。

```gradle

def enableNullAway = false
if (enableNullAway) {
    //error-prone相关配置
    apply plugin: "net.ltgt.errorprone"
}


dependencies {

    if (enableNullAway) {
        apt "com.uber.nullaway:nullaway:0.3.2"
        errorprone "com.google.errorprone:error_prone_core:2.1.3"
    }

}

tasks.withType(JavaCompile) {
    if (enableNullAway) {
        options.compilerArgs +=
                [
                        "-Xep:JavaLangClash:OFF",
                        "-Xep:EqualsHashCode:OFF",
                        "-Xep:ReferenceEquality:OFF",
                        "-Xep:DefaultCharset:OFF",
                        "-Xep:MissingCasesInEnumSwitch:OFF",
                        "-Xep:MissingOverride:OFF",
                        "-Xep:ClassCanBeStatic:OFF",
                        "-Xep:Overrides:OFF",
                        "-Xep:ImmutableEnumChecker:OFF",
                        "-Xep:NarrowingCompoundAssignment:OFF",
                        "-Xep:DoubleCheckedLocking:OFF",
                        "-Xep:SynchronizeOnNonFinalField:OFF",
                        "-Xep:NonAtomicVolatileUpdate:OFF",
                        "-Xep:FloatingPointLiteralPrecision:OFF",
                        "-Xep:CheckReturnValue:OFF",
                        "-Xep:ClassNewInstance:OFF",
                        "-Xep:JdkObsolete:OFF",
                        "-Xep:NullAway:WARN",
                        "-XepOpt:NullAway:AnnotatedPackages=package",
                        "-XepOpt:NullAway:ExcludedFieldAnnotations=com.google.gson.annotations.SerializedName,xxx.DatabaseField",
                        "-XepOpt:NullAway:CustomInitializerAnnotations=package.Initializer",
                ]
    }
}


```

> dependencies 中 apt 会加载Nullway，编译的时候检查@Nullable 的注解（javax.annotation.Nullable）在使用的时候判断是否为null
>task.withType(JavaCompile) 表示编译的时候设置Nullway参数。

- -Xep:NullAway:ERROR 设置编译提示 错误级别（Error WARN）
- -XepOpt:NullAway:AnnotatedPackages=package  设置要使用Nullway 检查的包名,支持多个包名，使用逗号分隔。
- -XepOpt:NullAway:UnannotatedSubPackages=package 设置从 nullaway检查的包名下面排除的子包。
- -XepOpt:NullAway:KnownInitializers=android.app.Activity.onCreate 设置方法全路径（不带参数） 表示为初始化方法
- -XepOpt:NullAway:ExcludedClassAnnotations= 导致类从空分析中排除的注释列表
- -XepOpt:NullAway:ExcludedFieldAnnotations= 导致类从空分析中排除的字段列表
- -XepOpt:NullAway:CustomInitializerAnnotations= 设置初始化程序的注释

- -XepDisableAllChecks 
