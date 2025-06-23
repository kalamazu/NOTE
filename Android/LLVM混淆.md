
### 如何使用llvm
Souce Code -> Frontend->Optimizer->Backend->Machine Code
Frontend :词法分析，语法分析，语义分析，生成中间代码
Optimizer：优化
Backend：生成机器码

[OLLVM](https://www.cnblogs.com/huhuf6/p/14010717.html)


[配置成功方案AndroidStudio](https://pshocker.github.io/2022/05/04/%E7%94%A8ollvm%E6%B7%B7%E6%B7%86native%E4%BB%A3%E7%A0%81/)


[llvm pass编写   适用于Linux](https://www.less-bug.com/posts/llvm-implement-function-pass-from-scratch/)
编写MyPass.cpp,MyPsss.h

编写Makefile或者CMakelists.txt

生成Libmypass.so
在opt环节导入并使用


将原文件编译为.ll
clang -S -emit-llvm -O0 -Xclang -disable-O0-optnone foo.c -o foo.ll
将.ll经过pass变成新的.ll
opt -S -load-pass-plugin=libmypass.so -passes='mypass' foo.ll -o foo_passed.ll
将.ll变为可执行文件
clang foo.ll -o foo 


pass编写：
一，函数级操作
1.PreservedAnalyses类，作为run的返回值，有三种返回模式，确定自己的修改对分析结果有无影响
（保守做法：return PreservedAndalyses：：none()）
2.先创建一个class继承PassInfoMixin< self > 再声明run，再实现run，对每个函数进行操作

1.指令替换：
	1.使用dyn_cast< BinaryOperator >(&i) llvm 的类型检查转换,检查i是不是二元运算，是就转换为BinaryOperator类型的指针；
	2.简单的指令替换a+b -> a-(-b)
```cpp
namespace llvm {

static int f_number = 0;

PreservedAnalyses MyPass::run(Function& F, FunctionAnalysisManager& AM) {

  // rename function to f_number
  for(BasicBlock &BB:F){//遍历F中的基本块
    SmallVector<BinaryOperator* ,8> AddInsts;//创建加法指令收集容器
    for(Instruction &I:BB){//遍历基本块里的指令
      if(auto *Add=dyn_cast<BinaryOperator>(&I)){//检测指令是二元运算
        if(Add->getOpcode()==Instruction::Add){//检测是不是加法指令
          AddInsts.push_back(Add);//添加加法指令进容器
        }
      }
    }
    for(auto *Add:AddInsts){//遍历容器的指令
      IRBuilder<> Builder(Add);//创建IR构建器
      Value *NegB=Builder.CreateNeg(Add->getOperand(1),"neg.val");
      //获取这个指令的第二个操作数生成-b的负值计算指令CreateNeg
      Value *Sub=Builder.CreateSub(Add->getOperand(0),NegB,"sub.neg");
      //获取这个指令的第一个操作数，添加减法操作
      Add->replaceAllUsesWith(Sub);
      //将所有使用加法结果的地方替换为新的减法结果
      Add->eraseFromParent();
      //从IR中删除原来的加法指令
    }
  }
  return PreservedAnalyses::all();
}
}  
```


### Java混淆
[编写规则](https://www.guardsquare.com/manual/configuration/usage)

在模块级的build.gradle添加proguard设置
```build.gradle.kts
buildTypes {  
    release {  
        isMinifyEnabled = true  
        isShrinkResources=true  
        proguardFiles(  
            getDefaultProguardFile("proguard-android-optimize.txt"), 
             //默认配置位于SDK/tools
            "proguard-rules.pro"  
        )  
    }  
}
```







