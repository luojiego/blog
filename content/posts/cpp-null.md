---
title: C++ Mysql null 引发的血案
date: 2020-06-03 09:11:51
tags:
- C++
- MySQL
---
# 代码
## 出错代码段
```cpp
bool CGameServer::getAccountInfoByUserId(int user_id, UserAccountData& data) {                                                                                          
    if (connectToLocalDB()) {
        log_error("connect to db failed");
        return false;
    }

    TLIB_DB_LINK *pTempDBLink = &m_stLDBLink;

    snprintf(pTempDBLink->sQuery[0], sizeof(pTempDBLink->sQuery[0]), 
        "select bind_mark, account, password, state, device_mark, country, tunnel, \
        register_version, transfer_code, bind_equipment from account_data where \
        user_id=%d", user_id);
    //......
                if (pTempDBLink->stRow[i]) {
                    data._transfer_code = pTempDBLink->stRow[i++];
                }

                if (pTempDBLink->stRow[i]) {
                    data._bind_equipment = atoi(pTempDBLink->stRow[i++]);
                }
   //......
```
<!--more-->
## 问题代码段
```cpp
int CGameServer::setBindStatAndEquipment(int user_id, const std::string& equipment) {                                                                                   
    if (connectToLocalDB()) {
        log_error("connect to db failed");
        return -1;
    }    

    TLIB_DB_LINK *pTempDBLink = &m_stLDBLink;

    unsigned int equipment_size = 2 * equipment.size() + 2; 
    char* equipment_object = (char*)malloc(equipment_size);
    mysql_real_escape_string(&pTempDBLink->stMysqlConn.stMysql, (char*)equipment_object, 
        (char*)equipment.c_str(), equipment.size());

    snprintf(m_stLDBLink.sQuery[0], sizeof(m_stLDBLink.sQuery[0]), 
            "UPDATE account_data SET device_mark='%s', bind_equipment=1, \
            transfer_code=null WHERE user_id=%d", equipment_object, user_id);
            
    //......
}
```
# 错误分析
由于设置了 **transfer_code** 为 **null**，导致在获取时，由于数据为 **null**，导致 **bind_equipment** 也无法正常获取到值。

细心的小伙子，想必你也看到了，SET 的时候 **transf_code = null 和 bind_equipment** 恰好是一组，所以后续获取一定就出错了。

## 为什么叫血案
- 我自测未发现；
- 测试也未发现；

**最终这个问题被搞到线上环境了**

### 为什么？
日本方需求如下：
用户在 A 设备上获取“迁移码”，在 B 设备上使用该**迁移码**，A 设备则不能再登录游戏，此后该账号只能在 B 设备上使用。若想切换成其它设备，只需要在 B 设备上再获取“迁移码”在其它机器上使用即可

#### 错误复现
- 用户在初始获取迁移码时，**getAccountInfoByUserId** 会被调用，此时会创建新的迁移码；
- 在另外的设备上使用迁移码，**setBindStatAndEquipment** 会被调用；
- 此时逻辑是正确的；
- 但是在新的设备上获取迁移码时，**getAccountInfoByUserId** 会因为 **setBindStatAndEquipment** 设置的数据，导致获取的 **bind_equipment** 为 **0**，**导致设备绑定的逻辑失效**。


