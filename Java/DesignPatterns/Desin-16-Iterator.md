---
layout: default
title: Iterator Pattern
parent: Design Patterns
has_children: false
nav_order: 26
---

Here are java Iterator Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是解释器模式？

迭代器模式（Iterator Pattern）

迭代器模式是一种行为型设计模式，它提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露该对象的内部表示。

核心思想是：将遍历行为从聚合对象中分离出来，封装到一个独立的迭代器对象中，使得遍历算法可以与聚合对象解耦，并且可以为同一个聚合提供多种遍历方式。

> 生活中的类比：书架上的书。你不需要知道书架的内部结构（是按字母排序还是按出版日期排列），只需要一个“书签”或“手指”来一本本地翻阅。这个“书签”就是迭代器。

在没有迭代器模式的时代，如果你有一个数组，你用 for 循环遍历；如果你有一个 List，你用 get(index) 遍历；

如果你有一个 Set，你无法直接用索引遍历。每种集合都有自己的遍历方式，客户端代码必须了解集合的内部结构。

迭代器模式统一了遍历接口，让客户端可以用相同的方式遍历不同类型的集合，同时隐藏了集合的内部实现。

## 模式结构

```
┌──────────────────┐          ┌──────────────────────┐
│    Aggregate     │          │      Iterator        │
│ +createIterator()│────────▶ │ + hasNext(): boolean │
└────────┬─────────┘          │ + next(): Object     │
         ▲                    │ + remove()           │
         │ 实现                └─────────┬────────────┘
┌────────┴─────────┐                    ▲
│ ConcreteAggregate│                    │ 实现
│ +createIterator()│           ┌────────┴───────────┐
└──────────────────┘           │ ConcreteIterator   │
                               │ + hasNext()        │
                               │ + next()           │
                               └────────────────────┘
```

核心角色：

- Iterator（抽象迭代器）：定义访问和遍历元素的接口（hasNext()、next()、remove()）。
- ConcreteIterator（具体迭代器）：实现迭代器接口，跟踪遍历过程中的当前位置。
- Aggregate（抽象聚合）：定义创建迭代器的接口（createIterator()）。
- ConcreteAggregate（具体聚合）：实现创建迭代器的方法，返回一个合适的 ConcreteIterator 实例。

##  业务案例：音乐播放器播放列表

假设你有一个音乐播放器，播放列表可以有不同的存储方式（数组、链表、数据库游标），但用户希望用统一的方式遍历歌曲（上一首、下一首、查看所有歌曲）。

我们用迭代器模式来实现。

1. 迭代器接口
```java
public interface SongIterator {
    boolean hasNext();
    Song next();
    void reset();  // 回到开头
}
```

2. 具体迭代器：数组实现
```java
public class ArraySongIterator implements SongIterator {
    private Song[] songs;
    private int position = 0;

    public ArraySongIterator(Song[] songs) {
        this.songs = songs;
    }

    @Override
    public boolean hasNext() {
        return position < songs.length && songs[position] != null;
    }

    @Override
    public Song next() {
        if (!hasNext()) {
            throw new NoSuchElementException("没有更多歌曲");
        }
        return songs[position++];
    }

    @Override
    public void reset() {
        position = 0;
    }
}
```

3. 具体迭代器：链表实现
```java
import java.util.LinkedList;

public class LinkedListSongIterator implements SongIterator {
    private LinkedList<Song> songs;
    private int position = 0;

    public LinkedListSongIterator(LinkedList<Song> songs) {
        this.songs = songs;
    }

    @Override
    public boolean hasNext() {
        return position < songs.size();
    }

    @Override
    public Song next() {
        if (!hasNext()) {
            throw new NoSuchElementException("没有更多歌曲");
        }
        return songs.get(position++);
    }

    @Override
    public void reset() {
        position = 0;
    }
}
```

4. 聚合接口
```java
public interface Playlist {
    SongIterator createIterator();
    void addSong(Song song);
    void removeSong(Song song);
    int size();
}
```
5. 具体聚合：数组实现的播放列表
```java
import java.util.Arrays;

public class ArrayPlaylist implements Playlist {
    private Song[] songs;
    private int count = 0;
    private static final int DEFAULT_CAPACITY = 10;

    public ArrayPlaylist() {
        songs = new Song[DEFAULT_CAPACITY];
    }

    @Override
    public void addSong(Song song) {
        if (count >= songs.length) {
            songs = Arrays.copyOf(songs, songs.length * 2);
        }
        songs[count++] = song;
    }

    @Override
    public void removeSong(Song song) {
        // 简化实现：找到并移除
        for (int i = 0; i < count; i++) {
            if (songs[i].equals(song)) {
                System.arraycopy(songs, i + 1, songs, i, count - i - 1);
                songs[--count] = null;
                break;
            }
        }
    }

    @Override
    public int size() {
        return count;
    }

    @Override
    public SongIterator createIterator() {
        return new ArraySongIterator(Arrays.copyOf(songs, count));
    }
}
```

6. 具体聚合：链表实现的播放列表
```java
import java.util.LinkedList;

public class LinkedListPlaylist implements Playlist {
    private LinkedList<Song> songs = new LinkedList<>();

    @Override
    public void addSong(Song song) {
        songs.add(song);
    }

    @Override
    public void removeSong(Song song) {
        songs.remove(song);
    }

    @Override
    public int size() {
        return songs.size();
    }

    @Override
    public SongIterator createIterator() {
        return new LinkedListSongIterator(songs);
    }
}
```
7. 歌曲类
```java
public class Song {
    private String title;
    private String artist;

    public Song(String title, String artist) {
        this.title = title;
        this.artist = artist;
    }

    @Override
    public String toString() {
        return title + " - " + artist;
    }
}
```
8. 客户端（播放器控制器）
```java
public class MusicPlayer {
    public static void main(String[] args) {
        // 创建数组播放列表
        Playlist arrayPlaylist = new ArrayPlaylist();
        arrayPlaylist.addSong(new Song("晴天", "周杰伦"));
        arrayPlaylist.addSong(new Song("海阔天空", "Beyond"));
        arrayPlaylist.addSong(new Song("后来", "刘若英"));

        System.out.println("=== 数组播放列表遍历 ===");
        playAll(arrayPlaylist);

        // 创建链表播放列表
        Playlist linkedPlaylist = new LinkedListPlaylist();
        linkedPlaylist.addSong(new Song("Shape of You", "Ed Sheeran"));
        linkedPlaylist.addSong(new Song("Blinding Lights", "The Weeknd"));

        System.out.println("\n=== 链表播放列表遍历 ===");
        playAll(linkedPlaylist);
    }

    // 统一的遍历方法，不关心具体实现
    public static void playAll(Playlist playlist) {
        SongIterator iterator = playlist.createIterator();
        while (iterator.hasNext()) {
            Song song = iterator.next();
            System.out.println("正在播放：" + song);
        }
    }
}
```
输出：
```
=== 数组播放列表遍历 ===
正在播放：晴天 - 周杰伦
正在播放：海阔天空 - Beyond
正在播放：后来 - 刘若英

=== 链表播放列表遍历 ===
正在播放：Shape of You - Ed Sheeran
正在播放：Blinding Lights - The Weeknd
```

核心价值：

- 客户端（playAll 方法）完全不依赖播放列表的内部实现（数组还是链表）。
- 如果要新增一种集合类型（如数据库游标），只需新增一个迭代器实现，客户端代码无需修改。
- 可以轻松实现多种遍历方式（正序、倒序、跳跃遍历），每种遍历方式对应一个迭代器。



## 优缺点

优点：

- 简化客户端代码：客户端无需了解集合的内部结构，只需使用统一的迭代器接口。
- 支持多种遍历方式：同一个聚合可以提供多个不同的迭代器（如正向、反向、过滤）。
- 符合单一职责：将遍历行为从聚合对象中分离，聚合只负责存储，迭代器负责遍历。
- 易于扩展：新增集合类型只需新增对应的迭代器，无需修改现有代码。

缺点：

- 增加系统复杂度：对于简单的集合（如数组），引入迭代器可能显得过度设计。
- 性能开销：迭代器对象本身需要额外的内存，且方法调用有一定开销。
- 并发修改问题：如果在遍历过程中修改集合，迭代器可能抛出 ConcurrentModificationException（Java 的 fail-fast 机制）。


## 适用场景

- 需要以统一方式遍历不同类型的集合（如数组、List、Set、自定义数据结构）。
- 需要为同一个聚合提供多种遍历方式（如正序、倒序、按条件过滤）。
- 希望隐藏集合的内部实现，对外只暴露遍历接口。
- 需要支持对集合的并发遍历（快照迭代器、弱一致性迭代器等）。


## 迭代器模式 vs 组合模式

|对比项	| 迭代器模式	            | 组合模式        |
|-------|-------------------|-------------|
|目的	| 提供统一方式遍历集合元素| 表示部分-整体层次结构 |
|结构	| 线性遍历（顺序访问）    | 树形递归遍历      |
|侧重点	| 访问元素而不暴露内部表示 | 统一对待叶子与容器   |
|结合使用| 组合模式常配合迭代器模式遍历树形结构 | -   |

## 内部迭代器 vs 外部迭代器

|类型|特点|示例|
|---|----|----|
|外部迭代器|客户端控制遍历过程（主动调用 hasNext() 和 next()）|Java 的 Iterator|
|内部迭代器|迭代器控制遍历过程，客户端提供回调函数|JavaScript 的 forEach()、Java 的 Iterable.forEach()|

> 外部迭代器更灵活（可以暂停、跳过、反向），内部迭代器更简洁（不需要管理循环变量）。

## 现实应用

|领域	| 示例                                                              |
|-----|-----------------------------------------------------------------|
|正则表达式 |正则表达式引擎内部使用解释器模式解析模式串并匹配文本。    |
|SQL 解析 |许多数据库的 SQL 解析器将 SQL 语句解析为抽象语法树，然后解释执行。 |
|数学表达式计算 |计算器应用（如 2*(3+4)）使用解释器模式解析和求值。 |
|Spring SpEL |Spring 表达式语言（SpEL）内部使用解释器模式解析 #{bean.property} 等表达式。|
|规则引擎 |Drools、EasyRules 等规则引擎将规则（如 when ... then ...）解析为表达式树并执行。 |
|模板引擎 |FreeMarker、Thymeleaf 将模板中的 ${...} 表达式解析为 AST 并解释执行。 |

## 总结：

迭代器模式是现代编程中最常用的设计模式之一，几乎所有主流语言的标准库都内置了迭代器支持。

它的核心是“将遍历行为抽象化”，让客户端可以用统一的方式访问不同集合的元素。在 Java 中，你每天都在使用它——for (String s : list) 的背后就是迭代器模式。

记住一句话：“无论里面装的是什么，给我一个迭代器，我就能一个一个拿出来。”