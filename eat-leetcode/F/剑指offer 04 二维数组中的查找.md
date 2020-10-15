- Q

```markdown
在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整。

现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]

给定 target = 5，返回 true。
给定 target = 20，返回 false。
```

- 暴力 遍历每一个元素

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {

        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }

        int rows = matrix.length, columns = matrix[0].length;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < columns; j++) {
                if (matrix[i][j] == target) {
                    return true;
                }
            }
        }
        return false;
        }
    }
}
```

- 双指针  从右上角

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rows = matrix.length, columns = matrix[0].length;
        int row = 0, column = columns - 1;
        while (row < rows && column >= 0) {
            int num = matrix[row][column];
            if (num == target) {
                return true;
            } else if (num > target) {
                column--;
            } else {
                row++;
            }
        }
        return false;
    }
}

```

- 双指针 从左下角

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rows = matrix.length, columns = matrix[0].length;
        int row = 0, column = columns - 1;
        while (row < rows && column >= 0) {
            int num = matrix[row][column];
            if (num == target) {
                return true;
            } else if (num > target) {
                column--;
            } else {
                row++;
            }
        }
        return false;
    }
}
```

- 二分法

```java
// https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/solution/suo-xiao-fan-wei-er-fen-fa-cha-zhao-by-qiu-chen-i/
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        int up = 0;
        int down =matrix.length-1;
        int left =0;
        if(down<0)
            return false;
        int right =matrix[0].length-1;
        if(right<0)
            return false;
        while(right>=left&&up<=down)
        {
            int temp=binarySreach(matrix,left,right,target,true,up);
            if(matrix[up][temp]==target)//判断是否为target 不是就移动指针
                return true;
            else
                right=temp;
            temp=binarySreach(matrix,up,down,target,false,left);
            if(matrix[temp][left]==target)//判断是否为target 不是就移动指针
                return true;
            else
                down=temp;
            left++;
            up++;
        }
        return false;
    }

    /**
     * 二分法查找
     * @param matrix
     * @param begin 开始坐标
     * @param end 结束坐标
     * @param target
     * @param isLine 判断是否为行 反之为列
     * @param high 行/列 高
     * @return 找到返回坐标，没找到返回小于target的最大值坐标
     */
    public int binarySreach(int[][] matrix,int begin,int end,int target,boolean isLine,int high){
        while(begin<end)
        {
            int mid =((end-begin+1)>>>1)+begin;//==(begin+end+1)/2 防止相加溢出
            int temp;
            if(isLine)
                temp=matrix[high][mid];
            else
                temp=matrix[mid][high];
            if(temp==target)
                return mid;
            else if(temp>target)
                end =mid-1;
            else
                begin=mid;
        }
        return begin;
    }
}

```

- 区域递归 

```java
// https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/solution/mian-shi-ti-04java-san-chong-jie-fa-tu-jie-xiang-j/

public boolean searchMatrix2(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0)
            return false;
        return search(matrix, target, 0, matrix[0].length - 1, 0, matrix.length - 1);
    }

    private boolean search(int[][] matrix, int target, int left, int right, int top, int bottom) {
        if (left > right || top > bottom) // 已无迭代区域
            return false;
        if (target < matrix[top][left] || target > matrix[bottom][right]) // 目标值比矩阵的左上角小或者比矩阵的右小角大，肯定无法不能在矩阵中找到该值
            return false;
        int mid = (left + right) / 2;
        int row = top;
        while (row <= bottom && matrix[row][mid] <= target) { // 搜索中间列是否能找到target，如果找不到就使row停在该行中间元素比target大的位置
            if (matrix[row][mid] == target)
                return true;
            row++;
        }
        return search(matrix, target, left, mid - 1, row, bottom) ||
                search(matrix, target, mid + 1, right, top, row - 1);
    }


```





