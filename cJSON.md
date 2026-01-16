# 库文件解析

## 创建对象

```c
/*创建一类别的cJSON对象*/
CJSON_PUBLIC(cJSON *) cJSON_CreateNull(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateTrue(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateFalse(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateBool(cJSON_bool boolean);
CJSON_PUBLIC(cJSON *) cJSON_CreateNumber(double num);
CJSON_PUBLIC(cJSON *) cJSON_CreateString(const char *string);

/*创建一个字符串，其中valuestring引用一个字符串cJSON_Delete不会释放它*/
CJSON_PUBLIC(cJSON *) cJSON_CreateStringReference(const char *string);

/*创建一个只引用其元素的对象/数组，以便cJSON_Delete不会释放它们*/
CJSON_PUBLIC(cJSON *) cJSON_CreateObjectReference(const cJSON *child);
CJSON_PUBLIC(cJSON *) cJSON_CreateArrayReference(const cJSON *child);

/*这些实用程序创建计数项数组。参数计数不能大于数字数组中的元素数，否则数组访问将不受限制*/
CJSON_PUBLIC(cJSON *) cJSON_CreateIntArray(const int *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateFloatArray(const float *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateDoubleArray(const double *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateStringArray(const char *const *strings, int count);

/* 创建cJSON对象 */
CJSON_PUBLIC(cJSON *) cJSON_CreateRaw(const char *raw);
CJSON_PUBLIC(cJSON *) cJSON_CreateArray(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateObject(void);	
```



## 添加项目

```c
/*辅助函数用于同时创建项目和向对象添加项目。它们在失败时返回添加的项或NULL*/
/*
* object是cJSON对象
* name的格式是字符串，含义是该项目的名字
* number是bool类型的参数，是内容
*/
CJSON_PUBLIC(cJSON*) cJSON_AddBoolToObject(cJSON * const object, const char * const name, const cJSON_bool boolean);

/*
* object是cJSON对象
* name的格式是字符串，含义是该项目的名字，例如"number"
* number是double类型的数字，是内容
*/
CJSON_PUBLIC(cJSON*) cJSON_AddNumberToObject(cJSON * const object, const char * const name, const double number);

/*
* object是cJSON对象
* name的格式是字符串，含义是该项目的名字
* number是字符串类型的参数，是内容，字符串的表示一般是"xxx"带有双引号
*/
CJSON_PUBLIC(cJSON*) cJSON_AddStringToObject(cJSON * const object, const char * const name, const char * const string);
CJSON_PUBLIC(cJSON*) cJSON_AddNullToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddTrueToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddFalseToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddRawToObject(cJSON * const object, const char * const name, const char * const raw);
CJSON_PUBLIC(cJSON*) cJSON_AddObjectToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddArrayToObject(cJSON * const object, const char * const name);

/*将项附加到指定的数组/对象*/
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToObject(cJSON *object, const char *string, cJSON *item);

/*将项的引用附加到指定的数组/对象。当您想将现有cJSON添加到新cJSON中，但不想损坏现有cJSON时，请使用此选项*/
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemReferenceToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemReferenceToObject(cJSON *object, const char *string, cJSON *item);

```

## 输出

```c
/*将cJSON实体渲染为文本以进行传输/存储*/
/*
* 形参item是cJSON对象
* 返回值是char *类型的指针
* 返回的内容就是将cJSON的内容赋值给char *指针
*/
CJSON_PUBLIC(char *) cJSON_Print(const cJSON *item);

/*将cJSON实体渲染为文本，以便在不进行任何格式化的情况下进行传输/存储*/
CJSON_PUBLIC(char *) cJSON_PrintUnformatted(const cJSON *item);

/*使用缓冲策略将cJSON实体渲染为文本。预缓冲区是对最终大小的猜测。猜对可以减少重新分配。fmt=0表示未格式化，=1表示格式化*/
CJSON_PUBLIC(char *) cJSON_PrintBuffered(const cJSON *item, int prebuffer, cJSON_bool fmt);

/*使用已在内存中分配的具有给定长度的缓冲区将cJSON实体渲染为文本。成功时返回1，失败时返回0*/
/*注意：cJSON在估计它将使用多少内存方面并不总是100%准确，因此为了安全起见，请比实际需要多分配5个字节*/
CJSON_PUBLIC(cJSON_bool) cJSON_PrintPreallocated(cJSON *item, char *buffer, const int length, const cJSON_bool format);




```

## 获取

```c
/*返回数组（或对象）中的项数*/
CJSON_PUBLIC(int) cJSON_GetArraySize(const cJSON *array);

/*从数组“array”中检索项目编号“index”。如果失败，则返回NULL*/
CJSON_PUBLIC(cJSON *) cJSON_GetArrayItem(const cJSON *array, int index);

/*从对象中获取项目“string”。不区分大小写*/
/*
* 参数object->cJSON对象
* 参数string->想要获取的项目名字
* 返回值为(cJSON *) 类型的指针
*对大小写不敏感
*/
CJSON_PUBLIC(cJSON *) cJSON_GetObjectItem(const cJSON * const object, const char * const string);

/*从对象中获取项目“string”。不区分大小写*/
/*
* 参数object->cJSON对象
* 参数string->想要获取的项目名字
* 返回值为(cJSON *) 类型的指针
*对大小写敏感
*/
CJSON_PUBLIC(cJSON *) cJSON_GetObjectItemCaseSensitive(const cJSON * const object, const char * const string);
CJSON_PUBLIC(cJSON_bool) cJSON_HasObjectItem(const cJSON *object, const char *string);

/*用于分析失败的解析。这将返回一个指向解析错误的指针。您可能需要查看几个字符才能理解它。当cJSON_Parse（）返回0时定义。cJSON_Parse（）成功时为0*/
CJSON_PUBLIC(const char *) cJSON_GetErrorPtr(void);

/*检查项目类型并返回其值*/
CJSON_PUBLIC(char *) cJSON_GetStringValue(const cJSON * const item);
CJSON_PUBLIC(double) cJSON_GetNumberValue(const cJSON * const item);


```



## 删除

```c
/*从数组/对象中删除/分离项目*/
/*Detach从JSON对象或数组中移除项，但不释放其内存。*/
/*Delete从JSON对象或数组中移除项，并释放其内存。*/
/*CaseSensitive表示的是对大小写敏感*/
CJSON_PUBLIC(cJSON *) cJSON_DetachItemViaPointer(cJSON *parent, cJSON * const item);
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromArray(cJSON *array, int which);
CJSON_PUBLIC(void) cJSON_DeleteItemFromArray(cJSON *array, int which);
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromObject(cJSON *object, const char *string);
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromObjectCaseSensitive(cJSON *object, const char *string);
CJSON_PUBLIC(void) cJSON_DeleteItemFromObject(cJSON *object, const char *string);
CJSON_PUBLIC(void) cJSON_DeleteItemFromObjectCaseSensitive(cJSON *object, const char *string);
```





## 其他函数

```c
/*以字符串形式返回cJSON的版本*/
CJSON_PUBLIC(const char*) cJSON_Version(void);

/*为cJSON提供malloc、realloc和free函数*/
CJSON_PUBLIC(void) cJSON_InitHooks(cJSON_Hooks* hooks);


/*内存管理：调用者始终负责从cJSON_Parse（使用cJSON_Delete）和cJSON_Print（根据需要使用stdlib free、cJSON_Hooks.free_fn或cJSON_free）的所有变体中释放结果。cJSON_PrintPreallocated是个例外，调用者对缓冲区负有全部责任*/
/*提供一个JSON块，这将返回一个可以查询的cJSON对象*/
CJSON_PUBLIC(cJSON *) cJSON_Parse(const char *value);   //解析数据包
CJSON_PUBLIC(cJSON *) cJSON_ParseWithLength(const char *value, size_t buffer_length);

/*ParseWithOpts允许您要求（并检查）JSON以null结尾，并检索指向解析的最后一个字节的指针*/
/*如果你在return_parse_end中提供了一个ptr并且解析失败，那么return_parce_end将包含一个指向错误的指针，因此将与cJSON_GetErrorPtr（）匹配*/
CJSON_PUBLIC(cJSON *) cJSON_ParseWithOpts(const char *value, const char **return_parse_end, cJSON_bool require_null_terminated);
CJSON_PUBLIC(cJSON *) cJSON_ParseWithLengthOpts(const char *value, size_t buffer_length, const char **return_parse_end, cJSON_bool require_null_terminated);

/*
*删除cJSON实体和所有子实体
*参数item是cJSON对象
*/
CJSON_PUBLIC(void) cJSON_Delete(cJSON *item);

/*这些函数检查项目的类型*/
CJSON_PUBLIC(cJSON_bool) cJSON_IsInvalid(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsFalse(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsTrue(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsBool(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNull(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNumber(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsString(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsArray(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsObject(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsRaw(const cJSON * const item);


```

## JSON *

```c
/* cJSON结构体 */
typedef struct cJSON
{
/*next/prev允许您遍历数组/对象链。或者，使用GetArraySize/GetArrayIntem/GetObjectItem*/
    struct cJSON *next;
    struct cJSON *prev;
   /*数组或对象项将有一个子指针指向数组/对象中的项链*/
    struct cJSON *child;

   /*项目的类型*/
    int type;

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw，则表示项目的内容 */
    char *valuestring;
    /*写入valueint是DEPRECATED，请改用cJSON_SetNumberValue*/
    int valueint;
   /*如果类型==cJSON_number，则表示项目的内容*/
    double valuedouble;

    /*如果该项是对象的子项或在对象的子项目列表中，则表示项目的内容*/
    char *string;
} cJSON;
```



# 创建JSON数据包

```c
cJSON *root = cJSON_CreateObject();   //创建一个cJSON对象
cJSON_AddStringToObject(root, "name", "STM32");    //添加字符串项目到cJSON对象中
cJSON_AddNumberToObject(root, "age",5);    //添加数字项目到cJSON对象中
cJSON_AddBoolToObject(root, "is_running", cJSON_True);  //添加Bool类型项目到cJSON对象

char *json_string = cJSON_Print(root);    //创建一个字符型指针，并通过cJSON_Print()函数赋值
printf("%s\n", json_string);
```



# 解析JSON数据包

```c
cJSON *parsed_json = cJSON_Parse(json_string);    //解析数据包
if (parsed_json == NULL) {     //检查数据包是否为空
    // 解析失败处理  
    printf("Parse error\n");    
} else {  
    cJSON *name = cJSON_GetObjectItem(parsed_json, "name");    //
    if (name != NULL) {  
        printf("Name: %s\n", name->valuestring);  
    }  
    // 继续解析其他数据...  
    cJSON_Delete(parsed_json);  
}
```



# 例子

```c
int8_t Parse_MqttCmd(uint8_t *data)
{
	// 寻找JSON数据的开始位置  
	const char *json_start = strstr((char *)data, "{"); 
	if (json_start == NULL) 
	{  
		printf("JSON data not found in the received string.\n");  
		return -1;  
	}  
	size_t json_length = strlen(json_start);
	
	// 分配内存并复制JSON数据  
	char *json_data = (char *)malloc(json_length + 1);  
	if (json_data == NULL) {  
		printf("Memory allocation failed.\n");  
		return -1;  
	}  
	strncpy(json_data, json_start, json_length);  
	json_data[json_length] = '\0'; // 添加null终止符  
 
	//解析JSON数据  
	cJSON *root = cJSON_Parse(json_data);  
	if (root == NULL)
	{  
		printf("Failed to parse JSON data.\n");  
		cJSON_free(json_data);  
		return -1;  
	}  
	// ... 在这里处理JSON数据 ...  
		
	// 获取并打印"command_name"字段的值  
	cJSON *name = cJSON_GetObjectItemCaseSensitive(root, "command_name");
 
	//判断"command_name"字段的值选择控制类型
	if (name && cJSON_IsString(name) && (name->valuestring != NULL))
	{  
		char * command_name = name->valuestring;
		printf("Name: %s \r\n", command_name);
		
		//灯光控制命令
		if(strstr(name->valuestring,"LED"))
		{
			cJSON *paras = cJSON_GetObjectItemCaseSensitive(root, "paras");/*获取obj中的paras的json*/
			if (paras && cJSON_IsObject(paras))
			{
				cJSON *sw_item = cJSON_GetObjectItemCaseSensitive(paras, "sw");/*开关状态*/  
				if (sw_item != NULL && cJSON_IsBool(sw_item))
				{  
					int sw_value = sw_item->valueint; // cJSON使用int来表示bool值  
					printf("sw: %d \r\n", sw_value); // 输出sw的值，1代表true，0代表false 
				}else
				{  
					printf("Failed to get 'sw' value or it's not a boolean.\r\n");  
				}  
				cJSON *val_item = cJSON_GetObjectItemCaseSensitive(paras, "val");  
				if (val_item != NULL && cJSON_IsNumber(val_item))
				{  
					int val = val_item->valueint; // cJSON使用int来表示bool值  
					printf("val: %d \r\n", val); // 输出sw的值，1代表true，0代表false  
				}else
				{  
					printf("Failed to get 'val' value or it's not a number. \r\n");  
				} 
			}					
		}
	}
	// 释放资源 
	cJSON_Delete(root);  
	cJSON_free(json_data);  
	return 1;  
}
```



