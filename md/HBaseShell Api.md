# HBase Shell Rest Api

## List

### GET /list

* Response 200 (application/json)

	* Body

			{
				"data": {
			    	"table": [
			      		"Table Name 1",
			      		"Table Name 2",
			      		"Table Name 3",
			      		...
			      		"Table Name n"
			    	]
			  	},
			  	"code": 0,
			  	"success": true
			}

## Put

### POST /put/tables/{Table Name}

	注意事項：
		1. 使用該 API 須設定 Content-Type

* Request (application/json)
	* Parameters
	
		| Parm       | Type   | Required | Descript     |
		| ---------- | ------ | -------- | -----------  |
		| Table Name | Path   | true     | Table Name   |
 
	* headers
		
			Content-Type: application/json
		 
	* Body

			{
				"data": {
					"rows": {
						"Row Key 1": {
			      			"Column Name 1": "value",
							"Column Name 2": "value",
			      			...
							"Column Name n": "value"
			    		},
						...
						"Row Key n": {
			      			"Column Name 1": "value",
			      			"Column Name 2": "value",
							...
							"Column Name n": "value"
						}
					}
			  	}
			}

* Response 200 (application/json)

	* Body

			{
				"data": {},
			  	"code": 0,
			  	"success": true
			}

## Scan

### GET /scan/tables/{Table Name}{?start, stop, columns, reserved, filter}

	注意事項：
		1. 如 limit 大於 3000 將視為 3000

* Request
	* Parameters
	
		| Parm       | Type   | Required | Descript     | Default value |
		| ---------- | ------ | -------- | -----------  | ------------- |
		| Table Name | Path   | true     | Table Name   | -             |
		| start      | Query  | -        | Start RowKey | -             |
		| stop       | Query  | -        | Stop RowKey  | -             |
		| limit      | Query  | -        | -            | 10            |
		| columns    | Query  | -        | -            | -             |
		| reserved   | Query  | -        | -            | false         |
		| filter     | Query  | -        | Scan Filter  | -             |
 			
* Response 200 (application/json)

	* Body

			{
				"data": {
					"rows": {
						"Row Key 1": {
			      			"Column Name 1": "value",
							"Column Name 2": "value",
			      			...
							"Column Name n": "value"
			    		},
						...
						"Row Key n": {
			      			"Column Name 1": "value",
			      			"Column Name 2": "value",
							...
							"Column Name n": "value"
						}
					}
			  	},
			  	"code": 0,
			  	"success": true
			}
			
## Get

### GET /get/tables/{Table Name}/rows/{Row Key}{?columns}

* Request
	* Parameters
	
		| Parm       | Type   | Required | Descript     |
		| ---------- | ------ | -------- | -----------  |
		| Table Name | Path   | true     | Table Name   |
		| Row Key    | Query  | true     | Row Key      |
		| columns    | Query  | -        | -            |
 
* Response 200 (application/json)

	* Body

			{
			  	"data": {
					"rows": {
						"{Row Key}": {
			      			"Column Name 1": "value",
							"Column Name 2": "value",
			      			...
							"Column Name n": "value"
			    		}
					}
			  	},
			  	"code": 0,
			  	"success": true
			}

## Delete

### DELETE /delete/tables/{Table Name}/rows/{Row Key}{?columns}

* Request
	* Parameters
	
		| Parm       | Type   | Required | Descript     |
		| ---------- | ------ | -------- | -----------  |
		| Table Name | Path   | true     | Table Name   |
		| Row Key    | Query  | true     | Row Key      |
		| columns    | Query  | -        | -            |
 	
* Response 200 (application/json)

	* Body

			{
				"data": {},
			  	"code": 0,
			  	"success": true
			}