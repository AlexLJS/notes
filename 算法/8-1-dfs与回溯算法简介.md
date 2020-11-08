---
title: 8-1 dfs与回溯算法简介
date: 2020-10-04 19:38:21
tags:
categories:
---

## 经典回溯 和 dfs


dfs 递归（或者暴搜），总之就是前进一层， 要用方法参数记录当前位置， 可以记录path。 

回溯则是在 dfs 基础上，一般在退出递归回到上一层后进行 path 的 remove。


## 技巧1： 记忆化回溯（或者记忆递归）

设定一个全局 boolean[] 数组（或者 int[][]）记忆路径，进行剪枝。

一般在在进入 dfs 时，会设定退出条件。 在此条件中通过 check 记忆数组，决定是否需要剪枝。  


## 技巧2： 想不清时要画递归树

递归树有助于判断， 索引从哪位开始， 从哪层开始去重


## 例 ： 牛客网 和为目标值的组合

```
    ArrayList<ArrayList<Integer>> res = new ArrayList<>();
    public ArrayList<ArrayList<Integer>> combinationSum2(int[] num, int target) {
        //经典回溯问题
        Arrays.sort(num);
        process(num, -1, target, new ArrayList<>());
        return res;
    }
    
    public void process(int[] arr, int index, int t, ArrayList<Integer> path){
        if ( index >= arr.length ) return;
        if (t == 0){
            ArrayList<Integer> temp = new ArrayList<>(path);
            Collections.sort(temp);
            res.add(temp);
            return;
        }
        if (t < 0) return;
        
        for (int i = index + 1; i < arr.length; i++){
            //去重
            if (i > index + 1 && arr[i] == arr[i - 1]) continue;
            path.add(arr[i]);
            process(arr, i, t - arr[i], path);
            path.remove(path.size() - 1);
        }
    }
```