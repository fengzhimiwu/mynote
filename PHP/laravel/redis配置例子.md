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

default????????????Redis????????????????????????????????????????????????app('redis.connection')???????????????????????????????????????

mydefine??????????????????Redis??????????????????????????????app('redis')->connection('mydefine')??????????????????????????????

mycluster1??????????????????Redis??????????????????????????????app('redis')->connection('mycluster1')??????????????????????????????