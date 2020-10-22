---
layout: post
title: 将扁平列表数据转换成树状列表结构数据
subtitle: 将扁平列表数据转换成树状列表结构数据的工具
date: 2020-10-22 17:55:55.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
 - Java
 - 工具集
---

数据库中对于具备「层级结构」数据的存储一般都是存储成扁平的结构，通过父ID来表示父子节点数据的层级关系。例如从数据库查询出来的数据列表是这样的（已转成json格式）：

```json
[
  {
    "code": "USER_MGR",
    "name": "用户管理",
    "parentCode": ""
  },
  {
    "code": "USER_DETAIL",
    "name": "用户详情页",
    "parentCode": "USER_MGR"
  },
  {
    "code": "SYS_CONFIG",
    "name": "系统设置",
    "parentCode": ""
  },
  {
    "code": "AUTH_CONFIG",
    "name": "权限设置",
    "parentCode": "SYS_CONFIG"
  },
  {
    "code": "ROLE_CONFIG",
    "name": "角色设置",
    "parentCode": "AUTH_CONFIG"
  }
]
```

我们需要把这些数据通过页面展示出该有的层级出来，类似这样的：

![tree.png](/assets/images/2020-09/tree.png)

所以我们给到页面的数据应该带有层级的json数据，像这样的：

```json
[
  {
    "code": "USER_MGR",
    "name": "用户管理",
    "parentCode": "",
    "child": [
      {
        "code": "USER_DETAIL",
        "name": "用户详情页",
        "parentCode": "USER_MGR",
        "child": []
      }
    ]
  },
  {
    "code": "SYS_CONFIG",
    "name": "系统设置",
    "parentCode": "",
    "child": [
      {
        "code": "AUTH_CONFIG",
        "name": "权限设置",
        "parentCode": "SYS_CONFIG",
        "child": [
          {
            "code": "ROLE_CONFIG",
            "name": "角色设置",
            "parentCode": "AUTH_CONFIG",
            "child": []
          }
        ]
      }
    ]
  }
]
```

为了方便将扁平的列表数据转换成树状的层级结构数据，自己写了个通用的抽象工具类：[点击下载](/assets/files/java/AbstractTreeNode.java)

```java
package com.pandaq.utils;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 扁平列表数据结构转换成树状数据结构的工具。<br /><br />
 * 使用方式：<br /><br />
 *
 * 1. 原始数据节点继承 <code>AbstractTreeNode<code/> 抽象类：<code>public class MyNode extends AbstractTreeNode&lt;String, MyNode&gt;</code> ；<br/>
 * 2. 原始数据节点覆写 <code>nodeID()</code> 和 <code>parentNodeID()</code> 两个方法，分别返回节点ID和父节点ID；<br/>
 * 3. 假设有原始节点数据列表 <code>List&lt;MyNode&gt; input;</code> 使用 <code>List&lt;MyNode&gt; output = MyNode.transfer(input)</code> 静态方法调用方式得到output，即树状结构列表数据。
 *
 * @author zhanghong.lu@foxmail.com
 * @date 2020-10-21 21:56
 */
public abstract class AbstractTreeNode<ID, N> {

    private List<N> child = new ArrayList<>();

    /**
     * 获取节点的子节点列表
     * @return 返回节点的子节点列表
     */
    public List<N> getChild() {
        return child;
    }

    /**
     * 获取节点的ID，需要具备唯一性。子类须实现该方法，返回节点ID
     * @return 返回节点ID
     */
    protected abstract ID nodeID();

    /**
     * 获取父节点ID，需要具备唯一性。子类须实现该方法，返回父节点ID
     * @return 返回父节点ID
     */
    protected abstract ID parentNodeID();

    /**
     * 转换方法工具
     * @param list 扁平的原始数据列表
     * @return 返回树状结构数据list
     */
    public static <ID, N extends AbstractTreeNode<ID, N>> List<N> transfer(List<N> list) {
        // 将list数据映射到Map中，Key为节点ID，Value为节点实例N
        Map<ID, N> allMap = new HashMap<>();
        for (N node : list) {
            allMap.put(node.nodeID(), node);
        }
        // 转换成树状结构数据
        List<N> nodeList = new ArrayList<>();
        for (N node : list) {
            // 获取当前节点的父节点
            N parentNode = allMap.get(node.parentNodeID());
            if (parentNode == null) {
                // 没有父节点，说明这个节点是顶级节点：将这个节点直接放入结果中
                nodeList.add(allMap.get(node.nodeID()));
            } else {
                // 有父节点，取父节点的child并将该节点添加到父节点的child列表中
                parentNode.getChild().add(allMap.get(node.nodeID()));
            }
        }
        return nodeList;
    }

}
```

使用例子：

```java
public class MyNode extends AbstractTreeNode<String, MyNode> {
    private String code;
    private String name;
    private String parentCode;
    // getter/setter ...
    @Override
    public String nodeID() {
        return this.getCode();
    }

    @Override
    public String parentNodeID() {
        return this.getParentCode();
    }
}

public class Test {
    public static void main(String[] args) {
        List<MyNode> myNodeList = getFromDB(); // 从DB中获取到的扁平数据列表
        List<MyNode> treeList = MyNode.transfer(myNodeList);
        System.out.println(JSON.toJSONString(treeList));
    }
}
```

完。