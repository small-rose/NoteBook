---
layout: default
title: Memento Pattern
parent: Design Patterns
has_children: false
nav_order: 28
---

Here are java Memento Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 什么是备忘录模式？

备忘录模式（Memento Pattern）

备忘录模式是一种行为型设计模式，它允许在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后可以将对象恢复到原先保存的状态。

核心思想是：“快照”机制——把对象的关键状态抽离出来单独存储，使得对象可以随时回滚到历史状态。

> 生活中的类比：游戏存档。你在打Boss前存个档（备忘录），如果打输了就读取存档回到战斗前的状态。存档文件本身不暴露游戏内部的复杂数据，只用来恢复。

考虑一个文本编辑器：用户输入文字、删除文字、修改格式，需要支持 Ctrl+Z 撤销 和 Ctrl+Y 重做。
如果不使用备忘录模式，你可能需要在编辑器类中维护一个历史状态列表，这会导致：

- 编辑器类既要负责编辑操作，又要负责状态管理，职责过重。
- 每次保存状态都需要暴露内部私有字段（如文本内容、光标位置、选中区域），破坏了封装性。
- 如果要保存的状态很复杂（如整个文档的DOM树），直接复制会带来性能问题。

备忘录模式将“状态保存”和“状态恢复”的责任委托给专门的备忘录对象，而原始对象只需提供获取状态和恢复状态的接口，且这些接口可以设计为只对备忘录对象可见（窄接口），对外部完全隐藏。

## 模式结构

```
┌──────────────────┐     ┌──────────────────┐
│   Originator     │     │    Memento       │
│ - state          │────▶│ - state          │
│ + saveToMemento()│     │ + getState()     │
│ + restoreFrom()  │     └────────┬─────────┘
└──────────────────┘              │
                                  │ 管理
                         ┌────────▼─────────┐
                         │   Caretaker      │
                         │ - mementoList    │
                         │ + addMemento()   │
                         │ + getMemento()   │
                         └──────────────────┘
```

核心角色：

- Originator（原发器）：需要保存和恢复状态的对象。它创建一个包含其当前内部状态的备忘录对象，也可以使用备忘录恢复状态。
- Memento（备忘录）：存储原发器内部状态的对象。它应该只被原发器访问（窄接口），对外部只提供必要的信息（如时间戳、名称）。
- Caretaker（负责人）：负责保存备忘录，但不能对备忘录的内容进行操作或检查。它可以存储多个备忘录（如历史记录列表）。



##  业务案例：文本编辑器的撤销/重做功能

实现一个简单的文本编辑器，支持输入文字、撤销、重做操作。我们使用备忘录模式保存每次操作后的编辑器状态。

1. 备忘录（Memento）
```java
public class EditorMemento {
    private final String content;
    private final int cursorPosition;
    private final long timestamp;

    public EditorMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
        this.timestamp = System.currentTimeMillis();
    }

    // 只提供 getter，不提供 setter —— 不可变
    public String getContent() {
        return content;
    }

    public int getCursorPosition() {
        return cursorPosition;
    }

    public long getTimestamp() {
        return timestamp;
    }

    // 对外部只显示基本信息
    @Override
    public String toString() {
        return "[" + content.substring(0, Math.min(content.length(), 20)) + "...] @" + cursorPosition;
    }
}
```

2. 原发器（Originator）
```java
public class TextEditor {
    private StringBuilder content = new StringBuilder();
    private int cursorPosition = 0;

    // 插入文字
    public void insertText(String text) {
        content.insert(cursorPosition, text);
        cursorPosition += text.length();
    }

    // 删除光标前的一个字符
    public void deleteChar() {
        if (cursorPosition > 0 && content.length() > 0) {
            content.deleteCharAt(cursorPosition - 1);
            cursorPosition--;
        }
    }

    // 移动光标
    public void moveCursor(int position) {
        if (position >= 0 && position <= content.length()) {
            this.cursorPosition = position;
        }
    }

    // 创建备忘录（保存当前状态）
    public EditorMemento save() {
        return new EditorMemento(content.toString(), cursorPosition);
    }

    // 从备忘录恢复状态
    public void restore(EditorMemento memento) {
        this.content = new StringBuilder(memento.getContent());
        this.cursorPosition = memento.getCursorPosition();
    }

    // 显示当前内容
    public void display() {
        System.out.println("内容: \"" + content.toString() + "\"");
        System.out.println("光标位置: " + cursorPosition);
        // 可视化光标
        StringBuilder sb = new StringBuilder(content);
        if (cursorPosition <= content.length()) {
            sb.insert(cursorPosition, "|");
        }
        System.out.println("视图: " + sb.toString());
    }
}
```

3. 负责人（Caretaker）
```java
import java.util.Stack;

public class HistoryManager {
    private Stack<EditorMemento> undoStack = new Stack<>();   // 撤销栈
    private Stack<EditorMemento> redoStack = new Stack<>();   // 重做栈

    // 保存一个新状态
    public void saveState(EditorMemento memento) {
        undoStack.push(memento);
        redoStack.clear();  // 新操作后清空重做栈
    }

    // 撤销：返回上一个状态
    public EditorMemento undo() {
        if (!undoStack.isEmpty()) {
            EditorMemento current = undoStack.pop();
            redoStack.push(current);
            if (!undoStack.isEmpty()) {
                return undoStack.peek();  // 返回上一个状态（但不弹出）
            }
        }
        return null;
    }

    // 重做：返回下一个状态
    public EditorMemento redo() {
        if (!redoStack.isEmpty()) {
            EditorMemento next = redoStack.pop();
            undoStack.push(next);
            return next;
        }
        return null;
    }

    public boolean canUndo() {
        return undoStack.size() > 1;  // 至少有两个状态才能撤销（当前 + 历史）
    }

    public boolean canRedo() {
        return !redoStack.isEmpty();
    }
}
```

4. 客户端
```java
public class EditorClient {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        HistoryManager history = new HistoryManager();

        // 初始状态（空文档）
        history.saveState(editor.save());

        // 操作1：输入 "Hello"
        editor.insertText("Hello");
        history.saveState(editor.save());
        editor.display();

        // 操作2：输入 " World"
        editor.insertText(" World");
        history.saveState(editor.save());
        editor.display();

        // 操作3：删除一个字符（删除 'd'）
        editor.deleteChar();
        history.saveState(editor.save());
        editor.display();

        // 撤销一次（回到 "Hello World"）
        System.out.println("\n=== 撤销 ===");
        EditorMemento prev = history.undo();
        if (prev != null) {
            editor.restore(prev);
            editor.display();
        }

        // 再次撤销（回到 "Hello"）
        System.out.println("\n=== 再次撤销 ===");
        prev = history.undo();
        if (prev != null) {
            editor.restore(prev);
            editor.display();
        }

        // 重做一次（回到 "Hello World"）
        System.out.println("\n=== 重做 ===");
        EditorMemento next = history.redo();
        if (next != null) {
            editor.restore(next);
            editor.display();
        }
    }
}
```

输出：
```
内容: "Hello"
光标位置: 5
视图: Hello|

内容: "Hello World"
光标位置: 11
视图: Hello World|

内容: "Hello Worl"
光标位置: 10
视图: Hello Worl|

=== 撤销 ===
内容: "Hello World"
光标位置: 11
视图: Hello World|

=== 再次撤销 ===
内容: "Hello"
光标位置: 5
视图: Hello|

=== 重做 ===
内容: "Hello World"
光标位置: 11
视图: Hello World|
```

核心价值：

- TextEditor 不需要自己维护历史列表，只需提供 save() 和 restore() 方法。
- 备忘录对象 EditorMemento 是不可变的，外界无法篡改历史状态。
- HistoryManager 独立管理撤销/重做逻辑，职责清晰。



## 优缺点

优点：

- 保持封装边界：原发器的内部状态对外部完全隐藏，只有原发器自己能访问备忘录中的状态（通过窄接口）。
- 简化原发器：原发器无需管理历史版本，职责更单一。
- 支持撤销/重做：这是备忘录模式最典型的应用场景。
- 状态快照可持久化：备忘录可以序列化到磁盘，实现程序重启后恢复。

缺点：

- 消耗内存：如果原发器状态很大，频繁保存备忘录会导致内存暴涨。可以通过增量备份或压缩优化。
- 增加系统复杂度：需要额外维护备忘录类和负责人类。
- 窄接口实现复杂：某些语言（如Java）需要通过包级访问权限或内部类来实现真正的窄接口，稍显麻烦。


## 适用场景

- 需要实现撤销/重做功能（文本编辑器、图形编辑器、IDE）。
- 需要保存对象的历史状态以便回滚（事务处理、游戏存档）。
- 需要在不破坏封装的前提下获取对象内部状态（调试工具、序列化快照）。
- 需要实现“检查点”机制（长时间运行的任务中途保存进度）。

## 备忘录模式 vs 命令模式

| 对比项	  | 中介者模式 | 观察者模式      |
|-------|-----------------|------------|
| 焦点	 | 保存和恢复对象状态| 封装操作请求 |
| 撤销实现	   |直接恢复历史快照   | 通过执行逆操作（undo）     |
| 内存占用	   | 保存完整状态快照，可能较大 |保存操作差异，通常较小   |
| 典型应用  |游戏存档、编辑器撤销 | 遥控器、任务队列  |

>两者可以结合使用：命令模式记录操作，备忘录模式保存命令执行前后的状态快照。

## 实现注意事项

- 窄接口 vs 宽接口：备忘录应对外部（Caretaker）只暴露最少信息（如时间戳），而对原发器暴露完整状态。在Java中可以通过包级访问权限或内部类实现。
- 克隆 vs 序列化：如果状态对象是可变的，备忘录必须进行深拷贝。可以使用克隆（实现Cloneable）或序列化（Serializable）。
- 增量备份：对于大型状态，可以只保存变化的部分（差异快照），而不是每次都保存全量快照。


## 现实应用

|领域	| 示例                                                      |
|-----|---------------------------------------------------------|
|文本编辑器 |Word、VS Code、Sublime Text 的 Ctrl+Z / Ctrl+Y 功能。 |
|图形编辑器 |Photoshop、Figma 的历史记录面板，可回退到任意步骤。|
|游戏开发 |游戏存档（保存角色位置、血量、背包物品）。|
|数据库事务|数据库的 UNDO 日志（回滚段）本质上是备忘录模式。|
|IDE 重构 |Eclipse/IntelliJ IDEA 的重构操作支持撤销。 |

## 总结：

备忘录模式的核心是“拍照留底，随时回放”。它让我们能够在不破坏封装的前提下，优雅地实现对象状态的保存和恢复。

虽然会增加内存开销，但在需要撤销/重做、历史回溯的场景中，它是无可替代的利器。

记住一句话：“存个档，打不过就读档。”