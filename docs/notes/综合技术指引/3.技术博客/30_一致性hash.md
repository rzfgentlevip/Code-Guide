---
title: 30、一致性Hash
icon: material-icon-theme:stackblitz
order: 30
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/一致性Hash/
---

## 为什么需要一致性Hash？

传统Hash的致命缺陷:

```java
// 传统Hash取模算法
public class TraditionalHash {
    public static void main(String[] args) {
        // 3台服务器
        String[] servers = {"S1", "S2", "S3"};
        
        // 10个数据键
        String[] keys = {"key1", "key2", "key3", "key4", "key5",
                        "key6", "key7", "key8", "key9", "key10"};
        
        System.out.println("=== 3台服务器时的分布 ===");
        for (String key : keys) {
            int serverIndex = Math.abs(key.hashCode()) % 3;
            System.out.println(key + " → " + servers[serverIndex]);
        }
        
        System.out.println("\n=== 增加1台服务器（共4台） ===");
        // 服务器数量变化
        for (String key : keys) {
            int serverIndex = Math.abs(key.hashCode()) % 4;  // 模数改变
            System.out.println(key + " → " + servers[serverIndex % 3]);
            // 注意：75%的数据需要重新分配！
        }
    }
}
```

问题分析：

1. 服务器从N台变为N+1台时，大约 N/(N+1) 的数据需要迁移 
2. 服务器从N台变为N-1台时，大约 (N-1)/N 的数据需要迁移 
3. 大规模数据迁移带来网络风暴和性能抖动


## 一致性Hash的核心思想

###  Hash环的构建
一致性Hash将数据和服务器都映射到一个环形空间（通常0~2^32-1）

```java
public class HashRingVisualization {
    // Hash环：0 到 2^32-1（43亿）
    private static final long RING_SIZE = 4294967295L;
    
    // 将Hash空间视为一个环
    // 0 --- 2^32-1
    // |           |
    // 环上的点可以无限添加
}
```

### 基础算法步骤

```java
public class BasicConsistentHash {
    /**
     * 一致性Hash核心算法步骤：
     * 1. 构建Hash环（0 ~ 2^32-1）
     * 2. 将服务器节点Hash到环上
     * 3. 将数据键Hash到环上
     * 4. 数据顺时针找到的第一个服务器节点
     */
    
    static class Node {
        String name;
        long hash;
        
        public Node(String name) {
            this.name = name;
            this.hash = hash(name);
        }
    }
    
    // 使用MD5作为Hash函数（示例）
    private static long hash(String key) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(key.getBytes());
            // 取前4字节作为32位整数
            long hash = 0;
            for (int i = 0; i < 4; i++) {
                hash = (hash << 8) | (digest[i] & 0xFF);
            }
            return hash & 0xFFFFFFFFL;  // 确保在0~2^32-1范围内
        } catch (Exception e) {
            return Math.abs(key.hashCode());
        }
    }
}
```

## 一致性Hash的核心原理

### 虚拟节点（Virtual Nodes）技术

解决节点分布不均问题：

```java
public class VirtualNodeConsistentHash {
    // 物理节点
    class PhysicalNode {
        String ip;
        List<VirtualNode> virtualNodes = new ArrayList<>();
        
        public PhysicalNode(String ip) {
            this.ip = ip;
        }
    }
    
    // 虚拟节点
    class VirtualNode {
        PhysicalNode physicalNode;
        int hashValue;
        
        public VirtualNode(PhysicalNode node, int index) {
            this.physicalNode = node;
            // 通过"IP#编号"的方式创建虚拟节点
            String virtualKey = node.ip + "#" + index;
            this.hashValue = Math.abs(virtualKey.hashCode());
        }
    }
    
    // 一致性Hash环
    class ConsistentHashRing {
        // 使用TreeMap实现有序环
        private TreeMap<Integer, PhysicalNode> ring = new TreeMap<>();
        private int virtualNodeCount = 150; // 每个物理节点150个虚拟节点
        
        public void addNode(PhysicalNode node) {
            for (int i = 0; i < virtualNodeCount; i++) {
                VirtualNode vNode = new VirtualNode(node, i);
                ring.put(vNode.hashValue, node);
            }
        }
        
        public PhysicalNode getNode(String key) {
            if (ring.isEmpty()) return null;
            
            int hash = Math.abs(key.hashCode());
            
            // 顺时针查找
            SortedMap<Integer, PhysicalNode> tailMap = ring.tailMap(hash);
            if (!tailMap.isEmpty()) {
                return tailMap.get(tailMap.firstKey());
            }
            // 环回到起点
            return ring.get(ring.firstKey());
        }
    }
}
```

### 数据定位算法

```java
// 完整的一致性Hash实现
public class ConsistentHash<T> {
    // 使用TreeMap模拟Hash环
    private final TreeMap<Integer, T> hashRing = new TreeMap<>();
    private final int virtualNodeCount;
    
    public ConsistentHash(int virtualNodeCount) {
        this.virtualNodeCount = virtualNodeCount;
    }
    
    /**
     * 添加节点
     */
    public void addNode(T node) {
        for (int i = 0; i < virtualNodeCount; i++) {
            String virtualNodeName = node.toString() + "#VN" + i;
            int hash = getHash(virtualNodeName);
            hashRing.put(hash, node);
        }
    }
    
    /**
     * 删除节点
     */
    public void removeNode(T node) {
        for (int i = 0; i < virtualNodeCount; i++) {
            String virtualNodeName = node.toString() + "#VN" + i;
            int hash = getHash(virtualNodeName);
            hashRing.remove(hash);
        }
    }
    
    /**
     * 根据key获取节点
     */
    public T getNode(String key) {
        if (hashRing.isEmpty()) {
            return null;
        }
        
        int hash = getHash(key);
        
        // 顺时针查找第一个节点
        Map.Entry<Integer, T> entry = hashRing.ceilingEntry(hash);
        if (entry != null) {
            return entry.getValue();
        }
        
        // 如果没有找到，返回环的第一个节点
        return hashRing.firstEntry().getValue();
    }
    
    /**
     * 计算Hash值（使用FNV1_32_HASH算法）
     */
    private int getHash(String key) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        
        for (int i = 0; i < key.length(); i++) {
            hash = (hash ^ key.charAt(i)) * p;
        }
        
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;
        
        return hash & 0x7FFFFFFF;  // 确保为正数
    }
    
    /**
     * 统计分布情况
     */
    public void printDistribution(Map<String, Integer> data) {
        Map<T, Integer> distribution = new HashMap<>();
        
        for (String key : data.keySet()) {
            T node = getNode(key);
            distribution.put(node, 
                distribution.getOrDefault(node, 0) + data.get(key));
        }
        
        System.out.println("数据分布情况:");
        for (Map.Entry<T, Integer> entry : distribution.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue() + " 条数据");
        }
    }
}
```

## 一致性Hash的应用场景

1. 分布式缓存系统
2. 负载均衡
3. 分布式数据库分片

## 总结

### **一致性Hash的核心价值**

1. **扩展性**：添加/删除节点只影响相邻数据
2. **负载均衡**：通过虚拟节点实现相对均匀分布
3. **容错性**：节点故障影响最小化

### **选择一致性Hash的场景**

- 分布式缓存系统（Redis、Memcached）
- 负载均衡器
- 分布式存储系统
- 数据库分库分表
- CDN内容分发网络

### **不适用场景**

- 节点数量极少（<3个）
- 数据量极小
- 对数据分布均匀性要求极高的场景


一致性Hash的核心思想是通过环形Hash空间和虚拟节点技术，实现了,三大核心优势：
1. 最小化数据迁移：节点变化时只影响相邻数据 
2. 负载均衡：通过虚拟节点实现近似均匀分布 
3. 可扩展性：支持动态添加/删除节点

**四个关键技术**
1. Hash环结构：将线性空间变为环形 
2. 虚拟节点：解决分布不均问题 
3. 权重支持：适应不同性能节点 
4. 高效查找：O(logN)的节点定位


> 一致性Hash是现代分布式系统的基石之一，理解其原理和实现方式对于设计高可用、可扩展的分布式架构至关重要。