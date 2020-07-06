# php实现几种常见算法

* 排序
 * [冒泡排序](#冒泡排序)
 * [快速排序](#快速排序)
 * [选择排序](#选择排序)
* 查找
 * [二分法查找](#二分法查找)
 * [递归](#递归)
 * [顺序查找](#顺序查找)
* 其它
 * [乘法口诀](#乘法口诀)
 * [寻最小的n个数](#寻最小的n个数)
 * [寻相同元素](#寻相同元素)
 * [抽奖](#抽奖)
 * [数组反转](#数组反转)
 * [随机打乱数组](#随机打乱数组)
 * [寻找最小元素](#寻找最小元素)

### 公共方法
```php
//创建数据
function createData($num) {
    $arr = [];
    for ($i = 0; $i < $num; $i++) {
        $arr[$i] = rand($i, 1000);
    }
    return $arr;
}

//打印输出数组
function printSortArr($fun, $num = 10) {
    $data = createData($num);
    $dataString = implode(',', $data);
    echo "原数据:{$dataString}" . PHP_EOL;
    $arr = $fun($data);
    echo "算法[{$fun}]数据打印" . PHP_EOL;
    foreach ($arr as $key => $value) {
        # code...
        echo $value . PHP_EOL;
    }
}

```

### 冒泡排序

冒泡排序的原理：一组数据，比较相邻数据的大小，将值小数据在前面，值大的数据放在后面。

思路就是递归，当前一个数大于(或小于)后一个数的时候，将这两个数互换位置，最大的数就在最上面，下一个循环就是直接少一遍循环
步骤：

比较相邻的元素。如果第一个比第二个大，就交换他们两个。
对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
针对所有的元素重复以上的步骤，除了最后一个。
持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较

```php
<?php

/**
 @desc 冒泡排序(像水里的气泡一样最大的先出来，依次次大的出来，最后全部排序成功)
 原理：1.循环大一轮两辆比较将最大的一个数排到末尾结束
 2.循环第二轮，不过需要注意的是这个时候元素已经变成了除去最大数之外的循环，循环同上
 3.依次类推最后依次将最大的拍到后面，所有循环结束，则数据就是从大到小的排序
 **/

//冒泡排序
function mp_sort($arr) {
    $count = count($arr);
    for ($i = 0; $i < $count; $i++) {
        for ($j = 1; $j < $count - $i; $j++) {
            if ($arr[$j - 1] > $arr[$j]) {
                $temp = $arr[$j - 1];
                $arr[$j - 1] = $arr[$j];
                $arr[$j] = $temp;
            }
        }
    }
    return $arr;
}



printSortArr('mp_sort', 10);

// 其他
$arr = [1,43,54,62,21,66,32,78,36,76,39];
    function maopao($arr){
        $len = count($arr);
        for($i=1;$i<$len;$i++){
            echo $i.'aaaaaaa<br>';
            for($k=0;$k<$len-$i;$k++){
                echo $k.'bbbbbb<br>';
                if($arr[$k] > $arr[$k+1]){//当前一个数大于(或小于)后一个数的时候，将这两个数互换位置，最大的数就在最上面，下一个循环就是直接少一遍循环
                    $tmp = $arr[$k+1];
                    $arr[$k+1] = $arr[$k];
                    $arr[$k] = $tmp;
                }
            }
        }
        return $arr;
    }
    $a = maopao($arr);
    print_r($a);

```

### 快速排序

快速排序是对冒泡排序的一种改进。

实现思想是：通过一趟排序将待排记录分割成独立的两部分，其中一部分的关键字均比另一部分记录的关键字小，则可分别对这两部分记录继续进行快速排序，整个排序过程可以递归进行，以达到整个序列有序的目的。

简单来说就是：找到当前数组中的任意一个元素（一般选择第一个元素），作为标的，新建两个空数组，遍历整个数组元素，如果遍历到的元素比当前的元素要小，那么就放到左边的数组，否则放到右面的数组，然后再对新数组进行同样的操作。

```php
<?php

/***
 *@desc 快速排序
 *思想:通过一趟排序将要排序的数据分隔成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，
 然后再按此方法对这两部分的数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。
 **/
function quickSort(&$arr){
    if(count($arr)>1){
        //定义一基准数,再定义一个小于基准数的数组，和一个大于基准数的数组，然后再递归进行快速排序
        $k = $arr[0];//定基准数
        $x = [];//小于基准数的数组
        $y = [];//大于基准数的数组
        $size = count($arr);
        for($i=1;$i<$size;$i++){
            if($arr[$i]>$k){
                $y[] = $arr[$i];
            }elseif($arr[$i]<=$k){
                $x[] = $arr[$i];
            }
        }
        $x = quickSort($x);
        $y = quickSort($y);
        return array_merge($x,array($k),$y); 
    }else{
        return $arr;
    }
}

$arr = [2,78,3,23,532,13,67];
print_r(quickSort($arr));

```

### 选择排序

也是递归的思路，就是相当于拿出第一个数，然后依次跟后面数比较，判断最小值或者最大值的所在位置，然后把第一个数的位置跟比较出来的数的位置互换，依次轮询，排序

```php
<?php

/**
 *@desc 选择排序
 *原理：每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，知道全部待排序的数据元素排完。
 **/
function xz_sort($arr) {
    $len = count($arr);
    for ($i = 0; $i < $len - 1; $i++) {
        //每次找出最小的值的放到最前面，知道排序完
        $min = $i;
        for ($j = $i + 1; $j < $len; $j++) {
            if ($arr[$min] > $arr[$j]) {
                $min = $j;
            }
        }
        if ($min != $i) {
            $temp = $arr[$min];
            $arr[$min] = $arr[$i];
            $arr[$i] = $temp;
        }
    }
    return $arr;
}

printSortArr('xz_sort', 10);

// 其他
$arr = [1,43,54,62,21,66,32,78,36,76,39];
    function xuanze($arr){
        $len = count($arr);
        for($i=0;$i<$len;$i++){
            $pai = $i;
            for($k=$i+1;$k<$len;$k++){
                if($arr[$pai] > $arr[$k]){
                    $pai = $k;
                }
            }
            if($pai != $i){
                $tmp = $arr[$i];
                $arr[$i] = $arr[$pai];
                $arr[$pai] = $tmp;
            }

        }
        return $arr;
    }
    $a = xuanze($arr);
    print_r($a);

```
###插入排序

简单理解一下就是让第一个数跟下一个数比较，如果第一个数小于第二个数，然后两个数互换位置，否则直接跳出循环，这样就形成了排序列，然后再取出第二个数，让他从第二个循环的后面开始与整个之前的排序列一一比较，只要小于就互换位置，知道插入到合适的位置跳出循环

从第一个元素开始，该元素可以认为已经被排序

取出下一个元素，在已经排序的元素序列中从后向前扫描

如果该元素（已排序）大于新元素，将该元素移到下一位置

重复步骤3，直到找到已排序的元素小于或者等于新元素的位置

将新元素插入到该位置中

重复步骤2

```php
function charu($arr){
        $len = count($arr);
        for($i=1;$i<$len;$i++){
            $tmp = $arr[$i];
            for ($j=$i-1; $j >=0 ; $j--) { 
                //dump("{$i}----{$j}----{$tmp}----{$arr[$j]}");
                if($tmp > $arr[$j]){
                    $arr[$j+1] = $arr[$j];
                    $arr[$j] = $tmp;
                }else{
                    break;
                }
                dump($arr);
            }
        }
    }
    charu($arr);
```





### 二分法查找

实现思想：将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；否则利用中间位置记录将表分成前、后两个子表，如果中间位置记 录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。

```php
<?php

/***
 *@desc 二分法查找
 *条件：要求数组是有序的数组
 *原理：每次查找取中，跟要查找的目的数进行比较，如果小则在数组的开始端和结束端取中进行比较，如果
 如果大则中间段跟数组结尾端再次取中进行比较，依次类推。
 ***/
function dichotomyFindvalue($arr,$num){
    $end = count($arr);
    $start = 0;
    $middle = floor(($start+$end)/2);
    $i = 0;
    while($start<$end-1){
       if($arr[$middle]==$num){
            $i++;
            return [$middle+1,$i];
       }elseif($arr[$middle]<$num){
            $start = $middle;
            $middle = floor(($start+$end)/2);
       }else{
            $end = $middle;
            $middle = floor(($start+$end)/2);
       }
       $i++;
    }
    return false;
}


$arr = [
    24,37,42,59,69,78,82,84,91,93,96,102,103,106,113,116,117,118,125,128,130,131,133,138,139,140,142,144,146,150,155,156,157,158,166,167,168,
    172,174,175,177,178,179,181,186,187,189,190,191,192,194,198,199,200,201,202,204,205,206,207,213,218,220,223,224,226,227,228,230,231,
    232,233,236,238,241,242,244,245,246,247,249,251,252,255,257,258,259,260,263,264,265,266,267,268,270,272,273,274,275,278,280,281,282,
    283,285,286,287,288,290,291,292,297,299,300,302,304,305,306,307,308,309,310,311,312,313,314,315,317,318,319,320,321,324,325,326,327,
    328,332,334,336,337,338,339,340,341,342,343,344,346,347,348,349,350,351,352,353,354,356,357,358,360,361,362,363,364,365,366,367,368,
    369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,
    402,403,404,405,406,407,409,410,411,412,413,414,415,416,417,418,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,437,
    438,439,440,441,442,443,444,445,446,447,449,451,452,453,454,455,456,457,458,460,461,462,463,464,465,467,469,470,472,473,474,477,478,
    479,481,483,484,485,487,488,489,491,495,501,505,506,508,509,517,520,522,523,528,532,533,542,543,551,553,554,563,571,576,577,583,594
];

//数组要查找的数组的位置
$num = 312;//查找数字在数组中的第多少个位置
$res = dichotomyFindvalue($arr,$num);
if($res){
    echo "数字".$num.'在数组中第'.$res[0].'个中被找到,该查找共循环次数:'.$res[1];
    exit;
}
echo "要查询的数字不在数组中";
```
### 递归

```php
<?php

$areaList = [
    ['id'=>1,'name'=>'湖北省','pid'=>0,'son'=>''],
    ['id'=>2,'name'=>'广东省','pid'=>0,'son'=>''],
    ['id'=>3,'name'=>'湖南省','pid'=>0,'son'=>''],
    ['id'=>4,'name'=>'武汉市','pid'=>1,'son'=>''],
    ['id'=>5,'name'=>'荆州市','pid'=>1,'son'=>''],
    ['id'=>6,'name'=>'宜昌市','pid'=>1,'son'=>''],
    ['id'=>7,'name'=>'咸宁市','pid'=>1,'son'=>''],
    ['id'=>8,'name'=>'仙桃市','pid'=>1,'son'=>''],
    ['id'=>9,'name'=>'潜江市','pid'=>1,'son'=>''],
    ['id'=>10,'name'=>'深圳市','pid'=>2,'son'=>''],
    ['id'=>11,'name'=>'广州市','pid'=>2,'son'=>''],
    ['id'=>12,'name'=>'珠海','pid'=>2,'son'=>''],
    ['id'=>13,'name'=>'佛山市','pid'=>2,'son'=>''],
    ['id'=>14,'name'=>'长沙市','pid'=>3,'son'=>''],
    ['id'=>15,'name'=>'岳阳市','pid'=>3,'son'=>''],
    ['id'=>16,'name'=>'株洲市','pid'=>3,'son'=>''],
    ['id'=>17,'name'=>'衡阳市','pid'=>3,'son'=>''],
];

function recursive($arr,$pid=0){
    $tree = [];
    foreach($arr as $k=>$v){
       if($v['pid'] == $pid){
           $v['son'] = recursive($arr,$v['id']); 
           $tree[] = $v;
       }    
        
    }
    return $tree;
}
print_r(recursive($areaList));
```
### 顺序查找

```php
<?php

/**
 *@desc 顺序查找某个数是否在数组中
 顺序查找：即循环查找
 **/

$arr = [2,100,89,67,34,78,900,234,675];
$findNum = 34;
$flag = false;
foreach($arr  as $k=>$v){
    if($v==$findNum){
        echo "查到了，键值为$k";
        $flag = true;
        break;
    }
}
if($flag==false){
    echo "未找到该数";
}

```
### 乘法口诀
```php
<?php
/**
 *@desc 乘法口诀
 **/
for($i=1;$i<=9;$i++){
    for($j=1;$j<=$i;$j++){
        if($j==$i){
            echo $j.'*'.$i.'='.$j*$i."\n";
        }else{
            echo $j.'*'.$i.'='.$j*$i.'  ';
        }
      }
}
echo "\n\n---------------------------\n\n";
$word = ['1'=>'一','2'=>'二','3'=>'三','4'=>'四','5'=>'五','6'=>'六','7'=>'七','8'=>'八','9'=>'九',
    '10'=>'十','12'=>'十二','14'=>'十四','15'=>'十五','16'=>'十六','18'=>'十八','20'=>'二十','21'=>'二十一','24'=>'二十四','25'=>'二十五','27'=>'二十七','28'=>'二十八','30'=>'三十',
   '31'=>'三十一', '32'=>'三十二','35'=>'三十五','36'=>'三十六','40'=>'四十','42'=>'四十二','45'=>'四十五','48'=>'四十八','49'=>'四十九','52'=>'五十二','54'=>'五十四','56'=>'五十六',
    '63'=>'六十三','64'=>'六十四','72'=>'七十二','81'=>'八十一'
];
for($i=1;$i<=9;$i++){
    for($j=1;$j<=$i;$j++){
        $num = $j*$i;
        if($j==$i){
            echo $word[$j].$word[$i].'得'.$word[$num]."\n";
        }else{
            echo $word[$j].$word[$i].'得'.$word[$num].'    ';
        }
        unset($num);
      }
}
```
### 寻最小的n个数

```php
<?php
/**
 *@desc 选择数组中最小的n个数
 ***/
function get_min_array($arr, $k)
{
    $n = count($arr);
    $min_array = array();
    for ($i = 0; $i < $n; $i++) {
        if ($i < $k) {
            $min_array[$i] = $arr[$i];
        } else {
            if ($i == $k) {
                $max_pos = get_max_pos($min_array);
                $max = $min_array[$max_pos];
            }
            if ($arr[$i] < $max) {
                $min_array[$max_pos] = $arr[$i];
                $max_pos = get_max_pos($min_array);
                $max = $min_array[$max_pos];
            }
        }
    }
    return $min_array;
}
/* 获取数组中值最大的位置
* @param array $arr
* @return array
    */
function get_max_pos($arr)
{
    $pos = 0;
    //echo '$pos:'.$pos.PHP_EOL;
    for ($i = 1; $i < count($arr); $i++) {
        if ($arr[$i] > $arr[$pos]) {
            //echo '$i:'.$i.'--$post:'.$pos.PHP_EOL;
            $pos = $i;
        }
    }
    return $pos;
}
$array = [1, 100, 20, 22, 33, 44, 55, 66, 23, 79, 18, 20, 11, 9, 129, 399, 145,88,56,84,12,17];
//$min_array = get_min_array($array, 10);
//print_r($min_array);
$arr = [130,2,4,100,89,8,99];
$num = get_max_pos($arr);
echo $num;
print_r($arr[$num]);

```
### 寻相同元素

```php
<?php
$arr1 = [2,4,5,7,8,9,17];
$arr2 = [6,7,9,12,15,16,17];
/**
 *@desc 找出两个有序数组的相同元素出来
 **/
function findCommon($arr1,$arr2)
{
    $sameArr = [];
    $i = $j = 0;
    $count1 = count($arr1);
    $count2 = count($arr2);
    while($i<$count1 && $j<$count2){
        if($arr1[$i]<$arr2[$j]){
            $i++;
        }elseif($arr1[$i]>$arr2[$j]){
            $j++;
        }else{
            $sameArr[] = $arr1[$i];
            $i++;
            $j++;
        }
    }
    if(!empty($sameArr)){
        $sameArr = array_unique($sameArr);
    }
    return $sameArr;
}
$result = findCommon($arr1,$arr2);
print_r($result);
```
### 抽奖

```php
<?php
$arr = [
    ['id'=>1,'name'=>'特等奖','v'=>1],
    ['id'=>2,'name'=>'二等奖','v'=>3],
    ['id'=>3,'name'=>'三等奖','v'=>5],
    ['id'=>4,'name'=>'四等奖','v'=>20],
    ['id'=>5,'name'=>'谢谢参与','v'=>71],
];
function draw($arr)
{
    $result = [];
    //计算总抽奖池的积分数
    $sum = [];    
    foreach($arr as $key=>$value){
       $sum[$key] = $value['v'];
    }
    $randSum = array_sum($sum);
    $randNum = mt_rand(1,$randSum);
    error_log('随机数数：'.$randNum);
    $count = count($arr);
    $s = 0;
    $e = 0;
    for($i=0;$i<$count;$i++){
        if($i==0){
            $s = 0;
        }else{
            $s += $sum[$i-1];
        }
        $e += $sum[$i];
        if($randNum>=$s && $randNum<=$e){
            $result = $arr[$i];      
        }
    }
    unset($sum);
    return $result;
}
print_r(draw($arr));
```
### 数组反转

```php
<?php
$arr = ['好','好','学','习','天','天','向','上'];
/**
 *@desc 将一维数组进行反转
 ***/
function reverse($arr)
{
   $n = count($arr);
   $left = 0;
   $right = $n-1;
   $temp = [];
   //首尾依次替换元素的值实现数组反转
   while($left<$right){
       $temp = $arr[$left];
       $arr[$left] = $arr[$right];
       $arr[$right] = $temp;
       $left++;
       $right--;
   }
   return $arr;
}
$result = reverse($arr);
print_r($result);
```
### 随机打乱数组

```php
<?php
/***
 *@desc 将一个数组中的元素随机打算
 **/
function randShuffle($arr)
{
   $n = count($arr);
   $temp = [];
   for($i=0;$i<$n;$i++){
       $randNum = rand(0,$n-1);
       if($randNum!=$i){
           $temp = $arr[$i];
           $arr[$i] = $arr[$randNum];
           $arr[$randNum] = $temp;
           unset($temp);
       }   
   }
   return $arr;
}
$arr = ['a','c','test','cofl',1,9,48];
$result = randShuffle($arr);
//print_r($result);
/*
 *@desc 洗牌算法
 **/
function wash_card($cardNum){
   $n = count($cardNum);
   $temp = [];
   for($i=0;$i<$n;$i++){
       $randNum = rand(0,$n-1);
       if($randNum!=$i){
           $temp = $cardNum[$i];
           $cardNum[$i] = $cardNum[$randNum];
           $cardNum[$randNum] = $temp;
           unset($temp);
       }   
   }
   return $arr;
}
$cardNum = [];
```
### 寻找最小元素

#### 方式一
```php
<?php
$arr = [9,56,789,45,35,12,2,88,852,963,456,785,123,456,852,423,965,3,444,555,654,743,982];
$count = count($arr);
$minValue = 0;
$minIndex = 0;
for($i=0;$i<$count;$i++){
    $n = $i+1;
    if($n<$count){
        if($arr[$minIndex]<$arr[$n]){
            $minIndex = $minIndex;
            $minVlaue = $arr[$minIndex];
        }else{
            $minIndex = $n;
            $minValue = $arr[$n];
        }
    }
}
echo "最小值的索引为：".$minIndex.',最小值为:'.$minValue;
```

#### 方式二

```php
<?php
$arr = [10,15,2,7,8];
$count = count($arr);
$min = 0;//最小元素索引值标示
for($i=1;$i<$count;$i++){
    if($arr[$min]>$arr[$i]){
        $min = $i;
    }
}
echo "数组中最小值的索引为:".$min.',最小值为:'.$arr[$min];
```

### 约瑟夫环问题

```php
// 方法一
function test1($arr, $n)
{
    $i = 1;
    while(1) {
        if (count($arr) == 1) return $arr[$i - 1];
        if ($i % $n != 0) {
            array_push($arr, $arr[$i - 1]);
        }
        unset($arr[$i - 1]);
        $i++;
    }
}
$arr = [1,2,3,4,5];
var_dump(test1($arr, 3));

// 方法二

```

### 斐波那契数列

```php
//方法一  使用原始方法
function fibo(i)
{
    if(i == 0){
        return 1;
    }else if(i == 1){
        return 1;
    }else{
        total = fibo(i-1) + fibo(i-2);
        return total;
    }
}
//方法二 使用迭代解法, 将每一次执行的数据保存起来,时间会大大缩短
function  fibo1(i)
{
    if(i<=1){
        return 1;
    }
    var pre = 1;
    var prepre = 1;
    for(j = 2; j <= i; j++){
        total = pre + prepre;
        prepre = pre;
        pre = total;
    }
    return total;
}
```





### 扩展阅读

- [PHP 冒泡排序](https://www.cnblogs.com/wgq123/p/6529450.html)
- [php实现快速排序](https://www.cnblogs.com/wangjingwangjing/p/5241486.html)
- [PHP实现各种经典算法](https://www.cnblogs.com/hellohell/p/5718175.html)
- [PHP常见算法-面试篇](http://www.cnblogs.com/zswordsman/p/5824599.html)
- [php实现二分查找法](https://www.cnblogs.com/wangjingwangjing/p/5206711.html)