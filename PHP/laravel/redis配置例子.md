```
'redis' => [                                                                                    
    'client' => 'predis',                                                                       
    'default' => [                                                                              
         'host' => env('REDIS_HOST', '127.0.0.1'),                                               
         'password' => env('REDIS_PASSWORD', null),                                              
         'port' => env('REDIS_PORT', 6379),                                                      
         'database' => 0,                                                                        
    ], 
    'mydefine' => [                                                                              
         'host' => env('REDIS_HOST', '127.0.0.1'),                                               
         'password' => env('REDIS_PASSWORD', null),                                              
         'port' => env('REDIS_PORT', 6379),                                                      
         'database' => 4,                                                                        
    ],
    'clusters' => [                                                                             
        'mycluster1' => [                                                                       
            [                                                                                   
                'host' => env('REDIS_HOST', '127.0.0.1'),                                       
                'password' => env('REDIS_PASSWORD', null),                                      
                'port' => env('REDIS_PORT', 6379),                                              
                'database' => 1,                                                                
            ],                                                                                  
            [                                                                                   
                'host' => env('REDIS_HOST', '127.0.0.1'),                                       
                'password' => env('REDIS_PASSWORD', null),                                      
                'port' => env('REDIS_PORT', 6379),                                              
                'database' => 2,                                                                
            ],
            [                                                                                   
                'host' => env('REDIS_HOST', '127.0.0.1'),                                       
                'password' => env('REDIS_PASSWORD', null),                                      
                'port' => env('REDIS_PORT', 6379),                                              
                'database' => 3,                                                                
            ], 
       ],                                                                                      
    ],          
],
```

default是默认的Redis连接对象名，值是连接对象的参数；app('redis.connection')返回的就是该默认连接对象；

mydefine是笔者定义的Redis连接对象名；通过执行app('redis')->connection('mydefine')可以获取该连接对象；

mycluster1是笔者定义的Redis集群对象名；通过执行app('redis')->connection('mycluster1')可以获取该集群对象；