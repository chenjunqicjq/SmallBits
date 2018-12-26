#题目
---
把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
#思路
---
通过分析，丑数的只能包含质因子2，3，5，那么每个数都是由x=2^a+3^b+5^c构成；那么我就从第一个丑数1开始，分别乘以2，3，5；得到的数保存在一个集合里，里面选出最小的数，将他加入结果队列里，再进行一次乘以2，3，5；一次循环；
例如：result 1
arr2:2
arr3:3
aer5:5
取出最小值2放入结果队列，再用这个数乘以2，3，5
   result 1,2
arr2:4
arr3:3,6
arr4:5,10
取出最小值3放入结果队列，再用这个数乘以2，3，5
result 1,2,3
arr2:4,6
arr3:6,9
arr5:5,10,15
..........
一次下去，知道结果队列的长度等于需要的第n个丑数
这个方法用四个集合来存放，而且回出现相同的数，所以进行简化，尝试只用一个集合来操作，将集合arr2,arr3,arr5都放到结果集合里。因为三个暂时集合里的数都是丑数，只是暂时未加入结果集合，那么可以用结果集合表示暂时集合。
用p2,p3,p5表示暂时集合在结果集合中的位置下标
result 1
p2=p3=p5=0;
result[p2]=1*2=2
result[p3]=result[0]*3=3
result[p5]=result[0]*5=5;
将2加入result ,移动p2指针
result 1,2
p2=1;p3=p5=0;
result[p2]=2*2=4
result[p3]=result[0]*3=3
result[p5]=result[0]*5=5;
将3加入结果集合，移动p3指针
........
**代码如下**
`import java.util.ArrayList;
public class Solution {
    public int GetUglyNumber_Solution(int index) {
        if(index<7){
            return index;
        }
        int newNumber=1;
        ArrayList<Integer>arr=new ArrayList<Integer>();
        int p2,p3,p5;
        p2=p3=p5=0;
        arr.add(1);
        for(int i=0;i<index;i++){
            newNumber=Math.min(arr.get(p2)*2,Math.min(arr.get(p3)*3,arr.get(p5)*5));
            if(newNumber==arr.get(p2)*2){
                p2++;
            }
            if(newNumber==arr.get(p3)*3){
                p3++;
            }
            if(newNumber==arr.get(p5)*5){
                p5++;
            }
            arr.add(newNumber);
        }
        return arr.get(index-1);
    }
}
`






