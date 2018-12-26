#题目
---
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。
#思路
---
1.第一想法就是先进行排序，然后遍历输出符合结果的数。选用冒泡排序来做，理解简单，复杂度为O(n*k)
`import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList <Integer>result=new ArrayList<Integer>();
        if(k>input.length){
            return result;
        }
        for(int i=0;i<k;i++){
            for(int j=0;j<input.length-1-i;j++){
                if(input[j]<input[j+1]){
                    int temp=input[j];
                    input[j]=input[j+1];
                    input[j+1]=temp;
                }
            }
            result.add(input[input.length-1-i]);
        }
        return result;
    }
}`
2.第二反应则是用一个大小为k的大顶堆，先将前k个数用来建立大顶堆，然后以后的数跟堆顶比较，如果该数小于堆顶，则删除堆顶元素，将该数加入堆，调整堆，一次遍历，最后得到的堆就为最小的k个数。
`import java.util.*;
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer>result=new ArrayList<Integer>();
        if(k>input.length||k==0){
            return result;
        }
        Comparator <Integer>datacomparator=new Comparator<Integer>(){
            public int compare(Integer o1,Integer o2){
                return o2.compareTo(o1);
            }
        };
        PriorityQueue<Integer>maxheap=new PriorityQueue<Integer>(k,datacomparator);
        for(int i=0;i<input.length;i++){
            if(maxheap.size()!=k){
                maxheap.offer(input[i]);
            }else if(maxheap.peek()>input[i]){
                int temp=maxheap.poll();
                maxheap.offer(input[i]);
            }
        }
        for(Integer i:maxheap){
            result.add(i);
        }
        return result;
    }
}`