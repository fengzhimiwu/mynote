```php
<?php

/**
 * 计算两点之间的距离
 * @param $lng1 经度1
 * @param $lat1 纬度1
 * @param $lng2 经度2
 * @param $lat2 纬度2
 * @param int $unit m，km
 * @param int $decimal 位数
 * @return float
 */
function getDistance($lng1, $lat1, $lng2, $lat2, $unit = 2, $decimal = 2)
{

    $EARTH_RADIUS = 6370.996; // 地球半径系数
    $PI           = 3.1415926535898;

    $radLat1 = $lat1 * $PI / 180.0;
    $radLat2 = $lat2 * $PI / 180.0;

    $radLng1 = $lng1 * $PI / 180.0;
    $radLng2 = $lng2 * $PI / 180.0;

    $a = $radLat1 - $radLat2;
    $b = $radLng1 - $radLng2;

    $distance = 2 * asin(sqrt(pow(sin($a / 2), 2) + cos($radLat1) * cos($radLat2) * pow(sin($b / 2), 2)));
    $distance = $distance * $EARTH_RADIUS * 1000;

    if ($unit === 2) {
        $distance /= 1000;
    }

    return round($distance, $decimal);
}

// 起点坐标
$lng1 = 118.34593200683594;
$lat1 = 33.9527587890625;

// 终点坐标
$lng2 = 118.301612;
$lat2 = 33.936781;

$distance = getDistance($lng1, $lat1, $lng2, $lat2, 1);
echo $distance . 'm' . PHP_EOL; // 4457.64m

$distance = getDistance($lng1, $lat1, $lng2, $lat2);
echo $distance . 'km' . PHP_EOL; // 4.46km
```

