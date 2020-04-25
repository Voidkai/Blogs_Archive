# Java参数传递之一场由Java值传递引发的“血案”

> 车祸发生于公元二零二零年四月二十五日上午十一时许，事故犯罪团伙名为力扣每日一题之46全排列，具体案件将于后续公布...

## "事故犯罪团伙"：算法题之没有感情的全排列

给定一个没有重复数字的序列，返回其所有可能的全排列。

示例：

```
输入： [1,2,3]

输出：

[

​	[1,2,3],

​	[1,3,2],

​	[2,1,3],

​	[2,3,1],

​	[3,1,2],

​	[3,2,1]
]
```



## 解题思路

1. 暴力法：数组中有多少个数就需要有多少层for循环嵌套，超时的结果就理所应当。
2. 深度优先，递归调用：同《啊哈！算法》第四章第一节摆放扑克牌问题，将问题分步考虑，一个数字一个数字的考虑，为了保证不重复选取数组中的数据，我们需要一个标记数组 `int[] isChoose`，保存数字使用情况。在每一步中遍历数组中的每一个数，如果数字未被使用，则选择该数，递归调用每一步的处理函数。同时在循环中处理函数运行完后还需要重新将该数设置为未被调用，一遍下一个循环中调用过程中对该数的选取。



## Java代码实现

```Java
public class solution {
    void dfs(int len, int[] nums,int[] isChoose,List<Integer> p,List<List<Integer>> res){
        if(p.size() == len){
            List<Integer> tmp=new LinkedList<>();
            for(int k=0;k<len;k++){
                tmp.add(p.get(k));
            }
            res.add(tmp);

            return;
        }

        for(int i=0;i<len;i++){
            if(isChoose[i] == 0){
                p.add(nums[i]);
                isChoose[i] = 1;
                dfs(len,nums,isChoose,p,res);
                p.remove(p.size()-1);
                isChoose[i] = 0;
            }
        }
    }

    public List<List<Integer>> permute(int[] nums) {
        int len = nums.length;
        int[] isChoose = new int[len];
        List<Integer> p = new LinkedList<>();
        List<List<Integer>> res = new LinkedList<>();
        dfs(len,nums,isChoose,p,res);

        return res;
    }

    public static void main(String[] args){
        List<List<Integer>> res = new solution().permute(new int[]{1,2,3});

        System.out.println(res);
    }

}
```

输出结果：

`[[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]`



## 错误代码示范

```Java
public class solution {
    void dfs(int len, int[] nums,int[] isChoose,List<Integer> p,List<List<Integer>> res){
        if(p.size() == len){
            res.add(p);
            return;
        }

        for(int i=0;i<len;i++){
            if(isChoose[i] == 0){
                p.add(nums[i]);
                isChoose[i] = 1;
                dfs(len,nums,isChoose,p,res);
                p.remove(p.size()-1);
                isChoose[i] = 0;
            }
        }
    }

    public List<List<Integer>> permute(int[] nums) {
        int len = nums.length;
        int[] isChoose = new int[len];
        List<Integer> p = new LinkedList<>();
        List<List<Integer>> res = new LinkedList<>();
        dfs(len,nums,isChoose,p,res);

        return res;
    }

    public static void main(String[] args){
        List<List<Integer>> res = new solution().permute(new int[]{1,2,3});

        System.out.println(res);
    }

}
```

错误输出

`[[], [], [], [], [], []]`



## 错误分析之Java参数传递方法

比较两段代码发现：

在`void dfs(int len, int[] nums,int[] isChoose,List<Integer> p,List<List<Integer>> res)`函数中，当我们找到完整的一个排列准备add的时候，这里有一些不同，从而造成了结果上的不同



正确代码：

```Java
/*正确代码
新创建一个List对象，将预选排列list中的数字一一放入新建的List对象中，随后将新建的List加入到结果List中返回，*/
if(p.size() == len){
            List<Integer> tmp=new LinkedList<>();
            for(int k=0;k<len;k++){
                tmp.add(p.get(k));
            }
            res.add(tmp);

            return;
        }

/*错误代码
直接将list p 放入结果List中返回*/
if(p.size() == len){
            res.add(p);
            return;
        }
```



### 为什么会有这样的结果？

大多数时间会觉得正确解法中的操作有点多此一举，其实不然。

Java的传参方式是按值传递，针对基本数据类型，传参时，传入的是该数据类型具体的值。而针对引用数据类型，如数组，类等，传参时，传入的也是该数组或者类的值，只不过这个值为该数组，类的地址，因此，也将这种情况看做是Java 的按引用传递。

上述的代码就是由地址传递所导致的错误，如果直接将List p 传入List res 中，在List res 中其实保存的是List p 的地址，而在整个程序中，这个List p是用来暂时存放多种排列的具体情况，是会动态改变的。如果直接加P放入res，则会导致后续P中数据的改变也会相应改变我们认为的已经放入了res中的排列情况，同时在错误情况中，我们最后的P因为没有的可以选择的数据从而变为空list，这也最终导致了错误输出全为空的结果。

相较而言，正确代码将保存p的地址改变， 为一个拷贝了p的新List的地址，从而避免了覆盖的问题。