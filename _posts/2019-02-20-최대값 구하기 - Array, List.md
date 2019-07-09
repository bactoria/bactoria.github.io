---
layout: post
title:  "최대값 구하기 - Array, List"
categories: Java
author: bactoria
---

### 1. Array

```java
        int[] array = new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

        // 1. for
        int result1 = -1;
        for(int i=0; i < array.length; i++) result1 = Math.max(result1, array[i]);

        // 2. foreach
        int result2 = -1;
        for(int i : array) result2 = Math.max(result2, i);

        // 3. stream
        int result3 = Arrays.stream(array).reduce((a, b) -> a > b ? a : b).orElse(-1);

        System.out.println(result1); // 10
        System.out.println(result2); // 10
        System.out.println(result3); // 10
```

&nbsp;
&nbsp;

### 2. Collections

```java
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 1. for
        int result1 = -1;
        for (int i = 0, length = list.size(); i < length; i++) result1 = Math.max(result1, list.get(i));

        // 2. foreach
        int result2 = -1;
        for (int i : list) result2 = Math.max(result1, i);

        // 3. stream
        int result3 = list.stream().reduce((a, b) -> a > b ? a : b).orElse(-1);

        // 4. Collections
        int result4 = Collections.max(list);

        System.out.println(result1); // 10
        System.out.println(result2); // 10
        System.out.println(result3); // 10
        System.out.println(result4); // 10
```