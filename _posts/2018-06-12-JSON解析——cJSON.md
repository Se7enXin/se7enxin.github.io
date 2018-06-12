---
title: JSON解析——cJSON
description: How to use cJSON lib
categories:
- JSON
tags:
- JSON
- C
---

# 一、简介

### [cJSON Source Code](https://github.com/DaveGamble/cJSON)

> cJSON 是一个以ANSI C (C89) 标准编写的轻量级JSON解析库。

# 二、How To Use

cJSON包含两个文件：`cJSON.c`、`cJSON.h`。在代码中include`cJSON.h`即可。例如：

```c
#include "cJSON.h"
```

目录结构：

![cJSON-1](https://i.imgur.com/CzZhFcP.png)

#### cJSON_test.c:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "cJSON.h"

#define JSON_FILE_PATH    "./test.json"
#define JSON_FILE_SIZE    1024

int readValueFromFile(char* fileName, char* buff, int len)
{
    int ret = -1;
    FILE *fp = fopen(fileName, "r");
    if (fp == NULL) {
        printf("Unable to open file %s", fileName);
        return -1;
    } else {
        if (fread(buff, sizeof(char), len, fp)>0) {
            ret = 0;
        }
    }
    fclose(fp);
    return ret;
}

/**
 * [printJson 遍历JSON]
 */
void printJson(cJSON * root)
{
    char buf[500];
    memset(buf, 0, sizeof(buf));
    //遍历最外层json键值对
    for(int i=0; i<cJSON_GetArraySize(root); i++) {
        cJSON * item = cJSON_GetArrayItem(root, i);
        //如果对应键的值仍为cJSON_Object就递归调用printJson
        if(cJSON_Object == item->type) {
            printJson(item);
        }
        //值不为json对象就直接打印出键和值
        else {
            printf("%s:%s\n", item->string, cJSON_Print(item));
        }
    }
}

int main()
{
    cJSON *root = NULL;
    cJSON *item = NULL;//cjson对象

    char jsonStr[JSON_FILE_SIZE];
    memset(jsonStr, 0, sizeof(jsonStr));
    readValueFromFile(JSON_FILE_PATH, jsonStr, sizeof(jsonStr));
    // printf("jsonStr=%s\n", jsonStr);

    root = cJSON_Parse(jsonStr);
    if (!root)
    {
        printf("Error before: [%s]\n",cJSON_GetErrorPtr());
    }
    else
    {
        printf("\n%s\n", "print all:");
        printJson(root);
    }
    return 0;
}
```

#### JSON:
```javascript
{
    "station_name": "Module",
    "structure": {
        "module": {
            "num": 1,
            "rule": "023M",
            "sn_len": 24
        },
        "cell": {
            "num": 7,
            "rule": "NULL",
            "sn_len": 14
        }
    }
}
```

#### 输出结果：

```shell
print all:
station_name:"Module"
num:1
rule:"023M"
sn_len:24
num:7
rule:"NULL"
sn_len:14
```
