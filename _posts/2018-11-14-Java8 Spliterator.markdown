---
layout:     post
title:      "Introduction to Spliterator in Java"
subtitle:   "Java8 Spliterator tutorial"
date:       2018-11-14
author:     "baeldung"
header-img: "img/post-bg-js-version.jpg"
tags:
    - Java8
    - Stream
    - 翻译
---

# Java8 Spliterator

## 1. 概览

Java8引进了**Spliterator**接口，*用于队列的遍历和分区*，是**Stream**类的基础工具类，特别是在并行这方面。

本文涵盖了用途、特征、方法并举例如何实现一个自定义的**Spliterator**。



## 2. Spliterator API

#### 2.1 tryAdvance

#### 2.2 trySplit

#### 2.3 estimatedSize

#### 2.4 hasCharacteristics 



## 3. Spliterator特征

JDK8 API定义了8种Spliterator的特性用于描述该对象。这些特性能为外部工具在使用Spliterator时提供便捷：

* *SIZED* - 能否够使用estimateSize（）方法返回确切数量的元素

* *SORTED* - 是否迭代自排列有序的源

* *SUBSIZED* - 通过*trySplit()*方法获得的的**Spliterators**是否*SIZED*

* *CONCURRENT* - 

* *DISTINCT*

* *IMMUTABLE*

* *NONNULL*

* *ORDERED*

---

## 4. 实现一个自己的Spliterator

### 4.1场景

首先，我们假设存在如下场景：

我们有个`Article`类，它有个`authors`的域（一篇文章可能有多个作者 ），是一个`List`。此外，建立一个简单的关系，`Author`通过`relatedArticleId`域映射`Article`的`id`关联`Article`。

`Author` 如下：

```java
public class Author {
    private String name;
    private int relatedArticleId;

    // standard getters, setters & constructors
}
```



该类在检索authors stream时统计作者个数，使用这个流完成一个推导过程。

`RelatedAuthorCounter` 如下：

```java
public class RelatedAuthorCounter {
    private int counter;
    private boolean isRelated;
 
    // standard constructors/getters
 
    public RelatedAuthorCounter accumulate(Author author) {
        if (author.getRelatedArticleId() == 0) {
            return isRelated ? this : new RelatedAuthorCounter( counter, true);
        } else {
            return isRelated ? new RelatedAuthorCounter(counter + 1, false) : this;
        }
    }

    public RelatedAuthorCounter combine(RelatedAuthorCounter RelatedAuthorCounter) {
        return new RelatedAuthorCounter(
          counter + RelatedAuthorCounter.counter, 
          RelatedAuthorCounter.isRelated);
    }
}
```

