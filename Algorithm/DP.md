# 01DP

```java
public static int getMaxValue(int[] weight, int[] value, int w, int n) {
        int[][] table = new int[n+1][w + 1];
        for (int i = 1; i <= n; i++) { //物品
            for (int j = 0; j <= w; j++) {  //背包大小
//                if (weight[i] > j) {
                    //当前物品i的重量比背包容量j大，装不下，肯定就是不装
//                    table[i][j] = table[i - 1][j];
//                     System.out.print(table[i][j]+ " ");
//                } else { //装得下，Max{装物品i， 不装物品i}
                    table[i][j] = Math.max(table[i-1][j], table[i-1][j - weight[i]] + value[i]);
                }
            }
        }
        return table[n][w];
    }



// 一维
public static int getMaxValue(int[] weight, int[] value, int w, int n) {
        int[] dp = new int[w + 1];
        for (int i = 1; i <= n; i++) { //物品
            for (int j = w; j >= 0; j--) {  //背包大小
//                if (weight[i] > j) {
                    //当前物品i的重量比背包容量j大，装不下，肯定就是不装
//                    table[i][j] = table[i - 1][j];
//                } else { //装得下，Max{装物品i， 不装物品i}
                    dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i]);
                }
            }
        }
        return table[w];
}
```



# 完全背包

> ```java
> f[i][v]=max{f[i-1][v],f[i][v-c[i]]+w[i]}
> 
> for i=1..N
>     for v=0..V
>         f[v]=max{f[v],f[v-cost]+weight}
> ```