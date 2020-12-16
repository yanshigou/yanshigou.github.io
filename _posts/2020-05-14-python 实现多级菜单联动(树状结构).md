---
title: "python 实现多级菜单联动(树状结构)"
date: 2020-05-14 18:58
author: dzt
subtitle: python 实现多级菜单联动(树状结构)
tags:
  - Django
  - python
---


python 实现多级菜单联动(树状结构)

```python
level = request.query_params.get('level', "99")
                menu = MenuModel.objects.filter(menu_level_id__lte=level).values().order_by('menu_level_id')
                menuList = []
                for m in menu:
                    menuList.append({
                        "menuId": m.get('menu_id'),   # 菜单id
                        "menuPath": m.get('menu_path'),
                        "menuName": m.get('menu_name'),
                        "menuStatus": m.get('menu_status'),
                        "menuLevelId": m.get('menu_level_id'),  #  等级int
                        "menuParentId": m.get('menu_parent_id'),  # 父级菜单id
                        "createTime": m.get('create_time'),
                    })
                # print(roleList)
                menuDict = {}
                for mDict in menuList:
                    mDict['children'] = []
                    menuDict[mDict.get('menuId')] = mDict

                # print(roleDict)
                menuTreeList = []

                for menuId, mDict in menuDict.items():
                    roleParentId = mDict.get('roleParentId')  # 父级ID
                    if not roleParentId:  # 根节点为空
                        menuTreeList.append(mDict)
                    else:
                        mDict[roleParentId]['children'].append(mDict)

                print(menuTreeList)
                return http_response(data={"data": menuTreeList})
```



数据

```
{
    "data": [
        {
            "roleId": "ro40c0f15a952411eaae044ccc6affa1c1",
            "roleCode": "superadmin",
            "roleName": "超级管理员",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-12T12:00:00",
            "children": [
                {
                    "roleId": "ro40c0f15a952411eaae044ccc6affa1c2",
                    "roleCode": "admin",
                    "roleName": "管理员",
                    "roleLevelId": 2,
                    "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c1",
                    "roleSsStaff": true,
                    "createTime": "2020-05-12T12:00:00",
                    "children": [
                        {
                            "roleId": "ro40c0f15a952411eaae044ccc6affa1c3",
                            "roleCode": "user",
                            "roleName": "用户",
                            "roleLevelId": 3,
                            "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c2",
                            "roleSsStaff": false,
                            "createTime": "2020-05-12T12:00:00",
                            "children": [
                                {
                                    "roleId": "ro40c0f15a952411eaae044ccc6affa1c5",
                                    "roleCode": "teammember",
                                    "roleName": "队员",
                                    "roleLevelId": 4,
                                    "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c3",
                                    "roleSsStaff": false,
                                    "createTime": "2020-05-12T12:00:00",
                                    "children": []
                                }
                            ]
                        },
                        {
                            "roleId": "ro40c0f15a952411eaae044ccc6affa1c4",
                            "roleCode": "teamleader",
                            "roleName": "队长",
                            "roleLevelId": 3,
                            "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c2",
                            "roleSsStaff": false,
                            "createTime": "2020-05-12T12:00:00",
                            "children": []
                        },
                        {
                            "roleId": "ro40c0f15a952411eaae044ccc6affa1c6",
                            "roleCode": "referee",
                            "roleName": "裁判",
                            "roleLevelId": 3,
                            "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c2",
                            "roleSsStaff": false,
                            "createTime": "2020-05-12T12:00:00",
                            "children": []
                        }
                    ]
                },
                {
                    "roleId": "roa2fd2b5895d911eaadd14ccc6affa1c1",
                    "roleCode": "Cide",
                    "roleName": "测试",
                    "roleLevelId": 35,
                    "roleParentId": "ro40c0f15a952411eaae044ccc6affa1c1",
                    "roleSsStaff": true,
                    "createTime": "2020-05-14T19:54:16.317395",
                    "children": []
                }
            ]
        },
        {
            "roleId": "ro59ab898895d911ea99004ccc6affa1c1",
            "roleCode": "123",
            "roleName": "123",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-14T19:52:13.308735",
            "children": []
        },
        {
            "roleId": "ro0cb915ac95dd11eab3e14ccc6affa1c1",
            "roleCode": "124124124",
            "roleName": "123124124",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-14T20:18:42.199874",
            "children": []
        },
        {
            "roleId": "roa155f89895de11ea85e84ccc6affa1c1",
            "roleCode": "123",
            "roleName": "ceshi",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-14T20:30:01.027572",
            "children": []
        },
        {
            "roleId": "robe325a8895de11ea9e184ccc6affa1c1",
            "roleCode": "123",
            "roleName": "ceshi",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-14T20:30:49.448076",
            "children": []
        },
        {
            "roleId": "ro27b6a9ee95e011ea86e64ccc6affa1c1",
            "roleCode": "44",
            "roleName": "1234444",
            "roleLevelId": 1,
            "roleParentId": "",
            "roleSsStaff": true,
            "createTime": "2020-05-14T20:40:55.973691",
            "children": []
        }
    ],
    "success": true,
    "msg": null,
    "code": 200
}
```



事务回滚

在m保存成功之后，rbm保存失败时启动事务回滚 均不保存

```
from django.db import transaction

requestData = request.data.copy()
roleId = "ro" + "".join(str(uuid.uuid1()).split('-'))
requestData['roleId'] = roleId
m = RoleModelSerializer(data=requestData)
with transaction.atomic():  # 事务回滚
	if m.is_valid():
    	m.save()
        menuIdList = request.data.get("menuIdList")

		for menuId in menuIdList:
            rbm = RoleBindMenuModelSerializer(data={"menuId": menuId[0], "roleId": roleId})
            if rbm.is_valid():
                rbm.save()

            else:
                errors = m.errors.values()
                errors = rbm.errors.values()
                print(rbm.errors)
                print(m.errors)
                return http_response(data={"errors": errors}, msg="权限菜单绑定失败", success=False, code=400)
         return http_response(msg="新增权限成功")
```

