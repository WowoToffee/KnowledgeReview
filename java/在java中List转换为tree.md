# List 转换为tree

## 递归

```java
public static List<Zone> buildTree1(List<Zone> zoneList) {
    List<Zone> result = new ArrayList<>();
    for (Zone zone:zoneList) {
        if (zone.parentId.equals("0")) {
            result.add(zone);
            setChildren(zoneList, zone);
        }
    }
    return result;
}

public static void setChildren(List<Zone> list, Zone parent) {
    for (Zone zone: list) {
        if(parent.id.equals(zone.parentId)){
            parent.children.add(zone);
        }
    }
    if (parent.children.isEmpty()) {
        return;
    }
    for (Zone zone: parent.children) {
        setChildren(list, zone);
    }
}

```

## 利用Map,遍历两次，实现tree的组装

```java 
public static List<Zone> buildTree3(List<Zone> zoneList) {
    Map<String, List<Zone>> zoneByParentIdMap = zoneList.stream().collect(Collectors.groupingBy(Zone::getParentId));
    zoneList.forEach(zone->zone.children = zoneByParentIdMap.get(zone.id));
    return zoneList.stream().filter(v -> v.parentId.equals("0")).collect(Collectors.toList());
}
```

方法一: 用递归的方法，时间复杂度等于：O(n +（n-m）* n)，根据初始算法那篇文章的计算时间复杂度的方法，可以得到最终时间复杂度是O(n2)
方法二: 用两次遍历的方法，时间复杂度等于：O(3n)，根据初始算法那篇文章的计算时间复杂度的方法，可以得到最终时间复杂度是O(n)，但它的空间复杂度比前两种方法稍微大了一点，但是也是线性阶的，所以影响不是特别大。所以第三种方法是个人觉得比较优的一种方法

| 方法 | 代码执行次数     | 时间复杂度    | 代码复杂程度 |
| ---- | ---------------- | ------------- | ------------ |
| 递归 | O(n +（n-m）* n) | 平方阶,O(n2） | 一般         |
| map  | O(3n)            | 线性阶,O(n）  | 复杂         |