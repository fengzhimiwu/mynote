1) yum install epel-release  

2) yum install nodejs  

3) yum install nodejs npm  

4) npm install elasticdump  -g

5) cd node_modules/elasticdump/bin 


```
elasticdump  --input=http://localhost:9200/mfu_toutiao --output=/data/test.json --type=data
elasticdump  --input=http://localhost:9200/mfu_toutiao --output=http://123.59.100.115:29200/mfu_toutiao --type=mapping
elasticdump  --input=http://localhost:9200/mfu_toutiao --output=http://123.59.100.115:29200/mfu_toutiao --type=data
```



ES创建索引

PUT请求:

http:// 10.17.7.240:29200/chop_shop/

 

body:

{

​	"mappings":{

​			"user_order": {

​	      "properties": {

​	        "orderid": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "ordernum": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "ordername":{

​	        	"type": "string",

​	        	"index": "not_analyzed"

​	        },

​	        "product_number": {

​	          "type": "long"

​	        },

​	        "normid": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "shoppageid": {

​	          "type": "long"

​	        },

​	        "userid": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "mobile": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "channelid": {

​	          "type": "integer"

​	        },

​	        "shopid": {

​	          "type": "long"

​	        },

​	        "provinceid": {

​	          "type": "long"

​	        },

​	        "addr_realname": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "addr_phone": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "addr_province": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "addr_city": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "success": {

​	          "type": "integer"

​	        },

​	        "addr_address": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "deliverytype": {

​	          "type": "integer"

​	        },

​	        "shippingremark": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "shippingtime": {

​	          "type": "string"

​	        },

​	        "leavemsg": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "title": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "payprice": {

​	          "type": "integer"

​	        },

​	        "itemcount": {

​	          "type": "integer"

​	        },

​	        "price": {

​	          "type": "integer"

​	        },

​	        "transportprice": {

​	          "type": "integer"

​	        },

​	        "totalprice": {

​	          "type": "integer"

​	        },

​	        "paytype": {

​	          "type": "integer"

​	        },

​	        "payorderid": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "paytime": {

​	          "type": "string"

​	        },

​	        "ordertime": {

​	          "type": "string"

​	        },

​	        "tradetime": {

​	          "type": "string"

​	        },

​	        "order_create": {

​	          "type": "string"

​	        },

​	        "order_modified": {

​	          "type": "string"

​	        },

​	        "status": {

​	          "type": "integer"

​	        },

​	        "reason": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​	        "first_categoryid": {

​	          "type": "integer"

​	        },

​	        "second_categoryid": {

​	          "type": "integer"

​	        },

​	        "espageid": {

​	          "type": "string",

​	           "index": "not_analyzed"

​	        },

​					"esshippingtime": {

​	          "type": "long"

​	        },

​					"espaytime": {

​	          "type": "long"

​	        },

​	        "esordertime": {

​	          "type": "long"

​	        },

​	        "estradetime": {

​	          "type": "long"

​	        },

​	        "esorder_create": {

​	          "type": "long"

​	        },

​	        "esorder_modified": {

​	          "type": "long"

​	        }

​		    }

​			}

​	}

}